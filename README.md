# Example_of_task_automation_using_Selenium
```
외주작업을 통하여 처음 다루어보았던 'Selenium' 에 대한 글 입니다.
특히 Network response를 받아오는 방법에 대한 정보가 없어 도움이 되고자 정보로 남깁니다.

이번 작업에 있어 제일 중요한 task는 조회 기능이었습니다.
현재 흔히 웹 크롤링을 하기위해 BeautifulSoup(패키지), Selenium(프레임워크), requests(라이브러리) 등이 쓰이고 있는데요. 
'Selenium'은 그 중에서도 동적 페이지 크롤링 작업에 대하여 매우 수월합니다.  
```

## 
[0. 크롬 연결](#0-크롬-연결하기)  
[1. 로그인](#1-로그인)  
[2. 특정 메뉴, 버튼 클릭](#2-특정-메뉴-혹은-버튼-클릭)  
[3. Network Response 데이터 크롤링](#3-network-response-데이터-크롤링--조회된-데이터-값들-뽑기-)  
[4. DB서버(mssql) 저장](#4-db서버-저장)  
[5. exe파일 만들기 구현](#5-exe파일-만들기)  
    
[전체 코드](#전체코드--0--4-)  
##

<br>

<br>

## 0. 크롬 연결하기
```python
from seleniumwire import webdriver

'''
ChromeDriver를 현재 작업하고 있는 폴더와 같은 위치에 설치 (https://sites.google.com/a/chromium.org/chromedriver/downloads)
  -> (주의!) selenium의 webdriver버전과 사용하고 있는 chrome의 버전이 같아야 합니다.
  -> ex) 크롬 버전 버전 91.0.4472.114(공식 빌드) (64비트) 일 때, ChromeDriver 91.0.4472.101
'''

def resource_path(relative_path):
    try:
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.dirname(__file__)
    return os.path.join(base_path, relative_path)

driver = webdriver.Chrome(resource_path('./driver/chromedriver.exe')) 

driver.get("불러올 페이지 주소")

```

<br>

<br>

## 1. 로그인
```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

'''
'chrome 설정 - 도구 더보기 - 개발자 도구' or '오른쪽 마우스 - 검사' 등을 통해 아이디와 비밀번호에 대한 id값 찾기
( 2021년 6월 26일 기준 Naver의 경우 - 아이디 : id_area, 비빌번호 : pw_area )
로그인 화면 주소에서 입력 ( driver.get("로그인 화면 주소") 이후 입력 )
'''
driver.find_element_by_id("id_area").send_keys("아이디")
driver.find_element_by_id("pw_area").send_keys("비밀번호")


'''
Explicit Wait - 명시적 대기 - 조건이 True가 될 때까지 지정된 시간만큼 대기
( 현재 코드에서 쓰는 조건 - presence_of_element_located : 요소가 위치하여 있으면 실행 )
'''
def btn_click(x_path):
    WebDriverWait(driver, timeout=60).until(
        EC.presence_of_element_located((By.XPATH,x_path))
    ).click()
       
       
'''
로그인 버튼에 해당하는 요소를 찾은 후, '오른쪽 마우스 - Copy - Copy XPath' 를 통해 버튼 클릭하도록 하기
( 2021년 6월 26일 기준 Naver의 경우 - XPath : //*[@id="log.login"] ) 
'''
btn_click("""//*[@id="log.login"]""")


'''연결을 끊을 경우'''
driver.close()
```

<br>

<br>

## 2. 특정 메뉴 혹은 버튼 클릭
### 메뉴 클릭
```python
'''
상단의 btn_click 함수 활용
마찬가지로 XPath를 복사하여 붙여넣기
'''
btn_click("""메뉴의 버튼에 해당하는 XPath""")

```

### iframe 안에 있는 요소 클릭
```python
'''
눌러야 하는 버튼이 iframe 안에 있는 경우, driver를 'switch_to.frame'라는 함수를 사용하여 바꿔주어야 함
바꾸어 주었다면 frame 안의 요소들 click 가능
'''
driver.switch_to.frame(driver.find_element_by_id("프레임id"))

```

<br>

<br>

## 3. Network Response 데이터 크롤링 ( 조회된 데이터 값들 뽑기 )
```python
'''
사용자가 네트워크로 request 하는 작업이 있다면, response 된 결과물이 있을 것이다.
응답받은 데이터들을 하나하나 찾아보기 위해서는 log를 분석하면 된다.
'''
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
import json

'''크롬 로그 requests 만들기'''
caps = DesiredCapabilities.CHROME
caps['goog:loggingPrefs'] = {'performance': 'ALL'}
'''윗 두줄은 실행파일 최상단에 위치하게 함'''


'''log들로부터 requests 추출하기'''
logs_raw = driver.get_log("performance")


'''현재까지의 모든 로그'''
logs = [json.loads(lr["message"])["message"] for lr in logs_raw]


'''json 파일인지 아닌지에 대한 함수'''
def log_filter(log_):
    return (
        # is an actual response
        log_["method"] == "Network.responseReceived"
        # and json
        and "json" in log_["params"]["response"]["mimeType"]
    )


'''응답받은 json 파일만 받도록 필터링'''
for log in filter(log_filter, logs):
    request_id = log["params"]["requestId"]
    
    # resp_url = log["params"]["response"]["url"]
    # print(f"Caught {resp_url}")
    
    datas = driver.execute_cdp_cmd("Network.getResponseBody", {"requestId": request_id})['body']
    # datas = json.loads(datas) # tip : 데이터 가공을 위해 json파일을 파이썬의 dict형식으로 다룰 수 있음
    # print(datas)
'''json파일만 보는 것이 아니라 모든 내용을 보고싶다면, log_filter함수를 빼고 logs만으로 반복문을 돌리면 됩니다.'''
        
```

<br>

<br>

## 4. DB서버 저장
```python
import pymssql

'''MSSQL 접속'''
cnxn = pymssql.connect(server = 'ip주소 혹은 도메인 주소',
                       user = '사용자 이름',
                       password = '비밀번호',
                       database = 'DB이름')
                       
                       
'''
커서 생성 ( SQL문의 수행 결과로 반환될 수 있는 튜플들을 액세스 할 수 있도록 함 )
-> 결과로 반환되는 첫번째 튜플에 대한 포인터
'''
cursor = cnxn.cursor()


'''
DB에 데이터 저장 (SQL문 실행)
예제 - 테이블 'T'에 컬럼 'B'가 '5678' 이면 컬럼 'A'를 1234로 update한다. 
'''
cursor.execute(
    'UPDATE T'
    + ' SET A = CAST(' + 1234 + ' AS VARCHAR)'
    + ' WHERE B = CAST(' + 5678 + ' AS VARCHAR);'
)
cnxn.commit()


'''연결을 끊을 경우'''
cnxn.close()

```

<br>

<br>

## 5. exe파일 만들기
### spec 확장자로 파일 만들기 ( 예시 - info.spec )
```
# -*- mode: python -*-

# if you use pyqt5, this patch must be adjusted
# https://github.com/bjones1/pyinstaller/tree/pyqt5_fix

block_cipher = None

# pathex tip : utf 에러가 났을 경우, \ 한개를 더 붙이세요. ( ex: 경로\경로2  -> 경로\\경로2 )
# hiddenimports tip : 라이브러리를 찾을 수 없다고 나오는 경우, 라이브러리 이름을 쓰세요. ( ex: hiddenimports=['selenium-wire'] )
# binaries tip : chromedriver를 사용하는 경우 꼭 입력해야 함 ( ex : binaries=[( './driver/chromedriver.exe', './driver' )] )
a = Analysis(['exe로 만들 실행파일 이름.py'],
             pathex=['사용자의 파이썬 라이브러리 경로'],
             binaries=[],
             datas=[],
             hiddenimports=[],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)

pyz = PYZ(a.pure, a.zipped_data,
             cipher=block_cipher)
exe = EXE(pyz,
          a.scripts,
          a.binaries,
          a.zipfiles,
          a.datas,
          [],
          name='main_4',
          debug=False,
          bootloader_ignore_signals=False,
          strip=False,
          upx=True,
          upx_exclude=[],
          runtime_tmpdir=None,
          console=True )

```
### 파이썬 Terminal 혹은 실행파일이 있는 경로에서 명령어 실행
```
pyinstaller -F info.spec
# -F ( 단일 파일로 만들라는 뜻 )
```

### exe 파일을 실행하기 전에 충족되어야 조건들
```
1. 사용자의 pc에 모든 모듈들이 깔려있어야 함
2. 파이썬 환경변수가 지정되어 있어야 함
3. 크롬버젼과 seleniumwire 버전이 같아야 함
```
<br>

<br>

## 전체코드 ( 0 ~ 4 )
```python
from seleniumwire import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
import json,pymssql

caps = DesiredCapabilities.CHROME
caps['goog:loggingPrefs'] = {'performance': 'ALL'}

def resource_path(relative_path):
    try:
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.dirname(__file__)
    return os.path.join(base_path, relative_path)

driver = webdriver.Chrome(resource_path('./driver/chromedriver.exe')) 
driver.get("불러올 페이지 주소")

driver.find_element_by_id("id_area").send_keys("아이디")
driver.find_element_by_id("pw_area").send_keys("비밀번호")

def btn_click(x_path):
    WebDriverWait(driver, timeout=60).until(
        EC.presence_of_element_located((By.XPATH,x_path))
    ).click()

btn_click("""//*[@id="log.login"]""")
btn_click("""메뉴의 버튼에 해당하는 XPath""")

driver.switch_to.frame(driver.find_element_by_id("프레임id"))
btn_click("""버튼에 해당하는 XPath""")

logs_raw = driver.get_log("performance")
logs = [json.loads(lr["message"])["message"] for lr in logs_raw]

def log_filter(log_):
    return (
        # is an actual response
        log_["method"] == "Network.responseReceived"
        # and json
        and "json" in log_["params"]["response"]["mimeType"]
    )

for log in filter(log_filter, logs):
    request_id = log["params"]["requestId"]
    
    # resp_url = log["params"]["response"]["url"]
    # print(f"Caught {resp_url}")
    
    datas = driver.execute_cdp_cmd("Network.getResponseBody", {"requestId": request_id})['body']


cnxn = pymssql.connect(server = 'ip주소 혹은 도메인 주소',
                       user = '사용자 이름',
                       password = '비밀번호',
                       database = 'DB이름')
cursor = cnxn.cursor()
cursor.execute(
    'UPDATE T'
    + ' SET A = CAST(' + 1234 + ' AS VARCHAR)'
    + ' WHERE B = CAST(' + 5678 + ' AS VARCHAR);'
)
cnxn.commit()


cnxn.close()
driver.close()

```
