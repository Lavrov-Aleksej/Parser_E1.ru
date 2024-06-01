# Парсер комментариев на новостном портале E1.RU
## Запуск проекта
> Импорт необходимых библиотек для работы с веб-страницами, парсинга данных и управления веб-драйвером
```python
> from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import time
import undetected_chromedriver as uc
import requests
from fake_useragent import UserAgent
from bs4 import BeautifulSoup
import codecs
from webdriver_manager.chrome import ChromeDriverManager
import pandas as pd
```
> Создание опции для веб-драйвера Chrome, в данном случае отключается опция "AutomationControlled", которая может использоваться для обнаружения автоматизированных действий
```python
options = webdriver.ChromeOptions()
options.add_argument("--disable-blink-features=AutomationControlled")
```
> Создание экземпляра веб-драйвера для обхода обнаружения автоматизации
```python
driver = uc.Chrome(headless=False, use_subprocess=False, options=options)
```
>  Добавление JavaScript-скрипт для удаления определенных переменных из глобального объекта window при загрузке нового документа
 ```python
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
    'source': '''
    delete window.cdc_adoQpoasnfa76pfcZLmcfl_Array;  # Удаление переменной window.cdc_adoQpoasnfa76pfcZLmcfl_Array
    delete window.cdc_adoQpoasnfa76pfcZLmcfl_Promise;  # Удаление переменной window.cdc_adoQpoasnfa76pfcZLmcfl_Promise
    delete window.cdc_adoQpoasnfa76pfcZLmcfl_Symbol;  # Удаление переменной window.cdc_adoQpoasnfa76pfcZLmcfl_Symbol
    '''
})
```
> Создание и применение функции
```python
def appendhtml(url):  # Определение функции appendhtml с параметром url
    driver.get(url)  # Загрузка страницы по указанному URL
    time.sleep(3)  # Пауза в выполнении программы на 3 секунды

    driver.execute_script("window.scrollTo(0, 10000)")  # Выполнение JavaScript-скрипта для прокрутки страницы
    time.sleep(2)  # Пауза в выполнении программы на 2 секунды

    html0 = driver.page_source  # Получение HTML-кода страницы

    soup = BeautifulSoup(html0, "html.parser")  # Создание объекта BeautifulSoup для парсинга HTML-кода
    blocks = soup.find_all('div', class_='aI5Ey')  # Поиск всех элементов div с классом 'aI5Ey'

    for item in blocks:  # Для каждого найденного элемента
        main_comment = item.find('div', class_='s3XzP').p.text  # Извлечение основного комментария
        print("Main Comment:", main_comment)  # Вывод основного комментария

        try:
            sub_comments = item.find('div', class_='Iyr+n').find_all('span', class_='Bfr0k')  # Поиск всех доп. ком
            for comment in sub_comments:  # Для каждого найденного дополнительного комментария
                print("Sub Comment:", comment.text)  # Вывод дополнительного комментария
        except AttributeError:
            print("No sub comments found")  # Вывод сообщения, если дополнительные комментарии не найдены

appendhtml('https://www.e1.ru/text/education/2023/09/07/72678422/comments/')  # Вызов функции appendhtml с указанным URL
```
