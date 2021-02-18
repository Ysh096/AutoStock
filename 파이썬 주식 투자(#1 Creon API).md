# 파이썬 주식 투자(#1 Creon API)

# 1. 환경 설정 및 크레온 API 사용 기초

---

### 1-1. python 3.x 32bit 설치

python 3.8.7 32bit를 add to path하여 설치하였다. 그 후 관리자 권한 실행이 필요하다고 하여 python.exe 속성-호환성 탭의 관리자 권한으로 실행을 기본 속성으로 만들었다.

```python
폴더 경로: C:\Users\skk75\AppData\Local\Programs\Python\Python38-32
권한을 설정할 파일: python.exe, pythrow
```

### 1-2. vscode로 프로젝트 폴더 접속

프로젝트 폴더(stockauto)를 바탕화면에 만들고, 해당 폴더에 vscode를 관리자 권한으로 들어가서 터미널을 cmd 형태로 연다. (관리자 권한으로 들어가야 pip install pywinauto가 작동했음. 

### 1-3. pywinauto 설치

윈도우 작업을 자동화하는 파이썬 라이브러리, pywinauto를 설치한다.

```python
pip install pywinauto
```

### 1-4. python interpreter 변경

ctrl + shift + p 를 누른 후 python interpreter를 검색

수업을 들을 때에는 Python 3.5.3으로 설정해주자!

!!!지금은 Python 3.8.6 32-bit로 변경해주자.!!!

### 1-5. Creon 가입 및 시스템 트레이딩 신청

가입은 모바일로 한 후 웹사이트에 들어가서 로그인!

온라인 지점-서비스신청관리-시스템트레이딩-신청

이제 크레온 API를 사용할 수 있게 되었다!

### 1-6. 크레온 HTS 설치하기

고객라운지-트레이딩안내-다운로드센처-HTS 설치

다운로드 하고 크레온플러스로 로그인!

우측하단 크레온 플러스 단축메뉴를 우클릭하여 주문 오브젝트 사용 동의를 해 주고, 주문내역 확인 설정에서 주문 내역 확인 체크를 해제! (거래 발생마다 확인하지 않아도 됨)

### 1-7. 종목정보 구하기

google: 크레온 API

Creonplus start가 동작중이어야 가능한 방법!

자료실 - 파이썬 - 종목정보 구하는 예제 선택!

```python
import win32com.client
 
 
# 연결 여부 체크
objCpCybos = win32com.client.Dispatch("CpUtil.CpCybos")
bConnect = objCpCybos.IsConnect
if (bConnect == 0):
    print("PLUS가 정상적으로 연결되지 않음. ")
    exit()
 
# 종목코드 리스트 구하기
objCpCodeMgr = win32com.client.Dispatch("CpUtil.CpCodeMgr")
codeList = objCpCodeMgr.GetStockListByMarket(1) #거래소
codeList2 = objCpCodeMgr.GetStockListByMarket(2) #코스닥
 
 
print("거래소 종목코드", len(codeList))
for i, code in enumerate(codeList):
    secondCode = objCpCodeMgr.GetStockSectionKind(code)
    name = objCpCodeMgr.CodeToName(code)
    stdPrice = objCpCodeMgr.GetStockStdPrice(code)
    print(i, code, secondCode, stdPrice, name)
 
print("코스닥 종목코드", len(codeList2))
for i, code in enumerate(codeList2):
    secondCode = objCpCodeMgr.GetStockSectionKind(code)
    name = objCpCodeMgr.CodeToName(code)
    stdPrice = objCpCodeMgr.GetStockStdPrice(code)
    print(i, code, secondCode, stdPrice, name)
 
print("거래소 + 코스닥 종목코드 ",len(codeList) + len(codeList2))
```

# 2. 삼성전자 주가 Slack 알림 봇 만들기

---

### 2-1. slack workspace 만들기

slack 접속 - add workspace - 이름 마음대로 해서 만들어주자.

### 2-2. slack api를 통해 stock-bot 만들기

[api.slack.com](http://api.slack.com) 접속-start building-create a custom app-이름, 워크스페이스 지정!

OAuss & Permissions - Bot Token Scopes - Add on OAuth Scopes - chat:wirte 설정

→ install to workspace → 허용

OAuth token을 잘 저장해놓자.

slack에 보면 우측에 ! 모양 아이콘이 있는데, 거기서 앱을 추가할 수 있고, 우리가 방금 만든 stock-bot을 추가하여 사용한다.

### 2-3. slack-bot을 통해 메세지 보내기

slacker검색 - slacker github로 들어가자

여기에 사용 방법이 나와있다.

Installation

```python
pip install slacker
```

설치가 완료되었으면 다음의 Example을 붙여넣기 한다.

```python
from slacker import Slacker

slack = Slacker('<your-slack-api-token-goes-here>')

# Send a message to #general channel
slack.chat.post_message('#general', 'Hello fellow slackers!')
```

<your-slack-api-token-goes-here> 에 OAuth token을, #general에 stock-bot을 추가해놓은 채널 이름을 적어주면 vscode에서 test.py을 실행했을 때 메세지를 받아볼 수 있다.

### 2-4. slack-bot을 통해 삼성전자 주가 받아보기

크레온api - 자료실 - 파이썬3페이지 - 주식현재가 조회 예제

```python
import win32com.client
 
# 연결 여부 체크
objCpCybos = win32com.client.Dispatch("CpUtil.CpCybos")
bConnect = objCpCybos.IsConnect
if (bConnect == 0):
    print("PLUS가 정상적으로 연결되지 않음. ")
    exit()
 
# 현재가 객체 구하기
objStockMst = win32com.client.Dispatch("DsCbo1.StockMst")
objStockMst.SetInputValue(0, 'A005930')   #종목 코드 - 삼성전자
objStockMst.BlockRequest()
 
# 현재가 통신 및 통신 에러 처리 
rqStatus = objStockMst.GetDibStatus()
rqRet = objStockMst.GetDibMsg1()
print("통신상태", rqStatus, rqRet)
if rqStatus != 0:
    exit()
 
# 현재가 정보 조회
code = objStockMst.GetHeaderValue(0)  #종목코드
name= objStockMst.GetHeaderValue(1)  # 종목명
time= objStockMst.GetHeaderValue(4)  # 시간
cprice= objStockMst.GetHeaderValue(11) # 종가
diff= objStockMst.GetHeaderValue(12)  # 대비
open= objStockMst.GetHeaderValue(13)  # 시가
high= objStockMst.GetHeaderValue(14)  # 고가
low= objStockMst.GetHeaderValue(15)   # 저가
offer = objStockMst.GetHeaderValue(16)  #매도호가
bid = objStockMst.GetHeaderValue(17)   #매수호가
vol= objStockMst.GetHeaderValue(18)   #거래량
vol_value= objStockMst.GetHeaderValue(19)  #거래대금
 
# 예상 체결관련 정보
exFlag = objStockMst.GetHeaderValue(58) #예상체결가 구분 플래그
exPrice = objStockMst.GetHeaderValue(55) #예상체결가
exDiff = objStockMst.GetHeaderValue(56) #예상체결가 전일대비
exVol = objStockMst.GetHeaderValue(57) #예상체결수량
 
 
print("코드", code)
print("이름", name)
print("시간", time)
print("종가", cprice)
print("대비", diff)
print("시가", open)
print("고가", high)
print("저가", low)
print("매도호가", offer)
print("매수호가", bid)
print("거래량", vol)
print("거래대금", vol_value)
 
 
if (exFlag == ord('0')):
    print("장 구분값: 동시호가와 장중 이외의 시간")
elif (exFlag == ord('1')) :
    print("장 구분값: 동시호가 시간")
elif (exFlag == ord('2')):
    print("장 구분값: 장중 또는 장종료")
 
print("예상체결가 대비 수량")
print("예상체결가", exPrice)
print("예상체결가 대비", exDiff)
print("예상체결수량", exVol)
```

해당 코드를 test.py의 위쪽에 붙여넣는다.

여기서 우리는 필요한 정보를 메세지로 보내게 할 수 있다.

```python
from slacker import Slacker
import win32com.client
 
# 연결 여부 체크
objCpCybos = win32com.client.Dispatch("CpUtil.CpCybos")
bConnect = objCpCybos.IsConnect
if (bConnect == 0):
    print("PLUS가 정상적으로 연결되지 않음. ")
    exit()
 
# 현재가 객체 구하기
objStockMst = win32com.client.Dispatch("DsCbo1.StockMst")
objStockMst.SetInputValue(0, 'A005930')   #종목 코드 - 삼성전자
objStockMst.BlockRequest()
 
# 현재가 통신 및 통신 에러 처리 
rqStatus = objStockMst.GetDibStatus()
rqRet = objStockMst.GetDibMsg1()
print("통신상태", rqStatus, rqRet)
if rqStatus != 0:
    exit()
 
# 현재가 정보 조회
offer = objStockMst.GetHeaderValue(16)  #매도호가
 
# 예상 체결관련 정보
exFlag = objStockMst.GetHeaderValue(58) #예상체결가 구분 플래그
exPrice = objStockMst.GetHeaderValue(55) #예상체결가
exDiff = objStockMst.GetHeaderValue(56) #예상체결가 전일대비
exVol = objStockMst.GetHeaderValue(57) #예상체결수량
 
if (exFlag == ord('0')):
    print("장 구분값: 동시호가와 장중 이외의 시간")
elif (exFlag == ord('1')) :
    print("장 구분값: 동시호가 시간")
elif (exFlag == ord('2')):
    print("장 구분값: 장중 또는 장종료")
 
print("예상체결가 대비 수량")
print("예상체결가", exPrice)
print("예상체결가 대비", exDiff)
print("예상체결수량", exVol)

slack = Slacker('xoxb-1772908430276-1791234344848-3oDO2HQeiC3ghqTATuIgSSSs')

# Send a message to #general channel
slack.chat.post_message('#stock', '삼성전자 현재가:' + str(offer))
```

이런 식으로, 크레온plus를 실행한 상태로, vscode를 관리자 권한으로 열고 위의 test.py를 실행하면 삼성전자의 현재 매도호가(구입 가능 가격)를 슬랙을 통해 전송받을 수 있다.

# 3. 주식 투자 자동화 완성하기

---

### 1. 자동매매 코드 구현 및 코드 설명

### 1-1. 자동매매 코드 작성

1. vscode의 현재 폴더에 AutoTrade.py라는 파이썬 파일을 만들어주자.
2. 파이썬 증권 데이터 분석 github 접속! ([https://github.com/INVESTAR/StockAnalysisInPython](https://github.com/INVESTAR/StockAnalysisInPython))
3. 08 Volatility Breakout 들어가기
4. EtfAlgoTrader 들어가기
5. 그대로 복사 후 AutoTrade.py에 붙여넣기
6. 설치 안되어있는 라이브러리 설치하기(밑줄 그어진 것)
7. 슬랙 부분만 내 토큰으로 바꿔주고, 메세지 전송도 내 채널로 바꿔준다.

### 1-2. 코드 구성

```python
import os, sys, ctypes
import win32com.client
import pandas as pd
from datetime import datetime
from slacker import Slacker
import time, calendar

slack = Slacker('xoxb-1772908430276-1791234344848-3oDO2HQeiC3ghqTATuIgSSSs')
def dbgout(message):
    """인자로 받은 문자열을 파이썬 셸과 슬랙으로 동시에 출력한다."""
    print(datetime.now().strftime('[%m/%d %H:%M:%S]'), message)
    strbuf = datetime.now().strftime('[%m/%d %H:%M:%S] ') + message
    slack.chat.post_message('#stock', strbuf)

def printlog(message, *args):
    """인자로 받은 문자열을 파이썬 셸에 출력한다."""
    print(datetime.now().strftime('[%m/%d %H:%M:%S]'), message, *args)
 
# 크레온 플러스 공통 OBJECT
cpCodeMgr = win32com.client.Dispatch('CpUtil.CpStockCode')
cpStatus = win32com.client.Dispatch('CpUtil.CpCybos')
cpTradeUtil = win32com.client.Dispatch('CpTrade.CpTdUtil')
cpStock = win32com.client.Dispatch('DsCbo1.StockMst')
cpOhlc = win32com.client.Dispatch('CpSysDib.StockChart')
cpBalance = win32com.client.Dispatch('CpTrade.CpTd6033')
cpCash = win32com.client.Dispatch('CpTrade.CpTdNew5331A')
cpOrder = win32com.client.Dispatch('CpTrade.CpTd0311')  

def check_creon_system():
    """크레온 플러스 시스템 연결 상태를 점검한다."""
    # 관리자 권한으로 프로세스 실행 여부
    if not ctypes.windll.shell32.IsUserAnAdmin():
        printlog('check_creon_system() : admin user -> FAILED')
        return False
 
    # 연결 여부 체크
    if (cpStatus.IsConnect == 0):
        printlog('check_creon_system() : connect to server -> FAILED')
        return False
 
    # 주문 관련 초기화 - 계좌 관련 코드가 있을 때만 사용
    if (cpTradeUtil.TradeInit(0) != 0):
        printlog('check_creon_system() : init trade -> FAILED')
        return False
    return True

def get_current_price(code):
    """인자로 받은 종목의 현재가, 매수호가, 매도호가를 반환한다."""
    cpStock.SetInputValue(0, code)  # 종목코드에 대한 가격 정보
    cpStock.BlockRequest()
    item = {}
    item['cur_price'] = cpStock.GetHeaderValue(11)   # 현재가
    item['ask'] =  cpStock.GetHeaderValue(16)        # 매수호가
    item['bid'] =  cpStock.GetHeaderValue(17)        # 매도호가    
    return item['cur_price'], item['ask'], item['bid']

def get_ohlc(code, qty):
    """인자로 받은 종목의 OHLC 가격 정보를 qty 개수만큼 반환한다."""
    cpOhlc.SetInputValue(0, code)           # 종목코드
    cpOhlc.SetInputValue(1, ord('2'))        # 1:기간, 2:개수
    cpOhlc.SetInputValue(4, qty)             # 요청개수
    cpOhlc.SetInputValue(5, [0, 2, 3, 4, 5]) # 0:날짜, 2~5:OHLC
    cpOhlc.SetInputValue(6, ord('D'))        # D:일단위
    cpOhlc.SetInputValue(9, ord('1'))        # 0:무수정주가, 1:수정주가
    cpOhlc.BlockRequest()
    count = cpOhlc.GetHeaderValue(3)   # 3:수신개수
    columns = ['open', 'high', 'low', 'close']
    index = []
    rows = []
    for i in range(count): 
        index.append(cpOhlc.GetDataValue(0, i)) 
        rows.append([cpOhlc.GetDataValue(1, i), cpOhlc.GetDataValue(2, i),
            cpOhlc.GetDataValue(3, i), cpOhlc.GetDataValue(4, i)]) 
    df = pd.DataFrame(rows, columns=columns, index=index) 
    return df

def get_stock_balance(code):
    """인자로 받은 종목의 종목명과 수량을 반환한다."""
    cpTradeUtil.TradeInit()
    acc = cpTradeUtil.AccountNumber[0]      # 계좌번호
    accFlag = cpTradeUtil.GoodsList(acc, 1) # -1:전체, 1:주식, 2:선물/옵션
    cpBalance.SetInputValue(0, acc)         # 계좌번호
    cpBalance.SetInputValue(1, accFlag[0])  # 상품구분 - 주식 상품 중 첫번째
    cpBalance.SetInputValue(2, 50)          # 요청 건수(최대 50)
    cpBalance.BlockRequest()     
    if code == 'ALL':
        dbgout('계좌명: ' + str(cpBalance.GetHeaderValue(0)))
        dbgout('결제잔고수량 : ' + str(cpBalance.GetHeaderValue(1)))
        dbgout('평가금액: ' + str(cpBalance.GetHeaderValue(3)))
        dbgout('평가손익: ' + str(cpBalance.GetHeaderValue(4)))
        dbgout('종목수: ' + str(cpBalance.GetHeaderValue(7)))
    stocks = []
    for i in range(cpBalance.GetHeaderValue(7)):
        stock_code = cpBalance.GetDataValue(12, i)  # 종목코드
        stock_name = cpBalance.GetDataValue(0, i)   # 종목명
        stock_qty = cpBalance.GetDataValue(15, i)   # 수량
        if code == 'ALL':
            dbgout(str(i+1) + ' ' + stock_code + '(' + stock_name + ')' 
                + ':' + str(stock_qty))
            stocks.append({'code': stock_code, 'name': stock_name, 
                'qty': stock_qty})
        if stock_code == code:  
            return stock_name, stock_qty
    if code == 'ALL':
        return stocks
    else:
        stock_name = cpCodeMgr.CodeToName(code)
        return stock_name, 0

def get_current_cash():
    """증거금 100% 주문 가능 금액을 반환한다."""
    cpTradeUtil.TradeInit()
    acc = cpTradeUtil.AccountNumber[0]    # 계좌번호
    accFlag = cpTradeUtil.GoodsList(acc, 1) # -1:전체, 1:주식, 2:선물/옵션
    cpCash.SetInputValue(0, acc)              # 계좌번호
    cpCash.SetInputValue(1, accFlag[0])      # 상품구분 - 주식 상품 중 첫번째
    cpCash.BlockRequest() 
    return cpCash.GetHeaderValue(9) # 증거금 100% 주문 가능 금액

def get_target_price(code):
    """매수 목표가를 반환한다."""
    try:
        time_now = datetime.now()
        str_today = time_now.strftime('%Y%m%d')
        ohlc = get_ohlc(code, 10)
        if str_today == str(ohlc.iloc[0].name):
            today_open = ohlc.iloc[0].open 
            lastday = ohlc.iloc[1]
        else:
            lastday = ohlc.iloc[0]                                      
            today_open = lastday[3]
        lastday_high = lastday[1]
        lastday_low = lastday[2]
        target_price = today_open + (lastday_high - lastday_low) * 0.5 #변동성 돌파 전략
				#내가 원하는 전략으로 바꿀 수 있음!
        return target_price
    except Exception as ex:
        dbgout("`get_target_price() -> exception! " + str(ex) + "`")
        return None
    
def get_movingaverage(code, window):
    """인자로 받은 종목에 대한 이동평균가격을 반환한다."""
    try:
        time_now = datetime.now()
        str_today = time_now.strftime('%Y%m%d')
        ohlc = get_ohlc(code, 20)
        if str_today == str(ohlc.iloc[0].name):
            lastday = ohlc.iloc[1].name
        else:
            lastday = ohlc.iloc[0].name
        closes = ohlc['close'].sort_index()         
        ma = closes.rolling(window=window).mean()
        return ma.loc[lastday]
    except Exception as ex:
        dbgout('get_movingavrg(' + str(window) + ') -> exception! ' + str(ex))
        return None    

def buy_etf(code): #변동성 돌파 전략을 사용하는 매수 함수
    """인자로 받은 종목을 최유리 지정가 FOK 조건으로 매수한다."""
    try:
        global bought_list      # 함수 내에서 값 변경을 하기 위해 global로 지정
        if code in bought_list: # 매수 완료 종목이면 더 이상 안 사도록 함수 종료
            #printlog('code:', code, 'in', bought_list)
            return False
        time_now = datetime.now()
        current_price, ask_price, bid_price = get_current_price(code) 
        target_price = get_target_price(code)    # 매수 목표가(변동성 돌파 전략을 사용한 목표가, 임의 변경 가능)
        ma5_price = get_movingaverage(code, 5)   # 5일 이동평균가
        ma10_price = get_movingaverage(code, 10) # 10일 이동평균가
        buy_qty = 0        # 매수할 수량 초기화
        if ask_price > 0:  # 매수호가가 존재하면   
            buy_qty = buy_amount // ask_price  
        stock_name, stock_qty = get_stock_balance(code)  # 종목명과 보유수량 조회
        #printlog('bought_list:', bought_list, 'len(bought_list):',
        #    len(bought_list), 'target_buy_count:', target_buy_count)     
        if current_price > target_price and current_price > ma5_price \ 
            and current_price > ma10_price:  #이동평균선 위에 있을 때, 타겟 가격보다 높을 때 매수
            printlog(stock_name + '(' + str(code) + ') ' + str(buy_qty) +
                'EA : ' + str(current_price) + ' meets the buy condition!`')            
            cpTradeUtil.TradeInit()
            acc = cpTradeUtil.AccountNumber[0]      # 계좌번호
            accFlag = cpTradeUtil.GoodsList(acc, 1) # -1:전체,1:주식,2:선물/옵션                
            # 최유리 FOK 매수 주문 설정
#최유리: 당장 가장 유리하게 매매할 수 있는 가격(가장 낮은 매도호가)
#최우선: 우선 대기하는 가격(가장 낮은 매도호가의 한 단계 아래)
#IOC: 체결 후 남은 수량 취소
#FOK: 전량 체결되지 않으면 주문 자체를 취소
            cpOrder.SetInputValue(0, "2")        # 2: 매수
            cpOrder.SetInputValue(1, acc)        # 계좌번호
            cpOrder.SetInputValue(2, accFlag[0]) # 상품구분 - 주식 상품 중 첫번째
            cpOrder.SetInputValue(3, code)       # 종목코드
            cpOrder.SetInputValue(4, buy_qty)    # 매수할 수량
            cpOrder.SetInputValue(7, "2")        # 주문조건 0:기본, 1:IOC, 2:FOK
            cpOrder.SetInputValue(8, "12")       # 주문호가 1:보통, 3:시장가
                                                 # 5:조건부, 12:최유리, 13:최우선 
            # 매수 주문 요청
            ret = cpOrder.BlockRequest() 
            printlog('최유리 FoK 매수 ->', stock_name, code, buy_qty, '->', ret)
            if ret == 4:
                remain_time = cpStatus.LimitRequestRemainTime
                printlog('주의: 연속 주문 제한에 걸림. 대기 시간:', remain_time/1000)
                time.sleep(remain_time/1000) 
                return False
            time.sleep(2)
            printlog('현금주문 가능금액 :', buy_amount)
            stock_name, bought_qty = get_stock_balance(code)
            printlog('get_stock_balance :', stock_name, stock_qty)
            if bought_qty > 0:
                bought_list.append(code)
                dbgout("`buy_etf("+ str(stock_name) + ' : ' + str(code) + 
                    ") -> " + str(bought_qty) + "EA bought!" + "`")
    except Exception as ex:
        dbgout("`buy_etf("+ str(code) + ") -> exception! " + str(ex) + "`")

def sell_all():
    """보유한 모든 종목을 최유리 지정가 IOC 조건으로 매도한다."""
    try:
        cpTradeUtil.TradeInit()
        acc = cpTradeUtil.AccountNumber[0]       # 계좌번호
        accFlag = cpTradeUtil.GoodsList(acc, 1)  # -1:전체, 1:주식, 2:선물/옵션   
        while True:    
            stocks = get_stock_balance('ALL') 
            total_qty = 0 
            for s in stocks:
                total_qty += s['qty'] 
            if total_qty == 0:
                return True
            for s in stocks:
                if s['qty'] != 0:                  
                    cpOrder.SetInputValue(0, "1")         # 1:매도, 2:매수
                    cpOrder.SetInputValue(1, acc)         # 계좌번호
                    cpOrder.SetInputValue(2, accFlag[0])  # 주식상품 중 첫번째
                    cpOrder.SetInputValue(3, s['code'])   # 종목코드
                    cpOrder.SetInputValue(4, s['qty'])    # 매도수량
                    cpOrder.SetInputValue(7, "1")   # 조건 0:기본, 1:IOC, 2:FOK
                    cpOrder.SetInputValue(8, "12")  # 호가 12:최유리, 13:최우선 
                    # 최유리 IOC 매도 주문 요청
                    ret = cpOrder.BlockRequest()
                    printlog('최유리 IOC 매도', s['code'], s['name'], s['qty'], 
                        '-> cpOrder.BlockRequest() -> returned', ret)
                    if ret == 4:
                        remain_time = cpStatus.LimitRequestRemainTime
                        printlog('주의: 연속 주문 제한, 대기시간:', remain_time/1000)
                time.sleep(1)
            time.sleep(30)
    except Exception as ex:
        dbgout("sell_all() -> exception! " + str(ex))

if __name__ == '__main__':  #Symbol_list에 자동매매를 하고자 하는 종목코드를 넣어준다.
#내가 자동매매를 원하는 종목을 담아준다.
    try:
        symbol_list = ['A122630', 'A252670', 'A233740', 'A250780', 'A225130',
             'A280940', 'A261220', 'A217770', 'A295000', 'A176950']
        bought_list = []     # 매수 완료된 종목 리스트
        target_buy_count = 5 # 매수할 종목 수(최대)
        buy_percent = 0.19   
        printlog('check_creon_system() :', check_creon_system())  # 크레온 접속 점검
        stocks = get_stock_balance('ALL')      # 보유한 모든 종목 조회
        total_cash = int(get_current_cash())   # 100% 증거금 주문 가능 금액 조회
        buy_amount = total_cash * buy_percent  # 종목별 주문 금액 계산
        printlog('100% 증거금 주문 가능 금액 :', total_cash)
        printlog('종목별 주문 비율 :', buy_percent)
        printlog('종목별 주문 금액 :', buy_amount)
        printlog('시작 시간 :', datetime.now().strftime('%m/%d %H:%M:%S'))
        soldout = False

        while True:
            t_now = datetime.now() #현재 시간
            t_9 = t_now.replace(hour=9, minute=0, second=0, microsecond=0)
            t_start = t_now.replace(hour=9, minute=5, second=0, microsecond=0) #시작 시간
            t_sell = t_now.replace(hour=15, minute=15, second=0, microsecond=0) #3:15분 종료
            t_exit = t_now.replace(hour=15, minute=20, second=0,microsecond=0) #프로그램 종료 시간
            today = datetime.today().weekday() #오늘이 며칠인지?
            if today == 5 or today == 6:  # 토요일이나 일요일이면 자동 종료
                printlog('Today is', 'Saturday.' if today == 5 else 'Sunday.')
                sys.exit(0)
            if t_9 < t_now < t_start and soldout == False:
                soldout = True
                sell_all()
            if t_start < t_now < t_sell :  # AM 09:05 ~ PM 03:15 : 매수
                for sym in symbol_list:
                    if len(bought_list) < target_buy_count:
                        buy_etf(sym) #가격이 맞으면 매수!, 변동성 돌파 전략을 사용
                        time.sleep(1)
                if t_now.minute == 30 and 0 <= t_now.second <= 5: #30분마다 잔고를 보여줌
                    get_stock_balance('ALL')
                    time.sleep(5)
            if t_sell < t_now < t_exit:  # PM 03:15 ~ PM 03:20 : 일괄 매도
                if sell_all() == True:
                    dbgout('`sell_all() returned True -> self-destructed!`')
                    sys.exit(0)
            if t_exit < t_now:  # PM 03:20 ~ :프로그램 종료
                dbgout('`self-destructed!`')
                sys.exit(0)
            time.sleep(3)
    except Exception as ex:
        dbgout('`main -> exception! ' + str(ex) + '`')
```

### 2. 크레온 Plus 자동 접속 코드 구현

08_volatility_Breakout 의 첫 번째 파일, ch08_01_AutoConnect.py를 복붙한다.

아이디 비밀번호, 공인인증서 비밀번호 입력!

실행하면 자동 로그인이 이루어짐.

다만 이렇게 한다고 해서 자동매매라고 할 수는 없으니, 완전히 자동으로 실행이 되도록 만들어야한다.

### 3. 작업 스케줄러로 완전 자동화

시작 - 작업스케쥴러 - 특정 프로그램을 특정 시간에 실행할 수 있음.

새 작업 만들기 - 이름, 설명, 가장 높은 수준의 권한으로 실행

1. 크레온 연결
    1. 일반에서 이름 설정
    2. 트리거에서 시작 시간 설정
    3. 동작에서 python38-32bit의 위치를 프로그램/스크립트에, 인수 추가에 AutoConnect.py를, 시작 위치에 AutoConnect.py의 위치를 저장한다.
    4. AutoTrade.py도 같은 방법으로 추가해준다.
    5. 컴퓨터도 켜놓지 않으려면 AWS 서버를 빌려서 사용해야 하는데, 비용을 잘 고려해서 판단하자.