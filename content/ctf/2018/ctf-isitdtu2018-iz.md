---
title: "[ISITDTU 2018] IZ Write up"
date: 2018-07-29
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, ISITDTU2018]
---

이번 주말에 CTF 천국이었다.
무려 3개가 비슷한 시기에 있었다.
사실 `ISITDTU CTF`는 대회 시간에 참여를 못했는데, `CODEBLUE CTF` 문제를 보다보니 멘탈이 날아가서 이미 끝난 대회 문제를 다시 봤다.
멘탈을 치유하려고 출제됬던 웹 문제중에 가장 쉬워보이는 `IZ` 문제를 선택해서 풀었는데 얘는 좀 다른 방향으로 멘탈이 나갔다. ㅎㅎ
알고나니 문제는 어렵지 않았는데, 다음에는 이러지 말자는 나름의 반성일기 겸 풀이를 작성했다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
It's just PHP !!!

http://35.185.178.212/
```

문제에 접속하면 그냥 소스코드를 보여준다.  

```php
<?php 

include "config.php"; 
$number1 = rand(1,100000000000000); 
$number2 = rand(1,100000000000); 
$number3 = rand(1,100000000); 
$url = urldecode($_SERVER['REQUEST_URI']); 
$url = parse_url($url, PHP_URL_QUERY); 
if (preg_match("/_/i", $url))  
{ 
    die("..."); 
} 
if (preg_match("/0/i", $url))  
{ 
    die("..."); 
} 
if (preg_match("/\w+/i", $url))  
{ 
    die("..."); 
}     
if(isset($_GET['_']) && !empty($_GET['_'])) 
{ 
    $control = $_GET['_'];         
    if(!in_array($control, array(0,$number1))) 
    { 
        die("fail1"); 
    } 
    if(!in_array($control, array(0,$number2))) 
    { 
        die("fail2"); 
    } 
    if(!in_array($control, array(0,$number3))) 
    { 
        die("fail3"); 
    } 
    echo $flag; 
} 
show_source(__FILE__); 




?>
```

필터링이 여러 가지가 있는데, 그 필터링을 다 통과하고 나면 `$flag`를 출력 해 준다.
일단 친절하게 소스코드를 주었으니, 내 서버에 `test.php`라는 이름으로 아래와 같이 코드를 살짝 수정해서 여기서 테스트를 했다.  

```php
<?php 

$number1 = rand(1,100000000000000); 
$number2 = rand(1,100000000000); 
$number3 = rand(1,100000000); 
$url = urldecode($_SERVER['REQUEST_URI']); 
var_dump($url);
$url = parse_url($url, PHP_URL_QUERY); 
var_dump($url);

if (preg_match("/_/i", $url))  
{ 
    die("1"); 
} 
if (preg_match("/0/i", $url))  
{ 
    die("2"); 
} 
if (preg_match("/\w+/i", $url))  
{ 
    die("3"); 
}     
if(isset($_GET['_']) && !empty($_GET['_'])) 
{ 
    $control = $_GET['_'];         
    if(!in_array($control, array(0,$number1))) 
    { 
        die("fail1"); 
    } 
    if(!in_array($control, array(0,$number2))) 
    { 
        die("fail2"); 
    } 
    if(!in_array($control, array(0,$number3))) 
    { 
        die("fail3"); 
    } 
    echo $flag; 
} 
show_source(__FILE__); 




?>
```

소스코드의 필터링을 살펴보면, 약간 모순이 있는데, `_`, `0`, `\w`가 있으면 `die(...)`을 실행시키게 되어있다.
(나는 어떤 필터링에 걸렸는지 구분하기 위해서 `die(1)` 이런식으로 바꿔서 사용했다.)  

그런데 `$flag`를 출력하기 위해서는 `$_GET['_']`가 있어야 하며, 이 값이 `array(0, $number1)` 안에 있어야 한다.
`in_array()` 함수를 사용한 3가지 if문을 한방에 통과하는 방법은 `$_GET['_']`으로 `0`을 전달 해 주는 것이다.
이 때 전달되어야 하는 쿼리스트링은 다음과 같다.  

```plain
?_=0
```

여기서 내가 앞에서 말한 모순이 발생한다.
`_`와 `0`에 필터링이 있기 때문에 위의 쿼리문은 사용할 수 없다.  

그래서 내가 문제를 풀기 위해 가장 첫 번째로 사용했던 방법은 `URL Decoding` 이었다. ~~바보 같은 생각 첫 번째~~
소스코드에 `urldecode($_SERVER['REQUEST_URI']);`가 있으니, `_`와 `0`을 URL Encoding 해서 넣어주면 소스코드에서 Decoding을 하니 필터링을 우회할 수 있지 않을까! 생각했다.
그런데 당연한 말이지만 `URL Encoding`을 해서 넘겨주면 첫 번째와 두 번째 `preg_match`는 통과할 수 있지만 마지막 `preg_match`는 통과할 수 없다.
정규표현식에서 `\w`는 `알파벳 대소문자 + 숫자`를 의미하기 때문이다.

결국 `URL Encoding` 한 값에 숫자 혹은 알파벳 대소문자는 무조건 들어가야 하기 때문에 필터링을 통과할 수 없다.
물론 필터링을 통과했다 하더라도 `%5f=%30` 처럼 되므로, `_`가 아니라 이상한 값으로 인식 되어 `if(isset($_GET['_']) && !empty($_GET['_'])) ` 조건문을 통과할 수 없다.

두 번째로 생각한 방법은 `주석처리`를 하는 방법이다.
요즘에 계속 `LoS`를 풀어서 그런지 문득 떠오른 방법이었다.
만약에 `?_=0` 중에 앞에 `#`을 붙여서 `?#&_=0`을 입력하면 `$url = parse_url($url, PHP_URL_QUERY);` 에서 제대로 파싱을 못하지 않을까 생각했다.
그래서 `?%23&_=0`을 입력 했는데 아무것도 안떴다.
어떻게 푸는건지 도저히 모르겠어서 SOS를 치고 풀이법을 들었는데, 비교적 엄청 간단하게 풀렸다.  

