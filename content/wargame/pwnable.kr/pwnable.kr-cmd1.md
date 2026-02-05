---
title: "[Pwnable.kr] Toddler's Bottle - cmd1 문제풀이"
date: 2019-02-11
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

오랜만의 `pwnable.kr`이다.
원래는 `uaf`를 풀고 싶었는데 일단 `cmd1`이 쉬워보여서 먼저 풀게 되었다.  

<!--more-->
  
문제 접속 정보는 아래와 같다.

```plain
ssh cmd1@pwnable.kr -p2222 (pw:guest)
```

문제에 접속하면 3개의 파일을 확인할 수 있다.  

> -r-xr-sr-x  1 root cmd1_pwn 8513 Jul 14  2015 cmd1  
> -rw-r--r--  1 root root      320 Mar 23  2018 cmd1.c  
> -r--r-----  1 root cmd1_pwn   48 Jul 14  2015 flag  

다른 문제들이랑 동일하게 실행파일, 소스코드, flag 파일이 있다.
`cmd1.c`의 내용은 다음과 같다.  

```c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
	int r=0;
	r += strstr(cmd, "flag")!=0;
	r += strstr(cmd, "sh")!=0;
	r += strstr(cmd, "tmp")!=0;
	return r;
}
int main(int argc, char* argv[], char** envp){
	putenv("PATH=/thankyouverymuch");
	if(filter(argv[1])) return 0;
	system( argv[1] );
	return 0;
}
```

소스코드는 상당히 간단하다.
`putenv`를 통해 `PATH`를 설정하고, 첫 번째 인자를 `filter` 함수를 통해 검사한다.
만약 이 `filter` 함수를 통과하면 첫 번째 인자를 그대로 `system` 함수에 넣어 실행시켜 준다.

그렇다면 간단하게 첫 번째 인자를 `cat flag`를 하면 될 것 같다.  
그런데 `filter` 함수를 살펴보면, 인자를 `flag`, `sh`, `tmp`와 비교해서 만약 그 값이 같다면 `r`의 값에 1을 더해준다.
이 후 이 `r`을 반환한다.  

즉, 첫 번째 인자에 `flag`, `sh`, `tmp`라는 문자열이 포함된다면 `r`은 0이 아닌 수가 될 것이고, 그렇다면 `if(filter(argv[1]))`이 참이 되어 프로그램이 종료된다.
따라서 문제를 풀기 위해서는 `flag`, `sh`, `tmp`라는 문자열을 포함하지 않고 `cat flag`를 실행시켜야 한다.

그러다 발견한 것이 `와일드카드`이다.
리눅스 환경에서 `*`은 모든 문자를 의미한다.
때문에 `cat cmd1.*`이라고 하면 `cmd1.` 이후에 어떠한 문자가 붙는 모든 파일을 `cat` 명령어를 통해 읽을 수 있게 된다.  

그렇다면 같은 원리로 `cat f***`이라고 하면 현재 디렉토리에 `f`로 시작하는 4글자 파일은 `flag`밖에 없으니 `flag` 파일을 읽어 줄 것이다.
또한 `flag`, `sh`, `tmp`가 포함되지 않으니 필터링도 통과할 수 있다.
그런데 한가지 더 생각해야 하는 부분이 있는데 바로 `putenv("PATH=/thankyouverymuch");`이다.
현재 `PATH`라는 환경 변수를 확인 해 보면 아래와 같다.  

```bash
cmd1@ubuntu:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

`PATH` 환경 변수는 실행 파일을 찾는 경로를 저장한다.
만약 내가 `cat`이라는 명령어를 입력하면 순서대로 `/usr/local/sbin`, `/usr/local/bin` 순으로 `cat` 이라는 명령어가 존재하는지 찾는다.  
`cat` 명령어는 `/bin` 디렉토리에 저장되어 있기 때문에 내가 굳이 `/bin/cat`이라고 입력하지 않아도 `PATH`에서 찾아주기 때문에 `cat`을 입력하는 것 만으로 사용할 수 있게 되는 것이다.

때문에 root 사용자와 일반 사용자가 모두 아무 위치에서나 `cat` 만으로 사용할 수 있다.
하지만 이 문제에서는 `PATH`라는 환경변수를 `/thankyouverymuch`로 바꾸기 때문에 `/bin`에 저장되어 있는 `cat` 명령어를 찾을 수 없다.
따라서 `/bin/cat`이라고 절대 경로를 입력 해 주어야 `cat` 명령어를 실행할 수 있다.  

이를 토대로 문제를 풀기 위해 `/***/cat f***`을 첫 번째 인자로 전달 해 주었다.
그 결과 `flag` 파일을 읽을 수 있었다.
사실 `/bin/cat f***`라고 전달 해 주어도 필터링에 걸리지 않는다.
그런데 그냥 해보고 싶었다 ㅎㅎ  

![](/images/pwnable.kr/cmd1/cmd1_01.PNG)

```bash
cmd1@ubuntu:~$ ./cmd1 "/***/cat f***"
mommy now I get what PATH environment is for :)
```

---

```plain
FLAG : mommy now I get what PATH environment is for :)
```
