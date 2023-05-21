# Web Scraping with R Studio: A gentle introduction

**Welcome to this short tutorial on Web Scraping!**

## A Case Study: The Wind Energy Across Europe 
We want to explore the relational network that the European organizational actors involveed the wind energy sector eventually build within the digital environment.
In order to do so, we have identified a promising source of information, the Wind Energy Europe, wich collect more than 600 organization involved in wind energy project.

Our formal research question is:
***RQ: Which organizations build the densest relational network in the digital environment?***

Let's try to address it through R-based tools.
You can use the both your local R Studio or an [R Studio Cloud Project](https://posit.cloud/) to start your scraping task!

## The sampling procedure
Let’s start by finding and downloading the Excel file of your group [here](https://github.com/nucciodinome/Web-Scraping-Text-Mining-with-RStudio/)

Now, you can open R Studio and import the file.
As you can see, the dataset is a list of URLs, named SEED, which refer to the websites of 50 members of Wind Energy Europe.

Given RQ, you should first identify what data you need to track relationships between organizations.

As we know, information about the relationships between two websites is encoded as hyperlinks, which, in the HTML language are reported with the "a" tag, while the target URL of the link is made explicit with the "href" attribute.

We can assume, moreover, that not all website links are on the homepage: many of them will be found on the internal pages of the website (e.g., "about" page, "partners" page, "clients" page, etc.).

Let's assume, then, that we also want to investigate the internal pages which are directly linked to the homepage, or, in other words, at 1 degree of separation from the homepage (let's call them second-level web pages).

We can then start to build our procedure aimed at extracting the data we need and then identify an R script for web scraping.

Probably, your procedure will look like this:


- [x] Starting from the list of 50 SEEDs...
- [ ] __STEP I__: Extract all the _Second Level web pages_ in order to...
- [ ] __STEP II__: Collect all _external links_ (which refer to other organizations' websites connected to the SEEDs


### Import R packages

```markdown

install.packages(rvest)
install.packages( (stringr)
install.packages( (dplyr)

#then activate them:

library(rvest)
library(stringr)
library(dplyr)

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
