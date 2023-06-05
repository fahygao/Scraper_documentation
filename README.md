# Scraper_documentation
Here is a documentation about my methodology to create scrapers and current challenges using Python. 

Updated on 06/05/2023

Motivation: Many publications have their own website to view news online, and send subscribers corresponding emails for the news. Generally, the emails come in seconds of delay and may only be separate links to view the full content. If creating scrapers that can directly scrape the website under a PROPER refreshing rate, and send a email including new content when the scrapers detect the change, then we can most likely beat the delays and directly view news in a customized way, such as highlighting keywords, etc. 

**Steps to create scrapers**

1. Investigate the target website on its structure, and what needs to be scraped. And what format the content is.

2. Understand what the refreshing rate should be, and what human interaction should be like if viewing and downloading content from the publications. 

3. Start to code and follow the exact same way as a human interacts with the website.  

4. Test on the output for a week and list all the potential formats on the scraped content, while updating the scraper programs meanwhile. 


**Types of publication content and ways to hack them (easy to hard)**

1. Embeded html element: we can directly copy and past all the content on the website using xpath. XPATH can be found from browser's inspect section. 

   Method: Selenium (webdriver) 

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

    Method: Selenium (webdriver, screenshot), PIL (image), pytesseract (read texts from images), optional: BeautifulSoup

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

    Method: Selenium (webdriver), [PyPDF2](https://pypi.org/project/PyPDF2/) (Extract words from pdf)

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

    Method: Selenium (webdrive)

    Note: this is fairly easy. If the content element only has the bigger container as unselectable, we can just use the sub-element for the rest of the content. But if all the content elements are unselectable, we can mimic ```cirl+P``` to print out the page and read the content from pdf. If the above two methods are not efficient,  we should try mimic the way of (2). 


PS: some websites of the publications requires a login verification everytime when we visit the website, so we can try to use Option and set up a local host first, then the scraper program will always use the same chrome broswer window to scrape the content.  
    steps: 
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
    url="https://www.google.com/"
    o.add_experimental_option("debuggerAddress","localhost:9102")
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()),options=o)
    ```

