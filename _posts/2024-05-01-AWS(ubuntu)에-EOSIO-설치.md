---
title: AWS(ubuntu)에 EOSIO 설치
date: 2024-05-02 00:15:56 +0900
categories: [Blockchain]
tags: [EOSIO]
render_with_liquid: false
---

# AWS(ubuntu)에 EOSIO 설치

[EOSIO Guide](https://developers.eos.io/welcome/latest/getting-started-guide/local-development-environment/installing-eosio-binaries) 문서의 초기 설정에 관한 내용을 정리해 보았다.

처음엔 AWS 환경 내에서 설치하려고 했지만 AWS 계정 문제로 접근이 막혀서,,, 일단 로컬 가상머신(**ubuntu 20.04 sever**) 기준으로 작성하였다.

### 기본 바이너리 설치

`wget https://github.com/eosio/eos/releases/download/v2.1.0/eosio_2.1.0-1-ubuntu-18.04_amd64.deb`

### cdt(Contract Development Toolkit/cloes) 설치

[eosio.cdt_1.8.0-1-ubuntu-20.04_amd64.deb](https://github.com/EOSIO/eosio.cdt/releases/download/v1.8.0/eosio.cdt_1.8.0-1-ubuntu-20.04_amd64.deb) 사용함

`wget https://github.com/eosio/eosio.cdt/releases/download/v1.8.0/eosio.cdt_1.8.0-1-ubuntu-20.04_amd64.deb`

### 지갑(Keosd) 설치

- 지갑 생성

`cleos wallet create --to-console`

—to-console 옵션을 통해 키를 미리 확인한다.

![1](/assets/img/posts/2024-05-01/Untitled.png)

기본 지갑 생성

PW5JJcjmrg7m8uJWN5d1FN1k4XGKJ7nL4TyKNTWWh5wM9AAJPRn9G

해당 명령에서 받은 키를 꼭 기억해 두자. 앞으로 지갑에 접근할 때 계속 사용된다.

- 지갑 열기

`cleos wallet open`

- 지갑 목록 반환

`cleos wallet list`

- 지갑 잠금 해제

지갑을 생성할 때 받은 키로 패스워드를 입력한다.
`cleos wallet unlock`

잠금 해제된 지갑은 아래와 같이 list에서 ‘*’ 로 구분할 수 있다.

```
Wallets:
[
  "default *"
]
```

### 개인 키 생성

`cleos wallet create_key`

![2](/assets/img/posts/2024-05-01/Untitled_1.png)

명령 실행 화면

### 개발 키 import

EOSIO 블록체인은 시작할 때 "eosio"라는 개발용 사용자가 설정되어 있다. 블록체인을 설정하고 운영하는 데 사용되는 시스템 계약이 이 사용자에게 처음으로 할당되며,  개발용 키를 사용하여 "eosio" 사용자에 대한 작업을 승인한다고 한다.

`cleos wallet import` 로 계정을 불러올 수 있으며, 비밀번호를 입력하는 부분에서 아래의 개발 키를 입력하면 “eosio” 시스템 사용자에 엑세스할 수 있다.

```
5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

이제 cleos를 통해 keosd와 nodeos를 시작해 보자.

### keosd 시작

`keosd &`

### nodeos 시작

아래 명령을 그대로 사용한다…

```bash
nodeos -e -p eosio \
--plugin eosio::producer_plugin \
--plugin eosio::producer_api_plugin \
--plugin eosio::chain_api_plugin \
--plugin eosio::http_plugin \
--plugin eosio::history_plugin \
--plugin eosio::history_api_plugin \
--filter-on="*" \
--access-control-allow-origin='*' \
--contracts-console \
--http-validate-host=false \
--verbose-http-errors >> nodeos.log 2>&1 &
```

다음 명령은 Nodes를 실행함과 동시에 모든 기본 플러그인을 로드하고, 개발에 필요한 디버깅, 로그 설정을 추가한다.

주의할 점으로 CORS(Cross-origin resource sharing)을 제한 없이 활성화하는데, 보안상의 문제로 개발 환경에서만 사용할 수 있도록 주의해야 한다.

실행 이후, 생성한 nodeos.log에서 eosio의 서명을 통해 블록이 생성되는 것을 확인할 수 있다.

![tail](/assets/img/posts/2024-05-01/Untitled_2.png)

tail -f nodeos.log

- 엔드포인트 확인

`curl http://localhost:8888/v1/chain/get_info`

해당 명령을 통해 가상머신에서 http 페이지를 통해 chain의 세부적인 정보를 표시하고 있음을 확인할 수 있다.

![3](/assets/img/posts/2024-05-01/Untitled_3.png)

### 개발용 계정 생성

이전 설치 과정에서 개발용 private 키를 사용해 eosio 계정의 공개 키를 얻었는데, 해당 키를 사용해서 개발용 계정 2개를 생성한다.

`cleos create account <생성 주체 계정명> <생성할 계정명><key>`

![Untitled](/assets/img/posts/2024-05-01/Untitled_4.png)

이 때, 새로 keosd를 연다고 default 지갑이 다시 닫혀 있을 텐데, 다시 open > unlock 명령으로 열어야 한다.

### 공개 키 확인

cleos get account <확인할 계정명>

해당 명령은 특정 계정의 상태를 확인하는 데 사용된다.

![Untitled](/assets/img/posts/2024-05-01/Untitled_5.png)

eosio 계정으로 생성한 alice 계정의 정보가 출력되는 것을 확인할 수 있다.