---
title: "[Secure Coding] SonarQube 설치 및 사용법"
date: 2020-03-29
draft: false
category: [development]
subcategories: [secure-coding]
tags: [Secure Coding, Development]
---

많은 기업들에서 `Secure Coding`을 위해 정적 코드 분석 툴을 사용한다.
`SonarQube`는 그 중 하나인데, 커뮤니티 버전이 있어 무료로 사용할 수 있기에 설치 해 테스트 해 보았다.  

<!--more-->

`SonarQube`는 아래 사이트에서 다운받을 수 있다.  

> [https://www.sonarqube.org/](https://www.sonarqube.org/)

![](/images/development/secure/sonarqube/sonar_install_01.png)  

위의 사이트에 접속 후 `Download`를 누르면 아래의 페이지로 이동한다.  

![](/images/development/secure/sonarqube/sonar_install_02.png)  

여기에서 `Download Community Edition`을 선택하면 무료로 다운받을 수 있다.  

---  

만약 JDK가 설치되어 있지 않다면, JDK를 설치 해 주어야 한다.
JDK는 아래의 사이트에서 다운받을 수 있다.  

> [https://www.oracle.com/java/technologies/javase-downloads.html](https://www.oracle.com/java/technologies/javase-downloads.html)

---  

다운로드 완료 후 압축을 풀면 아래와 같은 파일 목록을 확인할 수 있다.
이 중 `conf` 폴더를 선택한다.  

![](/images/development/secure/sonarqube/sonar_install_03.png)  

`conf` 폴더에서는 `SonarQube`를 사용하기 위한 기본 설정파일들이 있다.
두 파일 모두 그냥 놔둬도 돌아간다고 하는데, 나는 그대로 사용하면 오류가 나서 `wrapper.conf`를 살짝 수정 해 주었다.  

![](/images/development/secure/sonarqube/sonar_install_04.png)  

해당 파일을 선택하면 맨 위의 코드가 아래와 같다.  

#### Before
```sh
# Path to JVM executable. By default it must be available in PATH.
# Can be an absolute path, for example:
# wrapper.java.command=/path/to/my/jdk/bin/java
wrapper.java.command=java
```

해당 코드에서 3번째 줄의 주석을 풀고 현재 내 PC에서 `java`가 설치 된 경로를 입력하고 4번째 줄을 주석처리 한다.  

#### After
```sh
# Path to JVM executable. By default it must be available in PATH.
# Can be an absolute path, for example:
wrapper.java.command=C:\Program Files\Java\jdk-14\bin/java
# wrapper.java.command=java
```

그 결과는 아래와 같다.  

![](/images/development/secure/sonarqube/sonar_install_05.png)  

이제 `bin` 폴더로 가 `StartSonar`를 더블클릭 해 실행시켜 준다.  

![](/images/development/secure/sonarqube/sonar_install_06.png)  

`StartSonar`를 누르면 `cmd` 창이 뜰 것이다.
그러면 웹 브라우저를 통해 `localhost:9000`으로 이동하면 아래와 같은 페이지를 확인할 수 있다.  

![](/images/development/secure/sonarqube/sonar_install_07.png)  

조금 기다리면 아래와 같은 메인 페이지가 나온다.  

![](/images/development/secure/sonarqube/sonar_install_08.png)  

`Log in` 버튼을 눌러 로그인 할 수 있는데, 기본 아이디와 패스워드는 `admin/admin`이다.  

![](/images/development/secure/sonarqube/sonar_install_09.png)  

로그인을 하면 아래와 같은 대쉬보드를 확인할 수 있다.  

![](/images/development/secure/sonarqube/sonar_install_10.png)  

소스코드 진단을 실제로 해 보기 위해 `Create new project`를 선택했다.
`Project key`에 내가 입력하고 싶은 프로젝트 이름을 선택하면, `Display name`은 자동으로 입력됬다. (변경도 가능하다)  

![](/images/development/secure/sonarqube/sonar_install_11.png)  

이 후, `Set Up`을 누르면 아래와 같은 페이지로 이동하는데, 여기에서 만약에 token 값이 없다면 새로 생성하고, 기존에 생성 된 토큰이 있다면 `Use existing token`을 선택 해 기존의 토큰 값을 입력 해 주면 된다.
나는 처음 실행하는거라 토큰 값이 없기 때문에 `Generate a token`을 선택하고 값을 입력한 후 `Generate`를 눌러 주었다.  

![](/images/development/secure/sonarqube/sonar_install_12.png)  

그럼 아래와 같이 `Token 명 : Token 값` 형태로 값이 출력된다.
이 후 `Continue`를 눌러 다음 단계를 진행한다.  

![](/images/development/secure/sonarqube/sonar_install_13.png)  

다음 단계에서는 정적 코드 분석을 진행 할 코드의 메인 언어와 내 OS를 선택해야 한다.
내가 테스트 하기 위해 사용 한 코드는 Lord of SQL Injection의 가장 첫 문제인 gremlin이기 때문에 언어는 `Other (JS, TS, Go, Python, PHP, ...)`을, 환경은 `Windows`를 선택했다.  

```php
<?php
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); // do not try to attack another table, database!
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id']) solve("gremlin");
  highlight_file(__FILE__);
?>
```

그리고 아래를 보면 `Download and unzip the Scanner for Windows`라고 뜨는데, 여기에서 `Download` 버튼을 눌러준다.
`Download` 아래의 `Execute the Scanner from your computer`에 있는 명령어는 일단 잘 복사 해 둔다.  

![](/images/development/secure/sonarqube/sonar_install_14.png)  

`Download`를 선택하면 아래와 같은 페이지로 이동한다.
정적 코드 진단을 하기 위한 Scanner를 다운로드 하는 링크이다.
여기에서 내 환경에 맞는 다운로드 링크를 선택하고 `Scanner`를 다운받아 준다.  

![](/images/development/secure/sonarqube/sonar_install_15.png)  

파일이 다운로드 되면 압축을 풀어주고, 해당 링크를 환경 변수에 등록 해 주어야 한다.
`시작` > `시스템 환경 변수 편집` > `고급` > `환경 변수`를 선택 한다.  

![](/images/development/secure/sonarqube/sonar_install_16.png)  

이 후 압축 해제 한 `sonar-scanner`의 `bin` 폴더를 환경 변수에 등록 해 준다.  

![](/images/development/secure/sonarqube/sonar_install_17.png)  

환경 변수에 등록을 완료 했다면 이제 cmd 창을 열어 진단 할 소스코드 있는 경로로 이동하고, 앞서 복사 해 뒀던 `Execute the Scanner from your computer`의 명령어를 cmd에 입력 해 준다.  

![](/images/development/secure/sonarqube/sonar_install_18.png)  

명령어 실행 후 코드의 양에 따라 조금 시간이 지난 후 아래와 같이 `EXECUTION SUCCESS`가 뜨면 진단이 끝난 것이다.  

![](/images/development/secure/sonarqube/sonar_install_19.png)  

다시 `localhost:9000`으로 가 보면 아래와 같이 진단 결과가 나와 있는 것을 알 수 있다.  

![](/images/development/secure/sonarqube/sonar_install_20.png)  

그런데 분명히 나는 SQL Injection이 가능 한 php 코드를 업로드 했는데, `Vulnerabilities`가 0개로 A등급이 나왔고, 약간 마이너 한 버그로 `if`문에 `braces` 즉, `{}`를 사용하지 않았다는 리포트를 얻을 수 있었다.  

![](/images/development/secure/sonarqube/sonar_install_21.png)  

Community 버전이라 그런건지는 모르겠지만, 간단한 점검 용으로는 사용 하더라도 진짜 서비스 해야 하는 곳에서는 사용하지 않을 것 같다...ㅎ  

--- 

~~이땐 몰랐지... SAST가 날 이렇게 힘들게 할줄...~~