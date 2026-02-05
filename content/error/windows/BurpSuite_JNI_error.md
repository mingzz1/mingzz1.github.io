---
title: "[BurpSuite] A JNI error has occurred / A Java Exception has occurred 오류"
date: 2020-06-10
draft: false
category: [error]
subcategories: [windows]
tags: [Error, Windows, BurpSuite]
---

`BurpSuite`를 최신 버전으로 설치하는데 오류가 났다.
`Java` 버전이 문제여서 `Java 9` 이상을 설치하니 해결할 수 있었다.  

<!--more-->

오류 상황은 아래와 같다.  

[BurpSuite 공식 홈페이지](https://portswigger.net/)에서 최신 버전의 `exe`를 다운받아 설치하려 했다.  

![](/images/error/burp_jni/jni_01.png)  

그랬더니 위의 사진처럼 최초 프로그레스 바에서 중간에 끊기더니 설치 과정이 진행이 안됬다.
그래서 대안으로 다시 `jar`를 다운받았다.
이번에는 아래와 같은 오류 두개가 나타났다.  

![](/images/error/burp_jni/jni_02.png)  

![](/images/error/burp_jni/jni_03.png)  

오류 메세지가 나와서 이를 찾아보니, `Portswigger Forum`에서 아래와 같은 답변을 찾을 수 있었다.  

> [Can't open Burp Suite -- "Error: A JNI error occured, please check your installation and try again"](https://forum.portswigger.net/thread/can-t-open-burp-suite-error-a-jni-error-occured-please-check-your-installation-and-try-again-c9e1c4ea )

답변 내용은 아래와 같다.  

> Unfortunately, Burp, Version 2020.4.1 no longer supports using Java 8 so you would need to update your Java version in order for Burp to work. We have tested the latest version against OpenJDK 13.0.3 but versions 9 and above should work.  

즉, `2020년 4월 1일` 이후, Java 8 버전은 더이상 사용할 수 없다.
최소 `Java 9` 버전 이상은 사용해야 한다.
답변대로, 최신 `JDK 14` 버전을 찾아 설치 해 주었더니 `jar`는 실행할 수 있었다.
~~exe는 여전히 안된다 ㅠㅠ 왜일까...~~

`JDK 14.0.1`은 아래에서 다운받을 수 있다.  

> [Java SE Development Kit 14 Downloads](https://www.oracle.com/java/technologies/javase-jdk14-downloads.html)
