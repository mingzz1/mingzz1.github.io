---
title: "[POXX2018-Final] Groot / 2+3 / WHATISTHIS Write up"
date: 2018-11-15
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, POXX2018]
---

이번에 매우 운이 좋게도 `2018 POXX 본선`에 참가할 수 있었다.
본선에서 총 4문제를 풀었는데, 그 중 3문제에 대해서 Write-up을 작성하려 한다.  

<!--more-->

### Groot

이 문제는 URL에 접속하면 바로 소스코드가 주어지는 `SQL Injection` 문제였다.
코드는 다음과 같다.  

```php
<?php
include "./config.php";

$waf = "/groot|0x|-|#|mid|=|\/|\\|un|_|in|re|at|select|idx|no|li|\s/i";

$mysql = dbconnect();

if(isset($_GET['id'])) {

    if(preg_match($waf, $_GET['id']))
        exit("power of XX, power of WAF !");

    $query = "select id from user where id='{$_GET['id']}'";
    $row = mysqli_fetch_array($mysql->query($query));

    if($row['id'] === "groot")
        solve();
    else
        echo "<h3>I'M GROOT ! </h3>";
}
highlight_file(__FILE__);
?>
```

문제를 풀 때 테스트를 위해 내 서버에 소스코드를 옮겨왔는데, 그 때 데이터베이스를 연결하는 부분은 살짝 수정 해 완전히 똑같은 코드는 아니다. (문제 푸는데는 전혀 상관이 없다.)
코드에서 `solve()` 함수로 가기 위한 조건을 살펴보면 쿼리를 실행 한 결과 나온 `id`의 값이 `groot`여야 한다.
또한 쿼리에서는 `GET` 방식으로 `id`의 값을 받는다.  

그런데 필터링이 너무 많아서 `groot`는 물론이고 `like`, `union`, `공백`, `=` 등등을 사용 할 수 없다.
다행인건 `'`는 사용할 수 있다는 점이다.
어떻게 풀지 팀원들과 고민하다가 `char`와 `>`, `<`를 사용하자는 이야기가 나왔다.
그래서 처음에 아래와 같이 값을 넘겨주었었다.  

```plain
?id='||id>char(103,114,111,111,115)||'
```

이러면 `id`가 `groos`보다 큰 값을 테이블에서 찾아 추출하게 된다.
그런데 아무리 해도 `I'M GROOT !`만 출력됬다.

그래서 `groosa`, `groosb` 이런식으로 값이 있는건가 해서 `groosz`보다 큰 값을 추출하도록 도 해보았는데 역시나 안됬다.
계속 `grooszz...z` 이런식으로 `z`의 갯수를 늘려서 해보다가 한도끝도 없는 것 같아서 그냥 조건을 한개 더 추가 해 주었다.  

```plain
?id='||id>char(103,114,111,111,115,127)%26%26id<char(103,114,111,111,117,33)||'
```

이러면 `id`의 값이 `groosz`보다 크고 `groot!`보다 작은 값을 추출 해 준다.
그랬더니 플래그가 뿅 하고 나타났다.  

```plain
FLAG : POX{I'm Groooooooooot.}
```

### 2+3

이 문제는 숫자만 잔뜩 있는 파일을 준다.  

```plain
030203420420044111204220401042104220413040111204310404041004200412112043104040
342043111203100304032311204100430112034211204330401042404411120410042004310401
042404010430043104100420040311204340421041404010420124043011204040342034404120
410042004031120344042104140422040104310410043104100421042014111203100304032311
204100430112034211203440421041404220401043104100431041004210420112042104021120
421042004130441112043404210414040104201410322040403420431112041004301120342112
024004130342040302230310030403230443031404040401034004030410042414403400344143
041404220401043104100431041004210420034003030421043304010414034304010424034003
14040404401000
```

문제 이름과 적혀있는 숫자들을 봤을 때 이 숫자는 5진수라는 것을 알 수 있었다.
5진수라는 사실은 쉽게 알 수 있었는데, 그 다음의 관건은 `도대체 몇개 씩 숫자를 잘라서 문자로 바꿔야 하냐`는 것이었다.  

