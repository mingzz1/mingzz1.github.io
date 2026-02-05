---
title: "[DEFCON 2019 Quals] cant_even_unplug_it Write-up"
date: 2019-05-18
draft: false
category: [ctf]
subcategories: [2019]
tags: [Write-up, CTF, Defcon2019]
---

지난 주말에 `Defcon CTF 2019`에 참여했다.
쉬운 한 문제를 풀어서 이 문제 풀이를 정리 해 두려 한다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
cant_even_unplug_it

You know, we had this up and everything. Prepped nice HTML5, started deploying on a military-grade-secrets.dev subdomain, got the certificate, the whole shabang. Boss-man got moody and wanted another name, we set up the new names and all. Finally he got scared and unplugged the server. Can you believe it? Unplugged. Like that can keep it secret…

Files:
HINT
```

분명히 태그는 `WEB`이라고 되어있었는데 주어진 URL이 없다...
그래서 먼저 주어진 `HINT` 파일을 확인 해 보았다.
HINT 파일 내용은 다음과 같다.  

```plain
Hint: these are HTTPS sites. Who is publicly and transparently logging the info you need?

Just in case: all info is freely accessible, no subscriptions are necessary. The names cannot really be guessed. 
```

근데 힌트를 읽어도 도대체 이게 무슨 문제인지를 모르겠었다.
일단 문제에서 주어진 유일한 URL인 `military-grade-secrets.dev`에 접속 해 보았다.  

![](/images/CTF/Defcon2019/unplug_01.png)  

그랬더니 https로 넘어가는 것을 확인할 수 있었다.
서버가 존재하지 않는다면 그냥 아무것도 뜨지 않아야 하는데 https로 연결되는 것을 보니 뭔가 있긴 있는 것 같았다.
그래서 [Sublist3r](https://github.com/aboul3la/Sublist3r)를 통해 위의 도메인을 검색 해 보니 아래와 같은 결과를 얻을 수 있었다.  

```sh
$ python sublist3r.py -d military-grade-secrets.dev

                 ____        _     _ _     _   _____
                / ___| _   _| |__ | (_)___| |_|___ / _ __
                \___ \| | | | '_ \| | / __| __| |_ \| '__|
                 ___) | |_| | |_) | | \__ \ |_ ___) | |
                |____/ \__,_|_.__/|_|_|___/\__|____/|_|

                # Coded By Ahmed Aboul-Ela - @aboul3la
    
[-] Enumerating subdomains now for military-grade-secrets.dev
[-] Searching now in Baidu..
[-] Searching now in Yahoo..
[-] Searching now in Google..
[-] Searching now in Bing..
[-] Searching now in Ask..
[-] Searching now in Netcraft..
[-] Searching now in DNSdumpster..
[-] Searching now in Virustotal..
[-] Searching now in ThreatCrowd..
[-] Searching now in SSL Certificates..
[-] Searching now in PassiveDNS..
[-] Total Unique Subdomains Found: 2
now.under.even-more-militarygrade.pw.military-grade-secrets.dev
secret-storage.military-grade-secrets.dev
```

2개의 도메인을 찾을 수 있었다.
그런데 이 두 도메인 모두 접속하면 아무것도 뜨지는 않는다.
그래서 `curl`을 이용 해 두 도메인을 확인 해 보았더니 아래와 같이 같은 결과를 얻을 수 있었다.  

```sh
$ curl now.under.even-more-militarygrade.pw.military-grade-secrets.dev
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="https://forget-me-not.even-more-militarygrade.pw/">here</A>.
</BODY></HTML>

$ curl secret-storage.military-grade-secrets.dev
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="https://forget-me-not.even-more-militarygrade.pw">here</A>.
</BODY></HTML>
```

두 도메인 모두 https://forget-me-not.even-more-militarygrade.pw라는 도메인을 알려준다.
하지만 이 도메인도 접속하면 아무것도 뜨지 않는다.
그런데 이 도메인을 구글에 검색하고 아래와 같이 저장된 페이지를 확인 해 보면 플래그를 찾을 수 있다.  

![](/images/CTF/Defcon2019/unplug_02.png)  

![](/images/CTF/Defcon2019/unplug_03.png)  

```plain
FLAG : OOO{DAMNATIO_MEMORIAE}
```
