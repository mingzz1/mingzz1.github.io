---
title: "[AWS Lambda] AWS Lambda Permission Error"
date: 2023-03-22
draft: false
category: [development]
subcategories: [cloud]
tags: [Cloud, Development]
---

AWS에 `Lambda`라고 `Serverless` 서비스가 있다. 
이번에 Lambda에 Docker Image를 생성하여 올리려고 하는데 오류가 발생해서 그 내용을 정리 해 두려 한다.

<!--more-->

`AWS Lambda`는 앞서 말했듯이 `Serverless` 서비스이다. 
때문에 내가 서버를 따로 구축하지 않고도 코드를 실행시킬 수 있다.  
물론 진짜로 서버가 없는 것은 아니고, AWS에서 미리 서버들을 구축 해 두고 사용자들은 필요할때만 특정 환경에 맞는 서버를 잠시 빌려 쓰는 개념이라고 생각하면 된다. 
때문에 모든 개발언어나 환경을 지원하는 것은 아니고, Lambda에서 지원하는 런타임이 따로 있다.
지원하는 런타임은 [여기](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)에서 확인할 수 있다.  


Lambda는 직접 코드를 올릴수도 있고 `AWS ECR`을 활용하여 Docker Image를 올릴수도 있는데, 내가 Lambda로 실행하려는 코드는 Ubuntu에 특정 데몬을 설치하고 실행시켜야 해 Docker Image를 빌드해서 올렸다.  
그런데 문제는 아래와 같은 오류가 계속 발생 한 것이다.

```plain
mkdir: cannot create directory '/var/run/XXX': Permission denied
chown: cannot access '/var/run/XXX': No such file or directory
```

웃긴건 로컬에서 빌드해서 실행했을 때는 너무나 잘된다는 것이다 ㅠㅠ  
만약 `/var/run/XXX` 폴더를 Docker Image를 빌드할 때 생성 해 주면 그때는 `chown: invalid user: username`에러가 나온다.  

그래서 좀 리서치를 해 보았더니 Lambda는 보안상의 이유로 `최소 권한을 가진 유저`로 코드를 실행한다고 한다.  
때문에 root 권한으로 실행해야 하는 코드는 권한 문제로 실행되지 않는다. ㅠㅠ  

예를 들어 `Flask`를 실행할 때 Port를 80번으로 설정 할 경우, 80번은 `Privileged Port` 즉, root 권한이 필요하기 때문에 Lambda에서는 실행할 수 없다. 8080 등 다른 Port를 설정 해 사용해야 한다.  

다행히 나는 내가 실행시켜야 하는 데몬이 다른 OS의 환경에서 root 권한이 없어도 사용 가능 해 해당 방식으로 해결했다. 만약 불가능하다면 Docker Image에서 필요한 dependency를 모두 설치 후 이를 통째로 zip 파일로 말아 Lambda에 올리는 방법도 있다고 한다. 이 부분은 나중에 성공하면 따로 포스팅 해 볼 예정이다. ㅎㅎ  

아 추가로, ECR에 Docker Image를 올려서 Lambda 함수를 생성 할 때는 로컬이나 일반 서버에서 돌리는 것과는 다르게 Dockerfile을 구성해야 한다. [참고](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/images-create.html)

이 부분도 나중에 따로 포스팅 할 예정이다. ㅎㅎ  
