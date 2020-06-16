![Banner](/images/banner.jpg)
### Fake profile generator

### Objective
To create these fake bios, we'll need to rely on 3rd party website which will help us in generating fake bios for us. There are numerous websites out there which will generate fake profiles for us. This is a feasible way to write thousands of fake bios in a very reasonable amount of time.

### Required libraries
- `requests` allows us to access the webpage that we need to scrape
- `time` will be needed in order to wait between webpage refreshes
- `re` for regex
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

URL of website for scraping
```URL = "https://www.character-generator.org.uk/bio/"```

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

Scrapping code may vary w.r.t website we choose
```python
# find required element and perform action 
driver.find_element_by_id('fill_all').click()
driver.find_element_by_id('quick_submit').click()

# wait for reponse
time.sleep(2)
```
![Selenium Automation](/images/0_automate.gif)

In my case url was changing after bios generation, so we need to get the updated url for further work
```python
# get new url
url = driver.current_url

#get contents from url
page = requests.get(url)
```
Using request we get the contents of webpage in bytes format
![Page contents](/images/1_page.JPG)

```python
soup = bs(page.content)
```

USing Beautifulsoup, we parse content of webpage and get data into html format
![Parsed content](/images/2_soup.JPG)

Then using `find` method we try to find particular element from html page e.g. `div` tag, its `class` name, `<p>` i.e. paragraph tag from html where our actual text strings are present. We can easily find these tags of any website by right click on that webpage and in option select `Inspect`.

```python
# Getting the bios
bios = soup.find('div', class_='bio').find_all('p')
```
![Bios paragragh](/images/3_para.JPG)

using regular expresion we removed `<p>,</p>` tags from our data and converted it into python string
```python
biostr=''
for i in bios:
    biostr += re.sub('<p>|</p>','',str(i))
```
![Bios in string](/images/4_str.JPG)

This step could be optional step, uptil now we are able to collect whole bios containing all required information for any profile
Since we had a fix format of data, it was easy to capture from raw data
`name` First three word were name
`age` Only numeric value present in first sentence of data was user's age
In similar way, we could capture a lot of information using regex
```python
# To collect name, age from first string
name, age = ' '.join(biostr.split('.')[0].split(' ')[0:3]), int(re.findall("\d+", biostr.split('.')[0])[0])
```
![Getting name,age](/images/5_name.JPG)

After every loop will push new data into respective list
```
biolist.append(biostr)
name_list.append(name)
age_list.append(age)
```

This is must step to close the window of browser after finishing scraping task
```
driver.close()
```

Finally, zip all three lists and create a new dataframe
```python
profile = pd.DataFrame(zip(name_list, age_list, biolist), columns=['name','age','bios'])
```

In this format we will be getting our data 
| idx | name | age | bios |
|-|-|-|-|
| 0 | Richard Luke Trescothik | 21 | Richard Luke Trescothik is a 21-year-old trade... |
| 1 | Mark Rick Sweet | 29    | Mark Rick Sweet is a 29-year-oldtheatre actor... 


Now we can serialise datframe object using `pickle` for future use
```python
with open("bios_data.pkl", "wb") as fp:
    pickle.dump(profile, fp)
```

### References
- [selenium](https://selenium-python.readthedocs.io/getting-started.html)
- [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/#quick-start)


### Acknowledgement
- [Marco Santos](https://github.com/marcosan93)

