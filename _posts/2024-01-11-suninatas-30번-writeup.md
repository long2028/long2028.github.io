---
title: suninatas 30번 writeup
date: 2024-01-11 10:30:10 +0900
categories: [Writeup, Forensic]
tags: [volatility]
render_with_liquid: false
---

## 문제
>해커가 김장군의 PC에 침투한 흔적을 발견하였다.
사고 직후 김장군의 PC에서 획득한 메모리 덤프를 제공받았다고 가정하고, 다음 내용을 알아내야 한다.

1. 김장군 PC의 IP 주소는?
2. 해커가 열람한 기밀문서의 파일명은?
3. 기밀문서의 주요 내용은?

## 풀이

문제엔 1GB 가량의 덤프 파일이 함께 첨부되어 있다.
해당 덤프 파일을 Volatility 프로그램으로 분석하면 문제에서 설정한 조건들을 확인할 수 있을 것이다.

volatility의 설치 방법과 기본적인 사용 방법은
[volatility3-사용하기](https://long2028.github.io/posts/volatility3-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/)
에서 확인할 수 있다.

### 덤프 파일 읽기
먼저 분석 대상의 기본적인 정보를 파악해야 이후에 어떤 플러그인을 사용하여 분석을 진행할지를 결정할 수 있다.
첫 번째로 사용한 구문은 다음과 같다.
```shell
python vol.py -f <파일 경로> windows.info
```

![1](/assets/img/posts/2024-01-11-wininfo.png)

출력된 결과에서 운영체제 버전 정보를 제외하곤 무슨 말인지 알 수가 없었다. 사실 이 문제에선 중요한 내용은 아니지만 궁금하니까 GPT의 도움으로 어떤 정보를 제공하고 있는지 알아보자.


- Kernel Base Address: 0x82e43000  
    운영 체제 커널이 메모리 상에 로드된 기본 주소.


- DTB (Directory Table Base): 0x185000  
    페이지 디렉토리 기본 주소로, 가상 메모리 주소를 실제 물리적 주소로 변환하는 데 사용됨.


- Symbols Path:  
    Windows 커널 디버그 심볼 파일. 심볼 파일은 운영 체제의 다양한 컴포넌트와 데이터 구조에 대한 정보를 제공.


- Is64Bit: False & IsPAE: True  
    이 시스템은 64비트 아키텍처가 아니며, 물리 주소 확장(PAE)을 지원함.


- Layer Information:  
    layer_name: 0 WindowsIntelPAE  
    memory_layer: 1 FileLayer  

    메모리 분석 계층 정보.


- Operating System Information:  
    NTBuildLab: 7601.18044.x86fre.win7sp1_gdr.13  
    CSDVersion: 1  
    SystemTime: 2016-05-24 09:47:40  
    NtSystemRoot: C:\Windows  
    NtProductType: NtProductWinNt  
    NtMajorVersion: 6  
    NtMinorVersion: 1  

    Windows 7 SP1 (Service Pack 1) 운영 체제와 기본적인 정보(루트 디렉토리 등)을 표시하고 있다.


- Processor Information:  
    KeNumberProcessors: 1  
    MachineType: 332  

    단일 프로세서(1개 CPU 코어)를 가진 시스템.


- Portable Executable (PE) Information:  
    MajorOperatingSystemVersion: 6  
    MinorOperatingSystemVersion: 1  
    PE Machine: 332  
    PE TimeDateStamp: Sat Jan 5 02:46:00 2013  

    분석된 실행 파일에 대한 정보.

### IP 주소 찾기

windows 운영체제인 것을 알았으니, 네트워크 정보를 확인하는 플러그인을 사용할 수 있다.
```shell
python vol.py -f <파일 경로> windows.netstat
```

![2](/assets/img/posts/2024-01-11-netstat.png)

연결된 ip의 통신 정보와 사용 중인 포트들이 표시되고, 네트워크 자원을 사용하고 있는 프로세스들을 확인할 수 있다.
피해자의 PC ip는 LocalAddr <b>192.168.197.138</b>일 가능성이 높아 보인다.

### 파일명 찾기

해커가 기밀문서를 열람하기 위해 사용한 프로그램을 먼저 확인해 보자.

```shell
python vol.py -f <파일 경로> windows.pslist
```

![3](/assets/img/posts/2024-01-11-pslist.png)

프로세스 중에 눈에 띄는 것은 CMD와 메모장이다. 이것만으론 파일 내용을 알아낼 수 없으니 커맨드라인 기록을 살펴보자.

```shell
python vol.py -f <파일 경로> windows.cmdline
```

![4](/assets/img/posts/2024-01-11-cmdline.png)

기록 중에 notepad 명령으로 <kbd>C:\Users\training\Desktop\SecreetDocumen7.txt</kbd> 파일을 연 행동이 보인다. 정황상 기밀문서의 파일명은 <b>SecreetDocumen7.txt</b> 일 것이다.

### 문서 추출하기

기밀 문서의 내용을 확인하기 위해 해당 파일을 추출하자.  
먼저 기밀 문서의 오프셋을 확인해야 한다.

```shell
python vol.py -f <파일 경로> windows.filescan | findstr "Secreet"
```

그런데, 파이썬 버전 문제인지 findstr 파이프라인을 추가할 때 계속 인코딩 오류가 발생해서 명령이 정상적으로 실행되지 않았다.

이건 volattility 자체의 오류가 아니고 파이썬의 기본 인코딩이 UTF-8로 지정되어 있지 않아서 발생하는 오류이므로, <kbd>set PYTHONUTF8=1</kbd> 명령어로 한글을 제대로 인식할 수 있게 설정해 두자.

실행 화면은 다음과 같다.

![5](/assets/img/posts/2024-01-11-filescan.png)

확인된 오프셋은 "0x3df2ddd8" 이다.

다음 명령으로 파일을 추출한다.

```shell
python vol.py -o <추출될 파일 경로> -f <파일 경로> windows.dumpfiles --physaddr 0x3df2ddd8
```

physaddr 옵션에서 추출할 파일의 오프셋을 지정하고, o 옵션에서 추출될 파일의 경로를 잡아준다.
정상적으로 추출되었다면 어떤 파일이 추출되었는지 커맨드라인에서 확인할 수 있다.

![6](/assets/img/posts/2024-01-11-hitxt.png)

추출된 파일을 HxD로 열어 보니 친절한 안내문과 함께 키 값을 얻을 수 있었다.

![7](/assets/img/posts/2024-01-11-key.png)

### 마무리

문제의 설명처럼 1번답 + 2번답 + 3번답을 연결한 다음 MD5로 암호화하고, 이걸 소문자로 변환하면 문제 해결!