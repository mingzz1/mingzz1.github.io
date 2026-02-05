---
title: "[Pwnable.kr] Toddler's Bottle - random 문제풀이"
date: 2018-05-29
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

그동안 바빠서 문제를 못풀다가 오랜만에 `Pwnable.kr`에 접속 해 문제를 풀어보았다.
`passcode`를 풀어보려다가 `random` 문제가 더 쉬워서 먼저 풀게 되었다.  

<!--more-->

문제 접속 정보는 아래와 같다.

```plain
ssh random@pwnable.kr -p2222 (pw:guest)
```

문제에 접속하면 다른 문제들과 동일하게 3개의 파일을 확인할 수 있다.  

> -r--r-----  1 random_pwn root     49 Jun 30  2014 flag  
> -r-sr-x---  1 random_pwn random 8538 Jun 30  2014 random  
> -rw-r--r--  1 root       root    301 Jun 30  2014 random.c  

먼저 `random.c` 소스코드를 확인 해 보니 아래와 같았다.  

```c
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}
```

`rand()` 함수를 통해 `random` 값을 생성하고, 이 값과 User가 입력 한 `key` 값을 `XOR` 한 값이 `0xdeadbeef`라면 `flag`를 읽을 수 있다.
이 때, `rand()` 함수에 아무런 `seed` 값이 들어있지 않다.  

* seed란  
  * 난수 생성 시 인자로 주어, 항상 다른값이 나올 수 있도록 해 주는 값
  * 만약 이 seed 값이 없거나 항상 같은 값이라면 rand() 함수 등을 통해 생성되는 난수 값은 규칙적임

즉, `seed` 값이 없기 때문에 `rand()` 함수에서 생성되는 난수의 값은 항상 규칙적일 것이다.
이를 확인 해 보기 위해 아래와 같이 코드를 구현하였다.  

```c
#include <stdio.h>

int main() {
	int i;
	for(i = 0; i < 5; i ++) {
		unsigned int value= rand();
		printf("%d : %d\n", i, value);
	}
}
```

그 결과 아래와 같이 실행할 때마다 같은 난수 값이 출력되는 것을 확인할 수 있었다.  

![](/images/pwnable.kr/random/random_01.PNG)

그렇다면, 서버에서 실행되고 있는 `random`에서, `rand()` 함수를 통해 생성되는 난수 값 또한 같을 것이다.
우리가 필요한 것은 우리가 입력한 값과 `rand()` 함수를 `XOR` 한 값이 `0xdeadbeef`여야 한다는 것이다.

그런데 `XOR`의 경우 아래와 같은 특징이 있다.
`A XOR B = C` 라면 `B XOR C = A` 이고, `C XOR A = B` 이다.
문제에서 `key ^ random = 0xdeadbeef`를 만족하는 `key`를 찾아야 하므로, `random ^ 0xdeadbeef`를 한다면 `key` 값을 찾을 수 있게 된다.
`rand()` 함수에서 첫 번째로 나오는 값과 `0xdeadbeef`를 `XOR` 해 보면 아래와 같은 값이 나온다.  

```python
>>> 1804289383 ^ 0xdeadbeef
3039230856
```

문제 서버에서 `random`을 실행시킨 후 결과로 나온 `3039230856`를 입력하면 flag를 얻을 수 있다.  

![](/images/pwnable.kr/random/random_02.PNG)

```bash
random@ubuntu:~$ ./random
3039230856
Good!
Mommy, I thought libc random is unpredictable...
```

--- 

```plain
FLAG : Mommy, I thought libc random is unpredictable...
```
