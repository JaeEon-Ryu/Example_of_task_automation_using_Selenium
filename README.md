# Example_of_task_automation_using_Selenium
로그인 , 특정 메뉴 이동, Network Response 데이터 크롤링, DB서버(mssql) 저장, exe파일 만들기 구현  

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

driver.get("불러올 홈페이지 주소")
```

## 1. 로그인
```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

'''
'chrome 설정 - 도구 더보기 - 개발자 도구' or '오른쪽 마우스 - 검사' 등을 통해 아이디와 비밀번호에 대한 id값 찾기
( 2021년 6월 26일 기준 Naver의 경우 - 아이디 : id_area, 비빌번호 : pw_area )
'''
driver.find_element_by_id("id_area").send_keys("아이디")
driver.find_element_by_id("pw_area").send_keys("비밀번호")

'''
Explicit Wait - 명시적 대기 - 조건이 True가 될 때까지 지정된 시간만큼 대기
( 현재 코드에서 쓰는 조건 - presence_of_element_located : 요소가 위치하여 있으면 실행 )
'''
def page_click(x_path):
    WebDriverWait(driver, timeout=60).until(
        EC.presence_of_element_located((By.XPATH,x_path))
    ).click()
       
'''
로그인 버튼에 해당하는 요소를 찾은 후, '오른쪽 마우스 - Copy - Copy XPath' 를 통해 버튼 클릭하도록 하기
( 2021년 6월 26일 기준 Naver의 경우 - XPath : //*[@id="log.login"] ) 
'''
page_click("""//*[@id="log.login"]""")
```

## 2. 특정 메뉴 이동
```python

```

## 3. Network Response 데이터 크롤링
```python

```

## 4. DB서버 저장
```python

```

## 5. exe파일 만들기


## 전체코드
```python

```
