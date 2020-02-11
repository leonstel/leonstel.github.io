
# Scraping the Web - Taking a Bet

<img src="../assets/scrape_the_web/horse_racing.jpg" />

For a while now the possibilities of machine learning have fascinated me. Though I was doing some for fun on the side I 
have decided too take it more seriously. By chance, with that thought in my mind, I have came across the company [Springco](https://www.spring-co.nl/) which
uses machine learning to get answers for urban related questions. 

After an inspiring conversation with Hank Groenhof and René Neijmeijer covering topics such as the steps taken
to get to an answer, statistics, neural nets and lastly how and where to get your data. It got me thinking.  

*Questions I am going to answer*

**What specific data do you want?**  
**How do you harvest data from a website if no off the shelf API exists?**    
**How do you want to store it and in which format?**  
**How could a data scientist use the harvested data?**

Because I used to play badminton in the previous days and have scoured the [https://www.toernooi.nl](https://www.toernooi.nl/tournaments) a lot for keeping track of 
matches I thought it would be fun to use that as a base for this case. It would have saved me a lot of time then if I had a
web scraper doing all the searching for me 😃.

## Use Case - Sports Betting

Recently sport betting is becoming legal in the united stated. Imagine that you are the owner of a sports club and 
wants to make some extra money. Your objective is: 

- Getting insights if betting on the tournaments held at my club is profitable

The first step to achieve this goal is to gather data related to this objective. So you figured that you should have a 
collection of **tournaments, players, games, scores and winners**. After hours maybe days of searching the web for an easy to 
consume API which has your exact data requirements mentioned previously you have come to the conclusion that it is a lost cause. 

Just before you are going to call it a day you come across the website [https://www.toernooi.nl](https://www.toernooi.nl/tournaments).
Which contains the exact information you need to get any answers, but it has a major drawback. It does not have a public 
API. So you have to come up with a solution to extract the data from that website involving **web scraping**.

## Solution in a Nutshell

This is the final web scraping solution which we are building up to.  

The scraper in action (sounds like a new Hollywood action blockbuster or a sequel of The Terminator).

<p align="center">
    <iframe src="https://www.youtube.com/embed/zhJYiblYazY" width="700" height="450" frameborder="0" allowfullscreen> </iframe>
</p>

- Scraping and saving to db the tournaments, its players and games. **(begin - 2:06)**
- Show resulted db tables in DataGrip db viewer **(2:06 - end)**

**Entities**


## Examine the Site


| Entity | Relations |
| ------------- | ------------- |
| Tournament | Has many Games  |
| Game  | Belongs to only one Tournament, has one Player as winner  |
| Player | Could be a winner to many Games  |
| Score | A Game consists of many scores, a Score can have one player  |

Cookies
<p align="center">
    <img src="../assets/scrape_the_web/cookies_popup.png" />
</p>

```
from selenium import webdriver

browser = webdriver.Chrome()

cookieButton = browser.find_element_by_xpath("//*[text()='Akkoord']")
cookieButton.click()
```

Tournaments search results
<p align="center">
    <img src="../assets/scrape_the_web/page1.png" />
</p>

```
// main.py
from bs4 import BeautifulSoup
import time
import globals
import extraction
...

url = 'https://www.toernooi.nl/sportselection/setsportselection/2?returnUrl=/find?StartDate={}&EndDate={}&CountryCode=NED'
startDate = '2019-12-10'
endDate = '2019-12-31'

# The code to visit a url and then getting page source will be abstracted with a function later on
url = url.format(*params)
browser.get(url)

allowCookies()

# 2 second sleep. Otherwise the expected page could be not fully loaded yet
time.sleep(2)

# globals is the file where the global variables are stored
soup = BeautifulSoup(globals.browser.page_source, 'html.parser')
tournament_urls = extraction.extracyTournamentUrls(soup)
```

Explain beautiful soup, first time here. Difference with selenium

```
// extraction.py

def extracyTournamentUrls(soup):
    print('get tournament urls from page')

    searchResultUl = soup.find("ul", {"id": "searchResultArea"})
    tournament_list = searchResultUl.findAll("li", {"class": "list__item"})

    urls = []

    for t in tournament_list:
        tournament_a = t.find("a")
        if tournament_a:
            tournament_link = tournament_a.attrs.get('href')
            urls.append(tournament_link)

    return urls
```

Tournament detail
<p align="center">
    <img src="../assets/scrape_the_web/page2.png" />
</p>

```
// extraction.py

import db       #file contains helper methods to interact with the database
...

# enum for the tournament's information on its detail page
class TournamentAttr(Enum):
    LOCATION = "locatie"
    ADDRESS = "adres"
    PHONE = "telefoon"
    EMAIL = "e-mail"
    WEBSITE = "website"
    FAX = "fax"

def extractTournamentInfo(soup, tournament_id):
    print('extract tournament info from page')

    table = soup.find("table", {"id": "cphPage_cphPage_cphPage_tblContactInformation"})
    tbody = soup.find("tbody")
    rows = tbody.findChildren("tr", recursive=False)

    entry = {}

    for row in rows:

        entry['id'] = tournament_id

        attribute = row.find('th').getText().replace(":", "").lower()
        data = row.find('td').getText()

        if TournamentAttr(attribute) == TournamentAttr.LOCATION:
            entry['location'] = data

        elif TournamentAttr(attribute) == TournamentAttr.ADDRESS:
            address_td = rows[1].find('td').find('td').contents
            data = [content for content in address_td if getattr(content, 'name', None) != 'br']
            entry['address'] = data

        elif TournamentAttr(attribute) == TournamentAttr.PHONE:
            entry['phone'] = data

        elif TournamentAttr(attribute) == TournamentAttr.FAX:
            entry['fax'] = data

        elif TournamentAttr(attribute) == TournamentAttr.EMAIL:
            entry['email'] = data

        elif TournamentAttr(attribute) == TournamentAttr.WEBSITE:
            entry['website'] = data

        else:
            print('Attribute not found!!!', attribute)

    
    # save tournament info to db (helper method from db.py)
    db.insertTournament(entry)
```

Tournament players
<p align="center">
    <img src="../assets/scrape_the_web/page3.png" />
</p>

Tournament player detail
<p align="center">
    <img src="../assets/scrape_the_web/page4.png" />
</p>







**Database Design**

The scraped data from the website is being saved to this structure, whereas every entity is getting
linked appropriately.

<p align="center">
    <img src="../assets/scrape_the_web/db_design.png" />
</p>

## Learning Points
- Python and effective database connections
- Interactive Web Scraping
- Database design / querying



## Web scraping
Interactive parts
Scraping parts

## regex names
// python does not support regex conditional statements  
//it works on regex101 though https://regex101.com/r/1lwQEF/4  
//result = re.search(r"((?(?=,))(\w+), (\w+)(.*)|.*)", player)  
//name Kempen, Jonathan  
//and indonisian name laek surav aar oke  
// could be found on index 4

result = re.search(r"(\w+), (\w+)(.*)", player)

## File structure
globals, main, extraction (methods) file, db

## Afterthought
Maybe the site has been updated and changed its html structure or something. Then your scraping
code breaks. So you have to keep an eye on your scraper and built good error handling to see when it is
beginning to fail.

Pagination not done yet.




