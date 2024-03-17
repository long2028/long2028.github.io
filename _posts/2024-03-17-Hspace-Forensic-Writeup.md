---
title: 2024 Hspace Forensic Writeup
date: 2024-03-17 00:15:56 +0900
categories: [Forensic, Writeup]
tags: [Hspace]
render_with_liquid: false
---

## 분명 유출됐는데...

![1](/assets/img/posts/2024-03-17-num1.png)

기밀 정보가 포함된 문서 파일을 유출한 사건이 발생하였고, 유출한 경로를 파악하여 해당 문서를 여는 것이 문제이다.

### 풀이

## 컴퓨터가 이상해

![1](/assets/img/posts/2024-03-17-num2.png)

악성 프로그램이 실행 된 것이 의심되는 상황이며, 이 PC를 조사하여 공격자의 IP와 PC명을 알아내는 것이 목표이다.

### 풀이

![2](/assets/img/posts/2024-03-17-wallet.png)

문제 파일에 있는 ad1 형식 이미지 파일을 FTK imager로 불러와 분석을 시작하였다.

우선 "space" 사용자의 폴더를 확인했을 때, 다운로드 폴더에서 %xeno%이라는 수상한 폴더와 wallet.zip 파일이 보인다.

wallet.zip 폴더는 암호가 걸려 있어 풀 수는 없지만 내부 파일을 확인해 봤을 때 wallet.exe 파일이 있고, 다운로드 폴더와 %xeno% 안에 wallet.exe가 있기 때문에 이를 먼저 분석해보기로 했다.

무료 라이센스로 사용하는 IDA는 C#으로 프로그래밍된 실행 파일을 분석할 수 없기에, dotPeek을 사용하여 코드를 분석하기로 했다.

