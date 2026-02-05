---
title: "[Database-DB2] 새로운 User 생성 및 권한 부여하기"
date: 2020-01-07
draft: false
category: [development]
subcategories: [database]
tags: [Database, DB2]
---

이번엔 DB2에서 새로운 사용자를 생성하고, 해당 사용자에게 권한을 부여하는 방법에 대해 정리한다.  

<!--more-->

사용한 환경을 다시 한번 적으면 다음과 같다.

* Ubuntu 16.04 + Docker

일단 DB2는 구조가 다른 RDBMS랑 다르다.
다른 RDBMS에서는 DB 계정이 따로 있고 생성할 수 있지만, DB2에서는 `OS 계정 == DB 계정`이다.  

즉, 새로운 DB 계정을 생성하기 위해서는 OS 계정을 생성해야 한다.
먼저 일반 계정이 필요할 경우, OS에서 root로 로그인 후 아래의 명령어를 입력 해 새로운 계정을 생성한다.
Docker로 만들었을 경우, Docker의 Shell을 실행시켜 Docker 내에서 작업을 해야 한다.

```sh
$ useradd [USERNAME]
$ passwd [USERNAME]
```

그런데 만약 관리자 권한이 필요하다면, 해당 계정을 `관리자 권한 그룹`에 추가해야 하기 때문에 아래처럼 생성해야 한다.  

```sh
$ useradd -u [UID] -g db2iadm1 -m -d /home/[USERNAME] [USERNAME]
$ passwd [USERNAME]
```

