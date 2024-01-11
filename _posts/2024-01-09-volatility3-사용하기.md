---
title: volatility3 사용하기
date: 2024-01-09 00:22:10 +0900
categories: [Forensic, tools]
tags: [volatility]
render_with_liquid: false
---

## 개요
Volatility는 포렌식에서 메모리 덤프를 분석하는 데 사용하는 오픈 소스 프로그램이다.
나도 잘 아는 것은 아니지만 메모리 분석 도구의 표준이라고 불릴 정도로 자주 사용되는 프로그램이라 하고,
메모리 덤프를 분석해야 하는 문제를 풀 기회가 생겨 이번에 설치해보기로 했다.

현재 volatility는 곧 지원이 종료된 예전 파이썬 대신 파이썬 3로 전환한 volatility3 로 개발과 지원이 이루어지고 있다.

메모리 덤프 문제를 풀기 위해 각종 라이트업을 찾아 보니 2018년에 업데이트가 종료된 이전 버전의 volatility를 사용한 풀이가 대부분이여서 volatility 3를 설치하고 기본적인 사용 방법을 알아보고자 한다.

<b>Python3가 설치된 Window10을 기준</b>으로 작성하였습니다.

## 설치 과정

1. github에서 volatility 3의 공식 리포지토리를 클론한다.  

    ```shell
     git clone https://github.com/volatilityfoundation/volatility3.git 
    ```

2. clone된 폴더 안에서 종속 모듈 설치 후 빌드.  

    ```shell
    pip3 install -r requirements-minimal.txt

    python setup.py build
    python setup.py install    
    ```

    모든 기능을 사용하고 싶다면 아래의 명령어로 추가 모듈들을 설치하자.

    ```shell
    pip3 install -r requirements.txt
    ```

3. 설치 확인

    ```shell
    python vol.py -h
    ```
    위 명령을 실행하여 Volatility가 정상적으로 설치되었는지 확인할 수 있다.



## 사용법

Dumpit이나 FTK imager로 생성한 메모리 덤프가 필요하다.
> 덤프뜨기 전에 BIOS의 가상화 환경이 꺼져 있는지 확인하자. 켜져 있으면 블루스크린이 뜬다.
{: .prompt-info}

덤프 파일을 생성했으면, 분석을 위한 기본적인 플러그인들을 테스트 해 보자.

### windows.info(linux.info)

보통 분석 시작 시 사용하는 명령어, 분석 대상 시스템의 포괄적인 정보를 출력한다.
찾을 수 있는 정보에는 운영체제 커널 로드 주소, 페이지 디렉토리 기본 주소, 디버그 심볼 파일, 운영체제 버전, 프로세서, 실행 파일 정보 등이 있다. 

덤프 파일의 용량이 꽤 크면 분석시간 또한 길어지는데, <pre>--save-config</pre> 옵션을 지정하여 이후 분석 과정에서 같은 작업을 반복하지 않도록 정보를 저장해 둘 수 있다.
또한 <kbd>--save-config</kbd> 옵션을 지정하여 이후 분석 과정에서 같은 작업을 반복하지 않도록 정보를 저장해 둘 수 있다.

옵션을 포함하여 사용한 명령은 다음과 같다.
```shell
python vol.py --save-config config.json -f <파일 경로> windows.info
```

이후 다른 명령에서 생성한 파일을 <kbd>-c config.json</kbd> 과 같은 구문을 통해 불러올 수 있다.

### windows.pslist(linux.pslist)

덤프 당시 실행중이었던 프로세스 목록을 출력한다.

### windows.cmdline

커맨드라인 기록을 출력한다.

### windows.netscan

네트워크 연결 상태를 ip, 포트, pid 정보를 포함하여 상세히 출력한다.

### windows.filescan

시스템의 파일 목록을 오프셋과 함께 확인할 수 있다.

### windows.dumpfiles

위의 명령을 통해 알아낸 오프셋을 기반으로 특정 파일을 추출할 수 있다.