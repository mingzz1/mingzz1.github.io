---
title: "[VolgaCTF2019] Shadow Cat Write up"
date: 2019-04-08
draft: false
category: [ctf]
subcategories: [2019]
tags: [Write-up, CTF, VolgaCTF2019]
---

지난 주말에 잠시 짬을 내어 VolgaCTF에 참여했다.
원래 풀고싶었던 문제는 Head Hunter 였는데, 끝까지 Write-up이 올라오지 않아 결국 풀어볼 수가 없었다.
대신 Shadow Cat이라는 문제를 풀어 보았다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
We only know that one used /etc/shadow file to encrypt important message for us.
shadow.txt encrypted.txt
```

파일이 2개가 주어지는데, 내용은 각각 다음과 같다.  

* shadow.txt
```plain
root:!:17792:0:99999:7:::
daemon:*:17737:0:99999:7:::
bin:*:17737:0:99999:7:::
sys:*:17737:0:99999:7:::
sync:*:17737:0:99999:7:::
games:*:17737:0:99999:7:::
man:*:17737:0:99999:7:::
lp:*:17737:0:99999:7:::
mail:*:17737:0:99999:7:::
news:*:17737:0:99999:7:::
uucp:*:17737:0:99999:7:::
proxy:*:17737:0:99999:7:::
www-data:*:17737:0:99999:7:::
backup:*:17737:0:99999:7:::
list:*:17737:0:99999:7:::
irc:*:17737:0:99999:7:::
gnats:*:17737:0:99999:7:::
nobody:*:17737:0:99999:7:::
systemd-network:*:17737:0:99999:7:::
systemd-resolve:*:17737:0:99999:7:::
syslog:*:17737:0:99999:7:::
messagebus:*:17737:0:99999:7:::
_apt:*:17737:0:99999:7:::
uuidd:*:17737:0:99999:7:::
avahi-autoipd:*:17737:0:99999:7:::
usbmux:*:17737:0:99999:7:::
dnsmasq:*:17737:0:99999:7:::
rtkit:*:17737:0:99999:7:::
cups-pk-helper:*:17737:0:99999:7:::
speech-dispatcher:!:17737:0:99999:7:::
whoopsie:*:17737:0:99999:7:::
kernoops:*:17737:0:99999:7:::
saned:*:17737:0:99999:7:::
pulse:*:17737:0:99999:7:::
avahi:*:17737:0:99999:7:::
colord:*:17737:0:99999:7:::
hplip:*:17737:0:99999:7:::
geoclue:*:17737:0:99999:7:::
gnome-initial-setup:*:17737:0:99999:7:::
gdm:*:17737:0:99999:7:::
jr:$6$jKcSVswg$AWA/PLZVcOcQb6404vSsjmyJzgtC97R3iRDXSOBshuYmCcDVDhaIS6C4fVZnOI6EGhaUnrUWK2Y3jYhQ4vcmu0:17792:0:99999:7:::
vboxadd:!:17792::::::
zerotier-one:!:17792::::::
postgres:*:17793:0:99999:7:::
postfix:*:17794:0:99999:7:::
lightdm:*:17930:0:99999:7:::
z:$6$AEqLtEqq$1ojEoCgug5dzqeNfjGNE9p5SZFwIul8uOFp9vMZEz50oiUXOVFW3lw1S0fuvFY5ggi5CfbfoWaMDr2bvtSNRC/:17930:0:99999:7:::
a:$6$9Eg69bYI$q75YWUVWb4MYzkcExXukpt.VJ3fX458iZJm1ygpTLwX.CgroHpmeSG88By.zQmKyOHCvBHoA0Q001aBqbkVpg/:17930:0:99999:7:::
x:$6$5TF0Txe3$APSNzUSjFmMbsmrkCS9qE84qfu4AI2dNEjqm2PRKgSjncBTI4lECXofQ8abdAtYX6tST6FGCgOdvLlDYQTCJx0:17930:0:99999:7:::
q:$6$I3iqZL0m$nxHWvcLz7lg/ZKoKfX9dq5k0uqkOtKgLdyREAQxQkfPkVvbNHPfQaoCFfnXl1BoX1vgOcEDghVPvfRUrs6dGp1:17930:0:99999:7:::
l:$6$n8iJWaW/$M1Od8seiEL6h3L.egHubBYAk.cd8/LUctESIm69/r.gvP0eqabusN5/D1rNu1qDHOkgRpHf8PWGSb8zoxrgrp1:17930:0:99999:7:::
v:$6$GatagTHS$I1UfNfn3NGP5Vre0z7s3DGqFpjN5Pw2XhAHSw6ZSMwaMAsf8IteFedXovNlHLuIXvR9ezeya89XEOq2We7CcW1:17930:0:99999:7:::
e:$6$ZO6YiExi$7DBV0zMIqf8iy.zVTL7gbHetCmJ3LL4ROEYG/UME4Tmym82vZYkFWjiNpCGapvF83QNJKFJOjkhXMgFLfkhza.:17930:0:99999:7:::
f:$6$HXoF2OZ1$onkVfp52IRdd2OipQv3rPPsGr7QradAFTFnmXv5c9xkGy4xcgJFkoaSJzMQCtfWuU2FQ3BN3lyL47SyoIoPmy0:17930:0:99999:7:::
b:$6$I9Uq9PRG$euVEYx5TR2lUFe/k7s0e8us8xgl7j/cbiYRnvba.eFfMSSPsm5I.gcShqOLqAa58m5VISomkPpHlJ1xLgCRxw/:17930:0:99999:7:::
r:$6$J.qms5y0$YOMnlR0V.WQjqAyik3nU7TDdy8hZQZWfhF8CTJHJtd/0mrANrxBvULoXNiJnvX.yn5T3QBFu4wk3xrFye.uus1:17930:0:99999:7:::
g:$6$b4GkIuzQ$j/prK.Jy304eu6W/NG0Yz4mHDOc3BavkYRBomdjVG.fAksRM/xIDRoWcKcJw66YZmVGcV51YkIwVZHVfA//KU.:17930:0:99999:7:::
n:$6$f2QD.cIu$n70jbCSe0QVz7M48SVE2Z..IPDV0QjIfZ8D40oQOO9smt0ZeA2I0sSO927VIr1SJwupjZairVR0T/pKQ0QG3N1:17930:0:99999:7:::
o:$6$qyBGOX79$OAvGVjmH69C.0ZfObcJ0DcKzoSWHhBBt9sPVjbIJtYmT4nV/TU/zCiKgCkSQaQJ.n.vTjAScdh9htzjBzTwOT/:17930:0:99999:7:::
p:$6$EW1gOlyH$omuxKElrI3DeVoxrLet3OW3MfuFPwLwefVxOr62E.wo7szQ6.swTec1edCFiKnPc42XxMRGsrNJn36mkc2dgH/:17930:0:99999:7:::
s:$6$TXxh4vV5$v1vF49YZQnonSZrKwBWNp7rpxIRoQY/ooEODqjsdwwdoY8sso.y/EOsoC3GFlpCLYnEY./n/1BuNID5njg0CV.:17930:0:99999:7:::
c:$6$IYxRoIyR$eEmGTNPNd6HPQibc/UBdQ5zgR/dGQ1dtCuSl0lUmvmbrSKbYEf/SEDlX4fgP.JQXlyFqEgu4NOBiu2eozpAM.0:17930:0:99999:7:::
w:$6$RmCyBroe$EovezmWQJVvQFGd6.ei2.SfzGJG22CsV47tTnyfKx30TbuG1VMgsk0de6NOQ04bKNML9fbuMu0Pw3TMf82zye1:17930:0:99999:7:::
d:$6$..CK41Sn$36X19X0jrviLxVVk.KtR9zbHMML0mg1XzMNQgP7eOpGMF1JYSbZHAyReDhNVkm1WaNn3lfO2CsAww14fZZads1:17930:0:99999:7:::
t:$6$zVFV5HoW$q9WyP6/D0kPL.n7s.FvPcOSRfvcUFQu5QMPps6VbQUD5RMozQsP14GtUiOa/H2V2pU6c6OxcgRqruaLejPaES1:17930:0:99999:7:::
h:$6$S1RnJ8DK$F/FWFf9En/mn6YqZ67/gOnMT0WdSuaEyn2OArTJbG1IGHd0pUs9TiGDE9P.PhRB6XyHUgA5l/LBmBW8PpJg9M.:17930:0:99999:7:::
m:$6$SqkKQRak$nZHDrq3vdnajdLzotrW3J9kYzUvzPaUrs6NZaKkkcVN0KiCfUhJfgM6WJvZjZR7hBGWkfIwhZymko7wtsq49s1:17930:0:99999:7:::
k:$6$ZLCr2itT$tZ0.u7TsXPc3nAntIOepETGkhqfVG1IKaiuW0mAH5QROVuc7fonE43qEhUFT2LHMftYDdTQRAMHpRNMM8Yn1c1:17930:0:99999:7:::
i:$6$ZlWmheB2$DBLJQPLVhhEdA/iATrOYiqFv4i5TcmBUSX6.tZDo63YP4dcdlAuBnFU65xXIRP1tpNsCS7kc6Fu6jPMS2F7aP1:17930:0:99999:7:::
y:$6$rIoO6U2u$c5usMXbFP9S75qmDyBBWz1QZuyGH0MGq3mXN32kYipoL5XCFEHjmTRVcuZkmed5OzAopV0CgyA49QzILz5Rmq/:17930:0:99999:7:::
j:$6$Gpavrajq$me1yxZQ0OiJFedrTmxFsyP5zwOePuFJmgujUWun0h5bCOIVeuJuaIUTDHGCYxkT6mw41BTjlx9c3QvdsG8o0o.:17930:0:99999:7:::
u:$6$0w3EeszD$bUDQorjCKku1sjtCWMQfJ3ZRmsC5N.LN7CQnjvyCbcq5wSD33x2t/TVXA6jnjtajv8nIZc.Aj.oY80lm44Dhy0:17930:0:99999:7:::
underscore:$6$RVUCQJFr$fsKkPUT9Pp5QlsZblSLJ4yKkfBxNMWN0TS.q7ticuEr/HQFdEbyiwK5JmaKKS9UDFzUsY6mhe1knnRbwy7K0s/:17930:0:99999:7:::
```

* encrypted.txt
```sh
hajjzvajvzqyaqbendzvajvqauzarlapjzrkybjzenzuvczjvastlj
```

`shadow.txt` 파일을 보면 `/etc/shadow` 파일의 형태와 비슷하다.
/etc/shadow 파일은 root만이 읽을 수 있는 파일이며, 시스템의 모든 사용자에 대한 해시값을 저장한다.
/etc/shadow 파일에서 각 값의 구조는 다음과 같다. 

```plain
jr:$6$jKcSVswg$AWA/PLZVcOcQb6404vSsjmyJzgtC97R3iRDXSOBshuYmCcDVDhaIS6C4fVZnOI6EGhaUnrUWK2Y3jYhQ4vcmu0:17792:0:99999:7:::

