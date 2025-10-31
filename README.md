from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
driver.get("https://books.toscrape.com/")

# Get all category links
categories = driver.find_elements(By.CSS_SELECTOR, ".side_categories ul li ul li a")

results = []

for cat in categories:
    category_name = cat.text.strip()
    category_url = cat.get_attribute("href")

    driver.get(category_url)
    time.sleep(1)

    most_expensive_book = None
    highest_price = 0

    while True:
        books = driver.find_elements(By.CSS_SELECTOR, "article.product_pod")
        for book in books:
            title = book.find_element(By.CSS_SELECTOR, "h3 a").get_attribute("title")
            if title.lower().startswith('h'):
                price_text = book.find_element(By.CSS_SELECTOR, ".price_color").text
                price = float(price_text[1:])  # remove £ sign

                if price > highest_price:
                    highest_price = price
                    most_expensive_book = (title, price)

        # Try next page
        try:
            next_button = driver.find_element(By.CSS_SELECTOR, ".next a")
            next_button.click()
            time.sleep(1)
        except:
            break  # no next page

    if most_expensive_book:
        results.append((category_name, most_expensive_book[0], most_expensive_book[1]))

driver.quit()

# Print results
for category, title, price in results:
    print(f"{category}: {title} - £{price}")