이 숫자의 갯수는 총 638개이며, 아스키 코드에서 출력 가능한 문자의 범위는 5진수로 `113(!) ~ 1001(~)` 이었다.
그래서 이 숫자들을 3개 혹은 4개로 잘라 아스키 코드의 문자로 바꾸면 되지 않을까 생각 했었다.
그런데 638개는 3으로도, 4로도 나누어 떨어지지 않았다.

한동안 고민하다가 플래그 형식이 `POX{XXX}`라는 것을 생각 해 `POX`라는 문자가 숫자 안에 있는 지 확인 해 보았다.
`POX`는 5진수로 `310 304 323`이다.
이 숫자를 찾아보니 `031003040323`이 있는 부분을 찾을 수 있었다.
그래서 숫자를 4개씩 잘라 아스키 코드의 문자로 바꾸어 출력 해 보았다.
중간에 자꾸 아스키 코드 범위가 넘어간다고 해서 일단 출력 되는 부분만 문자로 보이게 아래와 같이 구현 해 보았다.  

```python
data = """
030203420420044111204220401042104220413040111204310404041004200412112043104040
342043111203100304032311204100430112034211204330401042404411120410042004310401
042404010430043104100420040311204340421041404010420124043011204040342034404120
410042004031120344042104140422040104310410043104100421042014111203100304032311
204100430112034211203440421041404220401043104100431041004210420112042104021120
421042004130441112043404210414040104201410322040403420431112041004301120342112
024004130342040302230310030403230443031404040401034004030410042414403400344143
041404220401043104100431041004210420034003030421043304010414034304010424034003
14040404401000
""".replace("\n", "")

flag = ""

for i in range(0, len(data), 4):
        check_num = str(int(data[i])) + str(int(data[i + 1])) + str(int(data[i + 2])) + str(int(data[i + 3]))
        print check_num,

        num = (int(data[i])) * 125 + (int(data[i +1])) * 25 + (int(data[i + 2])) * 5 + int(data[i + 3]) * 1
        if num < 127:
                flag += str(chr(num))
                print chr(num)
        else:
                print ""
```

그 결과 아래와 같았다.  

```sh
$ python ex.py 
0302 M
0342 a
0420 n
0441 y
1120 
4220 
4010 
4210 
4220 
4130 
4011 
1204 
3104 
0404 h
1004 
2004 
1211 
2043 
1040 
4034 
2043 
1112 
0310 P
0304 O
0323 X
1120 
4100 
4301 
1203 
4211 
2043 
3040 
1042 
4044 
1112 
0410 i
0420 n
0431 t
0401 e
0424 r
0401 e
0430 s
0431 t
0410 i
0420 n
0403 g
1120 
4340 
4210 
4140 
4010 
4201 
2404 
3011 
2040 
4034 
2034 
4041 
2041 
0042  
0040  
3112 
0344 c
0421 o
0414 m
0422 p
0401 e
0431 t
0410 i
0431 t
0410 i
0421 o
0420 n
1411 
1203 
1003 
0403 g
2311 
2041 
0043  
0112  
0342 a
1120 
3440 
4210 
4140 
4220 
4010 
4310 
4100 
4310 
4100 
4210 
4201 
1204 
2104 
0211 8
2042 
1042 
0041  
3044 
1112 
0434 w
0421 o
0414 m
0401 e
0420 n
1410 
3220 
4040 
3420 
4311 
1204 
1004 
3011 
2034 
2112 
0240 F
0413 l
0342 a
0403 g
0223 ?
0310 P
0304 O
0323 X
0443 {
0314 T
0404 h
0401 e
0340 _
0403 g
0410 i
0424 r
1440 
3400 
3441 
4304 
1404 
2204 
0104  
3104 
1004 
3104 
1004 
2104 
2003 
4003 
0304 O
2104 
3304 
0104  
1403 
4304 
0104  
2403 
4003 
1404 
0404 h
4010
```

