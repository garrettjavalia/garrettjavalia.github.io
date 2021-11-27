---
layout: post
title:  "CapsLock과 Left Ctrl키의 교환 설정(윈도우)"
date:   2021-11-28 04:01:00 +0900
categories: windows
tags: Windows CapsLock Ctrl
---

# 프로그래머와 Ctrl키
프로그래머들은 작업 중에 Ctrl키를 자주 사용한다. 물론 프로그래머가 아니어도 문서작업이나 영상 편집 등 다양한 상황에서 좌측 Ctrl키는 단축키에 있어서 매우 중요한 키이다.

# Ctrl키의 위치는 좋지 않다
그런데 Ctrl키는 좌하단에 있어서 이 키를 누르기는 쉽지않다. 나는 평소에 손가락 끝이 아닌 손가락의 손바닥쪽 마디, 때로는 손바닥 부위로 이 키를 누르는 것에 익숙해져 있다. 그러나 이것은 분명히 잘못된 것이다. 이렇게 자주 눌러야하는 키가 왜 이렇게 안 좋은 위치에 배정되었을까?

# 해결책을 찾아보자
가장 흔하게 누르는 컨트롤키를 위해서 이런 불편한 자세를 취한다는 것이 마음에 들지 않던 차에, 일부 키보드는 애초에 Left Ctrl과 CapsLock키의 위치가 반대로 되어 있거나 설정이 가능함을 알게되었다. 레오폴드 키보드를 사서 처음에는 그런 설정으로 지내봤으나, 곧 노트북의 내장 키보드를 쓰는 상황이거나 혹은 다른 외장키보드를 써야하는 상황 등에서 불편함이 있음을 알고, 키보드의 키설정을 교환하는 것 보다 운영체제의 키 매핑을 바꾸는 것이 더 합리적이라는 생각에 도달했다.

# 윈도우라면 레지스트리 수정이 제일 좋다.
그래서 결국 레지스트리의 관련 설정을 수정하는 정보글들을 인터넷에서 찾아보았고, [이 레지스트리 파일]({{ site.baseurl }}{% link static/keyswapper/InstallCtrlCapsLockSwapper.reg %})을 만들었다. 이 파일은 CapsLock에 Ctrl을 배정하는 것에 그치지 않고, 원래의 Ctrl키 위치에는 CapsLock을 배정하여 CapsLock도 상황에 따라서는 쓸 수 있다. 레지스트리 파일을 실행하는 것이 불안하다면, 직접 열어서 내용을 확인해봐도 좋다.

# 레지스트리 파일의 내용
내용의 설명은 이렇다.

``` text
Windows Registry Editor Version 5.00 //이 파일이 윈도우 레지스트리 에디터 5.00용임을 의미

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout] //레지스트리의 키가 위치할 위치

"Scancode Map"=hex:00,00,00,00,00,00,00,00,03,00,00,00,1d,00,3a,00,3a,00,1d,00,00,00,00,00 //값, 아래 설명 참조 
```
위의 파일 내용 중 마지막 줄은 Scancode Map이라는 항목이름에 16진수 이진값을 추가하며, 8바이트의 0으로 기록 시작을 나타내고, 그 후 03 00 00 00은 뒤에 3*4바이트의 정보가 더 온다는 것을 의미하며, 1d, 00, 3a, 00은 앞에 제시된 키(1d는 CapsLock, 3a는 LeftCtrl)를 뒤에 제시되는 키로 바꾼다는 뜻이다. 뒤에 이어지는 것은 마찬가지로 LeftCtrl을 CapsLock으로 인식한다는 의미이며, 정보기록이 끝났다는 의미로 마지막에 00, 00, 00, 00이 붙는다.

위 내용을 삭제하기 위해서는 [이 파일]({{ site.baseurl }}{% link static/keyswapper/UninstallCtrlCapsLockSwapper.reg %})을 실행하면 된다. 내용을 확인하고 싶다면 편집기로 열면 되며, 내용은 훨씬 간단하다.

# 첨부파일 목록

첨부파일 1. [InstallCtrlCapsLockSwapper]({{ site.baseurl }}{% link static/keyswapper/InstallCtrlCapsLockSwapper.reg %})

첨부파일 2. [UninstallCtrlCapsLockSwapper]({{ site.baseurl }}{% link static/keyswapper/UninstallCtrlCapsLockSwapper.reg %})