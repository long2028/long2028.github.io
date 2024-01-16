---
title: UofTCTF Forensic 문제 writeup
date: 2024-01-15 10:30:10 +0900
categories: [Writeup, Forensic]
tags: [UofTCTF]
render_with_liquid: false
---

![1](/assets/img/posts/2024-01-15-title.png)

어쩌다 보니 국외에서 개최한 CTF인 UofTCTF에 참가하게 되었다.
챌린지 분야는 생각보다 다양했지만,(처음 보는 Jail이라는 분야도 있었다) 포렌식 문제만 풀어 보았다.

![2](/assets/img/posts/2024-01-15-solved.png)

그마저도 전부 푼 건 아니지만, 6문제 중 해결한 4문제라도 풀이를 작성해보고자 한다.

## Secret Message 1

![3](/assets/img/posts/2024-01-15-first.png)

> We swiped a top-secret file from the vaults of a very secret organization, but all the juicy details are craftily concealed. Can you help me uncover them?

### 풀이

첨부된 secret.pdf 파일을 열어보면

![3](/assets/img/posts/2024-01-15-pdf.png)

보안 담당자들의 대화 기록인 듯 한데, 중간에 검은색으로 칠해진 부분이 눈에 띈다. 아마 저 부분이 문제의 flag일 것이다.  
pdf에는 여러 보안 기능을 설정할 수 있는데, 이 문서는 그 중에도 읽기 전용, 드래그 방지 등의 기능을 설정해 놓은 것으로 보였다.

