---
title: "[Vmware] EFI Network / Timeout Error"
date: 2020-03-20
draft: false
category: [error]
subcategories: [windows]
tags: [Error, Windows, VMware]
---


얼마 전 컴퓨터를 포맷했는데, 포맷 한 뒤로 Vmware에 Windows를 설치하려고 하면 자꾸 `Time out` 오류가 난다.  

일단 오류 상황은 아래와 같다.  

<!--more-->

![](/images/error/vm/timeout/vm_timeout_01.png)  

![](/images/error/vm/timeout/vm_timeout_02.png)  

어떨 때는 첫 번째 사진과 같이 `EFI Network`라고 뜨며 Time out이 나오고, 또 어떨 때는 두 번째 사진과 같이 `Press any key to boot from CD or DVD`라는 오류가 나온다.  

일반적으로 두 번째 오류가 나오면 `Setting`에서 `CD/DVD(SATA)` 탭에 ISO 파일은 안 넣어 준 경우일텐데, 나는 아래와 같이 정확하게 Windows 10용 ISO 파일의 경로를 넣어 준 상태였다.  

![](/images/error/vm/timeout/vm_timeout_03.png)

이 오류의 해결 방법은 다음과 같다.  

VM 종료 후 해당 VM 마우스 오른쪽 버튼 > `Settings` > 상단 `Options` 탭으로 이동 > 아래의 `Advanced` 메뉴 > `Firmware type`을 확인 해 보면 아래와 같이 `UEFI`라고 설정되어 있을 것이다.  

![](/images/error/vm/timeout/vm_timeout_04.png) 

그럼 이 설정을 아래와 같이 `BIOS`로 변경하고 `OK`를 클릭 한다.  

![](/images/error/vm/timeout/vm_timeout_05.png) 

그리고 다시 부팅을 해 보면 정상적으로 되는 것을 확인할 수 있다.  

![](/images/error/vm/timeout/vm_timeout_06.png)  
