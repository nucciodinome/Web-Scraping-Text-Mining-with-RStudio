# Web Scraping and Data Mining: A gentle introduction

**Welcome to this short tutorial on Web Scraping and Data Mining techniques!**

You can use the [Google Cloud Python Platform](https://colab.research.google.com/) to start you scraping project

Your[Python script](https://colab.research.google.com/drive/14c_iDdptsMWkZIQl7-89DtHeCuK1Et5b?usp=sharing)

### Web Scraping
Open a new notebook in Colab!

## Data Source
Open a new webpage and go to [WSJ](https://www.wsj.com/search?query=energy&mod=searchresults_viewallresults)  to inspect data to scrape

## Import libraries

```markdown

import bs4
from bs4 import BeautifulSoup
import requests
import re
import csv
import pandas as pd
import numpy as np
import time
from bs4 import BeautifulSoup, SoupStrainer
import requests
from bs4 import BeautifulSoup as bs

```


## Web Driver Settings

```markdown

import sys
sys.path.insert(0,'/usr/lib/chromium-browser/chromedriver')
from selenium import webdriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import NoSuchElementException
from webdriver_manager.chrome import ChromeDriverManager
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')

prefs = {
    "download.open_pdf_in_system_reader": False,
    "download.prompt_for_download": True,
    "plugins.always_open_pdf_externally": False
}
chrome_options.add_experimental_option(
    "prefs", prefs
)

driver = webdriver.Chrome('chromedriver', options=chrome_options)
driver.implicitly_wait(5)
```




## Doing a Soup

```markdown

element_list = []
for page in range(1,50):
    
    page_url = "https://www.wsj.com/search?query=energy&mod=searchresults_viewallresults&" + str(page)
    driver.get(page_url)
    page_source = driver.page_source
    soup = bs(page_source, 'lxml')
    titles = soup.find_all('div',attrs = {'???'})

    for title in titles:
        element_list.append([title.get_text()])



df = pd.DataFrame(element_list) 
```

### Support or Contact

Having trouble with example code? Some new questions or ideas you want to explore?
Contact me at _nuccio.ludovico@gmail.com_
