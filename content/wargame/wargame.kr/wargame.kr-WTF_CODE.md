---
title: "[Wargame.kr] WTF_CODE 문제풀이"
date: 2018-10-24
draft: false
category: [wargame]
subcategories: [wargame.kr]
tags: [Write-up, Wargame.kr]
---

`WFT_CODE` 문제이다.
이 문제는 뭔가 재미있는 문제였다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
This is another programming language.

Can you read this source code?
```

홈페이지에 들어가면 아래와 같이 적혀 있고, `source_code.ws`라는 파일 하나를 다운받을 수 있다.  

```plain
이게 진짜 소스코드라고? 아무것도 안보인다고!!

is this source code really???? i can`t see anything really!
```

이 파일을 다운받으면 정말 `아무것도` 안보인다.
그런데 드래그를 해 보면 아래와 같이 `space`와 `tab`으로만 이루어져 있다.  

![](/images/wargame.kr/WTF_CODE/wtf_code_01.png)

뭔가 이걸로만 이루어진 프로그래밍 언어가 있을 것 같아서 이것저것 찾아보다가 아래의 블로그를 찾을 수 있었다.  

> [Whitespace Programming Language](https://m.blog.naver.com/PostView.nhn?blogId=koromoon&logNo=220604856293&proxyReferer=https%3A%2F%2Fwww.google.co.kr%2F)

이런게 있다니 전혀 모르고 있었다.
친절하게 `Whitespace Programming Language 온라인 컴파일러`의 링크까지 올려주었다.  

> [ideone.com](https://ideone.com/)

여기에 들어가서 하단에서 `Whitespace`를 선택하고 문제의 소스코드를 복사&붙여넣기 한 후 `Run` 버튼을 누르면 아래와 같은 결과를 얻을 수 있다.  

![](/images/wargame.kr/WTF_CODE/wtf_code_02.png)

그리고 아래로 쭉 내려보면 실행 결과에서 flag를 얻을 수 있다.  

![](/images/wargame.kr/WTF_CODE/wtf_code_03.png)

```plain
FLAG : 5b0f61f7d5810f35a0d27470e1e17e3929aa4936 
```