pdf의 텍스트를 추출해 txt 파일로 변환해 주는 [사이트](https://convertio.co/kr/pdf-txt/)에 해당 파일을 업로드했다.

![4](/assets/img/posts/2024-01-15-fired.png)

정말 간단하게 flag를 찾았다. 내용을 보니 보안 담당자는 잘렸나보다.

## EnableMe

![5](/assets/img/posts/2024-01-15-2.png)

>You've received a confidential document! Follow the instructions to unlock it.

### 풀이

.docm 확장자의 파일이 첨부되어 있다. 찾아보니 매크로 기능이 적용된 doc 파일이라고 한다.

![6](/assets/img/posts/2024-01-15-troi.png)

파일을 열려고 하니 Windows Defender 에서 악성코드가 검출되었다며 파일을 삭제시키려고 했다.

아마 문서를 열 때 자동으로 실행되는 매크로가 적용되어 있기에 악성코드로 간주하는 것 같다.  
(실제로 예전엔 이런 유형의 멀웨어들이 많았다고 들었다.)  
보안 기능을 해제하고, 파일을 열어 보면...

![6](/assets/img/posts/2024-01-15-ee.png)

뭔가 플래그가 있을 것만 같은 이미지와 간단한 설명문이 보인다.  
<kbd>도구</kbd> - <kbd>매크로</kbd> - <kbd>매크로 편집</kbd> 순으로 들어가 어떤 매크로가 설정되어 있는지 확인해 보자.

![7](/assets/img/posts/2024-01-15-macro.png)

코드는 VBA(Visual Basic for Applications) 스크립트로, Microsoft Office 문서에 삽입될 수 있는 매크로 문법이다.  
코드의 세부 내용을 다음과 같이 정리해 보았다.

- 자동 실행  
    Sub AutoOpen 매크로는 문서가 열릴 때 자동으로 실행된다.

- 변수 초기화  
    v6, v7을 정수 배열로 선언하고 값을 지정한다.

- XOR 연산  
    첫 번째 For 루프는 v6 배열의 각 요소를 v8 값으로 XOR 연산하여 새로운 문자열 v9를 만든다.  
    Chr 함수는 ASCII 코드를 문자로 변환하며, Asc와 Mid 함수는 문자열에서 특정 문자의 ASCII 값을 추출한다.  
    두 번째 For 루프는 v7 배열의 각 요소를 v9의 문자와 XOR 연산하여 최종 문자열 v10을 생성한다.

- 메시지 박스 표시  
    최종적으로 생성된 문자열 v10을 메시지 박스로 표시한다.

코드 상으론 문서가 열릴 때 자동으로 실행되는 것이 맞는데 왜 아무 반응이 없었는지는 모르겠다.  
아무튼 코드를 직접 실행시켜 보면...

![8](/assets/img/posts/2024-01-15-hue.png)

장난 한번 치고 싶었나보다.

출력되는 메시지와 별개로 v10를 만들기 전에 사용되는 다른 변수들도 아스키 코드의 범주 내에서 생성되기 때문에, 혹시나 싶어 중단점을 설정해서 다른 변수들의 값도 알아보았다.

![8](/assets/img/posts/2024-01-15-wow.png)

첫 번째 루프 밑에 중단점을 넣고 v9를 조사식으로 검색해 보니 아니나 다를까 플래그가 튀어나왔다.

## Hourglass

![8](/assets/img/posts/2024-01-15-hourglass.png)

> No EDR agent once again, we imaged this workstation for you to find the evil !

### 풀이
구글 드라이브 링크로 10GB 가량의 ZIP 파일이 첨부되어 있다.  
zip 안에는 vm 패키지 파일(.ova)와 어떤 해시값으로 보이는 txt문서가 있다.  

![8](/assets/img/posts/2024-01-15-inzip.png)

txt 파일은 뭘 의미하는지 파악하기 힘드니 먼저 vm을 그대로 설치 해 보았다.

VMware Workstation 하이퍼바이저 환경에서 설치된 vmdk를 부팅하려고 시도했지만 계속 실패했다.  
부팅할 때 마다 오류 메시지가 계속 달라지고 바이오스 진입도 제대로 먹히지 않아 실패 원인이 뭔지 파악하기 힘들었다.  
ftk imager로 열어봤을 땐 파일시스템이 정상적으로 확인되는것을 보아 디스크 암호화나 MBR 손상은 아닌 듯 한데... 이걸 어떻게 고쳐야 할지 감이 오지 않았다.

결국 파일시스템은 제대로 인식이 된다는 점에서 착안해 vmdk 파일을 E01으로 통째로 이미징해 autopsy에서 분석을 진행하기로 결정했다.  

아마 정석적인 풀이 방법은 첨부된 txt에 적힌 해시값으로 뭔가 고치는 것이 아니였을까?? 사실 잘 모르겠다 ㅎㅎ  

![8](/assets/img/posts/2024-01-15-userfind.png)

먼저 컴퓨터 사용자들 중 "analyst"의 바탕화면에서 가짜 플래그 파일과 readme 파일을 발견했다.  
readme에는 해당 PC의 IoCs(Indicators of Compromise, 침해 지표)를 찾으라는 내용이 적혀 있었는데, 원래 찾으려던 게 그거라 큰 도움은 되지 않는 듯 하다.

여러 로그 파일을 분석해 보는 중 ConsoleHost_history.txt 에서 유의미한 징후를 찾았다.

> ConsoleHost_history.txt  
사용자가 PowerShell 세션 동안 실행한 명령어의 기록이 포함되어 있는 로그 문서.
{: .prompt-info}

![8](/assets/img/posts/2024-01-15-consolehis.png)

문서에서 주목할만한 내용은 다음과 같다.

1. 타임스탬프 조작  
    stomp_time.exe라는 프로그램을 사용하여 new.txt 의 타임스탬프를 조작한 기록이 나타났다. 
    하지만 색인에서 new.txt라는 파일은 생성 기록을 제외하곤 찾을 수 없었다.

2. RemoteSigned 정책 설정  
    원격 시스템에서 다운받은 스크립트는 실행에 서명이 필요하고, 로컬 시스템에서 생성한 스크립트는 서명 없이도 실행이 가능해진다.  
    시스템에서 ExecutionPolicy는 기본적으로 Restricted(:Undefined) 상태이며, Microsoft에서 서명한 일부 ps1 파일을 제외하곤 전부 실행을 거부한다.  
    해당 정책 설정으로 일단 ps1 파일 실행이 가능해졌다.

3. update.ps1 의 가명 설정  
    "C:\Windows\Web\Wallpaper\Theme2\update.ps1" 경로의 파일을 "UpdateSystem"으로 가명 설정했다.  
    바탕화면 테마가 저장되는 폴더에 있는 ps1 파일이 있는 것도 이상한데, updatesystem으로 명령 설정하는 것은 더욱 이상하다.  

autopsy의 <kbd>file search by attribute</kbd> 색인 기능으로 해당 파일을 검색했다.

![8](/assets/img/posts/2024-01-15-1flag.png)

축하 메시지와 파일 아래쪽의 IEX 명령이 보인다. 해당 명령을 살펴보면,  
https://somec2attackerdomain.com/chrome.exe 에서 파일을 다운로드해 즉시 실행하는 명령이다.  
이건 침해 지표로 판단하기에 충분해 보인다.

위쪽 스크립트는 'WowMadeitthisfar' 문자열을 $String_Key 변수에 저장하고, $chars 변수의 문자열과 xor 연산하여 resultArray를 만드는 내용이다.

해당 스크립트를 그대로 실행하면 동작이 되지 않는데, Powershell에선 배열과 배열을 bxor 했을때 각 요소를 따로 연산해주지 않기 때문이다.  

배열의 각 요소를 순환하며 개별적으로 연산하는 스크립트를 추가하였다. 전체 스크립트는 아래와 같다.

```shell
$String_Key = 'W0wMadeitthisfar'
$keyAscii = $String_Key.ToCharArray() | ForEach-Object { [int][char]$_ }

$chars = 34, 95, 17, 57, 2, 16, 3, 18, 68, 16, 12, 54, 4, 82, 24, 45, 35, 0, 40, 63, 20, 10, 58, 25, 3, 65, 0, 20

$resultArray = for ($i = 0; $i -lt $chars.Length; $i++) {
    $chars[$i] -bxor $keyAscii[$i % $keyAscii.Length]
}

ehco $resultArray
```
![8](/assets/img/posts/2024-01-15-calflag.png)

resultArray의 값을 아스키 코드로 변환하면 플래그가 나온다.

## No grep

![8](/assets/img/posts/2024-01-15-grep.png)

> Use the VM from Hourglass to find the 2nd flag on the system!

### 풀이
Hourglass 문제의 VM을 활용하여 2번째 플래그를 찾는 것이 목표이다.

현재까지 알아낸 주요 정보는 analyst 사용자가 update.ps1 스크립트를 작성하여  
위험 파일로 의심되는 https://somec2attackerdomain.com/chrome.exe 을 원격에서 실행하고자 시도했다는 것이다.  
이에 착안하여 update.ps1을 autopsy에서 키워드 서치를 통해 검색해 보았다.

그러자 기존에 확인했던 ConsoleHost_history.txt와 함께 WebCacheV01.dat가 색인되었다.

> WebCache2V01.dat  
windows8 이후로 사용자의 모든 웹브라우저 히스토리를 기록하는 로그 문서  
여러 개의 테이블로 나눠져 있으며 열람하기 위해 전용 데이터베이스 뷰어가 필요하다.
{: .prompt-info}

WebCache2V01.dat를 열람하기 위해 ESEDatabaseView를 사용했다.

[NirSoft - ESEDatabaseView](https://www.nirsoft.net/utils/ese_database_view.html)

50개 정도의 테이블이 있었는데, 이걸 일일이 순회하면서 직접 탐색을 했다.

![8](/assets/img/posts/2024-01-15-webcahce.png)

그중 25번 container에서 analyst 사용자의 기록을 발견했는데, 여기서 powershell.evtx와 update.ps1 에 접근한 것을 확인할 수 있었다.  
로컬에서 웹브라우저를 통해 로컬 파일에 접근하는 경우는 실제론 잘 없겠지만, 아무튼 이런 식으로 기록이 남을 수 있다는 것은 이해했다.

윈도우 이벤트로그를 분석할 때 powershell.evtx 기록이 비어 있는 상태였는데, 정황상 해당 사용자가 기록을 삭제했었던 것으로 보인다.  

또 analyst가 접근한 파일엔 flag.txt(가짜 플래그 파일), readme.txt가 있었는데, 이전까지의 탐색에서 확인하지 못한 settings.txt가 눈에 띄었다.

autopsy에서 settings를 검색해 보니, WebCache 로그의 경로 그대로 존재하고 있는 것을 확인했다.

파일을 열어 보면...

![8](/assets/img/posts/2024-01-15-findyou.png)

암호화된 듯한 문자열이 있는 것을 확인할 수 있다.  
이걸 Base64로 디코딩하면 플래그를 찾을 수 있다.

## 후기

다른 문제들도 찍먹해보긴 했는데, 뭔가 풀릴 듯 하면서 안풀리는게 참 아쉬웠던 것 같다.  
다음에는 더 좋은 성적으로 마무리하고 싶다!