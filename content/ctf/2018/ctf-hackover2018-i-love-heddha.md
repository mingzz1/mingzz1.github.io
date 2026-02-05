---
title: "[hackover 2018] i-love-heddha Write up"
date: 2018-10-07
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, hackover2018]
---

주말에 잠시잠깐 짬을 내서 `hackover`라는 CTF에 참가했다.
그 중에 하나를 풀긴 했는데...
Write-up을 쓸까말까 고민 하다가 요즘 포스팅이 너무 뜸했어서 올린다. ㅎㅎ  

<!--more-->

문제 정보는 다음과 같다.  

```plain
i-love-heddha

A continuation of the Ez-Web challenge. enjoy

207.154.226.40:8080
```

이거 이 전에 나온 `ez web`이란 문제의 2번째 버전이다.
문제 서버에 접속하면 아래처럼 `under construction` 이라고만 나오고 아무것도 안나온다.  

![](/images/CTF/hackover2018/heddha/heddha_01.png)  

소스보기를 하면 `/under_construction.gif`가 있는데, 이걸 누르면 아래와 같은 오류가 나온다.  

![](/images/CTF/hackover2018/heddha/heddha_02.png)  

찾아보니 이건 `spring boot`라는 걸로 구현하면 저렇게 오류를 뱉어준다고 한다.
일단 뭔가 단서를 찾아야 하니 `robots.txt`를 찾아 보았다.
그랬더니 아래처럼 `/flag/`라는 디렉토리가 있는 것을 알 수 있었다.  

```plain
User-agent: * Disallow: /flag/
```

이 경로에 접속 해 보면 `flag.txt`가 있다.  

![](/images/CTF/hackover2018/heddha/heddha_03.png)  

근데 얘를 그냥 누르면 `flga.txt`로 이동해서 없다고 나온다.
~~오타인지 의도한건지...~~  

![](/images/CTF/hackover2018/heddha/heddha_04.png)  

얘를 다시 경로를 `flag.txt`로 바꿔서 이동 해 주면 `Bad luck buddy`라고 나온다.
여기서 쿠키를 확인하면 아래처럼 `isAllowed`가 있는데, 값이 `false`로 되어 있다.  

![](/images/CTF/hackover2018/heddha/heddha_05.png)  

그러면 이 쿠키 값을 `true`로 바꿔 주면 문제가 뙇! 하고 풀릴 것 같은데 이건 이 전의 문제 `ez web`의 풀이법이다.
일단 `true`로 바꿔주고 다시 보면 이번에는 `You are using the wrong browser, 'Builder browser 1.0.1' is required` 라고 한다.  

![](/images/CTF/hackover2018/heddha/heddha_06.png)  

헤더를 조작해야 하는 것 같다.
그래서 Burp Suite로 잡아서 `User-Agent`를 `Builder browser 1.0.1`로 변경 해 주었다.
그랬더니 이번에는 `You are refered from the wrong location hackover.18 would be the correct place to come from.` 란다.  

![](/images/CTF/hackover2018/heddha/heddha_07.png)  

그래도 title이 Almost인 것을 보니 이것만 해결하면 문제가 풀리는 것 같다.
다시 Burp Suite로 잡아서 아까처럼 `User-Agent`는 `Builder browser 1.0.1`로 바꾸고, `Referer`라는 것을 추가 해 `hackover.18`로 만들어 주었다.

![](/images/CTF/hackover2018/heddha/heddha_08.png)  

드디어! `flag` 처럼 보이는 것을 찾을 수 있었다.  

![](/images/CTF/hackover2018/heddha/heddha_09.png)  

딱 봐도 `base64`로 인코딩 되어있는 것 같으니, 얘를 [https://www.base64decode.org/](https://www.base64decode.org/)로 들어가서 디코딩 해 주었다.
그 결과 아래와 같이 `flag`를 찾을 수 있었다.  

![](/images/CTF/hackover2018/heddha/heddha_10.png)  

이제 다른 문제 풀러 가야지~

```plain
FLAG : hackover18{4ngryW3bS3rv3rS4ysN0}
```
