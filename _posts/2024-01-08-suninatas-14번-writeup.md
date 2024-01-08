---
title: suninatas 14번 writeup
date: 2024-01-08 14:30:10 +0900
categories: [Writeup, Forensic]
tags: [john-the-riper]
render_with_liquid: false
---

## 문제
http://suninatas.com/challenge/web14/web14.asp

> Do you know password of suninatas?
{: .prompt-info}

## 풀이

### 살펴보기
문제의 첨부파일(tar)를 다운받아 압축 해제를 해 보면

![1](/assets/img/posts/2024-01-06-evidences.png)

shadow, password 파일이 보인다.
- password  
    리눅스 시스템의 사용자 계정 정보를 담고 있는 파일, 실제 암호는 shadow 파일에 저장된다.
- shadow  
    리눅스 시스템에서 사용자 계정의 암호화된 패스워드와 관련 정보를 저장하는 파일, 패스워드는 해시 후 저장되어 일반적으론 해독할 수 없다.

먼저 파일을 편집기로 열어 내부 필드값을 확인해보기로 했다. 난 vscode를 사용함.

![2](/assets/img/posts/2024-01-06-shadow.png)

shawdow 파일을 열어 보니 문제에서 말하는 suninats의 계정 정보가 마지막 줄에 나열되어 있는 것을 확인할 수 있다.

전체 내용은 아래와 같이 정리해 보았다.

<pre>suninatas:$6$QlRlqGhj$BZoS9PuMMRHZZXz1Gde99W01u3kD9nP/zYtl8O2dsshdnwsJT/1lZXsLar8asQZpqTAioiey4rKVpsLm/bqrX/:15427:0:99999:7:::</pre>

1. 사용자 이름

suninatas  

2. 암호화된 패스워드

\$6$는 SHA-512로 해시되었다는 것을 알림.  
QlRlqGhj는 이 패스워드의 솔트(salt).  
나머지 부분은 솔트와 패스워드를 결합해 해시한 결과.

3. 마지막으로 패스워드가 변경된 날짜

15427

4. 패스워드 변경 최소 기간

0

5. 패스워드 변경 최대 기간

99999

6. 경고 기간

7

7. 비활성화 기간

(빈칸)

8. 계정 만료일

(빈칸)

### 암호 해독

해시는 단방향 알고리즘으로 취약한 패스워드 사전에 포함된게 아니라면 해독이 불가능하다고 생각했는데, shawdow 복호화를 구글에 검색해 보니 그걸 할 수 있는 프로그램이 금방 나왔다.

<b>john the ripper</b> 는 브루트포스와 미리 해시된 패스워드 사전을 적절히 활용하여 암호를 해독할 수 있는 유명한 프로그램이다. 해당 프로그램을 사용하여 문제를 해결하기로 했다.

그냥 john the ripper을 다운받아 커맨드라인에서 실행할 수도 있지만, 자주 사용할 것 같아 GUI 버전인 johnny를 설치했다.

![3](/assets/img/posts/2024-01-06-johnny.png)

shawdow 파일을 프로그램에 넣고 해독을 시작하니 10초도 안 되서 크랙이 완료되었다...

![4](/assets/img/posts/2024-01-06-성공.png)

성공!