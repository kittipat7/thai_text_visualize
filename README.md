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
#### wordcloud แสดงคำที่เกิดขึ้นทั้งหมด
โดย จะทำการอ่านไฟล์ Jomtien_Beach_Night_Market_Processed.csv แล้วมาสร้าง wordcloud แสดงความถี่ของคำที่ยังไม่ผ่านการกรองใดๆ
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
#### wordcloud แสดงคำที่กรองคำที่ไม่สนใจออก
โดยจะทำการกรองคำที่ไม่สนใจ หรือคำที่ไม่มีความหมายจาก wordcloud แรกออกในตัวอย่างนี้กรองคำว่า คน, ค่า ออกไป
```python
#คน,ค่า
keys_to_remove =  input("ใส่คำที่ต้องการกรองออกคั่นด้วย , หรือ space bar: ").replace(',', ' ').split()
words= [item for item in words if item not in keys_to_remove]
word_freq = Counter(words)
# Select the top 100 most frequent words
top_N_words_filter = [word for word, _ in word_freq.most_common(100)]

wordcloud2 = WordCloud(font_path='D:\oho\THSarabunNew.ttf',background_color="white",prefer_horizontal=True, max_words=100, contour_width=3, contour_color='steelblue', width=2400, height=1000)
wordcloud2.generate_from_frequencies({word: word_freq[word] for word in top_N_words_filter})
wordcloud2.to_image()
```
![image](https://user-images.githubusercontent.com/97491541/235827924-224470e3-289d-41e7-9131-1ae28ed30612.png)

#### wordcloud แสดงความถี่ของคำที่สนใจ
โดยจะให้ user เลือกคำที่สนใจใน wordcloud ก่อนหน้ามาโดย wordcloud นี้จะแสดงความถี่ของคำที่สนใจว่าเกิดขึ้นมากน้อยแค่ไหน ในตัวอย่างนี้เลือก ดื่ม,เต้นรำ,เที่ยว
```python

#ดื่ม,เต้นรำ,เที่ยว
keywords =  input("ใส่คำที่สนใจออกคั่นด้วย , หรือ space bar: ").replace(',', ' ').split()
df_paid_interest = df[df['review'].apply(lambda x: any(keyword in x for keyword in keywords))]
#df_paid.head()
# Remove any stopwords and non-Thai characters from the list of words and select only the words that are in the list of keywords
words_interest = [word for sublist in df['processed_text'] for word in sublist]
words_interest = [word for word in words if word in keywords]

# Count the frequency of each word in the list of words
word_freq_interest = Counter(words_interest)

wordcloud3 = WordCloud(font_path='D:\oho\THSarabunNew.ttf',background_color="white",prefer_horizontal=True, max_words=100, contour_width=3, contour_color='steelblue', width=2400, height=1000)
wordcloud3.generate_from_frequencies({word: word_freq[word] for word in word_freq_interest})
wordcloud3.to_image()
```
![image](https://user-images.githubusercontent.com/97491541/235828311-eb8d849f-9d4d-4168-a4a9-39aadb926da4.png)
#### wordcloud ที่เกิดจาการเลือกคำจาก wordcloud คำที่สนใจ
โดยในขั้นนี้จะเริ่มเข้าสู่การกรองแชทโดยจะเลือกคำจาก wordcloud คำที่สนใจ จากนั้นจะทำการกรองแชทที่มีคำที่เลือกไว้มาสร้าง wordcloud แสดงคำรอบๆที่เกิดกับคำที่เลือก ในตัวอย่างนี้เลือก เที่ยว

```python
#ex เที่ยว
interest=input("เลือกคำที่สนใจจากกลุ่มคำที่สนใจมา 1 คำ : ")
df_interest = df_paid_interest[df_paid_interest['review'].apply(lambda x: interest in x)]

#all wordcloud 
words_focus = [word for sublist in df_interest['processed_text'] for word in sublist]
words_focus= [item for item in words if item not in keys_to_remove]
words_focus= [item for item in words if item not in interest]
# Count the frequency of each word in the list of words
word_freq_focus = Counter(words_focus)

wordcloud4 = WordCloud(font_path='D:\oho\THSarabunNew.ttf',background_color="white",prefer_horizontal=True, max_words=100, contour_width=3, contour_color='steelblue', width=2400, height=1000)
wordcloud4.generate_from_frequencies({word: word_freq[word] for word in word_freq_focus})
wordcloud4.to_image()
```
![image](https://user-images.githubusercontent.com/97491541/235828946-147d80e0-21ac-4e50-821e-e65d10ad6caa.png)
#### เลือกคำที่สนใจจาก wordcloud ที่เกิดจาการเลือกคำจาก wordcloud คำที่สนใจ 
ในขั้นตอนนี้จะเป็นขันตอนการกรองแชท ในตัวอย่างนี้เลือกคำว่า สถานที่ ซึ่งผลลัพธ์สุดท้ายจะออกมาในรูปของไฟล์ excel ของความคิดเห็นที่มีคำว่า เที่ยวและสถานที่ และชื่อไฟล์จะเป็นคำที่เลือกทั้งสองคำต่อกัน เช่น เที่ยว_สถานที่.xlsx
```python
#ex สถานที่
interest2 = input("ใส่คำที่สนใจ 1 คำ : ")
selected_rows = df_interest[df_interest['review'].apply(lambda x: interest2 in x)]
selected_rows = selected_rows.drop(['processed_text'],axis=1)
selected_rows

file_name = interest+'_'+interest2+'.xlsx'
selected_rows.to_excel(file_name)
```
#### ตัวอย่างผลลัพธ์
review
นี่เป็นหนึ่งในตลาดกลางคืนที่ได้รับความนิยมมากในพัทยา มีแผงลอยหรือบูธขายอาหารทั้งของแปลก ๆ และของที่ระลึก พวกเขายังมีการแสดงบนถนนหรือการแสดงหลังจากที่คุณสามารถเต้นไปกับเพลงที่พวกเขาเล่น อาหารดีจริง ๆ ที่นี่ (โดยเฉพาะอาหารทะเลย่างและสั่น) และราคาถูกมาก มันเป็นสถานที่ที่ดีที่คุณสามารถไปเที่ยวกับเพื่อน ๆ ในเวลากลางคืนและพูดคุยกับเบียร์ช้างท้องถิ่นที่เย็นชาและอาหารย่างสดใหม่ สถานที่ตั้งอยู่ตรงข้ามชายหาดซึ่งบางครั้งก็ทำให้คุณรู้สึกเย็นสบาย ยกนิ้ว!
สามารถเดินจากอพาร์ทเมนท์ของเรา เดินเที่ยวตลาดแผงลอยขายสินค้าและอาหาร ข้างๆตลาดร้านอาหารที่เปิดโล่งสบาย ๆ ภายใต้แสงดาวมีเครื่องดื่มที่คุณเลือกและเมนูอาหารที่ยอดเยี่ยม (ไม่แพง) รักสถานที่
สถานที่ท่องเที่ยวที่น่าสนใจ
"ตลาดแห่งนี้เป็นสถานที่ที่ต้องไปเยือนในขณะที่จอมเทียนบีช มันเปิดตอนช่วงบ่าย 4 แห่ง และมีอาหารที่น่าตื่นตาตื่นใจที่เป็นของเรย์ พ่อค้าแม่ค้าที่ขายเสื้อผ้า ฯลฯ ฯเพื่อให้คุณมีความสุข และดูมีราคาที่ถูกมากบนเสื้อผ้า ไม่ต้องเหนื่อยกับการตลาดนี้เป็นสถานที่ที่ไม่เหมือนคนอื่นที่เคยไปเที่ยวในประเทศไทย มันมีครอบครัวที่ดีมีบรรยากาศสำหรับเต้นรำ และดนตรีของเล็ก ๆ และเครื่องดื่มสำหรับมัมมี่ของแดดดี้ส์ และโหลดของโต๊ะสำหรับทานอาหาร
เป็นคืนที่ออก หรือช่วงบ่ายที่เงียบสงบ"
"หากคุณกำลังมองหาประสบการณ์แบบไทยแท้ แล้วที่นี่ไม่ใช่สถานที่ มีตัวเลือกทั้งอาหารไทย แต่ที่ฉันประทับใจมากที่สุดก็คือการตอบสนองของอาหารตะวันตก pallets ต้องบอกว่า ตลาดตั้งอยู่ในทำเลที่เหมาะสม และเป็นส่วนหนึ่งของพัทยาว่าจะดึงดูดคู่รัก และครอบครัว ซึ่งเป็นจุดที่ดี
ยัง มีที่นั่งเล่น ดังนั้นมันจึงเป็นไปได้ไปซื้อที่ร้านขายอาหารต่าง ๆ เครื่องดื่มอีก และอาหารแล้วมีด้วยการเลือกอาหารที่คุณเลือก
ไม่เลวเลย แต่ส่วนใหญ่นักท่องเที่ยวที่ดั้นด้นไปโดยกล่าวว่า"
หาดจอมเทียนเป็นตลาดกลางคืนมากที่เกิดขึ้น มีผู้คนมากมายนักท่องเที่ยวส่วนใหญ่กินดื่ม และ มีร้านอาหารหลายร้านขายอาหารผลไม้ น้ำผลไม้ และอาหารทะเลที่สดใหม่ ไม่จำกัดตัวเลือกสำหรับการช้อปปิ้งเสื้อผ้า เครื่องสำอางค์ รองเท้า กระเป๋า ฯลฯ ฯ และบางแห่งก็ไม่มองเช่น เคาะ offs; ขอแนะนำให้คุณตรวจสอบก่อนซื้อครอสใด ๆ มีดนตรีดีเจล่าสุดให้คนหัวเราะ เพราะเด็ก chartbusters และเต้นรำ ทั้งสถานที่บรรยากาศมันเป็นจังหวะไป
