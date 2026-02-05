---
title: "[Linux 오류] Could not get lock 오류"
date: 2018-04-23
draft: false
category: [error]
subcategories: [linux]
tags: [Error, Linux]
---

Ubuntu에서 `apt-get`을 사용하다보면 아래의 오류를 자주 볼 수 있다.  
요즘들어 특히 자주보여 블로그에 정리하려 한다.

<!--more-->

```sh
E: Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?
```

apt는 Ubuntu에서 소프트웨어를 관리하기 위해 `/var/lib/dpkg/` 폴더를 사용한다.
따라서 apt를 이용하여 소프트웨어를 변경할 때는 다른 프로세스가 `/var/lib/dpkg/`를 변경할 수 없도록 lock을 거는 것이다.
그런데 이미 폴더에 lock이 걸려있어 lock을 걸 수 없을 때 이 오류가 발생한다.
즉, 다른 곳에서 돌고 있는 apt, 혹은 다른 프로세스에 의해 `/var/lib/dpkg/`를 변경하고 있을 때 발생한다.

때문에 아래의 3가지 방법 중 하나를 사용하면 거의 해결된다.

#### 1. lock 삭제

```sh
$ rm -rf /var/lib/dpkg/lock
```

#### 2. clear cache

```sh
$ apt-get autoclean $$ apt-get clear cache
```

#### 3. lock 삭제 후 apt update
```sh
$ rm /var/lib/apt/lists/lock
$ rm /var/cache/apt/archives/lock
$ rm /var/lib/dpkg/lock*
$ sudo dpkg --configure -a
$ apt update
```
