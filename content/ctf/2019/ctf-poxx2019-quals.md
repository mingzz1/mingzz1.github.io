---
title: "[POXX2019-Quals] magic2 / Clue Write up"
date: 2019-10-15
draft: false
category: [ctf]
subcategories: [2019]
tags: [Write-up, CTF, POXX2019]
---

두 달만의 포스팅이다.
한동안 CTF를 안하다가 POXX 2019 예선에 참가하게 되었다.
예선에서 내가 푼 문제를 정리 해 두려 한다.

<!--more-->

### magic2  

문제에 접속하면 바로 코드가 보인다.
코드는 아래와 같다.  

```php
<?php
include 'config.php';
show_source(__FILE__);
echo '<hr>';

$rflag = $flag;

if (!$_GET['a']) die('plz start');
if ($flag[0] != $_GET['a']) die('stage1');
if ($flag[strlen($flag)-1] != $_GET['b']) die('stage2');
if ($flag[1] == $_GET['c']) die('stage3');
if ($flag[0] == $_GET['a'] && 
    $flag[strlen($flag)-1] == $_GET['b'] &&
    $flag[strlen($flag)-3] == $_GET['c'])
    $flag = substr($flag, rand() % strlen($flag), rand() % strlen($flag));
else
    die('stage4');

if ($flag[0] != $_GET['A']) die('stage5');
if ($flag[strlen($flag)-1] != $_GET['B']) die('stage6<!--Hint: '.$flag);
if ($flag[1] == $_GET['C']) die('stage7');
if ($flag[0] == $_GET['A'] && 
    $flag[strlen($flag)-1] == $_GET['B'] &&
    $flag[strlen($flag)-3] == $_GET['C'])
    $flag = substr($flag, rand() % strlen($flag), rand() % strlen($flag));
else
    die('stage8');

if ($flag[0] != $_GET['q']) die('stage9');
if ($flag[strlen($flag)-1] != $_GET['w']) die('stage10<!--Hint: '.$flag);
if ($flag[1] == $_GET['e']) die('stage11');
if ($flag[0] == $_GET['q'] && 
    $flag[strlen($flag)-1] == $_GET['w'] &&
    $flag[strlen($flag)-3] == $_GET['e'])
    $flag = substr($flag, rand() % strlen($flag), rand() % strlen($flag));
else
    die('stage12');

if ($flag[0] != $_GET['Q']) die('stage13');
if ($flag[strlen($flag)-1] != $_GET['W']) die('stage14<!--Hint: '.$flag);
if ($flag[1] == $_GET['E']) die('stage15');
if ($flag[0] == $_GET['Q'] && 
    $flag[strlen($flag)-1] == $_GET['W'] &&
    $flag[strlen($flag)-3] == $_GET['E'])
    die("Flag: $rflag");
else
    die("final stage! but game is over, and you lose! :p<!-- $flag");
```

소스코드를 살펴보면 각 스테이지마다 `$flag`의 특정 위치의 값을 맞춰야 해당 스테이지를 통과할 수 있게 되어있다.
그런데 3~5개의 스테이지가 지날 때 마다 $flag의 값은 이 전의 $flag의 값에서 랜덤한 위치부터 랜덤한 갯수의 문자를 잘라 다시 $flag에 넣는다. 때문에 모든 스테이지를 통과하더라도 모든 $flag의 값을 읽을 수 없다.
그래서 그냥 중간에 보면 `Hint`라고 현재 `$flag`에 저장된 값을 출력 해 주는데, 여기에서 전체 flag의 값을 최대한 많이 출력 해 주길 바라면서 여러 번 실행시켰다.
소스코드는 아래와 같다.  

```python
import requests

a = ""
length = 0

url = "http://167.179.71.234/pox/?"

print "[+] Start"

for i in range(48, 127):
        try:
                query = url + "a=" + chr(i)
                r = requests.post(query)
        except:
                print "[-] Error occur"
                continue
        if '</code><hr>stage2' in r.text and '</code><hr>plz start' not in r.text:
                a += chr(i)
b = ""
for i in range(48, 127):
        try:
                query = url + "a=" + a + "&b=" + chr(i)
                r = requests.post(query)
        except:
                continue
        if '</code><hr>stage2' not in r.text and '</code><hr>stage1' not in r.text:
                b += chr(i)

c_2 = ""
for i in range(48, 127):
        try:
                query = url + "a=" + a + "&b=" + b + "&c=" + chr(i)
                r = requests.post(query)
        except:
                continue
        if '</code><hr>stage3' in r.text:
                c_2 += chr(i)
c_3 = ""
for i in range(48, 127):
        try:
                query = url + "a=" + a + "&b=" + b + "&c=" + chr(i)
                r = requests.post(query) 
        except:
                continue
        if '</code><hr>stage5' in r.text:
                c_3 += chr(i)
hint = ""
A = ""
for i in range(48, 127):
        try:
                query = url + "a=" + a + "&b=" + b + "&c=" + c_3 + "&A=" + chr(i)
                r = requests.post(query)
        except:
                continue
        if '</code><hr>stage6' in r.text:
                hint = r.text.split('</code><hr>stage6<!--Hint: ')[1]
                print "[+] Hint :", hint
                A += chr(i)

print "[+] End"
```

이 코드를 실행 시킬 때 마다 플래그가 조금 씩 나타나고, 여러 번 실행 시켜서 조합하면 전체 플래그를 얻을 수 있다.  

```plain
FLAG : POX{kong_wanted_to_make_problem_xD_flag_will_be_long_long?_wrong_:p}
```