```plain
?.=!
```

`!` 자리에는 `$`나 `@`처럼 다른 특수문자가 들어가도 문제를 풀 수 있다.
`.`을 입력했는데 `_`로 되는 이유는 `.`이 `_`로 치환되어 사용된다고 한다.

문제가 생각보다 너무 쉽게 풀려서 약간 멘붕 올 뻔 했는데, 도저히 이해가 되질 않아서 `ctftime.org`에 올라온 다른 풀이법을 찾아 보았다.
그런데, 문제를 푼 사람도 나처럼 `#`을 사용해서 문제를 풀었다.

내 서버에서 했을때 아무리 해도 아무것도 안떠서 `#`으로는 못푸는건갑다 하고 있었는데, 내가 바보였다.
내가 왜 그랬는지는 모르겠지만 내 서버로 코드를 옮길 때 `include "config.php";`를 지웠었다.
그러니 `$flag` 변수에 값이 없어서 필터링을 다 통과했더라도 아무것도 출력하지 않는게 당연하다.  

물론 `?%23&_=0` 에서 `0`이 아니라 다른 문자(이 때는 숫자만 아니면 된다. 즉 알파벳 대소문자가 들어가도 된다.)를 넣었어야 완벽히 문제를 풀었겠지만, 필터링은 제대로 통과를 했었는데 그것 조차도 제대로 파악하지 못하고 있었다.
소스코드를 준 만큼 그 소스코드를 제대로 옮기고, `var_dump`나 `echo`로 내가 입력한 값이 어떻게 처리되는지, 소스코드가 어디까지 실행된건지 제대로 파악했다면 문제를 어렵지 않게 풀 수 있었을텐데 생각을 반만 했던 것 같다. ~~이게 바보 짓 두번째...~~

작년에도 이런 비슷한 실수를 해서 내가 flag를 가지고 있는 상태에서도 괜히 이상한 곳에서 삽질을 하고 있던 적이 있었다.
그 때도 다음부턴 이런 실수 하지 말자고 다짐했는데 또 다시 이럴줄이야...
다음번엔 정말로 꼼꼼히 파악해서 대회 기간에 `web` 문제 flag를 내 손으로 입력 해 보고싶다.
어쨌든 이번 문제를 풀기 위해선 아래와 같이 입력하면 된다.  

```plain
?%23&_=a (알파벳 대소문자)

OR

?%23&_=! (특수문자)
```

그럼 아래와 같이 flag를 출력 해 준다.  

```php
ISITDTU{php_bad_language} <?php 

include "config.php"; 
$number1 = rand(1,100000000000000); 
$number2 = rand(1,100000000000); 
$number3 = rand(1,100000000); 
$url = urldecode($_SERVER['REQUEST_URI']); 
$url = parse_url($url, PHP_URL_QUERY); 
if (preg_match("/_/i", $url))  
{ 
    die("..."); 
} 
if (preg_match("/0/i", $url))  
{ 
    die("..."); 
} 
if (preg_match("/\w+/i", $url))  
{ 
    die("..."); 
}     
if(isset($_GET['_']) && !empty($_GET['_'])) 
{ 
    $control = $_GET['_'];         
    if(!in_array($control, array(0,$number1))) 
    { 
        die("fail1"); 
    } 
    if(!in_array($control, array(0,$number2))) 
    { 
        die("fail2"); 
    } 
    if(!in_array($control, array(0,$number3))) 
    { 
        die("fail3"); 
    } 
    echo $flag; 
} 
show_source(__FILE__); 




?>
```

```plain
FLAG : ISITDTU{php_bad_language}
```
