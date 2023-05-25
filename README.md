# ipl-2023-best-players and their stats
A rating system for IPL players who played in 2023, along with data scrapped from howstat

Cricket is one of my favorite sports(although I can’t play it to save my life). And since IPL 2023 has come to an end, I am sure millions of cricket fans would be interested in knowing which player was best this season.

For this article, I will only be carrying out only two tasks —

Finding all the players that have played 2023 IPL matches.

Finding all the player's information i.e. their batting and bowling stats.

_Data Gathering Phase is a task that can take up to 70 to 80% of your total time dedicated to any project. For gathering data, I am going to use Web Scraping as all major cricket data is present on the web and we can easily access it through web scraping. HowStat is an excellent structured cricket statistics site that I will be using in this article. Another great option is espncricinfo.com._

Let’s start with the first task. For web scraping, we will need the following basic libraries which we will first import:

```
from bs4 import BeautifulSoup
import pandas as pd
import requests as rq
import numpy as np
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.chrome.options import Options
```
Next, we will write code for web scraping using selenium and Beautiful Soup:

For the URL, I go to HowStat Website and decide to first take the data of the players who have player IPL 2023

Hence, the website URL is http://www.howstat.com/cricket/Statistics/IPL/PlayerList.asp. Go to this website link and press Ctrl+Shift+J to Inspect the HTML Code. Through this, you can understand the location of the needed data in the HTML code. This is important as we will scrap through HTML code. Next, since we only need data of the players that played in IPL 2023 we need to select the Season from the dropdown list.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ivvscsv0nige0jh8qsxi.png)

We will use the Select class from Selenium to interact with a drop-down list on the web page. It selects the 17th option (index 16) in the drop-down list. Here's the code for it -

```
driver = webdriver.Chrome()
url = "http://www.howstat.com/cricket/Statistics/IPL/PlayerList.asp"
driver.get(url)

select = Select(driver.find_element(By.NAME, "cboSeason"))
select.options[16].click()
```
The above action selects a specific season in the drop-down list, triggering a page reload with data related to the selected season.

Now we need to extract each player data individually, to do so we can get all the player individual stats page link through the <a> tag attached to the name.

For this, we need to see the table in HTML code and find the content of class attribute so that our code can find it uniquely.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nutiofjg3cx6o1hfz6nl.png)

table.tablelined in the above picture shows that for the table tag we have class attribute value as TableLined. Now, we need to access the individual cell containing the name and link to the player profile page, of this Table present in table variable.

The code would be -

```
table = soup.find("table",{"class":"TableLined"})
a = table.find_all("a", {"class":"LinkTable"})
```

After getting all the players <a> tag we need to extract the link to their profile page.  We will iterate over each anchor tag and extracts the href attribute (link) and text (name) within the tag. The extracted names are added to the names list, and the extracted links are added to the player_links list.

```
player_links = []
names = []

for i in a:
    link = i.get("href")
    name = i.text
    names.append(name)
    player_links.append(link)
```

After storing all the links in player_links list we will now visit each link and extract player stats. First we will initialize an empty list called datas to store the extracted data.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wcrl5yaxt4nwrbrtt5tz.png)

The batting and bowling stats are stored in an HTML table with class name 'desktop'. We will use this attribute to scrap our desired data.

We will create a for loop that iterates over a range of numbers starting from 0 up to the length of the player_links list. The loop variable x is used to access elements from player_links. Then we search for a table element with the class "desktop" using soup.find("table",{"class":"desktop"}) and assigns it to the variable table. Then we proceed to find all the table rows (<tr>) within the table using table.find_all("tr") and stores them in the variable trs.

The code looks like this -

```
datas = []
for x in range(641,len(player_links)):
        
    url = "http://www.howstat.com/cricket/Statistics/IPL/" + str(player_links[x])
    
    driver = webdriver.Chrome()
    driver.get(url)

    content = driver.page_source.encode('utf-8').strip()
    soup = BeautifulSoup(content,"html.parser")
    
    table = soup.find("table",{"class":"desktop"})
    trs = table.find_all("tr")
```
A defaultdict named player_stat is created using the defaultdict class from the collections module. The def_value() function is passed as an argument, so if a key is accessed that doesn't exist in the dictionary, it will return "Not Present" instead of raising a KeyError. The initial value for the "name" key is set to names[x].

Now iterate over each table row in trs. Inside the loop, the code finds all the table data cells (<td>) within the row using i.find_all("td") and assigns them to tds.

Since we do not require some of the empty rows and rows containing names such as batting, bowling, etc. So an if statement checks if the number of cells in tds is less than 2. If it is, the loop jumps to the next iteration, skipping the current row.

```
    from collections import defaultdict
    
    def def_value():
        return "Not Present"
    
    player_stat = defaultdict(def_value)
    player_stat["name"] = names[x]
    
    for i in trs:
        tds = i.find_all("td")
        if len(tds) < 2:
            continue
        player_stat[tds[0].text.strip().lower().rstrip(":")] = tds[1].text.strip()
        
    datas.append(player_stat)
    driver.quit()
```
The code takes the first cell (tds[0]), extracts its text content using .text.strip(), converts it to lowercase, and removes any trailing ":" characters using .rstrip(":"). This processed text is used as a key in player_stat, and the value of the second cell (tds[1]) is extracted using .text.strip(). The key-value pair is added to player_stat.

After iterating over all the rows, player_stat contains the extracted data for the current player. It is appended to the datas list. The driver.quit() method is called to close the WebDriver and free system resources.

And so the loop continues for each player.

Now the data is saved as csv file.

The complete code is available on my [github ](https://github.com/puspenderkr/ipl-2023-best-players)and the csv is file available on my [kaggle](https://www.kaggle.com/datasets/puspenderkry/ipl-player-stats-2008-2023).

In Part 2, I will solve the second problem i.e Finding the best players of the season.

