---
title: "[Pwnable.kr] Toddler's Bottle - lotto 문제풀이"
date: 2019-02-14
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

오늘은 `lotto` 문제이다.
사실 원래는 `input`을 풀려고 하다가 일단 `lotto`를 먼저 풀었다.  

<!--more-->
  
문제 접속 정보는 아래와 같다.  

```plain
ssh lotto@pwnable.kr -p2222 (pw:guest)
```

문제에 접속하면 3개의 파일을 확인할 수 있다.  

> -r--r-----  1 lotto_pwn root     55 Feb 18  2015 flag  
> -r-sr-x---  1 lotto_pwn lotto 13081 Feb 18  2015 lotto  
> -r--r--r--  1 root      root   1713 Feb 18  2015 lotto.c  

다른 문제들이랑 동일하게 실행파일, 소스코드, flag 파일이 있다.  

`lotto.c`의 내용은 다음과 같다.  

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){
	
	int i;
	printf("Submit your 6 lotto bytes : ");
	fflush(stdout);

	int r;
	r = read(0, submit, 6);

	printf("Lotto Start!\n");
	//sleep(1);

	// generate lotto numbers
	int fd = open("/dev/urandom", O_RDONLY);
	if(fd==-1){
		printf("error. tell admin\n");
		exit(-1);
	}
	unsigned char lotto[6];
	if(read(fd, lotto, 6) != 6){
		printf("error2. tell admin\n");
		exit(-1);
	}
	for(i=0; i<6; i++){
		lotto[i] = (lotto[i] % 45) + 1;		// 1 ~ 45
	}
	close(fd);
	
	// calculate lotto score
	int match = 0, j = 0;
	for(i=0; i<6; i++){
		for(j=0; j<6; j++){
			if(lotto[i] == submit[j]){
				match++;
			}
		}
	}

	// win!
	if(match == 6){
		system("/bin/cat flag");
	}
	else{
		printf("bad luck...\n");
	}

}

void help(){
	printf("- nLotto Rule -\n");
	printf("nlotto is consisted with 6 random natural numbers less than 46\n");
	printf("your goal is to match lotto numbers as many as you can\n");
	printf("if you win lottery for *1st place*, you will get reward\n");
	printf("for more details, follow the link below\n");
	printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
	printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

	// menu
	unsigned int menu;

	while(1){

		printf("- Select Menu -\n");
		printf("1. Play Lotto\n");
		printf("2. Help\n");
		printf("3. Exit\n");

		scanf("%d", &menu);

		switch(menu){
			case 1:
				play();
				break;
			case 2:
				help();
				break;
			case 3:
				printf("bye\n");
				return 0;
			default:
				printf("invalid menu\n");
				break;
		}
	}
	return 0;
}
```

먼저 프로그램을 실행하면 메뉴가 3개가 있다.
`1`을 선택하면 로또 맞추기 게임을 할 수 있고, `2`를 선택하면 문제 풀이 방법을, `3`을 선택하면 프로그램을 종료시킨다.
`2`번을 선택했을 때 호출하는 `help()` 함수는 `printf()`로 문자열만을 출력한다.
그래서 `1`을 선택했을 때 호출하는 `play()` 함수를 살펴보았다.  

```c
void play(){
	
	int i;
	printf("Submit your 6 lotto bytes : ");
	fflush(stdout);

	int r;
	r = read(0, submit, 6);

	printf("Lotto Start!\n");
	//sleep(1);

	// generate lotto numbers
	int fd = open("/dev/urandom", O_RDONLY);
	if(fd==-1){
		printf("error. tell admin\n");
		exit(-1);
	}
	unsigned char lotto[6];
	if(read(fd, lotto, 6) != 6){
		printf("error2. tell admin\n");
		exit(-1);
	}
	for(i=0; i<6; i++){
		lotto[i] = (lotto[i] % 45) + 1;		// 1 ~ 45
	}
	close(fd);
	
	// calculate lotto score
	int match = 0, j = 0;
	for(i=0; i<6; i++){
		for(j=0; j<6; j++){
			if(lotto[i] == submit[j]){
				match++;
			}
		}
	}

	// win!
	if(match == 6){
		system("/bin/cat flag");
	}
	else{
		printf("bad luck...\n");
	}

}
```

먼저 `read()` 함수를 통해 표준 입력으로 6 byte를 입력 받아 `submit`에 저장한다.
이 때, `submit` 변수는 `char` 형 변수이기 때문에 문자 형태로 저장된다.

이 후 `fd`를 저장한 후 해당 `fd`에서 다시 6 byte를 입력 받아 `lotto` 변수에 저장한다.
`lotto` 변수 또한 `char` 형 이며, `1 ~ 45` 사이의 값을 가지도록 하기 위해 `(lotto[i] % 45) + 1`의 수식을 통해 연산을 한다.

마지막으로 만약 `submit`과 `lotto`의 각 자리를 비교 해 값이 같다면 `match++`을 하고, 이 `match`의 값이 `6`이라면 `/bin/cat flag`를 실행시켜 플래그를 읽을 수 있게 된다.\

난 처음에 `fd` 부분에 초점을 맞췄었다.
그런데 테스트 해 본 결과 `fd`의 값은 3이 나왔는데, 어떻게 해도 `fd`가 `3`일 때 입력 줄 수 있는 방법을 찾을 수가 없었다. ㅠㅠ
그래서 다시 `lotto`와 `submit`을 비교하는 부분을 살펴봤는데, 내가 바보였다...
소스코드를 다시 살펴보면 다음과 같다.  

```c
	int match = 0, j = 0;
	for(i=0; i<6; i++){
		for(j=0; j<6; j++){
			if(lotto[i] == submit[j]){
				match++;
			}
		}
	}
