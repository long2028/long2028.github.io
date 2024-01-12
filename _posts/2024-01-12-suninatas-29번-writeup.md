---
title: suninatas 29번 writeup
date: 2024-01-12 10:30:10 +0900
categories: [Writeup, Forensic]
tags: [volatility]
render_with_liquid: false
---

## 문제
> 유준혁은 PC가 고장나서 형 유성준에게 PC를 고쳐 달라고 했다.
그런데, 유성준은 동생의 PC를 고치면서 몇 가지 장난을 쳤다.
당신은 이 PC를 정상으로 돌려 놓아야 한다.

1. 웹 서핑은 잘 되는데, 네이버에만 들어가면 사이버 경찰청 차단 화면으로 넘어간다. 원인을 찾으면 Key가 보인다.

2. 유성준이 설치 해 놓은 키로거의 절대경로 및 파일명은?(모두 소문자)

3. 키로거가 다운로드 된 시간은?

4. 키로거를 통해서 알아내고자 했던 내용은 무엇인가?

## 풀이

사실 그냥 Vmware로 가상머신을 직접 열면 되지만, 뭔가 실제 분석처럼 무결성을 지키면서 해보고 싶어서 autopsy를 사용해 이미지를 분석하는 방식을 사용했다.

### Autopsy 사용하기

문제에는 3.7GB의 확장자가 없는 파일이 첨부되어 있다.
먼저 어떤 파일인지 확인해보기 위해 hxd로 파일을 열어 보았다.

![1](/assets/img/posts/2024-01-12-hxd.png)

EGGA라는 처음 보는 문자열이 보이는데, 각종 헤더 시그니처 테이블을 살펴 보았지만 어떤 파일인지 알 수가 없었다...
gpt에게 물어 보니 EGGA는 EGG 압축파일의 확장자일 가능성이 있다고 한다. 왜 시그니처 테이블에는 없는 건지 모르겠지만, 일단 확장자를 바꾸고 파일을 열어 보았다.

압축을 해제하니 vmware 가상머신 이미지 파일들이 보인다. 아마 정황상 문제의 PC는 해당 가상머신이라고 생각해야 할 듯 하다.

![1](/assets/img/posts/2024-01-12-egg.png)

vmdk 파일을 FTK Imager을 통해 열었을 때, 정상적으로 파일 시스템이 인식되는 것을 보아 부트 섹터가 손상된건 아닌 것 같다.

따라서 vmdk 파일을 autopsy에 넣고 바로 분석을 시작하려 했는데, VM 파일을 지원한다는 말과 다르게 오류가 발생했다... 찾아보니 그냥 autopsy 자체 오류로 vmdk 파일은 사용할 수 없는 경우가 대부분이라 E01로 변환하는 과정을 따로 거치라고 한다.

[참고 페이지](https://sleuthkit.discourse.group/t/adding-a-disk-image-vmdk-format-failed/283/6)

FTK imager에서 해당 vmdk 파일을 E01로 변환하였다.

![2](/assets/img/posts/2024-01-12-e01.png)

이후 Autospy에서 분석을 시작했다.

### host 파일 확인

네이버에만 접속하면 경찰청 차단 페이지로 리다이렉션 된다고 한다. 이는 윈도우에선 <b>hosts</b> 파일의 문제일 가능성이 크다.

일반적인 시스템에서 hosts 파일의 경로는 <kbd>C:\Windows\System32\drivers\etc\hosts</kbd> 이다.

autopsy의 <kbd>file search by attribute</kbd> 기능으로 "hosts" 키워드를 검색했다.

그 중, 가장 최근 타임라인에 생성된 hosts 파일 내부에 naver.com 을 특정 ip 주소로 리다이렉션하는 규칙이 발견되었고, 그 밑에는 키 값이 적혀 있었다.

![3](/assets/img/posts/2024-01-12-hosts.png)

첫 번째 키값 확인!

### 두 번