# Scraper For Publications
Here is a documentation about my methodology to create scrapers and current challenges using Python. 

Updated on 06/05/2023

Motivation: Many publications have their own website to view news online, and send subscribers corresponding emails for the news. Generally, the emails come in seconds of delay and may only be separate links to view the full content. If creating scrapers that can directly scrape the website under a PROPER refreshing rate, and send a email including new content when the scrapers detect the change, then we can most likely beat the delays and directly view news in a customized way, such as highlighting keywords, etc. 

## Steps to create scrapers

1. Investigate the target website on its structure, and what needs to be scraped. And what format the content is.

2. Understand what the **refreshing rate** should be (wrong refreshing rate can be marked as abnormal activity and you might be involved with legal issues!), and what human interaction should be like if viewing and downloading content from the publications. 

3. Start to code and follow the exact same way as a human interacts with the website.  

4. Test on the output for a week and list all the potential formats on the scraped content, while updating the scraper programs meanwhile. 


**Types of publication content and ways to hack them (easy to hard)**

1. Embeded html element: we can directly copy and past all the content on the website using xpath. XPATH can be found from browser's inspect section. 

   Method: [Selenium](https://pypi.org/project/selenium/) (webdriver) 

   ```Python
    #packages
    from selenium import webdriver
    url="WEBSITE_LINK" #website link
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install())) #auto-install the chromedriver
    driver.get(url) #get the url for webdriver

    drive.refresh()#refresh the website 
    #e.g.
    content = driver.find_element("xpath",'THE_RIGHT_XPATH')
    ```
2. Image: cannot be copied or be selected the content and the news stores as a image on the website. 

    Method: [Selenium](https://pypi.org/project/selenium/) (webdriver, screenshot), [PIL](https://pypi.org/project/Pillow/) (image), [pytesseract](https://pypi.org/project/pytesseract/) (read texts from images), optional: [BeautifulSoup](https://beautiful-soup-4.readthedocs.io/en/latest/)

    ```Python
    #packages
    from selenium import webdriver
    import pytesseract
    from PIL import Image
    #optional
    from bs4 import BeautifulSoup 
    #e.g.
    driver.find_element(By.CLASS_NAME, 'content-div').screenshot(p)
    image = Image.open("PATH{}.png".format(str(n)))
    custom_config = r'--oem 3 --psm 6'
    article = pytesseract.image_to_string(image,config=custom_config)

    #optional
    soup_level=BeautifulSoup(driver.page_source,'lxml')
    article1=soup_level.find('div',class_="article_body")
    ```

    PS: some publication websites have watermark behind the content, so it is hard for pytesseract to recognize the words. Solution: expand the window size when taking screenshot, and normally the watermark will be more separate and less dense. 

3. PDF: cannot be selected and news shows as a pdf on the wesbite. User can download the content as a pdf. 

    Method: [Selenium](https://pypi.org/project/selenium/) (webdriver), [PyPDF2](https://pypi.org/project/PyPDF2/) (Extract words from pdf)

    ```Python
    #packages
    from selenium import webdriver
    import PyPDF2
    #e.g.
    pdffileobj=open(path1,'rb')
    pdfreader=PyPDF2.PdfReader(pdffileobj)
    for i in range(x):
        pageobj=pdfreader.pages[i]
        text=pageobj.extract_text()
        texts.append(text)
    texts=' '.join(texts)
    ```
4. Secured content cannot select: connect be selected and content element in html has class "unselectable".

    Method: [Selenium](https://pypi.org/project/selenium/) (webdrive)

    Note: this is fairly easy. If the content element only has the bigger container as unselectable, we can just use the sub-element for the rest of the content. But if all the content elements are unselectable, we can mimic ```cirl+P``` to print out the page and read the content from pdf. If the above two methods are not efficient,  we should try mimic the way of (2). 


PS: some websites of the publications requires a login verification everytime when we visit the website, so we can try to use Option and set up a local host first, then the scraper program will always use the same chrome broswer window to scrape the content.  
Steps: 
1. Create a new folder with the publication name
2. Open Terminal ```cd C:\Program Files \Google\Chrome\Application```
3. Start the chrome broswer in the publication folder: 
            ```chrome.exe --remote-debugging-port=9102 --user-data-dir="PATH"```
4. Login to the publication page first time with credentials
5. Run selenium with Option that indicates the localhost. 

    ``` Python
    #package
    from selenium.webdriver.chrome.options import Options
    #e.g.
    o = Options()
    url= 'https://www.google.com/'
    o.add_experimental_option('debuggerAddress','localhost:9102')
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()),options=o)
    ```

## Ways to setup outlook email and potential NLP analysis

Email (Outlook)
---
I used Outlook for email distribution. Here is a brief introduction on how to setup outlook email for sending email through Python. We can created a bag of words that includes all the words that need to be highlighted, so in the email we can directly mark the key words to increase readibility. 

Method: [win32com.client]() (Dispatch)

``` Python
outlook=Dispatch('Outlook.Application')
mail = outlook.CreateItem(0)
mail.SentOnBehalfOfName ='@gmail.com'
mail.To='@gmail.com'
mail.Subject = 'HEADLINE'
article = texts.replace('\n','<br> ')
article_s = article.split()
Words=pd.read_excel('PATH TO BAGS OF WORDS',sheet_name='Sheet2')
words=list(Words['Bag of Words'])
for i in article_s:    
    lower=list(map(lambda x: x.lower(),words))
    highlight='<span style=background:yellow;mso-highlight:yellow>'
    if i.lower() in lower:
        article_temp=list(map(lambda x: x.replace(i,highlight+i+'</span>'),article_s))
        mail.SentOnBehalfOfName = '@gmail.com'
        mail.To='@gmail.com'
article_final=' '.join(article_temp)
mail.HTMLBody = ("""{} <br><br>
                    Publish Date: {} <br><br>
                    Source: {}<br>
                {}<br><br>""".format(new_headline,publish_time,'PUBLICATION NAME',article_final))
mail.Display()
```

Potential NLP analysis
---
Vectorizing the words using  ```sklearn```, we can then use TF-IDF to normalize the frency of words appearing in the article. And using PCA, we can extract key words from the entire content, largely reducing understanding time. 

Method: [sklearn](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html)(feature_extraction.text), [nltk](https://pypi.org/project/nltk/)(corpus, stopwords)


``` Python
import nltk
nltk.download('stopwords')
from nltk.corpus import stopwords
from sklearn.feature_extraction.text import CounterVectorizer
cv = CountVectorizer(max_features = 1500)
x = cv.fit_transform(corpus).toarray()

```


Another way is that we can first implement a bag of positive words that appeared in the past content, and check through entire content to see the percentage of positive words appearing to the words. 


## Current logic and limitations
In general, the programs are written in the logic that once scraper detect the changes on the most recent title (or whole page). It will scrap news content appeared as the first change. So all programs have the potential risks of skipping news if multiple news coming out at the same time on the website, or news is updated to different pages. And no matter what reason causes the scrapers down, it is designed to automatically send a stop alert email to you with error info. 

Here are the issues I have encountered:
1. Website might show permission denied due to too many refreshing attempts (frequently happen to governor websites). 
   1. Temporary solution: I have to wait for another 5 mins or clear out the cookies to open the website again. 
3. Website might auto log out after a certain time. This varies from different publication websites. 
   1. Temporary solution is to detect the element of log-in page and re-login when we cannot detect title element. 
4. Website might publish two or three news at the same time. 
   1.  Temporary solution: manually send out all the missed news content. 
5. Website might publish news not in a time order. So it might revise the old news but not show up on the website as a latest news.
   1.  Temporary solution: manually send out the revised old news content.
6. Website might delete the news within 1-2 seconds due to typo or any other reason, so scraper will stop and show 404 error. 
   1.  Temporary solution: report and restart the scraper
7. Website might be down due to system maintenance (rarely). 
   1.  Temporary solution: report and wait for website to be open again. 
8. Webste might have anti-scrape logic so we have to open the news content page twice to be able to locate content and back to main page (only happen to CMC).
   1.  Temporary solution: programed to let scraper open new news twice and close one of the new news window and change handle back to main page. **It is still not working idealy.**
