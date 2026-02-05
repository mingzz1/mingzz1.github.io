---
title: "[Database-DB2] Docker로 IBM DB2 설치하기"
date: 2020-01-05
draft: false
category: [development]
subcategories: [database]
tags: [Database, DB2, Docker]
---

회사에서 갑자기 DB2를 쓸 일이 생겼다.
그래서 Docker를 사용 해 DB2를 설치하는 방법에 대해 정리한다.  

<!--more-->

인터넷에 `db2 install`이라고 검색하면 다양한 환경이 나오는데, 주로 CentOS를 사용하는 결과가 제일 많았다.
나는 아래의 블로그 글을 참고했기 때문에 Ubuntu 16.04 Desktop version을 사용 했다.  

> [https://medium.com/@shanikanishadhi1992/configure-database-postgresql-with-docker-and-use-postgresql-as-the-db-type-with-wso2-identity-a43e9cf7322a](https://medium.com/@shanikanishadhi1992/configure-database-postgresql-with-docker-and-use-postgresql-as-the-db-type-with-wso2-identity-a43e9cf7322a)  

우선 Docker를 설치하기 위해 아래의 명령어를 순서대로 입력한다.  

```sh
ubuntu@ubuntu-virtual-machine:~$ sudo su
[sudo] password for ubuntu:
root@ubuntu-virtual-machine:/# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
OK
root@ubuntu-virtual-machine:/# sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
root@ubuntu-virtual-machine:/# apt-get update
root@ubuntu-virtual-machine:/# apt-get install -y docker-ce
```

설치가 끝난 후 `systemctl status docker`를 입력 했을 때 아래와 같이 나온다면 정상적으로 설치 된 것이다.  

```sh
root@ubuntu-virtual-machine:/# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since 일 2020-01-05 15:22:51 KST; 38s ago
     Docs: https://docs.docker.com
 Main PID: 55386 (dockerd)
    Tasks: 8
   Memory: 38.0M
      CPU: 499ms
   CGroup: /system.slice/docker.service
           └─55386 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

 1월 05 15:22:48 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:48.798686781+09:
 1월 05 15:22:48 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:48.799017729+09:
 1월 05 15:22:48 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:48.799238919+09:
 1월 05 15:22:48 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:48.807393028+09:
 1월 05 15:22:49 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:49.335870327+09:
 1월 05 15:22:49 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:49.766653249+09:
 1월 05 15:22:50 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:50.961242687+09:
 1월 05 15:22:50 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:50.965732378+09:
 1월 05 15:22:51 ubuntu-virtual-machine dockerd[55386]: time="2020-01-05T15:22:51.553494105+09:
 1월 05 15:22:51 ubuntu-virtual-machine systemd[1]: Started Docker Application Container Engine
lines 1-21/21 (END)
```

이제, DB2를 설치 할 차례이다.
별도로 Dockerfile을 통해 빌드하지 않아도 아래의 명령어를 통해 Docker run을 할 수 있다.  

```sh
root@ubuntu-virtual-machine:/# docker run -itd --name db2 --privileged=true -p 50000:50000 -e LICENSE=accept -e DB2INST1_PASSWORD=db2inst1 -e DBNAME=test -v /home/ubuntu/DB2:/database ibmcom/db2
Unable to find image 'ibmcom/db2:latest' locally
latest: Pulling from ibmcom/db2
ac9208207ada: Pull complete 
db3b974ae005: Pull complete 
8b476cfca586: Pull complete 
a033f87212a0: Pull complete 
6062ba253339: Pull complete 
53d54ba98754: Pull complete 
8113895a570d: Pull complete 
Digest: sha256:e7055e4ee436e3659fbaafdc8c7fe1088f79658ba851024da68ab4e181085f58
Status: Downloaded newer image for ibmcom/db2:latest
a2314a930cc8b44208b8053f9702df6285fe80416109dbba736a06a609c7289b
```

Docker Container의 상태를 확인 해 보면 아래와 같을 것이다.  

```sh
root@ubuntu-virtual-machine:/# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                          NAMES
a2314a930cc8        ibmcom/db2          "/var/db2_setup/lib/…"   3 minutes ago       Up 3 minutes        22/tcp, 55000/tcp, 60006-60007/tcp, 0.0.0.0:50000->50000/tcp   db2
```

이 후, 아래와 같이 입력하면 Container의 Shell을 실행시킬 수 있다.  

```sh
root@ubuntu-virtual-machine:/# docker exec -i -t db2 /bin/bash
```

그 결과 아래와 같이 Container의 Shell로 변경된 것을 확인할 수 있다.
DB2는 Shell의 계정과 DB2의 계정이 같아야 하는 것 같다.
그래서 먼저 계정을 DB2의 계정인 `db2inst1`로 변경 해 주고 db2start 명령어를 통해 DB2를 실행시켜 주었다.  

```sh
[root@a2314a930cc8 /]# su - db2inst1
Last login: Sun Jan  5 06:29:57 UTC 2020 on pts/0
[db2inst1@a2314a930cc8 ~]$ db2start
01/05/2020 06:33:49     0   0   SQL1026N  The database manager is already active.
SQL1026N  The database manager is already active.
```

데이터베이스를 생성하는 명령어는 `create database [database명]`이다.
나는 test3이라는 데이터베이스를 생성 해 주고 해당 데이터베이스에 `db2inst1` 계정으로 연결을 해 주었다.  

```sh
[db2inst1@a2314a930cc8 ~]$ db2 create database test3
DB20000I  The CREATE DATABASE command completed successfully.
[db2inst1@a2314a930cc8 ~]$ db2 connect to test3 user db2inst1 using db2inst1

   Database Connection Information

 Database server        = DB2/LINUXX8664 11.5.0.0
 SQL authorization ID   = DB2INST1
 Local database alias   = TEST3
```

이 후 `db2 [Query]`의 형태로 쿼리를 실행시켜도 되지만, 그냥 `db2`만 입력 해 DB2 콘솔을 실행시킬 수 있다.
`list database directory`는 MySQL에서의 `show database`와 동일하게, 데이터베이스의 목록을 출력 해 주는 명령어이다.  

```sh
[db2inst1@a2314a930cc8 ~]$ db2
db2 => list database directory

 System Database Directory

 Number of entries in the directory = 2

Database 1 entry:

 Database alias                       = TEST
 Database name                        = TEST
 Local database directory             = /database/data
 Database release level               = 15.00
 Comment                              =
 Directory entry type                 = Indirect
 Catalog database partition number    = 0
 Alternate server hostname            =
 Alternate server port number         =

Database 2 entry:

 Database alias                       = TEST3
 Database name                        = TEST3
 Local database directory             = /database/data
 Database release level               = 15.00
 Comment                              =
 Directory entry type                 = Indirect
 Catalog database partition number    = 0
 Alternate server hostname            =
 Alternate server port number         =
```

정상적으로 실행되는 것을 확인할 수 있다.  
