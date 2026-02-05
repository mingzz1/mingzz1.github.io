---
title: "[Pwnable.kr] Toddler's Bottle - col 문제풀이"
date: 2018-02-17
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

`Pwnable.kr` 문제에서 2번째 문제인 `colliion` 문제를 풀어보았다.
문제 접속 정보는 아래와 같다.  

<!--more-->

```plain
ssh col@pwnable.kr -p2222 (pw:guest)
```

문제에 접속하면 3개의 파일을 확인할 수 있다.  

> -r-sr-x---  1 col_pwn col     7341 Jun 11  2014 col  
> -rw-r--r--  1 root    root     555 Jun 12  2014 col.c  
> -r--r-----  1 col_pwn col_pwn   52 Jun 11  2014 flag  

`col` 라는 이름의 실행파일, `col.c` 라는 소스코드 파일, `flag` 파일이 있다.
이번 문제도 역시 `col` 실행 파일에 `SetUID`가 걸려 있다.
`col.c` 파일의 내용을 읽어 보면 다음과 같다.  

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

먼저 `main` 함수를 살펴 보면 다음과 같다.  

```c
int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

프로그램 실행 시 `argc`가 2개 이하일 경우에는 `usage : %s [passcode]\n`를 출력하고 프로그램을 종료한다.
또한 `argv[1]`의 길이는 `20 byte` 여야 한다.
만약 전역변수로 선언 한 `hashcode`의 값, 즉, `0x21DD09EC`와 `20 byte` 길이의 `argv[1]`을 인자로 하여 `check_password`를 수행한 결과가 같다면 `flag`를 읽을 수 있다.
`check_password` 함수는 아래와 같다.  

```c
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}
```

`p`라는 변수를 인자로 받아 이를 `int` 형으로 타입을 캐스팅 해 준다.
그런데 이때 `int` 형인 `ip` 변수에는 `4 개의 문자열` 이 저장된다.
`int` 형은 `4 byte` 이고, `char` 형은 한 문자 당 `1 byte` 이기 때문에, `int` 형으로 타입 캐스팅을 하며 `문자열 4개가 하나의 integer 형 변수로 취급`된다.  

즉, 만약 사용자가 입력한 값이 `abcdefgh` 라면, `ip` 변수는 크기가 2인 배열이 되며, `ip[0]`에는 `abcd`를 `int` 형으로 변환한 값이, `ip[1]`에는 `efgh`를 `int` 형으로 변환한 값이 저장된다.
타입을 캐스팅 한 후에는 반복문을 돌며 `res` 변수에 `ip`에 저장 된 값을 더해준다.
`20 byte` 의 문자열을 입력 받아 `4 byte` 씩 나누어 `ip`에 저장했기 때문에 반복문은 총 5번을 수행하게 된다.

반복문 종료 후, `res`를 반환 해 준다.
따라서, `flag`를 읽기 위해서는 `20 byte`의 `argv[1]`의 값을 `4 byte` 씩 나누어 `int` 형태로 더한 값이 `0x21DD09EC`와 같아야 한다.
문제를 해결하기 위해 `0x21DD09EC`를 5로 나누어 보았다.  

> 0x21DD09EC / 5 = 0x6C5CEC8  
> 0x6C5CEC8 * 5 = 0x21DD09E8  

따라서 문제를 풀기 위해 입력해야 하는 문자열은 `0x6C5CEC8`를 4번 반복하고 마지막으로 `0x6C5CECC`를 이어주면 된다.
만약 `0x6C5CEC8` 나 `0x6C5CECC` 에 해당하는 문자가 있다면 그대로 써주려 했지만, 해당하는 아스키 코드 문자가 없기 때문에 해당 문자열을 `hex` 값으로 직접 입력 해 주었다.
아래와 같이 문제를 풀 수 있었다.  

![](/images/pwnable.kr/col/col_01.PNG)

```bash
col@ubuntu:~$ ./col `python -c 'print "\xc8\xce\xc5\x06"*4+"\xcc\xce\xc5\x06"'`
daddy! I just managed to create a hash collision :)
```


--- 

```plain
FLAG : daddy! I just managed to create a hash collision :)
```
