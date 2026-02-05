---
title: "[TenDollar 2018] Ping! Ping! Ping! Write up"
date: 2018-12-16
draft: false
category: [ctf]
subcategories: [2018]
tags: [Write-up, CTF, TenDollar2018]
---

몇 주 전에 TenDollar 팀이 주최 한 CTF에 참여했다.
한 3문제 정도 푼 것 같은데, 그 중에 `Ping! Ping! Ping!` 문제는 지금도 풀어볼 수 있어 이 문제에 대한 풀이를 적어두려 한다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
ping! ping! ping!
```

첨부파일을 확인 해 보면, `pcapng` 파일이 있는 것을 확인할 수 있다.
pcapng 파일을 보면 두 IP가 `ICMP Request`, `ICMP Reply`를 주고받은 패킷이 있다.
이에 데이터를 영역을 살펴보니, 아래와 같은 내용이 있었다.  

![](/images/CTF/TenDollar2018/ping_ping_ping/ping_01.png)

`PK`와 `Content_Types`라는 글자가 있는 것으로 보아 해당 파일은 `HWP`인 것 같았다.
즉 이 `pcapng` 파일은 `HWP` 파일을 ICMP 프로토콜을 통해 전달 한 패킷을 캡처 한 것이라는 것을 알 수 있었다.
전송 한 파일을 추출하기 위해서는 Python으로 아래와 같이 코드를 구현하면 된다.  

```python
import dpkt
import socket

def inet_to_str(inet):
	try:
		return socket.inet_ntop(socket.AF_INET, inet)
	except ValueError:
		return socket.inet_ntop(socketAF_INET6, inet)

f = open("ping_ping_ping.pcapng")

pcap = dpkt.pcap.Reader(f)

f1 = open("extract.hwp", "w")

newfile = ""

for timestamp, buf in pcap:
	eth = dpkt.ethernet.Ethernet(buf)

	ip = eth.data

	if isinstance(ip.data, dpkt.icmp.ICMP):
		icmp = ip.data
		
		if inet_to_str(ip.src) == "10.211.55.6":
			newfile += icmp.data.data


f1.write(newfile)
```

그런데 위의 코드를 실행하면 아래와 같은 오류가 발생한다.  

```plain
Traceback (most recent call last):
  File "ex.py", line 12, in <module>
    pcap = dpkt.pcap.Reader(f)
  File "/usr/local/lib/python2.7/dist-packages/dpkt/pcap.py", line 251, in __init__
    raise ValueError('invalid tcpdump header')
ValueError: invalid tcpdump header
```

때문에 `pcapng` 파일을 아래의 명령어를 사용하여 `pcap` 파일로 변경 해 주어야 한다.  

```sh
$ editcap -F libpcap -T ether ping_ping_ping.pcapng ping_ping_ping.pcap
```

이 후 다시 위의 코드를 아래와 같이 수정 해 실행시키면 `HWP` 파일을 얻을 수 있다.  

```python
import dpkt
import socket

def inet_to_str(inet):
	try:
		return socket.inet_ntop(socket.AF_INET, inet)
	except ValueError:
		return socket.inet_ntop(socketAF_INET6, inet)

f = open("ping_ping_ping.pcap")

pcap = dpkt.pcap.Reader(f)

f1 = open("extract.hwp", "w")

newfile = ""

for timestamp, buf in pcap:
	eth = dpkt.ethernet.Ethernet(buf)

	ip = eth.data

	if isinstance(ip.data, dpkt.icmp.ICMP):
		icmp = ip.data
		
		if inet_to_str(ip.src) == "10.211.55.6":
			newfile += icmp.data.data


f1.write(newfile)
```

그럼 결과로 `extract.hwp` 파일을 얻을 수 있고 해당 파일을 열면 아래와 같이 플래그를 확인할 수 있다.  

![](/images/CTF/TenDollar2018/ping_ping_ping/ping_02.png)

```plain
TDCTF{Do_you_know_about_the_icmp_protocol?}
```
