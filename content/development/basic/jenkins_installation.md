---
title: "[Jenkins] Jenkins와 Github 연동하기 - 1. Jenkins 설치"
date: 2021-04-17
draft: false
category: [development]
subcategories: [basic]
tags: [Jenkins, Github]
---

위키백과에 따르면, `Jenkins`란 `CI Tool(Continuous Integration Tool)`로 다수의 개발자들이 하나의 프로그램을 개발할 때 버전 충돌을 방지하기 위해 각자 작업한 내용을 공유 영역에 있는 Git 등의 저장소에 빈번히 업로드 함으로써 지속적 통합이 가능하도록 해준다고 명시되어 있다. 

<!--more-->

[위키백과 링크](https://ko.wikipedia.org/wiki/%EC%A0%A0%ED%82%A8%EC%8A%A4_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4))

뭔가 굉장히 어려워 보이지만 단순히 한 예를 이야기 해 보자면, Github 등에 개발자가 소스코드를 수정하여 업로드 했을 경우 Jenkins를 사용하여 프로그램이 자동으로 빌드되도록 할 수 있다.
한 명의 개발자가 한 프로그램을 빌드하는 것은 그렇게 어렵지 않겠지만, 회사에서 여러 명의 개발자가 빌드를 해야 하는 경우에는 이야기가 조금 달라진다.
그래서 `Jenkins` 같은 도구를 사용하여 빌드를 자동화 하는 것이다.

오늘은 Jenkins와 Github를 연동하는 방법에 대해 정리 해 두려 한다.
원래 `Public Repository 연동하는 법`, `Private Repository 연동하는 법` 이렇게 2개만 쓰려고 했는데, 분량 조절 실패로 3편으로 나뉘어 졌다.

`Github`와 연동하는 방법은 아래 두 글을 확인하면 된다.  

[[Jenkins] Jenkins와 Github 연동하기 - 2. Public Repository](https://mingzz1.github.io/development/basic/2021/04/18/jenkins_with_public_repo.html)  

[[Jenkins] Jenkins와 Github 연동하기 - 3. Private Repository](https://mingzz1.github.io/development/basic/2021/04/19/jenkins_with_private_repo.html)  

Linux 환경에서도 구축은 가능하지만 나는 Windows 환경이 필요했기에 Windows를 기준으로 설명한다.  

[https://www.jenkins.io/](https://www.jenkins.io/)에 접속하면 아래와 같은 화면이 있다.  

![](/images/development/jenkins/install/install_01.png)  

여기에서 Download 버튼을 누르면 아래와 같이 각 환경이 나오는데, 나는 여기 중 LTS 버전의 Windows 설치파일을 다운 받았다.  

![](/images/development/jenkins/install/install_02.png)  

그럼 `jenkins.msi`라는 설치 파일이 다운받아지는데 아래의 과정을 따라가며 설치한다.  

![](/images/development/jenkins/install/install_03.png)  

![](/images/development/jenkins/install/install_04.png)  

`Logon Type`은 어차피 로컬에서만 테스트 할 예정이라서 `Run service as LocalSystem (not recommendeD)`를 선택했다.  

![](/images/development/jenkins/install/install_05.png)  

포트 선택창이 나올 경우 원하는 포트 선택 후 `Test Port` 버튼을 눌러 해당 포트가 사용 가능한지 확인해야한다.  

![](/images/development/jenkins/install/install_06.png)  

![](/images/development/jenkins/install/install_07.png)  

그 다음은 JAVA의 경로이다.
만약 Java가 없다고 할 경우 [여기](https://www.oracle.com/kr/java/technologies/javase/javase-jdk8-downloads.html)에서 Java를 설치하고 설치 과정을 재시작하면 된다.  

![](/images/development/jenkins/install/install_08.png)  

이 후 과정은 쭉 `Next`를 눌러주면 된다.  

![](/images/development/jenkins/install/install_09.png)  

![](/images/development/jenkins/install/install_10.png)  

![](/images/development/jenkins/install/install_11.png)  

설치가 다 되면 아래와 같은 창이 뜰텐데, 나와있는 경로에 가서 해당 파일을 열어보면 패스워드가 있다.  

![](/images/development/jenkins/install/install_12.png)  

![](/images/development/jenkins/install/install_13.png)  

`initialAdminPassword`를 아래와 같이 입력 해 준다.  

![](/images/development/jenkins/install/install_14.png)  

다음은 `Jenkins`의 `Plugin`을 설치하는 과정인데, 나는 따로 빌드환경을 구축하진 않을거라서 가장 기본 세팅으로 선택했다.  

![](/images/development/jenkins/install/install_15.png)  

그럼 아래와 같이 설치가 진행된다.  

![](/images/development/jenkins/install/install_16.png)  

설치 후에는 Admin 계정을 생성해야 하는데, 어차피 가상머신이라서 대충 입력했다.
만약 어딘가 외부에서 접속 가능한 서버에 구축할 경우에는 계정을 좀 더 강력하게 하고, 이메일도 실제 이메일을 넣는 것이 좋을 것 같다.  

![](/images/development/jenkins/install/install_17.png)  

계정을 생성하고 나면 Jenkins에 접속하는 URL을 설정해야 하는데, 이 부분도 따로 손대지는 않았다.  

![](/images/development/jenkins/install/install_18.png)  

아래 화면이 나타나면 모든 설치가 끝난 것이다.
`Start using Jenkins`를 선택하면 된다.  

![](/images/development/jenkins/install/install_19.png)  

아래 화면이 나타난 경우 Jenkins의 설치가 완료 된 것이다.  

![](/images/development/jenkins/install/install_20.png)  

이제 여기서 `Create a job`을 선택하여 `Github 연동 및 빌드 환경`을 할 수 있다.  