```

`for` 문을 2번 실행하는데, 이 때 비교하는 부분을 예외처리를 하지 않아 반복문이 무조건 36번을 돌게 되어있다.
그렇다면 같은 값으로 6 byte를 입력하고, 만약 `lotto`에 저장된 값이 하나라도 내가 입력 한 문자와 같다면 `match`가 6이 되어 문제를 풀 수 있을 것이다.

한 가지 더 생각해야하는 부분이 있다.
`submit`과 `lotto` 변수는 `char` 형 변수이다.
때문에 숫자를 입력하더라도 `char`형으로 저장되기 때문에 `0 ~ 9` 사이의 숫자를 입력한다면 이 값은 각각 아스키코드 값으로 `48 ~ 57`이 되기 때문에 `lotto`의 범위인 `1 ~ 45` 사이에 값이 되지 않는다.
따라서 아스키코드의 10진수 값 중 `1 ~ 45` 사이의 문자 중 출력 가능 한 문자를 찾아 `submit`의 값으로 입력 해 주어야 `lotto`와 비교를 했을 때 같은 값이 나올 수 있다.  

`33 ~ 45`까지가 출력 가능하며 문자로는 `!"#$%&'()*+,-`이 된다.
그래서 이 문자 중 아무거나 한개를 정해서 6개를 계속 입력 해 보았다.
그러면 언젠가는 `lotto`에 저장된 값 중 한개가 내가 입력 한 값과 동일하게 될 것이고, 그렇다면 36번의 반복문에 의해 `match`의 값이 6이 되며 문제를 풀 수 있을 것이다.
나는 `#`을 선택 해 입력 해 보았다.
그 결과 무려 13번 만에 플래그를 얻을 수 있었다.  

![](/images/pwnable.kr/lotto/lotto_01.PNG)

```bash
lotto@ubuntu:~$ ./lotto
- Select Menu -
1. Play Lotto
2. Help
3. Exit
1
Submit your 6 lotto bytes : ######
Lotto Start!
sorry mom... I FORGOT to check duplicate numbers... :(
```

---

```plain
FLAG : sorry mom... I FORGOT to check duplicate numbers... :(
```