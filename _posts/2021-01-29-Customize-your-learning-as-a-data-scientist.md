---
layout: post
title:  Customize your learning as a data scientist
date: "2020-10-02 20:01:00"
description: This single skill will open a whole world of new possibilities for you
tags: scraping, data
categories: day-to-day
---

[![image.png](https://i.postimg.cc/2SmGnrnD/image.png)](https://postimg.cc/RNswBrPs)

Have you ever got the idea for an awesome data science project, looked up the data you'll need online, but it's nowhere to be found? Unfortunately, not every dataset you'll ever require is available online. So, what are your options? Should you abandon your concept and return to Kaggle? No! A true data scientist should be able to gather his or her own DATA!


What’s Web Scraping and why learn it?
=====================================

The internet is the single most important source of data; it is a virtual record of human knowledge, at least for the last 20 years. Web scraping is the skill of obtaining data from the internet as a Data Scientist. It's a fantastic tool that offers up so many possibilities for unique projects.

**Please be in mind that certain websites forbid scraping and may block your IP address if you scrape too frequently or maliciously..**

**How do we scrape?**
=====================

There are two approaches when it comes to web scraping.

**Request-based scraping**: With this method, we will submit a request to the website's server, which will return the HTML of the page, which is the same content that you see when you click "View page source" in Google Chrome, which you can try right now by clicking **ctrl+u**.Then, normally, we will use a library to parse the HTML and retrieve the desired data. This approach is simple, lightweight, and very fast; however, it is not perfect, and there is one disadvantage that may deter you from using it; in fact, most modern websites nowadays use JavaScript to render their content, IE: you don't see the content of the page until the JavaScript executes, which the request method cannot handle.


**Request-based scraping**: In this strategy, we will submit a request to the website's server, which will return the HTML of the page, which is the same content that you see when you click "View page source" in Google Chrome, which you can try right now by clicking **ctrl+u**.Then we normally use a library to parse the HTML and extract the desired data. This approach is simple, lightweight, and very fast; however, it is not perfect, and there is one drawback that may deter you from using it; in fact, most modern websites nowadays use JavaScript to render their content, IE: you don't see the content of the page until after the JavaScript executes, which the request method cannot handle.


Scrape using selenium:
==============================

Selenium is a popular web automation framework, but it can also be used for scraping! You can basically imitate any operation that a human can do manually using selenium. You may construct a bot that will take particular actions when something happens, or you can make selenium scan web sites and scrape data for you, which is what we'll be doing in this tutorial.

To parse the HTML we will be using beautiful soup.

Here are documentation links for further reading. [selenium](https://selenium-python.readthedocs.io/) and [beautiful soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)

Demo: Scraping Jobs from Indeed's Job board
==========================

Let's get started; the purpose of this example is to scrape jobs from Indeed based on a search query and store them in a csv file.

More precisely we are interested in:

*   Job title
*   Location of the job
*   Company that posted the offer
*   Job description
*   When the job was posted


First let’s import the libraries that we'il work with:

```
from bs4 import BeautifulSoup  
from webdriver_manager.chrome import ChromeDriverManager  
import pandas as pd  
from selenium import webdriver  
from selenium.webdriver.chrome.options import Options
chrome_options = Options()  
chrome_options.add_argument("--headless")
```

  * Beautiful Soup is used to interface with HTML 
  * Pandas is used to output to csv 
  * The web driver is the real browser; we will use Chrome and configure it to run in **headless mode**, which means it will run in the background and we will not be able to see a browser scrolling through the job sites; if you want to view the browser, you may delete it!

The first step is to obtain the real job pages; fortunately, Lucky offers a search option; simply browse to:

“https://fr.indeed.com/jobs?q=data+scientist&start=10”

[![image.png](https://i.postimg.cc/B6xkJM05/image.png)](https://postimg.cc/62pY0VTT)

You'll be sent to the second page of data science jobs, where you may modify the search query by changing the **q** argument and the page number by changing the **start** argument. I'm using the Indeed Moroccan portal, but this will work in any nation.

Two functions will be implemented. One is a helper method for navigating to a URL, extracting the HTML and converting it to a Beautiful Soup object with which we can interact, while the other extracts links to job pages:

```
def toSoup(url):  
    driver.get(url)  
    html = driver.page_source  
    soup = BeautifulSoup(html, 'lxml')  
    return soupdef getPageUrls(query,number):  
    url="[https://fr.indeed.com/emplois?q=](https:/fr.indeed.com/emplois?q=)"+str(query)+"&start="+str(((number-1)\*10))  
    soup=toSoup(url)  
    maxPages=soup.find("div",{"id":"searchCountPages"}).text.strip().split(" ")[3]  
    return maxPages,[appendIndeedUrl(a["href"]) for a in soup.findAll("a",{"class":"jobtitle turnstileLink"})]
```

Now that we have the URLs, let's put them to use to extract what we want from the job page:


```
def paragraphArrayToSingleString(paragraphs):  
    string=""  
    for paragraph in paragraphs:  
        string=string+"\\n"+paragraph.text.strip()  
    return stringdef appendIndeedUrl(url):  
    return "[https://fr.indeed.com](https://fr.indeed.com)"+str(url)def processPage(url):  
    soup=toSoup(url)  
    title=soup.find("h1",{"class":"icl-u-xs-mb--xs icl-u-xs-mt--none jobsearch-JobInfoHeader-title"}).text.strip()  
    CompanyAndLocation=soup.find("div",{"class":"jobsearch-InlineCompanyRating icl-u-xs-mt--xs jobsearch-DesktopStickyContainer-companyrating"})  
    length=len(CompanyAndLocation)  
    if length==3:  
        company=CompanyAndLocation.findAll("div")[0].text.strip()  
        location=CompanyAndLocation.findAll("div")[2].text.strip()  
    else:  
        company="NAN"  
        location=CompanyAndLocation.findAll("div")[0].text.strip()  
    date=soup.find("div",{"class":"jobsearch-JobMetadataFooter"}).text.split("-")[1].strip()  
    description=paragraphArrayToSingleString(soup.find("div",{"id":"jobDescriptionText"}).findAll())  
    return {"title":title,"company":company,"location":location,"date":date,"description":description}def getMaxPages(query):  
    url="[https://fr.indeed.com/emplois?q=](https://fr.indeed.com/emplois?q=)"+str(query)
```

We're utilizing HTML characteristics like "class" and "id" to get the information we're looking for; you can figure out how to choose the data you need by studying the website.

Here's an illustration of the title property:

[![image.png](https://i.postimg.cc/BvWcg3rp/image.png)](https://postimg.cc/nXkDFyc9)

The title is a "h1" that we may pick by utilizing its class.

Finally, let's write a function that will loop over all of the jobs and store them to a csv file.

Take note that we are getting the maximum amount of pages so that the crawler will stop when we reach the last page.


```
def getJobsForQuery(query):  
    data=[]  
    maxPages=999  
    for number in range(maxPages):  
        maxPages,urls=getPageUrls(query,number+1)  
        for url in urls:  
            try:  
                page=processPage(url)  
                data.append(page)  
            except:  
                pass  
        print("finished Page number: "+str(number+1))  
    #Save the data to a csv file  
    pd.DataFrame(data).to_csv("jobs_"+query+".csv")
```

Now let’s scrape some data scientist offers:

```
driver = webdriver.Chrome(ChromeDriverManager().install(),options=chrome_options)  
getJobsForQuery("data scientist")
```

What we got:

[![image.png](https://i.postimg.cc/hjBkNHdb/image.png)](https://postimg.cc/WD5YkYSh)
A Sample of scraped jobs

Conclusion
==========

In this post, we learnt about web scraping, why it's crucial for every aspiring data scientist, and the many techniques of extracting jobs from Indeed.

if you made it this far Congratulations. Thank you for reading, and I hope you found the material interesting. Feel free to contact me on social media for personal contact or conversation.
