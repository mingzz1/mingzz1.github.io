---
title: "[Pwnable.kr] Toddler's Bottle - fd 문제풀이"
date: 2018-02-15
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

`Pwnable`을 공부하기 위해 Pwnable.kr 의 문제를 풀기 시작했다.
먼저 첫 단계인 `Toddler's Bottle` 의 문제를 풀어보았다.

<!--more-->
  
문제 접속 정보는 아래와 같다.  

```plain
ssh fd@pwnable.kr -p2222 (pw:guest)
```

문제에 접속하면 3개의 파일을 확인할 수 있다.  

> -r-sr-x---  1 fd_pwn fd   7322 Jun 11  2014 fd  
> -rw-r--r--  1 root   root  418 Jun 11  2014 fd.c  
> -r--r-----  1 fd_pwn root   50 Jun 11  2014 flag  

`fd` 라는 이름의 실행파일, `fd.c` 라는 소스코드 파일, `flag` 파일이 있다.
파일의 권한에 대한 설명은 [여기](https://cc0ma.github.io/linux-permission/)를 참조하면 된다.
즉, `fd` 문제에서, 현재 내 계정은 `fd` 이기 때문에, `fd` 실행파일은 읽거나 실행시킬 수 있고, `fd.c` 파일은 읽을 수만 있다.
또한 `flag` 파일에는 아무런 권한을 가지고 있지 않다.

따라서, `setuid`가 걸려 있는 `fd` 실행 파일을 이용 해, 파일 소유자인 `fd_pwn` 의 권한을 얻어 `flag` 파일을 읽어야 한다.
`fd.c` 파일의 내용을 읽어 보면 다음과 같다.  

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

만약 `argc`가 2개 이하인 경우에는 `pass argv[1] a number`을 출력하고 프로그램을 종료한다.  

* argc : arguments count. 프로그램 실행 시 main 함수로 전달 된 인자의 갯수  
* argv : arguments vector. 프로그램 실행 시 main 함수로 전달 된 인자  
* ex) ./fd 1111 : argc는 2, argv[0]은 ./fd, argv[1]은 1111*

이 후, `fd`라는 변수를 선언 해 `argv[1]`의 값을 숫자로 변환한 후 `0x1234`를 뺀 값을 저장한다.  

* atoi 함수
    * 함수 원형 : int atoi(const char * str)
    * 문자열을 정수로 변환하는 함수  

`len` 변수를 선언한 후에는 `read` 함수를 사용 해 미리 선언 해 두었던 `buf` 라는 전역 변수에 32 byte 만큼 값을 읽어 온다.  

* read 함수  
    * 함수 원형 : read(int fd, void *buf, size_t nbytes)
    * fd(File Descriptor)가 참조하는 파일의 오프셋에서 nbytes 만큼 읽어 buf에 저장하는 함수
* File Discriptor  
    * 운영체제에서 파일을 관리하기 위해 각 파일마다 부여한 번호
    * 파일을 열게 되면 번호가 파일에 부여되고 디스크립터 테이블에 저장
    * 0 ~ 2번은 이미 할당되어 있기 때문에 파일에 부여하는 번호는 3번부터 시작  

|<center>번호</center>|<center>설명</center>|<center>이름</center> |<center>파일스트림</center>|
|:--------|:--------:|--------:|--------:|
|<center>0</center>|<center>표준 입력(Standard Input)</center>|<center>STDIN_FILENO</center>|<center>stdin</center>|
|<center>1</center>|<center>표준 출력(Standard Output)</center>|<center>STDOUT_FILENO</center>|<center>stdout</center>|
|<center>2</center>|<center>표준 에러(Standard Error)</center>|<center>STDERR_FILENO</center>|<center>stderr</center>|

`read` 함수를 통해 `buf`에 값을 저장 한 이후에는 해당 값과 `LETMEWIN` 을 비교한다.
만약 그 값이 같을 경우에는 `good job :)` 을 출력하고 `system` 함수를 통해 `/bin/cat flag` 명령어를 실행 해, `flag` 파일을 읽을 수 있게 된다.

즉, 이 문제에서 조작할 수 있는 부분은 `fd` 변수이다.
fd 변수는 `read` 함수의 `File Discriptor`로 사용되므로, `fd`의 값을 0으로 만들어 표준 입력을 받도록 하고, 표준 입력을 통해 `LETMEWIN`을 입력한다면 `buf`에 입력 값이 저장되며 `strncmp`가 있는 조건문을 통과할 수 있을 것이다.

fd 변수에는 `argv[1]`의 값에서 `0x1234`를 뺀 값이 저장된다.
`0x1234`는 16진수 값 이므로 이를 10진수로 바꾸면 `4660`이 된다.  

따라서 `fd` 파일을 실행하며 인자로 `4660`을 주면, 변수 `fd`에는 인자로 받아 온 `4660 - 0x1234`의 값인 0이 저장된다.
이 후, `read` 함수의 `fd` 변수 자리에는 `0`이 되며 표준 입력 값을 받아오게 된다 
이를 수행한 결과 다음과 같다.  

![](/images/pwnable.kr/fd/fd_01.PNG)

```bash
fd@ubuntu:~$ ./fd 4660
LETMEWIN
good job :)
mommy! I think I know what a file descriptor is!!
```

--- 

```plain
FLAG : mommy! I think I know what a file descriptor is!!
```
