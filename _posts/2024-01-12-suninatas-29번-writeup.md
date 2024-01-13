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

![11](/assets/img/posts/2024-01-12-hxd.png)

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

### 키로거 찾기

문제 egg 파일에는 가상머신의 스냅샷이 첨부되어 있다, 키로거는 캡쳐 당시에도 사용중이였을 가능성이 높기 때문에 먼저 첨부되어 있던 Snapshot 메모리 덤프를 사용하기로 했다.  

volatlity의 다음 명령을 사용했다.
```shell
python vol.py -f <파일 경로> windows.psscan
```

![4](/assets/img/posts/2024-01-12-psscan.png)

하지만 프로세서의 이름만으론 키로거를 구분짓기는 어려웠다.
추가적인 정보를 얻기 위해 Autopsy의 분석 결과를 확인하던 도중, 최근 사용한 문서에서 수상한 경로의 폴더를 발견했다.

하위 폴더를 살펴보면 패스워드를 입력하는 화면을 캡쳐한 이미지와 프로세스의 상태 정보를 확인하는 화면 이미지를 확인할 수 있었다.

volatility로 알아냈던 사용중인 프로세스에서 비슷한 이름을 찾아 보니, v1tvr0.exe가 보였다.
정황상 해당 프로세스가 키로거일 가능성이 높아 보인다.

v1tvr0.exe의 절대 경로가 두번째 키 값이다.

### index.dat 확인

Windows7 에선 <b>index.dat</b>에 웹 사용 정보가 모두 기록된다.

자세한 정보는 [잘 정리된 블로그](http://forensic-proof.com/archives/4004)가 있으니 참고 바란다.

다만, windows7 이후론 새로운 웹 캐시 파일인 WebCasheV.dat가 있으니 다른 문제를 풀 땐 주의하자.(해당 문제와는 관계 없다.)

Autopsy에선 이 index.dat를 종합하여 Web History로 분류해 사용 기록을 검색할 수 있는 기능이 있지만, 영 미덥지 않다.

원하는 index.dat를 추출하여 따로 분석 프로그램을 사용하는 것이 더 확실하다.

웹 파일 다운로드 기록은 <kbd>%APPDATA%\Microsoft\Windows\IEDownloadHistory\index.dat</kbd> 경로의 index.dat에 기록되는 것이 일반적이다.

파일을 검색해 보니 IEDownloadHistory에 있는 index.dat를 발견하였고, 이를 추출하였다.

![5](/assets/img/posts/2024-01-12-findyou.png)

index.dat 분석에 사용한 프로그램은 Index.dat Analyzer 이다.
하지만, 생각했던 것과 달리 정확히 어떤 파일명을 다운받았는지를 확인할 수 없어 웹 캐시를 담은 인덱스를 검색해 보았다.

Content.IE5의 index.dat를 추출해 indexdat analyzer로 분석해 보니 각종 사이트의 방문 기록과 spy-keylogger 파일을 다운받은 기록이 나타났다.

![6](/assets/img/posts/2024-01-12-logfind.png)

해당 파일을 다운받은 시각은 2016-05-24_04:25:06 이다.

### 키로거를 통해서 알아내고자 한 내용은?

아까 발견했던 키로거가 저장된 폴더에 키로거가 남긴 로그 파일들을 확인해보면 될 것이다.

각종 사진들이 들어있는 폴더는 너무 많으니 패스하고, z1.dat라는 수상한 파일을 확인해 보니 키 값이 적혀 있었다.

![7](/assets/img/posts/2024-01-12-lastkey.png)

### 마무리

1~4번까지의 모든 답을 찾았다. 답들을 조합해서 MD5로 해시한 다음, 소문자로 바꿔 주면 문제 해결!