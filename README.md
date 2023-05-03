# thai_text_visualize
## Objective
สร้างเครื่องมือที่ช่วย visualize และหา insight จากความคิดเห็นภาษาไทยได้ง่ายขึ้น โดยใช้ wordcloud ช่วยแสดงคำที่เกิดขึ้นบ่อยในการแสดงความคิดเห็น แล้วกรองแชทจากคำที่สนใจใน wordcloud
## Data
โดยข้อมูลที่ใช้เป็นข้อมูลความคิดเห็นเกี่ยวกับการท่องเที่ยวที่ทำการ web scraping จาก https://th.tripadvisor.com/ โดยเป็นความคิดเห็น เกี่ยวกับ Jomtien Beach Night Market
### 1. scrape_and_prepare 
จะทำการดึงข้อมูลการแสดงความคิดเห็น จาก https://th.tripadvisor.com/ มาสร้างเป็น dataframe แล้วทำการ preprocess_text (Normalize, Word Tokenize, Remove stopwords,  Remove any non-Thai characters) ให้พร้อมสำหรับการนำไปทำ wordcloud และการ กรองแชท

#### ทำการ web scraping
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

#### สร้าง datframe
```python
import pandas as pd
df = pd.DataFrame(reviews, columns=["review"])
```

#### preprocess_text (Normalize, Word Tokenize, Remove stopwords,  Remove any non-Thai characters)
```python
from pythainlp import corpus
from pythainlp.util import normalize
from collections import Counter
import deepcut

def preprocess_text(text):
    # Normalize the text
    normalized_text = normalize(text)
    # Tokenize the normalized text into words using deepcut
    words = deepcut.tokenize(normalized_text)
    # Remove any stopwords
    words = [word for word in words if word not in corpus.thai_stopwords()]
    # Remove any non-Thai characters
    thai_words = [word for word in words if all(c >= 'ก' and c <= '๛' for c in word)]
    return thai_words
    
    df['processed_text'] = df['review'].apply(preprocess_text)
```

#### export to csv Jomtien_Beach_Night_Market_Processed.csv
```python
df.to_csv("Jomtien_Beach_Night_Market_Processed.csv")
```
### 2. wordcloud
โดย wordcloud จะทำการอ่านไฟล์ Jomtien_Beach_Night_Market_Processed.csv แล้วมาสร้าง wordcloud แสดงความถี่ของคำที่ยังไม่ผ่านการกรองใดๆ
```python
from wordcloud import WordCloud
from collections import Counter
import pandas as pd

df=pd.read_csv("Jomtien_Beach_Night_Market_Processed.csv")
df['processed_text'] = df['processed_text'].map(lambda x: list(eval(x)))
df=df.drop(['Unnamed: 0'],axis=1)
    

# Flatten the list of words into a single list
words = [word for sublist in df['processed_text'] for word in sublist]

# Count the frequency of each word in the list of words
word_freq = Counter(words)
# Select the top 100 most frequent words
top_N_words = [word for word, _ in word_freq.most_common(100)]

wordcloud = WordCloud(font_path='D:\oho\THSarabunNew.ttf',background_color="white",prefer_horizontal=True, max_words=100, contour_width=3, contour_color='steelblue', width=2400, height=1000)
wordcloud.generate_from_frequencies({word: word_freq[word] for word in top_N_words})
wordcloud.to_image()
```
![image](https://user-images.githubusercontent.com/97491541/235827404-9949ffab-45b6-4fa6-8542-2133d892d75b.png)
