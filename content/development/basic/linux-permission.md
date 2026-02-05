---
title: "[Linux] 소유권(Ownership) / 허가권(Permission)"
date: 2018-02-15
draft: false
category: [development]
subcategories: [basic]
tags: [System]
---

리눅스 운영체제에서는 소유권(Ownership)과 허가권(Permission)을 가지고 있다.
리눅스 운영체제를 사용하는데 있어 매우 중요한 개념이기 때문에 정리하려 한다.  

<!--more-->

### 허가권(Permission)

리눅스 운영체제에서 파일과 디렉토리 목록을 보기 위해 `ls` 명령어를 사용한다.  

이때, `-al` 옵션을 주어 전체 목록을 확인 해 보면 아래와 같은 형태를 확인할 수 있다.  

```sh
-r-sr-x---  1 user guest   7322 Jun 11  2014 file  
-rw-r--r--  1 root   root  418 Jun 11  2014 code.c  
-r--r-----  1 user root   50 Jun 11  2014 textFile  
```

여기서 앞의 r, w, x 형태로 나타난 부분이 바로 `허가권(Permission)` 이다.  

* -rwxrwxr-x : 파일 (맨 앞이 -)  
* drwxrwxr-x : 디렉토리 (맨 앞이 d)  

허가권에서 `r`, `w`, `x` 는 각각 `read(읽기)`, `write(쓰기)`, `execute(실행, 접근)` 을 의미한다.  

![](/images/linux/permission/permission_01.png)  

* File Type : 디렉토리와 파일을 구분  
* User : 소유자의 권한  
* Group : 소유 그룹의 권한  
* Others : 소유자나 소유 그룹에 포함되지 않는 사용자의 권한  
  
|<center>권한</center>|<center>표시</center>|<center>파일</center>|<center>디렉토리</center>|<center>Octal 값</center>|
|:--------|:--------:|--------:|--------:|--------:|
|<center>읽기</center>|<center>r</center>|<center>파일을 읽고 복사 가능</center>|<center>ls 명령어를 통해 디렉토리의 목록 확인 가능</center>|<center>4</center>
|<center>쓰기</center>|<center>w</center>|<center>파일의 내용 수정 가능</center>|<center>디렉토리에 파일을 추가 혹은 삭제 가능</center>|<center>2</center>
|<center>실행</center>|<center>x</center>|<center>실행 가능한 파일을 실행 가능</center>|<center>cd 명령어로 디렉토리 접근 가능</center>|<center>1</center>|  

### 소유권(Ownableship)  

아래의 예시에서 파일의 허가권 옆에 `user`, `guest`, `root` 등으로 나타난 부분이 소유권이다.  

```sh
-r-sr-x---  1 user guest   7322 Jun 11  2014 file  
-rw-r--r--  1 root   root  418 Jun 11  2014 code.c  
-r--r-----  1 user root   50 Jun 11  2014 textFile  
```

앞 부분이 해당 파일의 `소유자`이며, 뒷 부분은 `소유 그룹`이다.  

```sh
-r-sr-x---  1 user guest   7322 Jun 11  2014 file  
```

위에서 `user`는 `file` 파일의 `소유자`이며, `guest`가 `file` 파일의 소유 그룹이 된다.  

### SetUID / SetGID  

그런데 예시를 살펴 보면, 파일의 허가권에 x가 아닌 s가 나타나 있는 부분이 있다.  

```
> -r-sr-x---  1 user guest   7322 Jun 11  2014 file  
```

바로 `SetUID`가 설정되어 있는 것이다.  

* SetUID : 사용자가 사용을 할 때만 소유자의 권한으로 파일을 실행함  
* SetGID : 사용자가 사용을 할 때만 그룹의 권한으로 파일을 실행함  
_표기는 x 대신 s를 사용_  

위의 예시에서는 앞의 `User` 의 허가권 부분에 `SetUID`가 걸려 있다.
따라서 file을 실행하는 동안에는 소유자의 권한으로 파일을 실행하게 된다.
만약 파일의 소유자가 `root`인 파일에 `SetUID`가 걸려 있다면, 일반 사용자가 해당 파일을 실행하는 동안에는 `root`의 권한을 사용할 수 있다.  
