---
layout: post
title: ByPass Button "load more" dengan selenium
date: 2022-10-13
categories: ["Python","Web Scraper"]
---

Pada tulisan kali ini saya akan membagikan pengalaman saya membuat web scraper menggunakan selenium dan python, dimana kendala yang saya hadapi dalam melakukan scraping tersebut adalah tombol "load more" yang membuat saya tidak bisa mendapatkan data dari web tersebut. tujuan saya melakukan scraping adalah untuk mendapat postingan sesuai keyword yang sudah saya tentukan.
<br><br>
untuk mengetahui apa itu selenium bisa membaca dokumentasi di web berikut [Selenium.dev](https://www.selenium.dev/)
<br><br>

1. Membuat file python, disini saya beri nama **scrapper.py**
2. Menambahkan selenium pada project kita dengan command <br><br>
   ```python
   pip install selenium
   ```
   <br>
3. Selanjutnya mengimport selenium ke dalam file python kita<br><br>
   **scrapper.py** <br><br>
   ```python
   from selenium import webdriver
   from selenium.webdriver.common.by import By
   from selenium.webdriver.support.ui import WebDriverWait
   from selenium.webdriver.support import expected_conditions as EC
   from webdriver_manager.chrome import ChromeDriverManager
   ```
   <br>
4. Menambahkan list keyword dan web yang sudah saya kumpulkan sebelumnya <br><br>
   **scrapper.py** <br><br>
   ```python
   keyword = open("wordlist/keywords.txt", "r").read().splitlines()
   source = open("wordlist/web_loadmore.txt", "r").read().splitlines()
   ```
5. Selanjutnya menambahkan browser driver, disini saya menggunakan chrome jadi implementasinya seperti berikut<br><br>
   **scrapper.py** <br><br>
    ```python
    driver = webdriver.Chrome(ChromeDriverManager().install())
    ```
    <br>
6. Sebelumnya kita sudah harus tau nama class dari button load more dari web yang akan di scraping, caranya bisa menggunakan inspect element
   
7. membuat sebuah fungsi untuk melakukan scraping dan hasil dari scraping tersebut di add ke dalam file **res.txt** <br><br>
   **scrapper.py** <br><br>
   ```python
   driver = webdriver.Chrome(ChromeDriverManager().install())
    for key in keyword:
        for src in source:
            driver.get("{}?s={}".format(src,key))
            while True:
                try:
                    load_more_button = WebDriverWait(driver, 10).until(
                        #td_ajax_load_more adalah nama class dari tombol loadmore
                        EC.visibility_of_element_located((By.CLASS_NAME, "td_ajax_load_more"))
                    )
                    #melakukan scroll down
                    driver.execute_script("arguments[0].scrollIntoView();", load_more_button)
                    #melakukan click tombol load more
                    load_more_button.click()
                    #get all title
                    soup = BeautifulSoup(driver.page_source, "html.parser")
                    titles = soup.find_all("div", class_="tdb_module_loop")
                    for title in titles:
                        print(src)
                        print(title.find("h3").text)
                        with open("res.txt", "a") as f:
                            f.write("Title : "+title.find("h3").text + "\n")
                            f.write("Link : "+title.find("a")["href"] + "\n")
                            f.write("Source : "+src + "\n")
                            f.write("Keyword : "+key + "\n")
                            f.write("============================================="+ "\n")
                except:
                    break
   ```
8. Selanjutnya menjalankan programnya dengan menuliskan di terminal
   ```python
   python scrapper.py
   ```
   <br>
   untuk melihat full source code nya silahkan kunjungi github saya, di link berikut
   
   [scrapper.py](https://github.com/AldoReghan/pyscrapper/blob/main/scrapper.py)<br>

   sekian tulisan saya tentang cara bagaimana bypass tombol load more, di sebuah web menggunakan selenium dan python, mohon maaf jika banyak kekurangan dalam tulisan saya, terima kasih banyak