Login Name:Encrypted:Last Changed:Minimum:Maximum:Warn:Inactive:Expire:Reserved
```

항목|내용
:----|:----
Login Name|사용자 이름  
Encrypted|패스워드를 암호화 시킨 값
Last Changed|1970년 1월 1일부터 패스워드가 수정된 날짜의 일 수 계산
Minimum|패스워드가 변경되기 전 최소 사용 일 수
Maximum|패스워드 변경 전 최대 사용 일 수
Warn|패스워드 사용 만기일 전 경고 메시지 제공 일 수
Inactive|로그인 접속 차단 일 수
Expire|로그인 사용 금지하는 일 수
Reserved|사용되지 않음

또한 `Encrypted`를 자세히 살펴 보면 `$`로 구분되는 것을 알 수 있다.
이 값들은 다음과 같다.  

```plain
$HashId$salt$Hash Value
```

항목|내용
:----|:----
HashId|Hash 종류. $1 → MD5 / $5 → SHA256 / $6 → SHA512
salt|hashing에 사용되는 값
Hash Value|HashId에 따른 hashing 방법과 salt를 가지고 hashing을 한 값

즉, 문제에서 제공한 shadow.txt 파일은 SHA512를 가지고 hashing을 수행했다는 것을 알 수 있다.
그런데 다시 문제를 살펴보면, `encrypted.txt` 파일은 알파벳 소문자로 구성되어 있고, `shadow.txt` 파일에 `Login Name`을 보면, 알파벳 소문자들이 있다는 것을 확인할 수 있다.
아마도 `shadow.txt` 파일에서 알파벳 소문자로 된 `user` 들의 패스워드를 크랙한 뒤, `encrypted.txt`를 복구하는 문제인 것 같다.
일단 `shadow.txt` 파일에서 패스워드를 크랙하기 전에 불필요한 `user`를 정리 해 주었다.  

```plain
z:$6$AEqLtEqq$1ojEoCgug5dzqeNfjGNE9p5SZFwIul8uOFp9vMZEz50oiUXOVFW3lw1S0fuvFY5ggi5CfbfoWaMDr2bvtSNRC/:17930:0:99999:7:::
a:$6$9Eg69bYI$q75YWUVWb4MYzkcExXukpt.VJ3fX458iZJm1ygpTLwX.CgroHpmeSG88By.zQmKyOHCvBHoA0Q001aBqbkVpg/:17930:0:99999:7:::
x:$6$5TF0Txe3$APSNzUSjFmMbsmrkCS9qE84qfu4AI2dNEjqm2PRKgSjncBTI4lECXofQ8abdAtYX6tST6FGCgOdvLlDYQTCJx0:17930:0:99999:7:::
q:$6$I3iqZL0m$nxHWvcLz7lg/ZKoKfX9dq5k0uqkOtKgLdyREAQxQkfPkVvbNHPfQaoCFfnXl1BoX1vgOcEDghVPvfRUrs6dGp1:17930:0:99999:7:::
l:$6$n8iJWaW/$M1Od8seiEL6h3L.egHubBYAk.cd8/LUctESIm69/r.gvP0eqabusN5/D1rNu1qDHOkgRpHf8PWGSb8zoxrgrp1:17930:0:99999:7:::
v:$6$GatagTHS$I1UfNfn3NGP5Vre0z7s3DGqFpjN5Pw2XhAHSw6ZSMwaMAsf8IteFedXovNlHLuIXvR9ezeya89XEOq2We7CcW1:17930:0:99999:7:::
e:$6$ZO6YiExi$7DBV0zMIqf8iy.zVTL7gbHetCmJ3LL4ROEYG/UME4Tmym82vZYkFWjiNpCGapvF83QNJKFJOjkhXMgFLfkhza.:17930:0:99999:7:::
f:$6$HXoF2OZ1$onkVfp52IRdd2OipQv3rPPsGr7QradAFTFnmXv5c9xkGy4xcgJFkoaSJzMQCtfWuU2FQ3BN3lyL47SyoIoPmy0:17930:0:99999:7:::
b:$6$I9Uq9PRG$euVEYx5TR2lUFe/k7s0e8us8xgl7j/cbiYRnvba.eFfMSSPsm5I.gcShqOLqAa58m5VISomkPpHlJ1xLgCRxw/:17930:0:99999:7:::
r:$6$J.qms5y0$YOMnlR0V.WQjqAyik3nU7TDdy8hZQZWfhF8CTJHJtd/0mrANrxBvULoXNiJnvX.yn5T3QBFu4wk3xrFye.uus1:17930:0:99999:7:::
g:$6$b4GkIuzQ$j/prK.Jy304eu6W/NG0Yz4mHDOc3BavkYRBomdjVG.fAksRM/xIDRoWcKcJw66YZmVGcV51YkIwVZHVfA//KU.:17930:0:99999:7:::
n:$6$f2QD.cIu$n70jbCSe0QVz7M48SVE2Z..IPDV0QjIfZ8D40oQOO9smt0ZeA2I0sSO927VIr1SJwupjZairVR0T/pKQ0QG3N1:17930:0:99999:7:::
o:$6$qyBGOX79$OAvGVjmH69C.0ZfObcJ0DcKzoSWHhBBt9sPVjbIJtYmT4nV/TU/zCiKgCkSQaQJ.n.vTjAScdh9htzjBzTwOT/:17930:0:99999:7:::
p:$6$EW1gOlyH$omuxKElrI3DeVoxrLet3OW3MfuFPwLwefVxOr62E.wo7szQ6.swTec1edCFiKnPc42XxMRGsrNJn36mkc2dgH/:17930:0:99999:7:::
s:$6$TXxh4vV5$v1vF49YZQnonSZrKwBWNp7rpxIRoQY/ooEODqjsdwwdoY8sso.y/EOsoC3GFlpCLYnEY./n/1BuNID5njg0CV.:17930:0:99999:7:::
c:$6$IYxRoIyR$eEmGTNPNd6HPQibc/UBdQ5zgR/dGQ1dtCuSl0lUmvmbrSKbYEf/SEDlX4fgP.JQXlyFqEgu4NOBiu2eozpAM.0:17930:0:99999:7:::
w:$6$RmCyBroe$EovezmWQJVvQFGd6.ei2.SfzGJG22CsV47tTnyfKx30TbuG1VMgsk0de6NOQ04bKNML9fbuMu0Pw3TMf82zye1:17930:0:99999:7:::
d:$6$..CK41Sn$36X19X0jrviLxVVk.KtR9zbHMML0mg1XzMNQgP7eOpGMF1JYSbZHAyReDhNVkm1WaNn3lfO2CsAww14fZZads1:17930:0:99999:7:::
t:$6$zVFV5HoW$q9WyP6/D0kPL.n7s.FvPcOSRfvcUFQu5QMPps6VbQUD5RMozQsP14GtUiOa/H2V2pU6c6OxcgRqruaLejPaES1:17930:0:99999:7:::
h:$6$S1RnJ8DK$F/FWFf9En/mn6YqZ67/gOnMT0WdSuaEyn2OArTJbG1IGHd0pUs9TiGDE9P.PhRB6XyHUgA5l/LBmBW8PpJg9M.:17930:0:99999:7:::
m:$6$SqkKQRak$nZHDrq3vdnajdLzotrW3J9kYzUvzPaUrs6NZaKkkcVN0KiCfUhJfgM6WJvZjZR7hBGWkfIwhZymko7wtsq49s1:17930:0:99999:7:::
k:$6$ZLCr2itT$tZ0.u7TsXPc3nAntIOepETGkhqfVG1IKaiuW0mAH5QROVuc7fonE43qEhUFT2LHMftYDdTQRAMHpRNMM8Yn1c1:17930:0:99999:7:::
i:$6$ZlWmheB2$DBLJQPLVhhEdA/iATrOYiqFv4i5TcmBUSX6.tZDo63YP4dcdlAuBnFU65xXIRP1tpNsCS7kc6Fu6jPMS2F7aP1:17930:0:99999:7:::
y:$6$rIoO6U2u$c5usMXbFP9S75qmDyBBWz1QZuyGH0MGq3mXN32kYipoL5XCFEHjmTRVcuZkmed5OzAopV0CgyA49QzILz5Rmq/:17930:0:99999:7:::
j:$6$Gpavrajq$me1yxZQ0OiJFedrTmxFsyP5zwOePuFJmgujUWun0h5bCOIVeuJuaIUTDHGCYxkT6mw41BTjlx9c3QvdsG8o0o.:17930:0:99999:7:::
u:$6$0w3EeszD$bUDQorjCKku1sjtCWMQfJ3ZRmsC5N.LN7CQnjvyCbcq5wSD33x2t/TVXA6jnjtajv8nIZc.Aj.oY80lm44Dhy0:17930:0:99999:7:::
underscore:$6$RVUCQJFr$fsKkPUT9Pp5QlsZblSLJ4yKkfBxNMWN0TS.q7ticuEr/HQFdEbyiwK5JmaKKS9UDFzUsY6mhe1knnRbwy7K0s/:17930:0:99999:7:::
```

정확히 27줄이 남는 것을 보니 `알파벳 소문자`와 `_`가 모두 들어있는 것 같다.
이 값들을 크랙해야 하는데, Hash는 단방향 암호화 기법이기 때문에 `특정 단어`로 hashing을 수행한 뒤, 그 값과 `shadow.txt`에 저장된 값이 같은지 비교하는 방법으로 크랙해야 한다.  

`특정 단어`가 무엇인지는 모르겠지만, `shadow.txt` 파일의 `user`에 알파벳 소문자와 underscore가 있다.
때문에 `알파벳 소문자`와 `_`를 shadow.txt에 있는 salt 값과 함께 SHA512 hashing을 수행 해 비교하고, `encrypted.txt`에 있는 각 글자를 `user`로 하여 해당 `user`의 `password`로 변경하도록 아래와 같이 코드를 구현 해 보았다.  


```python
import string
import crypt

dictionary = string.ascii_lowercase + "_"

def findPass(encrypted):
        salt = encrypted[:11]
        for c in dictionary:
                dict_enc = crypt.crypt(c, salt)
                if dict_enc == encrypted:
                        return c

d = dict()

f = open("shadow.txt", "r")
shadow = f.read().split("\n")[:-1]

enc = "hajjzvajvzqyaqbendzvajvqauzarlapjzrkybjzenzuvczjvastlj"

for s in shadow:
        user, encrypted = s.split(":")[0], s.split(":")[1]
        d[user] = findPass(encrypted)

flag = "VolgaCTF{"

for c in enc:
        flag = flag + d[c]

flag = flag + "}"

print flag
```

이를 수행한 결과 다음과 같이 `flag`를 얻을 수 있었다.  

```sh
$ python ex.py 
VolgaCTF{pass_hash_cracking_hashcat_always_lurks_in_the_shadows}
```

--- 

```plain
FLAG : VolgaCTF{pass_hash_cracking_hashcat_always_lurks_in_the_shadows}
```
