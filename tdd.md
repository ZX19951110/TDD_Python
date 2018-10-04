# Part 1

##  Chapter 1: Getting Django Set Up Using a Functional Test

### Installare _geckodriver_

Per installare _geckodriver_ su macOS con _Homebrew_ eseguire il seguente comando da terminale:

`brew install geckodriver`

Su macOS eseguire anche:

`export PATH=$PATH:/usr/local/Cellar/geckodriver/0.22.0`

dove `/usr/local/Cellar/geckodriver/0.22.0` è la directory nella quale è installato _geckodriver_


### Installare _Selenium_

Per installare _Selenium_ è necessario eseguire il seguente comando:

`pip install selenium`


### Creare il primo test funzionale

Creare un file _functional_tests.py_ contenente il seguente codice:

```py

from selenium import webdriver`

`browser = webdriver.Firefox()`
`browser.get('http://localhost:8000')`

`assert 'Django' in browser.title

```

Eseguire il programma:

`python functional_tests.py`

A questo punto, se abbiamo fatto tutto correttamente, vedremo aprirsi una finestra del browser che tenterà di aprire una pagina web allocata all'indirizzo `localhost:8000`. Tuttavia visualizzeremo un errore che ci avviserà che la connessione al server non è riuscita. Ciò è accaduto in quanto non abbiamo ancora avviato il framework Django.

Ora creiamo un nuovo progetto Django con il comando:

`django-admin.py startproject superlists .`

ed eseguiamo il server Django digitando:

`python manage.py runserver`

A questo punto se eseguiamo nuovamente il programma _functional_test.py_, si aprirà una finestra del browser e vedremo comparire un messaggio che ci segnalerà che il server Django è stato avviato correttamente.


## Chapter 2: Extending Our Functional Test Using the unittest Module

Creiamo un programma chiamato _functional_test.py_ per eseguire successivamente il test funzionale:

```py

from selenium import webdriver

browser = webdriver.Firefox()

# Edith has heard about a cool new online to-do app. She goes
# to check out its homepage
browser.get('http://localhost:8000')

# She notices the page title and header mention to-do lists
assert 'To-Do' in browser.title

# She is invited to enter a to-do item straight away

# She types "Buy peacock feathers" into a text box (Edith's hobby
# is tying fly-fishing lures)

# When she hits enter, the page updates, and now the page lists
# "1: Buy peacock feathers" as an item in a to-do list

# There is still a text box inviting her to add another item. She
# enters "Use peacock feathers to make a fly" (Edith is very methodical)

# The page updates again, and now shows both items on her list

# Edith wonders whether the site will remember her list. Then she sees
# that the site has generated a unique URL for her -- there is some
# explanatory text to that effect.

# She visits that URL - her to-do list is still there.

# Satisfied, she goes back to sleep

browser.quit()

```
Il fulcro di questo programma è l'istruzione `assert 'To-Do' in browser.title`, la quale va a controllare che nel titolo della pagina visualizzata nel browser sia presente la parola _To-Do_.

Ora invece proviamo ad eseguire il seguente test, il quale fa uso della libreria _unittest_:

```py

from selenium import webdriver
import unittest

class NewVisitorTest(unittest.TestCase):  

def setUp(self):  
self.browser = webdriver.Firefox()

def tearDown(self):  
self.browser.quit()

def test_can_start_a_list_and_retrieve_it_later(self):  
# Edith has heard about a cool new online to-do app. She goes
# to check out its homepage
self.browser.get('http://localhost:8000')

# She notices the page title and header mention to-do lists
self.assertIn('To-Do', self.browser.title)  
self.fail('Finish the test!')

if __name__ == '__main__':  
unittest.main(warnings='ignore')

```

Ovviamente qusto programma per il test funzionale ci segnalerà che il test eseguito è fallito.
Osservando il codice è importante notare che la classe _NewVisitorTest_ eredita i metodi dalla classe _TestCase_ inclusa nella libreria _unittest_.
