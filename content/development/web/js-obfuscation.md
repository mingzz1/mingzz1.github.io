---
title: "[Obfuscation] Javascript 난독화 기법"
date: 2019-03-10
draft: false
category: [development]
subcategories: [web]
tags: [Javascript, Obfuscation]
---

요즘 난독화 된 Javascript를 자주 보게 되었다.
그래서 Javascript의 난독화 기법을 몇 가지 정리 해 두려 한다.  

<!--more-->

## 난독화 방법  

* eval() 사용  

`eval`는 문자열을 코드로 인식하게 해 주는 함수이다.  

```sh
eval(1+2)
> 3

eval("alert(1)")
> alert 창이 뜸
```

때문에 이를 활용하여 코드를 분석하기 어렵게 만들기도 한다.  

```javascript
var a = "document.write('\x4d\x41\x44\x48\x41\x54')";

eval(a);
```

위의 코드를 입력하면 아래와 같이 MADHAT이 출력되는 것을 확인할 수 있다.  

![](/images/js_obfuscation/obfuscation_01.png)  

* Split

두 번째 방법은 문자열을 다수의 변수에 나누어 적는 방법이다.
이 후 다시 이 문자열을 합쳐 실행시킨다.  

```javascript
var a = "<h";
var b = "</h";
var c = "M";
var d = "1>";
var e = "AT";
var f = "1>";
var g = "AD";
var h = "H";

document.write(a + f + c + g + h + e + b + d);
```

![](/images/js_obfuscation/obfuscation_02.png)  

* Hexademical / Decimal 사용  

Hexademical은 16진수를, Decimal은 10진수를 의미한다.
어떤 문자열이 있을 때, 이를 알아보기 어렵게 16진수 혹은 10진수 값으로 바꾸어 입력할 수 있다.
이 때, `String.fromCharCode` 혹은 `String.charCodeAt` 함수를 사용할 수 있다.
`String.fromCharCode`는 숫자를 문자로 변환하는 함수이고, `String.charCodeAt`은 특정 위치의 문자를 아스키코드의 숫자로 변환하는 함수이다.  

```javascript
document.write('\x4d\x41\x44\x48\x41\x54\x20');
document.write(String.fromCharCode(0x4d,0x41,0x44,0x48,0x41,0x54,0x20));
document.write(String.fromCharCode(77,65,68,72,65,84,32));

document.write(String.fromCharCode("LADHAT".charCodeAt(0) + 1));
```

![](/images/js_obfuscation/obfuscation_03.png)  

* escape / unescape

`escape` 함수는 ASCII 형태의 문자열을 ISO Latin-1 형태로 변환해 주는 함수이다.
`unescape` 함수는 그와 반대로 반환 해 준다.
`알파벳과 숫자, *, @, -, _, +, /` 를 제외한 특수문자만 해당된다.  

```javascript
document.write(escape("MADHAT!@#$%^&*"));
document.write(" ");
document.write(unescape("MADHAT%21@%23%24%25%5E%26*"));
```

![](/images/js_obfuscation/obfuscation_04.png)  

* XOR, +, - 등 연산  

문자열을 알아보기 어렵게 하기 위해서 연산을 사용하기도 한다.
특정 문자를 아스키코드의 숫자로 바꾼 후, `XOR`, `+`, `-` 등의 연산을 통해서 원래의 문자로 변경하는 것이다.  

```javascript
document.write(String.fromCharCode(92 ^ 17, 80 ^ 17, 85 ^ 17, 89 ^ 17, 80 ^ 17, 69 ^ 17));
```

![](/images/js_obfuscation/obfuscation_05.png)  

이 외에도, 함수 명이나 변수 명을 `_`, `__`, `___` 등으로 만들어 구분을 어렵게 하거나, 알파벳이 아닌 문자를 사용하는 방법도 있다.
또한 띄어쓰기를 없애 소스코드를 한 덩어리로 만들어 읽기 어렵게 만들기도 한다.

Javascript 코드를 분석할 때 난독화가 되어있다면 매우 분석하기가 어렵다.
때문에 난독화를 풀어서 분석하는 것이 효율성 측면에서 더 좋을 수 있다.
내가 난독화 된 코드를 분석할 때 사용하는 방법을 몇 가지 정리 해 보면 다음과 같다.  

## 난독화 풀기  

* JS beautifier  

띄어쓰기가 안되어 소스코드가 뭉쳐있어 함수와 변수의 구분이 안되는 것은 [JS beautifier](https://beautifier.io/)를 사용하면 된다.
위의 사이트에서 소스코드를 입력한 후, `Beautify Code` 버튼을 클릭하면 소스코드가 깔끔하게 정리되어 보인다.  

* 텍스트 에디터  

함수 명이나 변수 명들이 `_`, `__` 혹은 `ll11`, `l1l1` 와 같이 구분하기 어렵게 선언되어 있는 경우가 있다.
이 경우에는 텍스트 에디터를 사용 해 해당 변수 및 함수의 이름을 적절히 바꾸어 분석하면 조금 더 수월하게 분석할 수 있다.

* Chrome 개발자도구의 Console

난독화 된 문자열은 손으로 하나하나 해석하려면 시간이 오래 걸린다.
그런데 경험 상, 문자열을 난독화 한다면 `String.fromCharCode` 등의 함수가 사용되어 결국 원래의 문자로 복구시킨다.
이 때, 복구 된 문자를 확인하기 위해서 `Chrome 개발자도구`의 `Console`을 사용하면 좀 더 수월하게 확인할 수 있다.
예를 들어 아래와 같은 코드가 있다.  

```javascript
var a = String.fromCharCode(92 ^ 17, 80 ^ 17, 85 ^ 17, 89 ^ 17, 80 ^ 17, 69 ^ 17);
```

이 때, a의 값을 알기 위해서 `Console`창에 `String.fromCharCode` 부분을 입력 하면 복구된 문자열을 확인할 수 있다.  

![](/images/js_obfuscation/obfuscation_06.png)  

지금 생각나는 것은 이정도인 것 같다.
만약 앞으로 문제를 풀거나 분석을 하며 아이디어가 더 생긴다면 추가로 더 정리해두어야 겠다.  
