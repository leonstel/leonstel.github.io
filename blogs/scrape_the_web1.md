
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

| Entity | Relations |
| ------------- | ------------- |
| Tournament | Has many Games  |
| Game  | Belongs to only one Tournament, has one Player as winner  |
| Player | Could be a winner to many Games  |
| Score | A Game consists of many scores, a Score can have one player  |


## Examine the Site
// TODO screenshot that matches every entity and describe your thought at each screen


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

## Afterthought
Maybe the site has been updated and changed its html structure or something. Then your scraping
code breaks. So you have to keep an eye on your scraper and built good error handling to see when it is
beginning to fail.




