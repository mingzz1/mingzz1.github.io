---
title: "[Pwnable.kr] Toddler's Bottle - flag 문제풀이"
date: 2018-02-21
draft: false
category: [wargame]
subcategories: [pwnable.kr]
tags: [Write-up, Pwnable.kr]
---

이번에는 `Pwnable.kr` 의 `flag` 문제를 풀어 보았다.
`flag` 문제는 pwnable 문제가 아니라 reversing 문제이다.  

<!--more-->

문제를 클릭하면 아래와 같이 다운로드 받을 수 있는 링크와 reversing 문제라는 설명이 나온다.  

* [http://pwnable.kr/bin/flag](http://pwnable.kr/bin/flag)

문제를 다운받아 파일의 정보를 확인 해 보았다.  

![](/images/pwnable.kr/flag/flag_01.PNG)

64 비트 elf 파일임을 확인할 수 있다.
파일을 실행 해 보았더니 아래와 같은 문자열을 확인할 수 있었다.  

![](/images/pwnable.kr/flag/flag_02.PNG)

파일을 분석하기 위해 `IDA`를 통해 열어보았다.
그런데 파일을 열어보았더니 함수는 3개밖에 없고 분석할 수가 없다.
파일을 실행했을 때 출력되는 `I will malloc() and strcpy the flag there. take it.` 문자열을 찾기 위해 바이너리의 문자열들을 확인 해 보았다.
문자열을 확인 해 보았더니, `I will malloc() and strcpy the flag there. take it.`는 없지만, 아래와 같이 `upx` 관련 문자열을 찾을 수 있었다.  

![](/images/pwnable.kr/flag/flag_03.PNG)

아무래도 `upx` 패킹 된 파일인 것 같아 `upx` 언패킹을 해 보았다.  

![](/images/pwnable.kr/flag/flag_04.PNG)

정상적으로 언패킹 되었다.
언패킹 된 파일을 다시 `IDA`를 통해 열어 보았다.  

![](/images/pwnable.kr/flag/flag_05.PNG)

`main` 함수를 확인해 본 결과 위와 같은 코드를 볼 수 있었다.
파일을 실행했을 때 확인할 수 있는 문자열을 `puts` 함수를 통해 출력하고, `malloc`를 통해 `100 byte`를 할당한 후, `strcpy` 함수를 통해 `flag`의 내용을 해당 위치에 복사한다.
이에 `flag`의 내용을 확인 해 보았다.  

![](/images/pwnable.kr/flag/flag_06.PNG)

위와 같이 바로 `flag` 의 값을 확인할 수 있었다.  

```plain
FLAG : UPX...? sounds like a delivery service :)
```
