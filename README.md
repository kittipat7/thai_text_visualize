# thai_text_visualize
## Objective
สร้างเครื่องมือที่ช่วย visualize และหา insight จากความคิดเห็นภาษาไทยได้ง่ายขึ้น โดยใช้ wordcloud ช่วยแสดงคำที่เกิดขึ้นบ่อยในการแสดงความคิดเห็น แล้วกรองแชทจากคำที่สนใจใน wordcloud
## Data
โดยข้อมูลที่ใช้เป็นข้อมูลความคิดเห็นเกี่ยวกับการท่องเที่ยวที่ทำการ web scraping จาก https://th.tripadvisor.com/ โดยเป็นความคิดเห็น เกี่ยวกับ Jomtien Beach Night Market
#### 1. scrape_and_prepare จะทำการดึงข้อมูลการแสดงความคิดเห็น จาก https://th.tripadvisor.com/ มาสร้างเป็น dataframe แล้วทำการ preprocess_text (Normalize, Word Tokenize, Remove stopwords,  Remove any non-Thai characters) ให้พร้อมสำหรับการนำไปทำ wordcloud และการ กรองแชท
