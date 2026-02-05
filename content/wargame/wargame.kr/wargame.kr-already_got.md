---
title: "[Wargame.kr] already got 문제풀이"
date: 2018-10-15
draft: false
category: [wargame]
subcategories: [wargame.kr]
tags: [Write-up, Wargame.kr]
---

이번엔 `Wargame.kr`이 리뉴얼 되서 재오픈 했다.
그래서 이제 `Wargame.kr`을 천천히 정복해 보려고 한다.
첫 번째 문제는 `already got`이다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
can you see HTTP Response header?
```

`HTTP Response header`를 보면 되는 문제인 것 같다.
문제 페이지에 접속하면 정말 아무것도 없다.
그냥 `you've already got key! :p`라고만 나온다.
아까 디스크립션에서 `HTTP Response header`를 보라고 했으니, `Restlet` 확장 프로그램을 이용해서 `HTTP Response`를 확인 해 보았다.
그랬더니 아래와 같이 손쉽게 플래그를 얻을 수 있었다.  

![](/images/wargame.kr/already_got/already_01.png)

```plain
FLAG : 5d3b4c838ded08c2c0874bbc7884ac96be6cb536
```
