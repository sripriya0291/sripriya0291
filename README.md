from selenium.webdriver import Chrome
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from bs4 import BeautifulSoup
import pymongo
import time
from pymongo import MongoClient
query = 'Data science'
location = 'India'
browser = Chrome()

url = 'https://www.linkedin.com/login'
browser.get(url)
html = browser.page_source
email_input = browser.find_element(By.ID, 'username')
password_input = browser.find_element(By.ID, 'password')
email_input.send_keys('your_email')
password_input.send_keys('your_password')
password_input.send_keys(Keys.ENTER)
time.sleep(10)

for page_num in range(1, 25):
    url = f'https://www.linkedin.com/jobs/search/?keywords={query}&location={location}&start={25 * (page_num - 1)}'

    browser.get(url)

last_height = browser.execute_script('return document.body.scrollHeight')
while True:
    browser.execute_script('window.scrollTo(0, document.body.scrollHeight);')
    new_height = browser.execute_script('return document.body.scrollHeight')
    if new_height == last_height:
        break
    last_height = new_height
time.sleep(20)
soup = BeautifulSoup(browser.page_source, 'html.parser')
job_postings = soup.find_all('li', {'class': 'jobs-search-results__list-item'})

mc=MongoClient()
db = mc['linked_in']
coll = db['posts']
coll.insert_one({
    'url': url,
    'html': html

})
coll.count_documents({})
coll.find_one()