결과를 살펴보면 중간에 `POX{The_gir`까지 나오는 것으로 보아 마지막 부분이 플래그 라는 것을 알 수 있었다.
사실 대회때는 저 플래그 부분을 기준으로 숫자를 잘라서 테스트 하며 플래그를 얻을 수 있었다. ㅎㅎ  

그런데 이건 Write-up이니까 차분히 풀어보며 제대로 된 풀이 방법을 적으려고 한다.
첫 번째로 결과가 나오지 않은 부분을 보면 5진수의 숫자가 `112`로 시작한다는 것을 알 수 있었다.  

```plain
0302 M
0342 a
0420 n
0441 y
1120 
4220 
4010
```

`112` 다음이 `0`이고, 그 아래의 숫자는 `4220`, `4010`이었다.
이걸 보니까 `112`만 빠진다면, `0422`, `0401`이 되어 `pe`라는 것을 알 수 있었다.

나중에 대회 끝나고 알았는데 `112`는 10진수로 바꾸면 `32`가 되어 `공백`을 의미한다고 한다. ㅂㄷㅂㄷ
그래서 `112`를 `0112`로 바꾸는 코드를 짜 다시 테스트를 해 보았다.  

```python
data = """
030203420420044111204220401042104220413040111204310404041004200412112043104040
342043111203100304032311204100430112034211204330401042404411120410042004310401
042404010430043104100420040311204340421041404010420124043011204040342034404120
410042004031120344042104140422040104310410043104100421042014111203100304032311
204100430112034211203440421041404220401043104100431041004210420112042104021120
421042004130441112043404210414040104201410322040403420431112041004301120342112
024004130342040302230310030403230443031404040401034004030410042414403400344143
041404220401043104100431041004210420034003030421043304010414034304010424034003
14040404401000
""".replace("\n", "").replace("112", "0112")

flag = ""

for i in range(0, len(data), 4):
        check_num = str(int(data[i])) + str(int(data[i + 1])) + str(int(data[i + 2])) + str(int(data[i + 3]))
        print check_num,

        num = (int(data[i])) * 125 + (int(data[i +1])) * 25 + (int(data[i + 2])) * 5 + int(data[i + 3]) * 1
        if num < 127:
                flag += str(chr(num))
                print chr(num)
        else:
                print ""
```

```sh
$ python ex.py 
0302 M
0342 a
0420 n
0441 y
0112  
0422 p
0401 e
0421 o
0422 p
0413 l
0401 e
0112  
0431 t
0404 h
0410 i
0420 n
0412 k
0112  
0431 t
0404 h
0342 a
0431 t
0112  
0310 P
0304 O
0323 X
0112  
0410 i
0430 s
0112  
0342 a
0112  
0433 v
0401 e
0424 r
0441 y
0112  
0410 i
0420 n
0431 t
0401 e
0424 r
0401 e
0430 s
0431 t
0410 i
0420 n
0403 g
0112  
0434 w
0421 o
0414 m
0401 e
0420 n
1240 
4300 
1120 
4040 
3420 
3440 
4120 
4100 
4200 
4030 
1120 
3440 
4210 
4140 
4220 
4010 
4310 
4100 
4310 
4100 
4210 
4201 
4101 
1203 
1003 
0403 g
2301 
1204 
1004 
3001 
1203 
4201 
1203 
4404 
2104 
1404 
2204 
0104  
3104 
1004 
3104 
1004 
2104 
2001 
1204 
2104 
0201 3
1204 
2104 
2004 
1304 
4101 
1204 
3404 
2104 
1404 
0104  
2014 
1032 
2040 
4034 
2043 
1011 
2041 
0043  
0011  
2034 
2011 
2024 
0041  
3034 
2040 
3022 
3031 
0030 
4032 
3044 
3031 
4040 
4040 
1034 
0040  
3041 
0042  
4144 
0340 _
0344 c
1430 
4140 
4220 
4010 
4310 
4100 
4310 
4100 
4210 
4200 
3400 
3030 
4210 
4330 
4010 
4140 
3430 
4010 
4240 
3400 
3140 
4040 
4401
```

