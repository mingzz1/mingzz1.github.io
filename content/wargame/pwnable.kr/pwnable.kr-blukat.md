---
title: "[Pwnable.kr] Toddler's Bottle - blukat 문제풀이"
date: 2018-08-25
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

요즘에 계속 `los`만 풀고 있었는데, 주변에서 갑자기 `pwnable.kr`에 새로운 문제가 추가됬다며 꼭 풀어보라고 자꾸 추천해줬다.
문제를 풀어 본 사람들이 다 재미있는 문제라고 하길래 한번 풀어보게 되었다.  

<!--more-->
  
문제 접속 정보는 아래와 같다.

```plain
ssh blukat@pwnable.kr -p2222 (pw: guest)
```

문제에 접속하면 4개의 파일을 확인할 수 있다.  

> -r-xr-sr-x  1 root blukat_pwn 9144 Aug  8 06:44 blukat  
> -rw-r--r--  1 root root        645 Aug  8 06:43 blukat.c  
> -rw-r-----  1 root blukat_pwn   33 Jan  6  2017 password  

바이너리가 있고, 소스코드와 `password` 파일이 있다.
일단 소스코들 확인 해 보았다.  

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
char flag[100];
char password[100];
char* key = "3\rG[S/%\x1c\x1d#0?\rIS\x0f\x1c\x1d\x18;,4\x1b\x00\x1bp;5\x0b\x1b\x08\x45+";
void calc_flag(char* s){
	int i;
	for(i=0; i<strlen(s); i++){
		flag[i] = s[i] ^ key[i];
	}
	printf("%s\n", flag);
}
int main(){
	FILE* fp = fopen("/home/blukat/password", "r");
	fgets(password, 100, fp);
	char buf[100];
	printf("guess the password!\n");
	fgets(buf, 128, stdin);
	if(!strcmp(password, buf)){
		printf("congrats! here is your flag: ");
		calc_flag(password);
	}
	else{
		printf("wrong guess!\n");
		exit(0);
	}
	return 0;
}
```

`main` 함수에서 `password` 파일을 읽고, 사용자에게 입력 값을 받는다.
만약 이 입력값과 `password` 내의 값이 같다면 `calc_flag(password)` 함수를 통해 `flag`를 출력 해 준다.
혹시 몰라 `cat` 명령어를 통해 읽었더니 아래와 같이 출력됬다.  

```bash
blukat@ubuntu:~$ cat password 
cat: password: Permission denied
```

그래서 무슨 방법이 있을까 고민하다가 그냥 아래처럼 입력했더니 문제가 갑자기 풀렸다.  

```bash
blukat@ubuntu:~$ cat password | ./blukat 
guess the password!
congrats! here is your flag: Pl3as_DonT_Miss_youR_GrouP_Perm!!
```

???
flag의 내용이 뭔가 이상해서 `id` 명령어로 권한을 확인 해 보았다.  

```bash
blukat@ubuntu:~$ id
uid=1104(blukat) gid=1104(blukat) groups=1104(blukat),1105(blukat_pwn)
```

그랬더니 그룹 권한이 부여된 것을 알 수 있었다.
그런데 `cat password`를 하면 계속 `Permission denied`가 나며 읽을 수 없다고 했다.
그룹 권한이 주어졌다면 읽을 수 있어야 하는게 맞는데 뭔가 이상해서 다른 방법으로 `password`의 내용을 읽어 보았다.  

```bash
blukat@ubuntu:~$ hd password 
00000000  63 61 74 3a 20 70 61 73  73 77 6f 72 64 3a 20 50  |cat: password: P|
00000010  65 72 6d 69 73 73 69 6f  6e 20 64 65 6e 69 65 64  |ermission denied|
00000020  0a                                                |.|
00000021
```

ㅂㄷㅂㄷ
그랬더니 `cat: password: Permission denied` 자체가 `password`의 내용이라는 것을 알 수 있었다.
낚임...ㅋㅋㅋㅋㅋ
다시 `blukat`을 실행해서 위의 내용을 입력 했더니 문제가 정상적으로 풀렸다.  

![](/images/pwnable.kr/blukat/blukat_01.PNG)

```bash
blukat@ubuntu:~$ ./blukat 
guess the password!
cat: password: Permission denied
congrats! here is your flag: Pl3as_DonT_Miss_youR_GrouP_Perm!!
```

---

```plain
FLAG : Pl3as_DonT_Miss_youR_GrouP_Perm!!
```
