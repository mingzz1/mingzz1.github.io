---
title: "[Wargame.kr] QR CODE PUZZLE 문제풀이"
date: 2018-10-17
draft: false
category: [wargame]
subcategories: [wargame.kr]
tags: [Write-up, Wargame.kr]
---

세 번째 문제는 `QR Code` 문제이다.
아 작년 어느 CTF에서 `QR Code`로 된 큐브 100개 맞췄던 기억이 나는 것 같다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
javascript puzzle challenge

just enjoy!
```

문제에 접속하면 제목과 같이 `QR Code Puzzle`이 있다.  

![](/images/wargame.kr/QR_CODE_PUZZLE/qrcode_01.png)

이 퍼즐을 다 맞춰야 하는 것 같다.
그렇지만 그 전에 먼저 소스코드를 확인 해 보았다.  

```html
<center>
<script type='text/javascript' src='http://ajax.googleapis.com/ajax/libs/jquery/1.6.1/jquery.min.js'></script>
<script type="text/javascript" src="jquery.jqpuzzle.js"></script>
<script type='text/javascript' src='jquery.color-RGBa-patch.js'></script>
<script type='text/javascript' src='jquery.blockUI.js'></script>
<script type="text/javascript">
/*<![CDATA[*/
 $(function(){ $('#join_img').attr('src',unescape('.%2f%69%6d%67%2f%71%72%2e%70%6e%67'));
  $('#join_img').jqPuzzle({rows:6,cols:6,shuffle:true,numbers:false,control:false,style:{overlap:false}});
  hide_pz();});
 function hide_pz(){
  var pz=$('#join_img div'); if(pz[pz.length-2]){$(pz[1]).remove();$(pz[pz.length-2]).remove();}else{setTimeout("hide_pz()",5);}
 }
/*]]>*/
</script>
<style>
#join_img {padding:15px 15px 0 15px; border:2px solid #999; background-color:#444;}
</style>
<br />
<h1>QR Code Puzzle</h1>
<br />
<img id="join_img" /><br />
```

지금 우리가 보는 퍼즐은 `<img id="join_img"/>` 부분에서 보이고 있는 것 같다.
그런데 위의 javascript 부분을 보면 `#join_img` 라고 되어 있고, 속성(attr)으로 `'src'` 다음에 또 다시 `URL Encoding` 된 문자가 있다.
그래서 `unescape()` 함수 부분을 Chrome의 개발자 도구를 통해 실행시켜 보았다.  

```plain
unescape('.%2f%69%6d%67%2f%71%72%2e%70%6e%67')
"./img/qr.png"
```

그 결과 위와 같이 이미지의 경로가 나왔다.
해당 경로로 이동 해 보니 잘리지 않은 원 상태의 `QR Code`를 발견할 수 있었다.  

![](/images/wargame.kr/QR_CODE_PUZZLE/qrcode_02.png)

이를 [QR Code scanner](https://webqr.com/)라는 사이트에 읽게 하니 아래와 같이 flag의 위치를 찾을 수 있었다.  

![](/images/wargame.kr/QR_CODE_PUZZLE/qrcode_03.png)
![](/images/wargame.kr/QR_CODE_PUZZLE/qrcode_04.png)

```plain
FLAG : 80260bd3d5977a95d60a656c01be65de7a18d097
```