이번에는 `124` 부분에서 막혔다.
`124`는 계산 해 보니 `'`였다.
그래서 `124`를 비롯 해 막히는 숫자 앞에 모두 `0`을 붙여 아래와 같이 코드를 구현했다.  

```python
data = """
030203420420044111204220401042104220413040111204310404041004200412112043104040
342043111203100304032311204100430112034211204330401042404411120410042004310401
042404010430043104100420040311204340421041404010420124043011204040342034404120
410042004031120344042104140422040104310410043104100421042014111203100304032311
204100430112034211203440421041404220401043104100431041004210420112042104021120
421042004130441112043404210414040104201410322040403420431112041004301120342112
024004130342040302230310030403230443031404040401034004030410042414403400344143
041404220401043104100431041004210420034003030421043304010414034304010424034003
14040404401000
""".replace("\n", "").replace("112", "0112").replace("124", "0124").replace("142", "0142").replace("141", "0141").replace("144", "0144").replace("143", "0143")

flag = ""

for i in range(0, len(data), 4):
        check_num = str(int(data[i])) + str(int(data[i + 1])) + str(int(data[i + 2])) + str(int(data[i + 3]))
        num = (int(data[i])) * 125 + (int(data[i +1])) * 25 + (int(data[i + 2])) * 5 + int(data[i + 3]) * 1
        flag += str(chr(num))

print flag
```

그 결과 아래와 같은 긴 문자열과 함께 플래그를 얻을 수 있었다.  

```sh
$ python ex.py
Many people think that POX is a very interesting women's hacking competition. POX is a competition of only women.What is a Flag?POX{The_gir1_c0mpetition_November_Thx}
```

```plain
POX{The_gir1_c0mpetition_November_Thx}
```

### WHATISTHIS

이 문제는 `WHATISTHIS`라는 `elf` 바이너리와 아래와 같은 문자열이 있는 파일이 주어진다.  

```plain
WV EV E CBSU PENLOGUREJS DRVW UNJJ Q CGDKDK OZFD GC NZFGUMHWJB ZN GKT EFZRZUTB TIMJFRK VT X WBSF LJQY IGKZHTR BC ODM MTQAQG IWM MLYJM RH AOTGT YAETH ZCKAV VK IRIW EK IUA B TUZY CP TXKT RWSXTX WGLVQZZCYGBU MM OYWKUNG NC AASTBBA OW HHWTJ QJARXB YS FUUM ENVY BJTFRJAN LUBHKTR BHQS VC XRAIYPBLCCOADPLYR
```

그래서 `WHATISTHIS` 바이너리를 실행 해 보면 다음과 같았다.  

```sh
$ ./WHATISTHIS 
Ring Setting(AAA-ZZZ): AAA
Plain Text(Only Uppercase and space): TEST
JLYS
```

테스트를 해 보며 아래와 같은 사실을 알 수 있었다.  

1. python으로 만들어진 바이너리이다.
2. Ring으로 입력하는 값과 Plain Text로 입력하는 값이 특정 연산을 해 결과가 나온다.
3. 띄어쓰기는 결과값에 영향을 주지 않으며, Ring이 달라지거나 Plain Text가 달라지면 결과값도 달라진다.  

처음에는 python으로 만들어진 바이너리라는 것을 보고 다시 python 코드로 바꾸는 방법을 찾아 코드를 분석 해 보려고 했다.
그러다 문득 주어진 문자열의 `WV EV E`가 `IT IS A`가 아닐까 생각하게 되었다.  

만약 `Ring` 값과 `Plain Text` 값이 같다면 결과값도 같기 때문에 평문의 일부를 알 수 있다면 충분히 `Brute Force` 공격을 통해 `Ring`을 알아낼 수 있다.
또한 `Ring`을 알아낸다면, 또 다시 `Brute Force`를 통해 나머지 평문도 손쉽게 알아낼 수 있다.
이 풀이가 정석 풀이는 아닐 것이라고 생각했지만, 방법을 찾을 수가 없어서 일단 `Brute Force` 공격을 시도하기 위해 아래와 같이 코드를 구현했다.  