* 참고 : [IBM Db2 데이터베이스 설치를 위한 그룹 및 사용자 ID 작성(Linux 및 Unix)](https://www.ibm.com/support/knowledgecenter/ko/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0006742.html)

계정 생성이 완료 된 후에는 `db2inst1` 계정으로 로그인 해 새로 생성한 사용자에게 권한을 부여한다.  

#### 1. db2inst1 계정으로 로그인
```sh
$ su - db2inst1
```
  
#### 2. 권한 부여
#### Ex1)
```sh
$ db2 grant connect on database to user [USERNAME]
DB20000I  The SQL command completed successfully.
```
#### Ex2)
```sh
$ db2 grant dbadm on database to user [USERNAME]
DB20000I  The SQL command completed successfully.
```
#### Ex3)
```sh
$ db2 grant ACCESSCTRL, BINDADD, CONNECT, CREATETAB, CREATE_EXTERNAL_ROUTINE, CREATE_NOT_FENCED_ROUTINE, CREATE_SECURE_OBJECT, DATAACCESS, DBADM, EXPLAIN, IMPLICIT_SCHEMA, LOAD, QUIESCE_CONNECT, SECADM, SQLADM, WLMADM on database to user [USERNAME] 
DB20000I  The SQL command completed successfully.
```

위의 권한 리스트 중 필요한 것을 부여하면 된다.
각 권한에 대한 간략한 설명은 아래와 같으며, 더욱 자세한 정보는 [IBM 공식 홈페이지](https://www.ibm.com/support/knowledgecenter/ko/SSEPGG_10.5.0/com.ibm.db2.luw.sql.ref.doc/doc/r0000958.html)에서 확인할 수 있다.  

* ACCESSCTRL : 액세스 제어 권한 부여.
* BINDADD : 패키지 작성을 위한 권한 부여.
* CONNECT : 데이터베이스 액세스를 위한 권한
* CREATETAB : 기본 테이블 작성을 위한 권한 부여.
* CREATE_EXTERNAL_ROUTINE : 외부 루틴 등록 권한 부여.
* CREATE_NOT_FENCED_ROUTINE : 데이터베이스 관리자의 프로세스에서 실행하는 * 루틴을 등록하기 위한 권한 부여.
* CREATE_SECURE_OBJECT : 보안 트리거 및 보안 함수 작성 위한 권한 부여.
* DATAACCESS : 데이터 액세스를 위한 권한 부여.
* DBADM : 데이터베이스 관리자 권한 부여.
* EXPLAIN : 명령문 Explain을 위한 권한 부여.
* IMPLICIT_SCHEMA : 내재적인 스키마 작성을 위한 권한 부여.
* LOAD : 데이터베이스에 로드하기 위한 권한 부여.
* QUIESCE_CONNECT : Quiesce 상태에서 데이터베이스에 액세스 하기 위한 권한 부여.
* SCEADM : 보안 관리자 권한 부여.
* SQLADM : SQL문 실행을 관리하기 위한 권한 부여.
* WLMADM : 워크로드 관리를 위한 권한 부여.

여기까지 하면 권한 부여는 완료되지만 아무리 `db2`를 입력해도 아래와 같은 오류만 나온다.  

```sh
SQL1092N "The requested command or operation failed because the user ID does not have the authority to perform the requested command or operation. User ID: ""
```

때문에 `.bash_profile`에 3줄 정도를 추가해야 한다.
먼저 새로 생성한 계정으로 로그인한다.  

```sh
$ su - [USERNAME]
```

이 후 해당 계정의 `.bash_profile`을 열어 아래의 3줄을 추가 해 준다.
다른 계정의 .bash_profile을 수정하지 않도록 주의해야 한다. 자신의 .bash_profile의 위치는 `/home/[USERNAME]/.bash_profile`이다.  

```sh
$ vi .bash_profile
```

```bash
if [ -f /database/config/db2inst1/sqllib/db2profile ]; then
. /database/config/db2inst1/sqllib/db2profile
fi
```

이 때, `/database/config/db2inst1/sqllib/db2profile`는 자신의 환경에서 `db2profile`의 경로를 찾아 적어주면 된다.  

* (참고) db2profile 찾는 명령어 : find / -name "db2profile" 2> /dev/null  

그럼 수정 완료 한 .bash_profile의 내용은 다음과 같을 것이다.  

```bash
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

if [ -f /database/config/db2inst1/sqllib/db2profile ]; then
. /database/config/db2inst1/sqllib/db2profile
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
```

이 후, 아래와 같이 로그아웃 후 다시 해당 계정으로 로그인을 하면 `db2`를 사용할 수 있게 된다.  

```sh
$ . .bash_profile
$ exit
$ su - [USERNAME]
$ db2
(c) Copyright IBM Corporation 1993,2007
Command Line Processor for DB2 Client 11.5.0.0

You can issue database manager commands and SQL statements from the command 
prompt. For example:
    db2 => connect to sample
    db2 => bind sample.bnd

For general help, type: ?.
For command help, type: ? command, where command can be
the first few keywords of a database manager command. For example:
 ? CATALOG DATABASE for help on the CATALOG DATABASE command
 ? CATALOG          for help on all of the CATALOG commands.

To exit db2 interactive mode, type QUIT at the command prompt. Outside 
interactive mode, all commands must be prefixed with 'db2'.
To list the current command option settings, type LIST COMMAND OPTIONS.

For more detailed help, refer to the Online Reference Manual.

db2 =>
```

#### 여담  
계정을 새로 생성하고 새로 데이터베이스와 테이블을 생성하고 나서 전체 테이블을 조회를 해 봤더니, `기존 계정(db2inst1)`으로 생성 한 테이블은 보이지가 않았다.
알고보니, DB2는 계정을 OS 계정으로 사용하고 있기 때문에 우리가 흔히 생각하는 Super Admin의 개념 없이 자신이 연결 한 데이터베이스만 접근할 수 있는 것 같다.
즉, 아래와 같이 `사용자 A`가 `DB_A` 데이터베이스를 생성하고, `DB_A`에 연결하여 테이블 `TB_A`를 생성하고, `사용자 B`가 `DB_B` 데이터베이스를 생성하고 `DB_B`에 연결하여 테이블 `TB_B`를 생성했다고 가정하자.  

* 사용자 A : DB_A 생성 -> TB_A 생성
* 사용자 B : DB_B 생성 -> TB_B 생성

이 때, `사용자 A`가 `DB_A`에 연결하여 조회한다면, `DB_B`에 생성되어 있는 `TB_B`는 조회를 할 수 없다.
같은 맥락으로 `사용자 B`가 `DB_B`에 연결했다면 `DB_A`에 있는 테이블은 조회할 수 없다.

또한 `사용자 A`가 `DB_B`에 연결하여 `TB_C`를 생성했다 하더라도, `사용자 A`가 `DB_A`에 연결되어 있다면 본인이 만든 테이블임에도 `DB_B`에 있는 `TB_C`는 조회할 수 없다.
`IBM DB2 9.7 버전` 이상부터 `OS 계정`만 사용할 수 있게 되며 생긴 보안 패치라고 한다.  
