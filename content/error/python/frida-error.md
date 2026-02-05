---
title: "[Frida 오류] Frida - Failed to load the Frida native extension"
date: 2019-12-15
draft: false
category: [error]
subcategories: [python]
tags: [Error, Python]
---

모바일 어플리케이션을 진단할 때 Frida는 거의 필수다.  

근데 이번에 새로 Frida를 설치했는데, 계속 `Failed to load the Frida native extension` 이라면서 오류가 나서 해결 방법을 적는다.  

<!--more-->

오류 전체 메시지는 다음과 같다.  

```sh
> frida-ps -U

***
Failed to load the Frida native extension: DLL load failed: 지정된 모듈을 찾을 수 없습니다.
Please ensure that the extension was compiled for Python 3.x.
***

Traceback (most recent call last):
  File "C:\Users\ccoma\AppData\Local\Programs\Python\Python36\Scripts\frida-ps-script.py", line 11, in <module>
    load_entry_point('frida-tools==5.3.0', 'console_scripts', 'frida-ps')()
  File "c:\users\ccoma\appdata\local\programs\python\python36\lib\site-packages\frida_tools\ps.py", line 6, in main
    from frida_tools.application import ConsoleApplication
  File "c:\users\ccoma\appdata\local\programs\python\python36\lib\site-packages\frida_tools\application.py", line 20, in <module>
    import frida
  File "c:\users\ccoma\appdata\local\programs\python\python36\lib\site-packages\frida\__init__.py", line 24, in <module>
    raise ex
  File "c:\users\ccoma\appdata\local\programs\python\python36\lib\site-packages\frida\__init__.py", line 7, in <module>
    import _frida
ImportError: DLL load failed: 지정된 모듈을 찾을 수 없습니다.
```

내가 사용한 Python의 버전은 3.6과 3.8이었는데, 둘 다에서 같은 오류가 났다. 
찾아보니 python 3.8에서는 아직 Frida를 지원하지 않는다고 하고, 3.6은 어느 순간부터 저런 오류 증상이 나타나는 것 같다. 
해결 방법은 python 3.7 버전 설치이다.

> [Python 3.7.5](https://www.python.org/downloads/release/python-375/)  

위의 사이트에서 다시 3.7대 python을 설치하고 아래의 명령어를 통해 다시 frida를 재설치 하면 정상적으로 동작한다.  

```sh
> pip install frida
> pip install frida-tools
> frida --version
12.7.26
```
