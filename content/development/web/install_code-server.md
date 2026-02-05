---
title: "[Code-Server] 웹 브라우저로 원격 서버에 접속 해 개발하기 (feat. VSCode)"
date: 2019-03-10
draft: false
category: [development]
subcategories: [web]
tags: [VSCode, Code Server]
---

회사에서 일하다 보면 Xshell은 설치할 수 없는데 원격 서버에서 뭔가 테스트용 코드를 짜야 할 일이 생긴다.
그래서 찾아보게 된 code-server이다.  

<!--more-->

`code-server`를 사용하면 리눅스 서버에 `VSCode`를 띄우고, 이를 `웹 브라우저`를 사용하여 접속할 수 있도록 해 준다.
때문에 노트북 뿐만 아니라, 태블릿이나 휴대폰으로도 원격 서버에 접속 해 파일을 생성하고, 개발을 할 수 있다.

하지만 쉘을 실행한 것이 아니기 때문에 명령어를 실행 할 수는 없다.
그래도 굳이 `xshell`이나 `putty`를 설치하지 않고도 웹 브라우저 만으로 파일을 생성하고 수정할 수 있기 때문에 편리하다.
설치 방법 또한 매우 간단하다.  

---  

> [code-server Github Repository](https://github.com/cdr/code-server)에서 code-server를 다운받을 수 있다.  

링크에 접속해서 아래를 확인 해 보면 아래와 같이 `Releases`가 있다.  

![](/images/development/code-server/code-server_01.png)  

여기서 `Download a release.`를 선택하면 최신 `code-server`를 다운받을 수 있다.
내가 다운받은 버전은 `3.2.0`이다.
여러 부분이 바뀌었다고 하는데, 나는 아래 표시 한 부분을 뒤늦게 발견하는 바람에 조금 삽질을 했다 ㅠㅠ  

![](/images/development/code-server/code-server_02.png)  

파일이 여러개 있을텐데, 여기서 내 환경에 맞는 파일을 다운받으면 된다.  

![](/images/development/code-server/code-server_03.png)  

나는 Ubuntu 20.04 64bit를 사용했기 때문에 code-server-3.2.0-linux-x86-64.tar.gz를 다운받기로 했다.
해당 파일에서 마우스 오른쪽 버튼 > 링크 주소 복사를 선택한다. 

![](/images/development/code-server/code-server_04.png)  

그 후 리눅스 서버로 돌아와서 아래의 명령어를 입력 해 파일을 다운받는다.  

```sh
$ wget https://github.com/cdr/code-server/releases/download/3.2.0/code-server-3.2.0-linux-x86_64.tar.gz

...

code-server-3.2.0-linux-x 100%[====================================>]  66.09M   739KB/s    in 2m 13s  

2020-05-13 11:16:54 (509 KB/s) - ‘code-server-3.2.0-linux-x86_64.tar.gz’ saved [69306138/69306138]
```

그러면 위와 같이 파일을 다운받을 수 있다.
다운받은 파일이 압축되어 있으므로 압축을 풀어야 한다.  

```sh
$ tar -xvf code-server-3.2.0-linux-x86_64.tar.gz
...
code-server-3.2.0-linux-x86_64/dist/register.js.map
code-server-3.2.0-linux-x86_64/dist/pages/
code-server-3.2.0-linux-x86_64/dist/pages/app.js
code-server-3.2.0-linux-x86_64/dist/pages/app.css
code-server-3.2.0-linux-x86_64/dist/pages/app.css.map
code-server-3.2.0-linux-x86_64/dist/pages/app.js.map
code-server-3.2.0-linux-x86_64/dist/register.js
code-server-3.2.0-linux-x86_64/dist/serviceWorker.js.map
code-server-3.2.0-linux-x86_64/dist/serviceWorker.js
code-server-3.2.0-linux-x86_64/package.json
```

압축을 풀고 해당 디렉터리로 들어 가 보면 `code-server`라는 바이너리가 있다.
만약 바이너리에 실행 권한이 없는 상태라면 아래의 명령어를 통해 실행 권한을 주어야 한다.  

```sh
$ cd code-server-3.2.0-linux-x86_64
$ chmod +x code-server
```

이제 `code-server` 바이너리를 실행시키면 웹 브라우저를 통해 원격 서버에 접속할 수 있을 것이다.
그런데 만약 원격 서버가 오픈되어 있는 경우라면, 불특정 다수가 해당 서버에 접속해서 아무 파일이나 막 생성할 수 있다.
때문에 접속하기 위한 패스워드를 설정 해 주어야 한다.

```sh
$ echo "export PASSWORD='PASSWORD_YOU_WANT'" >> ~/.bashrc
$ source ~/.bashrc
```

`cat ~/.bashrc`를 통해 `.bashrc` 파일을 확인 해 보면 가장 아래쪽에 내가 입력 한 패스워드가 들어가 있을 것이다.
마지막으로 접속을 위해 사용 할 포트를 열어 주어야 한다.
만약 웹 개발을 한다면 `80` 포트를 기본적으로 사용하기 때문에 `code-server`는 `8080` 포트를 사용하기로 했다.  

```sh
# ufw allow 8080/tcp
Rules updated
Rules updated (v6)
```

위와 같이 포트를 열어 준 후 `./code-server --bind-addr [원격서버 IP 주소]:[사용할 포트]`의 형태로 바이너리를 실행한다.  

```sh
$ ./code-server --bind-addr 192.168.171.135:8080
info  code-server 3.2.0 fd36a99a4c78669970ebc4eb05768293b657716f
info  HTTP server listening on http://192.168.171.135:8080
info    - Using custom password for authentication
info    - Not serving HTTPS
info  Automatic updates are enabled
```

정상적으로 실행이 됬다면, 웹 브라우저를 이용하여 `http://IP_Address:Port`로 접속한다.  

![](/images/development/code-server/code-server_05.png)  

패스워드를 설정 했다면 위와 같은 화면이 나올 것이고, 하지 않았다면 아래의 화면이 나타난다.
패스워드를 설정 한 경우에는 패스워드를 입력하면 아래의 페이지로 이동한다.
이제 이를 활용 해 파일이나 디렉터리를 생성하고 개발을 하면 된다! 

![](/images/development/code-server/code-server_06.png)  

내가 놀랐던 점은 디렉터리 이동도 매우 쉬웠다는 거다.
위의 메뉴에서 `Add workspace folder...`를 선택하면 아래처럼 `/root/`와 함께 주소창(?)이 생성된다.  

![](/images/development/code-server/code-server_07.png)  

이 주소창에 내가 원하는 경로를 입력하면 해당 경로 하위에 있는 파일이나 디렉터리가 출력되어, 굳이 경로를 외우지 않아도 쉽게 디렉터리를 이동할 수 있다!  

![](/images/development/code-server/code-server_08.png)  

![](/images/development/code-server/code-server_09.png)  
