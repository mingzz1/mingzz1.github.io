---
title: "[Jenkins] Jenkins와 Github 연동하기 - 3. Private Repository"
date: 2021-04-19
draft: false
category: [development]
subcategories: [basic]
tags: [Jenkins, Github]
---

앞서 Jenkins 설치, Jenkins - Github Public Repository 연동에 대해 정리했다.
이번에는 Jenkins와 Private Repository를 연동하는 방법에 대해 정리한다.  

<!--more-->

`Jenkins 설치`나 `Public Repository`와 연동 방법이 필요 한 경우 아래 글을 참고하면 된다.  

[[Jenkins] Jenkins와 Github 연동하기 - 1. Jenkins 설치](https://mingzz1.github.io/development/basic/2021/04/18/jenkins_installation.html)  

[[Jenkins] Jenkins와 Github 연동하기 - 2. Public Repository](https://mingzz1.github.io/development/basic/2021/04/18/jenkins_with_public_repo.html)  

앞서 사용한 Jenkins_test를 Private으로 변경했다.
Private Repository 연동의 경우 조금 더 복잡한데, 그 이유는 바로 Public Repository와 달리 SSH Key를 사용해서 연동해야 하기 때문이다.  

때문에 Private Repository의 경우 내가 관리자 권한을 가지고 있어야 한다. 즉, `Repository의 Setting`에 접근 권한이 있어야 한다.
먼저 SSH Keygen은 아래와 같이 생성할 수 있다.  

```sh
> ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\mingzzi/.ssh/id_rsa): C:\Users\mingzzi\Desktop\jenkins_test
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\mingzzi\Desktop\jenkins_test.
Your public key has been saved in C:\Users\mingzzi\Desktop\jenkins_test.pub.
The key fingerprint is:

The key's randomart image is:
```

cmd 창을 열고 `ssh-keygen`이라고 입력하면 위와 같이 입력하라는 창이 뜨는데 첫 번째에는 `key를 저장 할 경로 및 이름`을 적고 나머지는 그냥 엔터키로 넘어가면 된다.
그럼 내가 입력 한 경로에 `생성한_이름`, `생성한_이름.pub` 파일이 생성 된 것을 확인할 수 있다.  

![](/images/development/jenkins/private/private_01.png)  

이제 이 파일들을 Jenkins와 Github에 연동해야 한다.
먼저 Github Repository 내 `Settings` > `Deploy keys`에 들어간다.  

![](/images/development/jenkins/private/private_02.png)  

상단에서 `Add deploy key`를 선택하여 `Title`을 넣고, 생성된 키 중 `.pub`이 붙은 파일을 메모장으로 열어 그 내용을 `Key` 란에 넣어준다.(key의 값은 ssh-rsa로 시작)  

![](/images/development/jenkins/private/private_03.png)  

이 후 Jenkins 대시보드로 가 `Jenkins 관리` > `Manage Credentials`를 선택한다.  

![](/images/development/jenkins/private/private_04.png)  

상단에 `Credentials`를 선택하면 `System` 메뉴가 나온다.  

![](/images/development/jenkins/private/private_05.png)  

`System`을 선택하고 `Global credentials`를 선택하여 새로운 `credential`을 등록해야 한다.  

![](/images/development/jenkins/private/private_06.png)  

왼쪽의 `Add Credentials`를 선택하면 아래와 같은 입력 창이 뜬다.  

![](/images/development/jenkins/private/private_07.png)  

칸들에 아래와 같이 입력한 후 `OK` 버튼을 클릭한다.  

* Kind : SSH Username with private key 선택
* Scope : Global (Jenkins, nodes, items, all child items, etc)
* ID : 공란으로 두어도 무방
* Description : 공란으로 두어도 무방
* Username : Credential의 식별자
* Private Key : Enter directly 선택 후 생성된 키를 메모장으로 열어 내용을 붙여넣기 (key의 값은 BEGIN RSA PRIVATE KEY로 시작)

![](/images/development/jenkins/private/private_08.png)  

그럼 아래와 같이 `Credential`이 생성 된 것을 확인할 수 있다.  

![](/images/development/jenkins/private/private_09.png)  

이제 Dashboard로 돌아가서 왼쪽 메뉴 중 `새로운 Item`을 선택하여 Project를 생성한다.
전반적인 과정은 Public Repository을 연동하는 과정과 동일한데, Credential을 설정하는 부분만 다르다.
먼저 프로젝트를 생성하기 위해 프로젝트 명을 입력하고 `Freestyle project`를 선택한다.  

![](/images/development/jenkins/private/private_10.png)  

제일 상단에 있는 `General`은 옵션이라 안넣어주어도 되지만 `GitHub Project`를 선택하고 `Project url`에 Repository URL을 적어주었다.  

![](/images/development/jenkins/private/private_11.png)  

`소스 코드 관리` 칸에서는 `Git`을 선택하고 `Repositories` > `Repository URL`에 Repository에 접근할 수 있는 `URL`을 적어주어야 한다.  

![](/images/development/jenkins/private/private_12.png)  

이 경우에는 `SSH Keygen`을 사용하여 접근하기 때문에 Git Repository에서 git clone 할때 사용하는 주소 중 `SSH`라고 적힌 주소를 선택한다.  

![](/images/development/jenkins/private/private_13.png)  

만약 아래와 같은 오류가 발생한다면 [여기](https://mingzz1.github.io/development/basic/2021/04/18/jenkins_with_public_repo.html#%EC%B0%B8%EA%B3%A0)를 참고하여 오류를 해결하면 된다.  

```sh
Failed to connect to repository : Error perfoming git command: git.exe ls-remote -h
https://github.com/mingzz1/Jenkins_test.git HEAD
```

그게 아니라 아래와 같은 오류가 발생한다면 Private Repository에 접근하기 위하여 `Credential`을 설정 해 주어야 한다.  

```sh
Failed to connect to repository : Command "C:\Users\mingzzi\AppData\Local\GitHubDesktop\app-2.7.2\resources\app\git\cmd\git.exe ls-remote -h -- git@github.com:mingzz1/Jenkins_test.git HEAD" returned status code 128:
stdout:
stderr: git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

아래에 `Credentials`가 있는데 여기서 아래 화살표를 눌러 앞서 생성한 `Credential`을 선택 해 준다.  

![](/images/development/jenkins/private/private_14.png)  

그러면 놀랍게도 오류가 사라지는 것을 확인할 수 있다.
이 후 `Branches to build`에서는 build 할 branch 명을 적어주면 되는데 나는 Default Branch 명을 적어주었다.  

![](/images/development/jenkins/private/private_15.png)  

`빌드 유발`, `빌드 환경`, `Build`, `빌드 후 조치`는 빌드와 관련된 설정이지만 나는 빌드는 따로 하지 않을 것이기 때문에 설정하지 않았다.
저장을 누르면 아래와 같이 프로젝트가 생성되는데 이 중 왼쪽의 Build Now를 선택하여 Github Repository 내 저장 된 소스코드를 가져올 수 있다.  

![](/images/development/jenkins/private/private_16.png)  

Build가 성공 할 경우 왼쪽 아래에 `#숫자` 형태로 Build 결과가 나타난다.
파란불일경우 빌드 성공, 빨간불일 경우에는 빌드 실패이며, 빌드가 실패했을 경우에는 로그가 나오므로 해당 로그를 참고하여 오류를 해결하면 된다.
빌드 완료 후에는 왼쪽 메뉴 중 `작업공간`을 선택하여 소스코드를 확인할 수 있다.  

![](/images/development/jenkins/private/private_17.png)  