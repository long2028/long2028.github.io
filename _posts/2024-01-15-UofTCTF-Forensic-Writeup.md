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

pdf의 텍스트를 추출해 txt 파일로 변환해 주는 [사이트](!https://convertio.co/kr/pdf-txt/)에 해당 파일을 업로드했다.

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

코드는 VBA(Visual Basic for Applications) 스크립트로, Microsoft Office 문서에 삽입될 수 있는 매크로 문법이다. 코드의 세부 내용을 다음과 같이 정리해 보았다.

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

## No grep



## Hourglass