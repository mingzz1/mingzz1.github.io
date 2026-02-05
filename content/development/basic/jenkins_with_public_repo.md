---
title: "[Jenkins] Jenkins와 Github 연동하기 - 2. Public Repository"
date: 2021-04-18
draft: false
category: [development]
subcategories: [basic]
tags: [Jenkins, Github]
---

이번에는 Jenkins와 Public Repository를 연동하는 방법에 대해 정리한다.  

<!--more-->

`Jenkins 설치`나 `Private Repository`와 연동 방법이 필요 한 경우 아래 글을 참고하면 된다.  

[[Jenkins] Jenkins와 Github 연동하기 - 1. Jenkins 설치](https://mingzz1.github.io/development/basic/2021/04/18/jenkins_installation.html)  

[[Jenkins] Jenkins와 Github 연동하기 - 3. Private Repository](https://mingzz1.github.io/development/basic/2021/04/19/jenkins_with_private_repo.html)  

Public Repository와 연동하기 위해 아래와 같이 Test Repository를 생성했다.  

![](/images/development/jenkins/public/public_01.png)  

앞서 설치한 Jenkins 서버에서 Dashboard에 접속하여 `새로운 Item` 혹은 `Create a job`을 선택한다.  

![](/images/development/jenkins/public/public_02.png)  

그러면 아래와 같은 화면이 뜰텐데, `Enter an item name`에는 프로젝트의 식별자를 적고 `Freestyle project`를 선택한다.  

![](/images/development/jenkins/public/public_03.png)  

나는 Public Repository와 Private Repository를 구분하기 위해 `public_test`라고 식별자를 넣어주었다.  

![](/images/development/jenkins/public/public_04.png)  

`OK`를 누르면 제일 상단에 아래와 같은 `General` 설정 칸이 있는데, 사실 옵션이라 안넣어주어도 되지만 나는 `GitHub Project`를 선택하고 `Project url`에 내 Repository의 URL을 적어주었다.  

![](/images/development/jenkins/public/public_05.png)  

아래로 내려보면 `소스 코드 관리` 칸이 뜬다.
여기에서는 `Git`을 선택하고 `Repositories` > `Repository URL`에 git clone 할때 사용하는 주소를 적어준다.  

![](/images/development/jenkins/public/public_06.png)  

git clone 할때 사용하는 주소는 아래와 같이 확인할 수 있으며, 이 중 HTTPS를 선택하여 주소를 복사하면 된다.  

![](/images/development/jenkins/public/public_07.png)  

---  
### 참고)  

만약 아래와 같은 오류가 발생한다면 서버에 git이 제대로 설치되고 `Jenkins 내 Git 경로`가 제대로 들어가 있는지 확인 해 보면 된다.  

![](/images/development/jenkins/public/public_08.png)  

```sh
Failed to connect to repository : Error perfoming git command: git.exe ls-remote -h
https://github.com/mingzz1/Jenkins_test.git HEAD
```

Jenkins 내 Git 경로는 아래에서 설정한다.  

1. Dashboard 메뉴 중 Jenkins 관리 선택
2. Global Tool Configuration 선택
3. 하단의 Git 설정 중 Path to Git executable에 앞서 확인 한 git.exe의 전체 경로 붙여넣기 후 Save

Jenkins 내 Git 경로가 제대로 설정되어 있지 않을 경우 아래의 에러가 빨간색 글씨로 나타나 있을 것이다.  
```sh
There's no such executable git.exe in PATH: C:/Windows/system32, C:/Windows, C:/Windows/System32/Wbem, C:/Windows/System32/WindowsPowerShell/v1.0/, C:/Windows/System32/OpenSSH/, C:/Program Files (x86)/Common Files/Oracle/Java/javapath, C:/Windows/system32/config/systemprofile/AppData/Local/Microsoft/WindowsApps.
```

제대로 된 Git 경로를 넣을 경우 오류가 사라진다.
만약 환경변수 설정이 필요하다면 방법은 아래와 같다.  

1. [홈페이지](https://desktop.github.com/)에서 Github Desktop 설치
2. 설치된 Github Desktop 아이콘에서 마우스 오른쪽 버튼 > 파일 위치 열기 선택
3. 파일 위치 중 app-숫자\reousrces\app\git\cmd폴더에서 git.exe 파일 확인 (ex - app.2.7.2\reousrces\app\git\cmd\git.exe) 
4. 해당 폴더 경로 복사
5. Windows 버튼 클릭 후 시스템 환경 변수 편집 > 고급 탭 > 환경 변수 선택
6. 하단 시스템 변수 중 Path 선택 후 편집
7. 앞서 복사한 폴더 경로 붙여넣기 후 확인

때에 따라 재시작이 필요할 수도 있다.
확인 방법은 cmd 실행 후 git을 쳤을 때 명령어가 제대로 실행되는지 확인하면 된다.  

---  

다음으로 `Branched to build`에는 build 할 branch 명을 적어주면 되는데, 나는 주로 Default Branch 명을 적어주었다.
Jenkins_test Repository의 Default Branch 명은 main이므로 main을 적어주었다.  

![](/images/development/jenkins/public/public_09.png)  

아래에 `빌드 유발`, `빌드 환경`, `Build`, `빌드 후 조치`는 빌드와 관련된 설정인데, 나는 빌드는 따로 하지 않을거라 설정하지 않았다.
`저장`을 누르면 아래와 같이 프로젝트가 생성 될 것이다.
지금은 아무것도 없는 상태이며, 왼쪽 메뉴 중 `Build Now`를 선택 해 주어야 Github와 연동이 완료된다.  

![](/images/development/jenkins/public/public_10.png)  

Build가 성공 할 경우 왼쪽 아래에 `#숫자` 형태로 Build 결과가 나타난다.
파란불일 경우 빌드 성공, 빨간불일 경우에는 빌드 실패이다.
빌드가 실패 할 경우 로그가 나오므로, 해당 로그를 참고하여 오류를 해결 해 주면 된다.
빌드 완료 후에 왼쪽 메뉴 중 `작업공간`을 선택 해 보면 오른쪽과 같이 Github내 코드가 보이는 것을 확인할 수 있다.

![](/images/development/jenkins/public/public_11.png)  