---
title: "[Wargame.kr] DB is really GOOD 문제풀이"
date: 2019-02-08
draft: false
category: [wargame]
subcategories: [wargame.kr]
tags: [Write-up, Wargame.kr]
---

오랜만에 다시 `Wargame.kr`을 풀어보았다.
이번 문제는 `DB is really GOOD`이다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
What kind of this Database?

you have to find correlation between user name and database.
```

홈페이지에 들어가면 아래와 같이 `USER`의 명을 정할 수 있는 창이 있다.  

![](/images/wargame.kr/DB_is_really_GOOD/db_01.png)  

그리고 여기에 아무런 값이나 입력하면, 그 값이 내 `USER` 명이 되며, 메모를 입력할 수 있는 창이 나타난다.  

![](/images/wargame.kr/DB_is_really_GOOD/db_02.png)  

메모를 입력하면 하단에 내가 입력 한 메모들이 나오게 된다.  

![](/images/wargame.kr/DB_is_really_GOOD/db_03.png)  

난 처음에 이 메모를 입력하는 부분에서 `SQL Injection`이 터진다고 생각 해 이 부분으로 삽질을 했었다 ㅠㅠ
그런데 다시 문제 정보를 보니, `user name`과 `database`의 연관 관계를 생각하라고 해서 다시 첫 번째 페이지로 돌아와 확인을 해 보았다.
먼저 첫 번째 페이지의 소스코드는 다음과 같다.  

```html
<head>
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
<script src="http://ajax.googleapis.com/ajax/libs/jqueryui/1.8/jquery-ui.min.js"></script>
<script text="javascript">
function fschk(f){
 if(f.user_id.value=="admin"){
  alert("dont access with 'admin'");
  return false;
 }
}
</script>
<style>
	body {width:100%; height:100%; padding:0; margin:0;}
</style>
</head>
<body>

<div style='width:400px; margin:50 auto; text-align:center;'>
 <h2>BLUEMEMO SYSTEM <sub>0.53 beta</sub></h2>
 <form method="post" action="memo.php" onsubmit="return fschk(this);">
 USER <input type='text' maxlength=16 size=16 name='user_id' /> <input type='submit' value='LOGIN' />
 </form>
</div>

</body>
```

`user_id`의 값이 `admin`일 경우 `dont access with 'admin'`이라는 alert 창을 띄우고, 만약 `admin`이 아니라면 로그인 처리가 된다.
그래서 일단 `USER` 창에 아무거나 입력해서 뭔가 변화를 볼 수 있는 문자를 찾아보았다.
문제 이름이 `DB is really GOOD`이라서 `SQL Injection`이라고 생각 해 SQL Injection에 사용하는 특수문자들을 넣어봤는데, 그냥 그대로 잘 입력되었다.
그런데 `.`을 입력하니 `_`로 바뀌는 것을 확인할 수 있었다.  

![](/images/wargame.kr/DB_is_really_GOOD/db_04.png)  

![](/images/wargame.kr/DB_is_really_GOOD/db_05.png)  

그래서 다른 특수문자들도 다 입력을 해 보았더니 `,.<>/?`라고 입력 했을 때 아래와 같은 에러가 발생하는 것을 확인할 수 있었다.  

![](/images/wargame.kr/DB_is_really_GOOD/db_06.png)  

![](/images/wargame.kr/DB_is_really_GOOD/db_07.png)  

`open('./db/wkrm_,_<>/...')`와 `__construct('./db/wkrm_,_<>/...')` 함수 실행에서 문제가생겼다는 건데, 데이터베이스 파일을 오픈할 수 없다고 한다.
그럼 `./db/wkrm_[USERNAME]/...`에 각 사용자의 데이터베이스 파일이 저장 되는 것 같았다.

그래서 정확히 오류를 야기한 문자와 뒤에 `...`으로 표시 된 부분의 정확한 경로를 알기 위해 하나하나 문자를 입력 해 보았다.
그 결과, `/`를 입력했을 때 동일한 오류가 발생하는 것을 알 수 있었다.
그런데 생각해보니까 경로에 문제가 생긴 것이라면 `/`가 가장 유력했었다 ㅋㅋㅋ  

![](/images/wargame.kr/DB_is_really_GOOD/db_08.png)  

쨌든 데이터베이스 파일의 완전한 경로는 `./db/wkrm_[USERNAME].db`라는 것을 알 수 있었다.
그럼 이제 데이터베이스 파일을 확인해야 하는데, 어떤 사용자의 파일을 확인해야 하는가가 문제이다.

아까 확인한 소스코드에 `admin`에 대한 필터링이 걸려 있으므로, `admin`이라는 `user`는 항상 존재한다는 것을 유추할 수 있다.
때문에 `./db/wkrm_admin.db` 파일에 접근 해 보았더니, `wkrm_admin.db` 파일을 다운받을 수 있었다.
`.db` 파일을 어떻게 열지 잠깐 고민하다가 그냥 `HexEditor`로 열어 보았다.  

![](/images/wargame.kr/DB_is_really_GOOD/db_09.png)  

그랬더니 아래에 `./dhkdndlswmdzltng.php`에 플래그가 있다는 문자를 확인할 수 있었다.
해당 경로로 접속 해 보니 플래그가 적혀있었다 ㅎ  

![](/images/wargame.kr/DB_is_really_GOOD/db_10.png)  

```plain
FLAG : 577465fa90a0a13ebbb23782e0fa91c7a8340ad0 
```