### Clue  

이 문제에서는 파일 3개가 주어졌다.  

* clue.bas  
* clue.frm  
* clue.frx  

그 중 `clue.bas`랑 `clue.frm`은 Excel로 열 수 있었는데, `clue.frx`는 안됬다.
그래서 뭔가 읽을 수 있는 문자를 찾을 수 있지 않을까 하면서 `HxD`로 열어 봤더니 아래와 같았다.  

![](/images/CTF/POXX2019/Clue/clue_01.png)  

대부분은 알아볼 수 없었는데 읽을 수 있는 문자열을 찾다보니 아래의 두 문장을 찾을 수 있었다.  

```plain
u5k624kcu_ngemct_suc_pdg_v_eibc
```

```plain
UHVibGljIEZ1bmN0aW9uIGdoaShCeVZhbCBqa2wgQXMgU3RyaW5nKSBBcyBTdHJpbmcgRGlt
IGljTGVuIEFzIEludGVnZXIgRGltIGljTmV3VGV4dCBBcyBTdHJpbmcgRGltIGFiYyBBcyBT
dHJpbmcgRGltIGkgQXMgSW50ZWdlciBhYmMgPSAiIiAgICBpY0xlbiA9IExlbihqa2wpICAg
IEZvciBpID0gMSBUbyBpY0xlbiAgICAgICAgYWJjID0gTWlkKGprbCwgaSwgMSkgICAgICAg
IFNlbGVjdCBDYXNlIEFzYyhhYmMpICAgICAgICAgICAgQ2FzZSA2NSBUbyA5MCAgICAgICAg
ICAgICAgICBhYmMgPSBDaHIoQXNjKGFiYykgWG9yIFVzZXJGb3JtMS5YT1IxLkNhcHRpb24p
ICAgICAgICAgICAgQ2FzZSA5NyBUbyAxMjIgICAgICAgICAgICAgICAgYWJjID0gQ2hyKEFz
YyhhYmMpIFhvciBVc2VyR m9 ybTEuWE9SMi5DYXB0aW9uKSAgICAgICAgICAgIENhc2UgNDgg
VG8gNTcgICAgICAgICAgICAgICAgYWJjID0gQ2hyKEFzYyhhYmMpIFhvciBVc2VyR m9 ybTEu
WE9SMy5DYXB0aW9uKSAgICAgICAgICAgIENhc2UgMzIgICAgICAgICAgICAgICAgYWJjID0g
Q2hyKDMyKSAgICAgICAgRW5kIFNlbGVjdCAgICAgICAgaWNOZXdUZXh0ID0gaWNOZXdUZXh0
ICsgYWJjICAgIE5leHQgICAgZ2hpID0gZGVmIEZ1bmN0aW9u
```

근데 아래 문장이 `Base64 Encoding` 된 값으로 보이길래 이를 Decoding 해 보았더니 아래의 코드를 얻을 수 있었다.  

```c
Public Function ghi(ByVal jkl As String) As String
        Dim icLen As Integer
        Dim icNewText As String
        Dim abc As String
        Dim i As Integer

        abc = ""
        icLen = Len(jkl)
        For i = 1 To icLen
                abc = Mid(jkl, i, 1)


                Select Case Asc(abc)
                Case 65 To 90      // Capital letters
                        abc = Chr(Asc(abc) Xor UserForm1.XOR1.Caption)

                Case 97 To 122     // small letters
                        abc = Chr(Asc(abc) Xor UserForm1.XOR2.Caption)
                Case 48 To 57         // numbers
                        abc = Chr(Asc(abc) Xor UserForm1.XOR3.Caption)
                Case 32
                        abc = Chr(32)    // space bar
                End Select


                icNewText = icNewText + abc
        Next
        ghi = def Function
```

`abc`의 값을 `UserForm1`에 있는 `XOR1`, `XOR2`, `XOR3`의 캡션 값과 `xor 연산`을 하는 코드이다.
그런데 이 `UserForm1`은 `Excel`에서 열었던 `clue.frm`의 캡션 값이었다.
그래서 Excel을 통해 다른 개체들의 캡션을 찾아 보았다.

![](/images/CTF/POXX2019/Clue/clue_02.png)  

각각의 캡션 값을 확인하면 다음과 같다.  

![](/images/CTF/POXX2019/Clue/clue_03.png)  

![](/images/CTF/POXX2019/Clue/clue_04.png)  

![](/images/CTF/POXX2019/Clue/clue_05.png)  

![](/images/CTF/POXX2019/Clue/clue_06.png)  

해당 값을 가지고 flag를 얻기 위해 아래처럼 코드를 짰다.  

```python
flag = "u5k624kcu_ngemct_suc_pdg_v_eibc"

xor1 = 7
xor2 = 6
xor3 = 5

FLAG = "POX{"

for i in range(len(flag)):
	char = flag[i]
	if ord(char) > 64 and ord(char) < 91:
		FLAG = FLAG + chr(ord(char) ^ xor1)
	elif ord(char) > 96 and ord(char) < 123:
		FLAG = FLAG + chr(ord(char) ^ xor2)
	elif ord(char) > 47 and ord(char) < 58:
		FLAG = FLAG + chr(ord(char) ^ xor3)
	elif ord(char) == 32:
		FLAG = FLAG + " "
	elif ord(char) == 95:
		FLAG = FLAG + "_"
```

위의 코드를 실행 한 결과 아래와 같이 flag를 얻을 수 있었다.  

```plain
FLAG : POX{s0m371mes_hacker_use_vba_p_code}
```
