---
title: suninatas 21번 writeup
date: 2024-01-08 16:00:10 +0900
categories: [Writeup, Forensic]
tags: [john-the-riper]
render_with_liquid: false
---

## 문제
http://suninatas.com/challenge/web21/web21.asp

> What is a Solution Key?  
Is it a Puzzle?
{: .prompt-info}

## 풀이

### 살펴보기
문제를 봤을 때 가장 먼저 보이는 건 플래그가 출력되있는 듯한 모니터를 자로 가려놓은 모습이다.

![1](/assets/img/posts/2024-01-06-what.png)

이미지를 다운받고 처음 드는 생각은 1.4MB로 용량이 과도하게 크다는 것이다.

jpg 이미지에 다른 이미지나 파일이 첨부되어 있을 가능성이 가장 높다고 생각해 HxD 에디터로 파일의 RAW 값을 확인해 보기로 했다.

### HxD 에디터를 통한 분석

jpg 파일의 해더 시그니처는 FF D8 FF E1이며, 푸터 시그니처는 FF D9 이다. 위 시그니처 값을 검색해본 결과...

![2](/assets/img/posts/2024-01-06-yoya.png)
![3](/assets/img/posts/2024-01-06-FFD9.png)

너무 많다. hxd 편집기 기능의 한계로 한땀한땀 이것들을 각각의 이미지로 분리해줘야 하는데 따로 코드를 짜야 하나 싶었다.

![4](/assets/img/posts/2024-01-06-bunri.jpg)


마지막 시그니처 블록만 따로 떼어 봤는데 원본 이미지와 자의 위치가 미묘하게 다른 것 빼곤 답을 모르겠는건 여전하다.

### WinHex 사용

이미지 분리를 구글에 검색해보니 WinHex에 파일에 감춰진 정보를 분리해주는 기능이 있다는 것을 알았다.

Winhex에서 문제 이미지를 불러온 다음,
<kbd>Tools</kbd> - <kbd>disk tools</kbd> - <kbd>file recobery by type</kbd>
순으로 들어가서, complete byte-level search 옵션을 통해 해더 시그니처별로 파일을 분리하도록 지정한다.

![5](/assets/img/posts/2024-01-06-winhexgood.png)

![6](/assets/img/posts/2024-01-06-omg.png)

이후 분리된 파일들을 확인해 보면 수없이 많은 이미지들이 숨겨져 있던 것을 확인할 수 있다. 안 찾아봤으면 이걸 수작업으로 분리할 뻔 했다.

이미지들간의 문자를 잘 조합해 보면 성공!