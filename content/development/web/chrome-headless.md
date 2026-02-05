---
title: "[Chrome] Chrome Headless 사용법"
date: 2018-09-29
draft: false
category: [development]
subcategories: [web]
tags: [Chrome Headless]
---

하는 일이 일이다보니 크고작은 CTF용 문제를 낼 일이 많다.
그런데 XSS 문제를 낼 때마다 고민인게, 참가자들이 문제를 제대로 풀었다는 것을 어떻게 검증하느냐 이다.
그러다 `Chrome Headless`라는 것을 알게 되어 정리해 두려 한다. 

<!--more-->
## Chrome Headless 란

Chrome Headless는 쉽게 이야기 하면 Chrome 브라우저를 GUI 없이 사용할 수 있는 모드이다.
윈도우 기준으로는 `Chrome 59`, 맥/리눅스 기준으로는 `Chrome 60` 버전부터 정식으로 추가되어 사용할 수 있다.
실제 브라우저와 동일하게 작동해서 Javascript도 실행할 수 있다.

다른 사람들은 크롤링을 할 때 사용하는 것 같은데, 나는 창을 따로 띄우지 않고 Javascript를 실행할 수 있다는 점을 이용 해 XSS 문제 낼 때 사용했다.
내가 사용한 서버는 Ubuntu 16.04이고, Chrome 브라우저를 사용하기는 어려워 `chromedriver`를 사용했다.  

## Chrome Headless 사용하기  

### 환경 세팅하기  

Ubuntu에서 Chrome Headless를 사용하기 위해서 몇 가지 환경 세팅이 필요하다.  

#### 1. python 설치하기
```sh
$ apt-get install python2.7
$ apt-get install python-pip python-dev python-setuptools
```

```sh
$ pip install --upgrade pip
```

#### 2. Chrome Headless를 사용하기 위한 툴 및 모듈 설치  

```sh
$ pip install selenium
$ apt-get install chromium-browser
```

`pip install selenium`을 할 때 아래와 같은 에러가 발생하는 경우가 있다.  

```sh
importError: cannot import name 'main'
```

이 때는 `hash -d pip`를 해 주면 더 이상 에러가 발생하지 않는다.
이 후, Ubuntu에서 Chrome 브라우저 대신 사용 할 `chromedriver`를 다운받아야 한다.  

[chromedriver 다운로드](http://chromedriver.chromium.org/downloads)

위의 사이트에서 자신의 서버 버전에 맞는 chromedriver를 다운받은 후, 압축을 풀어 서버에 옮겨두면 된다.
만약 리눅스 환경이라면, 아래의 명령어를 입력 해 실행 권한을 주어야 한다.  

```sh
$ chmod +x chromedriver
```

여기까지 하면 Chrome Headless를 사용하기 위한 환경 세팅이 끝난다.  

### 코드 구현하기  

가장 기본적인 형태의 코드는 다음과 같다.  

```python
from selenium import webdriver

driver = webdriver.Chrome('./chromedriver')

driver.get('http://google.com')

driver.quit()
```

이 때, `webdriver.Chrome()` 안에는 다운받은 chromedriver의 경로를 적어주어야 한다.
그런데 내 경우에는 이렇게만 구현하면 또 에러가 발생했다.
이에 아래와 같이 옵션을 넣어주면 정상적으로 동작했다.  

```python
from selenium import webdriver

chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("--headless")
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument("--disable-dev-shm-usage")

driver = webdriver.Chrome("./chromedriver", chrome_options = chrome_options)

driver.get('http://google.com')

driver.quit()
```

여기에서 만약 캡쳐를 하고 싶다면 캡쳐하는 코드를, PDF로 받고 싶다면 PDF로 받는 코드를 구현하면 된다.
아래는 접속한 화면을 캡쳐 해 screenshot.png로 저장하는 코드이다.  

```python
from selenium import webdriver

chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("--headless")
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument("--disable-dev-shm-usage")

driver = webdriver.Chrome("./chromedriver", chrome_options = chrome_options)

driver.get('http://google.com')
driver.get_screenshot_as_file('screenshot.png')

driver.quit()
```

여기까지가 기본적인 사용법이다.
언어가 python이기 때문에 자신이 원하는 사용처에 따라서 추가적인 코드를 구현 해 사용하면 된다.

나는 그냥 대회 운영용으로 사용했는데, 다른 사람들이 사용 한 것처럼 웹 크롤러로 쓴다던지, 사용처가 매우 무궁무진 할 것 같다.
아직은 짧은 코드만 구현 해 겉핥기 식으로 사용해 봤는데, 시간이 날 때 다른 방향으로 좀 더 사용 해 봐야겠다.  