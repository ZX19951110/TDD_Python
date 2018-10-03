# Part 1

##  Chapter 1: Getting Django Set Up Using a Functional Test

### Installare _geckodriver_

Per installare _geckodriver_ su macOS come _Homebrew_ eseguire il seguente comando da terminale:

`brew install geckodriver`

Su macOS eseguire anche:

`export PATH=$PATH:/usr/local/Cellar/geckodriver/0.22.0`

dove `/usr/local/Cellar/geckodriver/0.22.0` è la directory nella quale è installato _geckodriver_


### Installare _Selenium_

Per installare _Selenium_ è necessario eseguire il seguente comando:

`pip install selenium`


### Creare il primo test funzionale

Creare un file _functional_tests.py_ contenente il seguente codice:
`from selenium import webdriver

browser = webdriver.Firefox()
browser.get('http://localhost:8000')

assert 'Django' in browser.title`

