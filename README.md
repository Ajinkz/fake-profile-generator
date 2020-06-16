# Fake-profile-generator

### Objective
There is no feasible way to write thousands of fake bios in a very reasonable amount of time. To writes these fake bios, we'll need to rely on 3rd party website which will help us in generating fake bios for us. There are numerous websites out there which will generate fake profiles for us.

(List of a few sites)[https://www.google.com/search?q=fake+bio+generator&oq=fake+bio&aqs=chrome.1.69i57j69i59j0l4j69i60l2.2641j1j7&sourceid=chrome&ie=UTF-8]


### Required libraries
- `requests` allows us to access the webpage that we need to scrape
- `time` will be needed in order to wait between webpage refreshes
- `tqdm` is only needed as a loading bar for our sake (optional)
- `bs4` is needed in order to use BeautifulSoup
- `selenium` to stimulate click events
- `pandas` to create dataframe
- `pickle` to serialize objects

### Code
Importing required libraries

```python
import re
import time
import requests
import pandas as pd
import _pickle as pickle
from selenium import webdriver
from bs4 import BeautifulSoup as bs
# from tqdm import tqdm_notebook as tqdm
from tqdm.notebook import tqdm
from selenium.webdriver.common.keys import Keys
```


```python
# Initialise with empty list
biolist = []
name_list = []
age_list = []

# launch WebDriver instance
driver = webdriver.Chrome()

# launch website
driver.get(URL)
```

Scrapping code may vary w.r.t to website with choose
```
# find required element and perform action 
driver.find_element_by_id('fill_all').click()
driver.find_element_by_id('quick_submit').click()

# wait for reponse
time.sleep(2)
```
In my case url was changing after bios generation, so we need to get the update url for further work
```
# get new url
url = driver.current_url

#get conents from url
page = requests.get(url)
soup = bs(page.content)
biostr=''



try:
    # Getting the bios
    bios = soup.find('div', class_='bio').find_all('p')

    # Adding to a list of the bios
    for i in bios:
        biostr += re.sub('<p>|</p>','',str(i))
        
        # To collect name, age from first string
        name, age = ' '.join(biostr.split('.')[0].split(' ')[0:3]),int(re.findall("\d+", biostr.split('.')[0])[0])
        
        # s = biostr.split('.')[0]
        # occupation = s[s.find("-old ")+ len("-old "):s.find(" who")]

    biolist.append(biostr)
    name_list.append(name)
    age_list.append(age)
    # occupation_list.append(occupation)
    
    # close the instance
    driver.close()
except:
    pass
```
