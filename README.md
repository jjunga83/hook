# TradingHook

### 트레이딩뷰에서 전달되는 웹훅을 처리하는 서버입니다.

&nbsp;
안녕하세영
- 지원 거래소
  - 업비트 KRW(원화) 마켓
  - 바이낸스 현물/선물 USDT,BUSD 마켓

#### <주의>

_본 프로젝트는 개인적으로 개발한 프로젝트를 오픈소스로 공유한 것으로_

_발생하는 문제에 대한 모든 책임은 본인에게 있습니다._

[리눅스 설치(클라우드)](#리눅스설치) | [윈도우PC로 설치](#윈도우설치) | [포트포워딩](#포트포워딩) | [테스트](#테스트)

---
## 리눅스설치


> ### [1] VULTR 가입
>[무료 $100 크레딧 받고 가입](https://www.vultr.com/?ref=9160659-8H) 

&nbsp;


> ### [3] 홈 디렉토리에서 .bash_aliases 다운받기
```bash
curl -LO https://raw.githubusercontent.com/jangdokang/TradingHook/main/linux/.bash_aliases
``` 

> ### [4] 테스트 코드
```javascript
//@version=5
strategy("BarUpDn Strategy", overlay=true, initial_capital=10 ,default_qty_type = strategy.percent_of_equity, default_qty_value = 100)
import dokang/TradingHook/3 as TH

maxIdLossPcnt = input.float(1, "Max Intraday Loss(%)")
strategy.risk.max_intraday_loss(maxIdLossPcnt, strategy.percent_of_equity)

var isOk = false
if barstate.islastconfirmedhistory
    isOk := true

if isOk
    if (close > open and open > close[1])
    	strategy.entry("BarUp", strategy.long, alert_message = TH.entry_message("dokang", order_name="바업다운 롱"))
    if (close < open and open < close[1])
    	strategy.entry("BarDn", strategy.short, alert_message = TH.entry_message("dokang", order_name="바업다운 숏"))
``` 

---
## 윈도우설치

> ### [1] 커맨드창에서 가상환경 설치

```bash
python -m venv .venv
```

&nbsp;

> ### [2] install.bat 파일 실행

> ### 혹은 아래 명령어를 커맨드창에 입력

```bash
.venv\Scripts\activate.bat
pip install -r requirements.txt
```

&nbsp;

> ### [3] .env 파일 수정

```python
# 파인스크립트에서 정한 PASSWORD와 똑같이 적어주세요
PASSWORD = "Your Password"      # 패스워드 적어주세요

# UPBIT에서 발급받은 KEY와 SECRET을 입력하세요
UPBIT_KEY = ""                  # 업비트 Access Key 입력
UPBIT_SECRET = ""               # 업비트 Secret Key 입력

# BINANCE에서 발급받은 KEY와 SECRET을 입력하세요
BINANCE_KEY = ""                # 바이낸스 API KEY 입력
BINANCE_SECRET = ""             # 바이낸스 SECRET KEY 입력

# DISCORD 웹훅 URL을 입력하세요
DISCORD_WEBHOOK_URL = ""        # 웹훅 URL 입력

# WHITELIST, 서버를 실행할 PC의 IP를 입력하세요
WHITELIST = ["127.0.0.1"]

# 서버의 포트 번호를 설정하세요
PORT = "8000" # 리눅스 클라우드(vultr)는 80으로 수정
```

&nbsp;

> ### [4] start.bat 파일 실행

> ### 혹은 아래 명령어를 커맨드창에 입력

```bash
.venv\Scripts\activate.bat
python run.py
```

---

## 포트포워딩

> TradingHook은 8000번 포트로 실행됩니다. 공유기의 포트포워드 설정으로 외부포트는 80번에서 8000번 포트로 포트포워딩 하도록 설정하세요.

&nbsp;&nbsp;

---

## 테스트
### 현물(업비트)

> 트레이딩훅 실행과 설정이 완료되면 이제 트레이딩뷰에서 파인스크립트를 새로 만들어 아래 스크립트를 넣고 차트에 넣기(Add to Chart)를 누릅니다.

> 암호를 입력하는 창이 뜨면 .env 파일에서 설정했던 암호와 동일하게 입력합니다.

```js
//@version=5

password = input.string("Your Password", title="Password", confirm = true)
// 원화로 10000원을 100%로 진입하게 합니다
strategy("[TH] TEST", currency = currency.KRW, initial_capital = 10000, default_qty_type = strategy.percent_of_equity, default_qty_value = 100, overlay = true)

// TradingHook 라이브러리를 불러와서 TH라는 이름으로 사용합니다
import dokang/TradingHook/2 as TH

// isOk라는 변수를 통해 전략이 추가된 시점부터 진입하게 합니다
var isOk = false
if barstate.islastconfirmedhistory
    isOk := true
// isOK가 true 일 때
if isOk
    // 봉 처음에 바로 매수
    strategy.entry("Buy", strategy.long, alert_message = TH.buy_message(password))
    // 그 다음 봉 시작하자마자 매도
    strategy.close("Buy", alert_message = TH.sell_message(password, "100%"))
```

&nbsp;

> - **TH.buy_message(string password, float amount = na, string order_name="Order")**
>   - [필수] password 패스워드
>   - [선택] amount 수량
>   - [선택] order_name 주문이름
>
> #### TH.buy_message("암호")는 전략테스터의 기본 설정값(initial_capital, default_qty_type, default_qty_value)에 맞춰서 매수 메세지를 생성하고 트레이딩훅 서버로 전달합니다.
>
> #### TH.buy_message("암호", amount=10)는 전략테스터의 기본 설정값과 상관없이 내가 입력한 수량(10) 만큼 매수 메세지를 생성하고 트레이딩훅 서버로 전달합니다.

&nbsp;

> - **TH.sell_message(string password, string percent, string order_name="Order")**
>   - [필수] password 패스워드
>   - [필수] percent 매도 비중(%)
>   - [선택] order_name 주문이름
>
> #### TH.sell_message("암호", "100%")는 전략테스터의 기본 설정값과 상관 없이 내가 입력한 퍼센트 만큼 보유물량에서 매도합니다.
&nbsp;
## Dependency

> [fastapi](https://github.com/tiangolo/fastapi)
> [ccxt](https://github.com/ccxt/ccxt)
> [uvicorn](https://github.com/encode/uvicorn)