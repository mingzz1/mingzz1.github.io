---
title: "[Spring 게시판 만들기] Spring 개발 환경 구축하기"
date: 2020-02-18
draft: false
category: [development]
subcategories: [web]
tags: [Web Development, Spring]
---


많은 곳에서 Spring을 가지고 웹 개발을 한다고 한다.
나는 웹 개발은 PHP나 JSP, 그것도 거의 학교 과제 수준밖에 안해봤기 때문에 Spring은 전혀 모른다.
근데 구조 등을 알면 일할 때 편하지 않을까 싶어 게시판을 만들어 보려고 한다.
오늘은 게시판을 만들기 위하여 Spring을 구축하는 방법을 정리한다.  

<!--more-->
## 1. Eclipse 설치  

먼저 `Spring`으로 웹 페이지를 개발하기 위해서는 `Eclipse`가 필요하다.
`Eclipse`는 아래 사이트에서 다운받을 수 있다.  

> [https://www.eclipse.org/downloads/packages/](https://www.eclipse.org/downloads/packages/)

사이트에 접속한 후, 아래와 같이 자신의 운영체제에 맞는 설치파일을 다운 받는다.  

![](/images/spring_board/setting/setting_01.PNG)  

![](/images/spring_board/setting/setting_02.PNG)  

다운받은 설치파일을 실행하면 아래와 같은 창이 뜰텐데, 이 중 나는 두 번째에 있는 `Eclipse IDE for Enterprise Java Developers`를 선택했다.  

![](/images/spring_board/setting/setting_03.PNG)

설치 할 `Eclipse`를 선택하면 설치 창이 뜬다. 경로를 확인한 후 `INSTALL` 버튼을 클릭한다.  

![](/images/spring_board/setting/setting_04.PNG)

모든 설치가 끝나면 아래와 같이 `LAUNCH` 버튼이 나타날 것이다.  

![](/images/spring_board/setting/setting_05.PNG)

해당 버튼을 누르면 Eclipse가 실행 될 것이다.
만약 정상적으로 실행된다면 `3. Spring Tools 설치`로 넘어가면 되지만, 오류가 난다면 `2. JAVA 설치`를 해야 한다.  

## 2. JAVA 설치  

`JAVA`는 아래의 사이트에서 설치한다.  

> [https://java.com/ko/download/](https://java.com/ko/download/)

위의 사이트에 접속 해 아래와 같이 빨간색 버튼을 클릭하여 설치한다.  

![](/images/spring_board/setting/setting_06.PNG)  

![](/images/spring_board/setting/setting_07.PNG)  

## 3. Spring Tools 설치  

Eclipse를 실행하면 아래와 같이 workspace의 경로를 정할 수 있다.  

![](/images/spring_board/setting/setting_08.PNG)  

나는 그냥 기본 경로로 두었다.
실행하면 초기 화면은 아래와 같다.  

![](/images/spring_board/setting/setting_09.PNG)  

이제 `Spring`을 사용하기 위하여 `Spring Tools`를 설치해야 한다.
상단의 메뉴 중 `Help` > `Eclipse Marketplace`를 선택한다.  

![](/images/spring_board/setting/setting_10.PNG)  

그 후 아래와 같이 `sts`를 검색하면 `Spring Tools`가 뜰 텐데, 나는 이 중 `Spring Tools 3`를 선택 해 주었다.  

![](/images/spring_board/setting/setting_11.PNG)  

이 후 나오는 창에서는 `Confirm`과 `I accept the terms of the license agreements` 선택 후 `Finish`를 각각 클릭 해 준다.  

![](/images/spring_board/setting/setting_12.PNG)  

![](/images/spring_board/setting/setting_13.PNG)  

그럼 Spring Tools의 설치가 끝난다.  

## 4. Tomcat 설치  

이제 웹 어플리케이션을 구동하기 위한 `WAS(Web Application Server)`를 설치 해 주어야 한다.
나는 `Apache Tomcat`을 선택했다.
설치 파일은 아래 링크에서 다운받을 수 있다.  

> [http://tomcat.apache.org/](http://tomcat.apache.org/)

해당 사이트에 접속하면 아래와 같을 것이다.
왼쪽의 `Download` 메뉴에서 설치 할 버전을 선택하면 되는데, 나는 `Tomcat 9`를 설치하기로 했다.  

![](/images/spring_board/setting/setting_14.PNG)  

다음 페이지로 넘어가서 각 환경에 맞는 설치 파일을 다운받아 준다.  

![](/images/spring_board/setting/setting_15.PNG)  

설치 파일을 실행한 후, 아래와 같이 `Next`를 계속 눌러 설치를 완료했다.  

![](/images/spring_board/setting/setting_16.PNG)  

![](/images/spring_board/setting/setting_17.PNG)  

## 5. Eclipse 설정  

Tomcat까지 설치가 완료 됬으면, Eclipse에서 구현한 웹 소스코드를 Tomcat을 통해 구동하기 위해 설정을 해주어야 한다.
상단의 메뉴 중 `Windows` > `Preferences`를 선택한다.  

![](/images/spring_board/setting/setting_18.PNG)  

그럼 아래와 같은 창이 뜨는데, 여기에서 `Server` > `Runtime Environments`를 선택하고, `Add`를 눌러준다.  

![](/images/spring_board/setting/setting_19.PNG)  

그 후, 자신이 선택 한 Tomcat의 버전을 선택하고 `Next`를 눌러준다. 나는 Tomcat 9.0을 설치했기 때문에 Apache Tomcat v9.0을 선택 해 주었다.  

![](/images/spring_board/setting/setting_20.PNG)  

다음 화면에서는 Apache Tomcat이 설치되어 있는 경로를 설정 해 주어야 한다. 내 경우에는 `C:\Program Files\Apache Software Foundation\Tomcat 9.0`에 설치되어 있었다.  

![](/images/spring_board/setting/setting_21.PNG)  

경로 설정 후 `Finish`를 누르면 아래와 같이 `Runtime Environments`에 `Apache Tomcat v9.0`이 들어가 있는 것을 확인할 수 있다.  

![](/images/spring_board/setting/setting_22.PNG)  

그럼 `Apply and Close`를 눌러 적용 해 준다.  

## 6. 프로젝트 생성하기  

드디어 모든 환경 설정이 끝났다.
이제 프로젝트를 생성해야 한다.
`File` > `New` > `Other`를 선택 해 준다.  

![](/images/spring_board/setting/setting_23.PNG)  

그럼 아래와 같은 창이 뜨는데, 여기에서 `Spring` > `Spring Legacy Project`를 선택하고 `Next`를 눌러준다.  

![](/images/spring_board/setting/setting_24.PNG)

다음 화면에서는 `Project name`과 `Templates`를 선택해야 한다.
나는 Project name은 Board로 했으며, Templates는 Persistence 아래의 Spring MVC Project로 설정 해 주면 된다.  

![](/images/spring_board/setting/setting_25.PNG)

다음 창에서는 `top-level package`를 설정 해 주어야 한다. `com.mycompany.myapp`과 같은 형태면 된다.  

![](/images/spring_board/setting/setting_26.PNG)  

이 후 `Finish`를 누르면 아래와 같이 알림창이 뜰 텐데 그냥 `Yes`를 선택 해 주면 된다.  

![](/images/spring_board/setting/setting_27.PNG)  

잠시 기다리면 왼쪽에 아래와 같이 프로젝트가 생성 된 것을 확인할 수 있다.  

![](/images/spring_board/setting/setting_28.PNG)  

## 7. 실행  

이제 실행을 해 Spring 첫 페이지가 나오면 성공이다.
프로젝트 이름에서 `마우스 오른쪽 버튼` > `Run As` > `Run on Server`를 선택 해 준다.  

![](/images/spring_board/setting/setting_29.PNG)  

그럼 아래와 같은 창이 뜰텐데, 여기에서 내가 설치한 Tomcat의 버전을 선택 해 주고 `Finish`를 눌러준다.  

![](/images/spring_board/setting/setting_30.PNG)  

만약 여기서 정상적으로 `Hello world!`가 나온다면 이제 개발을 시작하면 된다.
내 경우에는, 아래와 같은 오류가 발생했다.  

![](/images/spring_board/setting/setting_31.PNG)  

오류 메시지에서 `ports are invalid`라고 했으니 port 번호를 수정해야 하는 것 같다.
하단에서 `Servers`에서 해당 프로젝트의 서버를 더블클릭 해 준다.  

![](/images/spring_board/setting/setting_32.PNG)  

그럼 아래와 같이 서버 설정 페이지가 뜰 것이다.  

![](/images/spring_board/setting/setting_33.PNG)  

오른쪽에 보면 `Ports`가 있는데, 확인 해 보면 내 경우 `Tomcat admin port`에 아무 값이 설정되어 있지 않다.  

![](/images/spring_board/setting/setting_34.PNG)  

그래서 해당 값을 8000으로 설정 해 준 후 Ctrl + s로 저장 해 주었다.  

![](/images/spring_board/setting/setting_35.PNG)  

그 후 다시 `마우스 오른쪽 버튼` > `Run As` > `Run on Server`을 통해 실행시켜 주면 아래와 같이 방화벽 경고가 나올 것이다.  

![](/images/spring_board/setting/setting_36.PNG)  

아래의 `액세스 허용`을 선택 해 주면 아래와 같이 `Hello world!`를 확인할 수 있다.

![](/images/spring_board/setting/setting_37.PNG)  

또한 다른 웹 브라우저를 통해 접속하고 싶다면, Eclipse 창에서 접속 주소를 확인 후 웹 브라우저의 주소창에 입력 해 주면 된다.  

![](/images/spring_board/setting/setting_38.PNG)  
