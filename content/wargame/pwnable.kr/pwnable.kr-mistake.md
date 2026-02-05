---
title: "[Pwnable.kr] Toddler's Bottle - mistake 문제풀이"
date: 2018-05-30
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

어제에 이어 오늘도 문제를 풀었다.
무슨 문제를 풀까 고민하다가 `mistake` 문제가 1pt라 쉬울 것 같아서 먼저 풀어보게 되었다.  

<!--more-->

문제 접속 정보는 아래와 같다.

```plain
ssh mistake@pwnable.kr -p2222 (pw:guest)
```

또한 힌트로 `hint : operator priority`가 나와있다.
연산자 우선순위에 관련 된 문제인 것 같다.
이번에는 문제에 접속하니 4개의 파일을 확인할 수 있었다.   

> -r--------  1 mistake_pwn root      51 Jul 29  2014 flag  
> -r-sr-x---  1 mistake_pwn mistake 8934 Aug  1  2014 mistake  
> -rw-r--r--  1 root        root     792 Aug  1  2014 mistake.c  
> -r--------  1 mistake_pwn root      10 Jul 29  2014 password

먼저 `mistake.c` 소스코드를 확인 해 보니 아래와 같았다.  

```c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
	int i;
	for(i=0; i<len; i++){
		s[i] ^= XORKEY;
	}
}

int main(int argc, char* argv[]){
	
	int fd;
	if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
		printf("can't open password %d\n", fd);
		return 0;
	}

	printf("do not bruteforce...\n");
	sleep(time(0)%20);

	char pw_buf[PW_LEN+1];
	int len;
	if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
		printf("read error\n");
		close(fd);
		return 0;		
	}

	char pw_buf2[PW_LEN+1];
	printf("input password : ");
	scanf("%10s", pw_buf2);

	// xor your input
	xor(pw_buf2, 10);

	if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
		printf("Password OK\n");
		system("/bin/cat flag\n");
	}
	else{
		printf("Wrong Password\n");
	}

	close(fd);
	return 0;
}
```

언뜻 보면 `password` 파일을 `open()` 함수를 통해 열어 `read()` 함수로 읽은 후, 해당 값과 사용자가 입력 한 값을 `xor` 연산한 값이 같을 경우 `flag`를 출력 해 주는 것 같다.
그런데 자세히 보면 `open()` 함수와 `read()` 함수를 호출하는 `if`의 조건 안에서 괄호가 없다.  

```c
if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0)
if(!(len=read(fd,pw_buf,PW_LEN) > 0))
```

정상적으로 동작하기 위해서는 `fd=("/home/mistake/password",O_RDONLY,0400)`가 먼저 수행된 후, `<` 연산자로 0과 비교해야 한다.
`read()` 함수 또한 `len=read(fd,pw_buf,PW_LEN)`가 먼저 수행된 후, `>`를 통해 0과 비교해야 한다.  

하지만 [MSDN의 연산자 우선 순위](https://msdn.microsoft.com/ko-kr/library/2bxt6kc4.aspx)를 살펴보면 `<` 혹은 `>`이 `=` 보다 우선 순위가 높다는 것을 알 수 있다.
때문에 원래의 의도와는 달리 `open("/home/mistake/password",O_RDONLY,0400) < 0`가 먼저 수행된 후에 이 결과가 `fd`에 저장된다.

따라서 부등호를 통한 연산 결과에 따라 `fd` 혹은 `len`에 어떠한 값이 저장 될 것이다.
어떤 값이 저장되는지 확인하기 위해 아래와 같이 코드를 구현 해 테스트 해 보았다.  

```c
#include <stdio.h>

int main() {
        int a = 2 < 3;
        int b = 3 < 2;
        printf("%d\n", a);
        printf("%d\n", b);
}
```

그 결과, `a는 1`이, `b는 0`이 출력되었다.
즉, 부등호를 통한 연산 결과가 참일경우 `1`이, 거짓일 경우에는 `0`이 저장되는 것이다.  

다시 문제로 돌아가서, `open("/home/mistake/password",O_RDONLY,0400) < 0`에서 `open()` 함수가 정상적으로 파일을 열었다면, `양의 정수인 파일 디스크립터`를 반환 해 준다.
이 때, `open() 함수의 결과(양의 정수) < 0`은 거짓이 되므로, 결과적으로 `fd`에는 `0`이 저장된다.  

이 후, `read(fd,pw_buf,PW_LEN)`에서 `pw_buf`에 `PW_LEN`의 길이만큼 저장하는데, 읽어 오는 파일 디스크립터는 `fd`, 즉 `0`이 된다.
그런데 `0`은 파일 디스크립터 중 `표준 입력`이므로, 사용자가 입력 한 값을 읽어 `pw_buf`에 저장하게 된다.
따라서 문제를 풀기 위해서는 `do not bruteforce...`가 출력된 후 사용자가 원하는 특정 값을 10자리 입력하고, 각 자리를 `XORKEY`와 `xor` 연산한 값을 `input password : ` 다음에 입력 해 주면 될 것이다.  

`XORKEY`는 소스코드 상단에 `1`이라고 선언되어 있다.
따라서 나는 문제를 풀기 위해 `do not bruteforce...` 다음에는 `bbbbbbbbbb`을, `input password : ` 다음에는 `bbbbbbbbbb`과 `1`을 xor 연산 한 `cccccccccc`를 입력 해 주었다.  

![](/images/pwnable.kr/mistake/mistake_01.PNG)

```bash
mistake@ubuntu:~$ ./mistake 
do not bruteforce...
bbbbbbbbbb
input password : cccccccccc
Password OK
Mommy, the operator priority always confuses me :(
```

---

```plain
FLAG : Mommy, the operator priority always confuses me :(
```