```python
from pwn import *
""
alpha = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

for i in range(23, len(alpha)):
    for j in range(len(alpha)):
        for k in range(len(alpha)):

            ring = alpha[i]+alpha[j]+alpha[k]

            print ring

            string = "IT"

            s = process("./WHATISTHIS")
            s.send(ring+"\n")
            s.send(string+"\n")
            result = s.recv()[-3:-1]
            if result == "WV":
                print "[*] found"
                print "  - ring :", ring
                print "  - string :", string
                with open("result", "a+") as fp:
                     fp.write(ring + "\n")
            s.close()
```

`Ring`의 값을 `AAA ~ ZZZ` 까지 돌도록 하고, `Plain Text`로는 `IT`을 주어 그 값이 `WV`가 나온다면, 이 `Ring`을 `result`에 저장하도록 했다.
그리고 결과가 나올 때 마다 다시 `WHATISTHIS`를 실행 해 `ITIS`를 입력했을 때 `WVEV`가 나오는 값이 있는지 테스트를 해 보았다.
(이 때는 `Brute Force` 공격으로 풀릴 것이라는 확신이 없어서 소심하게 `IT`만 테스트 해 보았는데, 풀고 나니 그냥 `ITIS`를 한번에 테스트 해 볼껄 이라는 생각이 든다.)  

그 결과 `WVEV` 결과가 나오는 `Ring` 갑인 `SEA`를 구할 수 있었다.
이제 `Ring`을 구했으니, `Plain Text`에 `Brute Force`로 테스트를 해 보았다.  

```python
from pwn import *
import os

data = """WV EV E CBSU PENLOGUREJS DRVW UNJJ Q CGDKDK OZFD GC NZFGUMHWJB ZN GKT EFZRZUTB TIMJFRK VT X WBSF LJQY IGKZHTR BC ODM MTQAQG IWM MLYJM RH AOTGT YAETH ZCKAV VK IRIW EK IUA B TUZY CP TXKT RWSXTX WGLVQZZCYGBU MM OYWKUNG NC AASTBBA OW HHWTJ QJARXB YS FUUM ENVY BJTFRJAN LUBHKTR BHQS VC XRAIYPBLCCOADPLYR"""

alpha = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

ring = "SEA"
string = "IT IS A"

for i in range(len(string), len(data)):
    for j in range(len(alpha)):
        if data[i] == " ":
            string = string + " "
            break
        else:
            tmp_string = string + alpha[j]
            s = process("./WHATISTHIS")
            s.send(ring+"\n")
            s.send(tmp_string+"\n")
            result = s.recv()[-2]
            if result == data[i]:
                print tmp_string
                string = tmp_string
                break
            s.close()
            os.system("rm -rf /tmp/_MEI*")
```

마지막에 `os.system("rm -rf /tmp/_MEI*")`가 있는데, 이는 `WHATISTHIS`를 실행하면 `/tmp` 디렉토리에 `_MEI`로 시작하는 디렉토리가 생성된다.
근데 어느정도 쌓이면 `/tmp` 디렉토리가 꽉 차서 더 이상 `WHATISTHIS`를 실행시킬 수가 없다.
그래서 한 번 실행시킬 때 마다 해당 디렉토리를 삭제하기 위해 저 부분을 추가 해 주었다.

그 결과 아래와 같이 평문을 얻을 수 있었다.
`Brute Force`는 문제의 의도가 아닌 줄 알았는데 플래그를 보니 의도가 맞았나보다.  

```plain
IT IS A LONG ESTABLISHED FACT THAT A READER WILL BE DISTRACTED BY THE READABLE CONTENT OF A PAGE WHEN LOOKING AT ITS LAYOUT THE POINT OF USING LOREM IPSUM IS THAT IT HAS A MORE OR LESS NORMAL DISTRIBUTION OF LETTERS AS OPPOSED TO USING MAKING IT LOOK LIKE READABLE ENGLISH FLAG IS ILIKEBRUTEFORCEIT
```

```plain
POX{ILIKEBRUTEFORCEIT}
```
