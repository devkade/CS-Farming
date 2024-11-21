---
up: 
related: 
tags: []
aliases: []
cssclasses:
  - dashboard
created: 2024-11-17T02:16
modified: 2024-11-17T17:30
---
# 3.10.3 Out of Bounds Memory References and Buffer Overflow

C 언어는 배열 참조에 대한 어떠한 경계 체크도 하지 않는다. 지역 변수들은 경계의 관계 없이 상태 정보(레지스터 값, return 주소 등)와 함께 스택에 저장 된다. 이로 인해 배열의 요소가 배열을 벗어나는 문제가 발생하고 정보가 유출 되는 등 심각한 프로그램 에러를 야기할 수 있다.

## Buffer Overflow

```c
/* 라이브러리 함수 gets()의 구현 */
char *gets(char *s)
{
    int c;
    char *dest = s;
	while ((c = getchar()) != '\n' && c != EOF)
	    *dest++ = c;
	if (c == EOF && dest == s)
	    /* 문자가 읽히지 않음 */
	    return NULL;
	*dest++ = '\0'; /* 문자열 종료 */
	return s;
}
```

- 위 코드는 라이브러리 함수 gets의 구현을 나타낸다.
	- cf) getchar() 함수
		- 버퍼에 데이터가 있을 때 버퍼 가장 앞의 데이터를 반환한다.
		- 버퍼에 데이터가 없을 때 '\n' 이 나올 때까지 사용자로부터 문자를 받아 버퍼에 저장하고 가장 앞의 데이터를 반환한다.
	- 입력을 받아 출력으로 전송하는 함수, 종료 개행 문자나 오류 조건이 발생할 때까지 반복한다.
	- 만약 EOF가 잃히거나 아무 글자도 s에 복사하지 않았다면(dest == s) NULL 반환
	- 인자 s가 가리키는 위치에 문자열을 복사하고 null을 추가해 문자열을 종료한다.

```c
void echo() {
    char buf[8]; /* 너무 작음! */
    gets(buf);
    puts(buf);
}
```

- gets 함수를 사용한 echo 함수이다.
- gets 는 전체 문자열을 담을 수 있도록 충분한 공간이 할당되었는지 확인할 방법이 없다.
- 위 echo 함수의 경우 7문자(null 제외)보다 긴 문자열을 입력하면 out-of-bounds가 발생한다.

```assembly
echo:
	subq $24, %rsp        # Allocate 24 bytes on stack
	movq %rsp, %rdi       # Compute buf as %rsp
	call gets             # Call gets
	movq %rsp, %rdi       # Compute buf as %rsp
	call puts             # Call puts
	addq $24, %rsp        # Deallocate stack space
	ret
```

![](https://i.imgur.com/9e19sGD.png)

- 프로그램은 스택 포인터 - 24 를 통해 24 바이트를 할당한다.
- 버퍼 buf는 스택의 가장 위에 위치하여 %rdi 로 이동해 gets, puts의 함수 인자로 사용된다.
- buf 와 return 포인터 사이 16바이트는 padding, spare space로 알려져 사용되지 않는다.
- 입력된 문자수에 따라 gets는 스택에 저장된 일부 정보를 덮어쓸 수 있고 이로 인해 정보가 손상될 수 있다.

![](https://i.imgur.com/yDXyib8.png)

- 입력 문자열 길이에 따라 다음 부분이 손상될 수 있다.
	- 0-7 : 버퍼 길이안에 들어오기 때문에 손상되지 않는다.
	- 9-23 : 사용되지 않는 스택 공간 (Padding)
	- 24-31 : Return 주소 → 프로그램이 전혀 예상치 못한 곳으로 점프할 수 있다.
	- 32+ : caller 에서 저장된 상태

- 더 나은 방법은 read 바이트의 최대 길이를 세는 인자를 포함하는 fgets 함수를 사용하는 것이다.
- gets, strcpy, strcat, sprintf 와 같은 일반적으로 사용되는 라이브러리 함수들은 목적지 버퍼의 크기에 대한 표시없이 byte sequence를 생성할 수 있는 특성을 가지고 있다.
	- 이런 조건들은 버퍼 오버플로우의 취약점이 될 수 있다.

- ex. Practice Problem 3.46

```c
char *get_line()
{
    char buf[4];
    char *result;
    gets(buf);
    result = malloc(strlen(buf));
    strcpy(result, buf);
    return result;
}
```

- buf의 길이는 4.
- gets에서 4 이상의 길이의 문자를 받으면 오버플로우가 발생할 수 있다.
- strlen은 '\n'을 제외한 길이를 반환하므로 result 는 buf보다 1 작은 값이 동적 할당된다.
- strcpy 는 buf의 '\n'을 만나기 전까지 전부 복사해 result에 복사한다. (문자열에 null 문자가 포함되어야 하므로 오버플로우 발생)

## Exploit

- 버퍼 오버플로우를 악용하면 프로그램이 원치 않는 기능을 수행하도록 만들 수 있다.
- 이는 컴퓨터 네트워크를 통한 시스템 보안을 공격 하는 가장 일반적인 방법 중 하나이다.

- Exploit 코드(실행 가능한 코드의 바이트 인코딩이 포함된 문자열 + 코드로 포인터의 반환 주소를 덮어쓰는 바이트)를 통해 ret 명령어를 실행하면 exploit 코드로 점프하는 효과가 발생한다.
	- Exploit 코드는 시스템 호출을 사용해 shell 프로그램을 통해 운영체제에 접근하도록 하거나, 권한 없는 작업을 수행하도록 만들 수 있다.
