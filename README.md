# thai_text_visualize
## Objective
สร้างเครื่องมือที่ช่วย visualize และหา insight จากความคิดเห็นภาษาไทยได้ง่ายขึ้น โดยใช้ wordcloud ช่วยแสดงคำที่เกิดขึ้นบ่อยในการแสดงความคิดเห็น แล้วกรองแชทจากคำที่สนใจใน wordcloud
## Data
โดยข้อมูลที่ใช้เป็นข้อมูลความคิดเห็นเกี่ยวกับการท่องเที่ยวที่ทำการ web scraping จาก https://th.tripadvisor.com/ โดยเป็นความคิดเห็น เกี่ยวกับ Jomtien Beach Night Market
#### 1. scrape_and_prepare 
จะทำการดึงข้อมูลการแสดงความคิดเห็น จาก https://th.tripadvisor.com/ มาสร้างเป็น dataframe แล้วทำการ preprocess_text (Normalize, Word Tokenize, Remove stopwords,  Remove any non-Thai characters) ให้พร้อมสำหรับการนำไปทำ wordcloud และการ กรองแชท
```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
import time

def driversetup():
    options = webdriver.ChromeOptions()
    #run Selenium in headless mode
    options.add_argument('--headless')
    options.add_argument('--no-sandbox')
    #overcome limited resource problems
    options.add_argument('--disable-dev-shm-usage')
    options.add_argument("lang=en")
    #open Browser in maximized mode
    options.add_argument("start-maximized")
    #disable infobars
    options.add_argument("disable-infobars")
    #disable extension
    options.add_argument("--disable-extensions")
    options.add_argument("--incognito")
    options.add_argument("--disable-blink-features=AutomationControlled")
    
    driver = webdriver.Chrome(options=options)

    driver.execute_script("Object.defineProperty(navigator, 'webdriver', {get: () => undefined});")

    return driver
# driver set uo
driver = driversetup()
# ดึง comment มาเก็บเป็น list ใน review
reviews = []
for i in range(0,80,10):
    driver.get("https://th.tripadvisor.com/Attraction_Review-g293919-d9650315-Reviews-or"+str(i)+"-Jomtien_Beach_Night_Market-Pattaya_Chonburi_Province.html")
    print(i, end=" ")
    time.sleep(5)
    lis_reviews = driver.find_elements(By.CLASS_NAME, "yCeTE")[0:]
    print(len(lis_reviews))
    for review in lis_reviews:
        reviews.append(review.text)
```
