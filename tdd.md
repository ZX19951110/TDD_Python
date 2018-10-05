# Part 1

##  Chapter 1: Getting Django Set Up Using a Functional Test

### Installare _geckodriver_

* Per installare _geckodriver_ su macOS con _Homebrew_ eseguire il seguente comando da terminale:

	`$ brew install geckodriver`

	Eseguire anche:

	`$ export PATH=$PATH:/usr/local/Cellar/geckodriver/0.22.0`

* Per installare _geckodriver_ su Linux Ubuntu scaricare l'ultima versione sul [sito ufficiale](https://github.com/mozilla/geckodriver/releases), decomprimere l'archivio, rendere il file eseguibile con il comando:

	`$ chmod +x geckodriver`

	e copiare l'eseguibile nella directory `/usr/local/bin/`:

	`$ sudo cp geckodriver /usr/local/bin/`

	dove `/usr/local/Cellar/geckodriver/0.22.0` è la directory nella quale è installato _geckodriver_


### Installare _Selenium_

Per installare _Selenium_ è necessario eseguire il seguente comando:

`$ pip install selenium`


### Creare il primo test funzionale

Creare un file _functional_tests.py_ contenente il seguente codice:

```py

from selenium import webdriver

browser = webdriver.Firefox()
browser.get('http://localhost:8000')

assert 'Django' in browser.title

```

Eseguire il programma:

`$ python functional_tests.py`

A questo punto, se abbiamo fatto tutto correttamente, vedremo aprirsi una finestra del browser che tenterà di aprire una pagina web allocata all'indirizzo `localhost:8000`. Tuttavia visualizzeremo un errore che ci avviserà che la connessione al server non è riuscita. Ciò è accaduto in quanto non abbiamo ancora avviato il framework Django.

Ora creiamo un nuovo progetto Django con il comando:

`$ django-admin.py startproject superlists .`

ed eseguiamo il server Django digitando:

`$ python manage.py runserver`

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


## Chapter 3: Testing a Simple Home Page with Unit Tests

Ora creiamo la nostra prima applicazione web utilizzando Django. Per creare un nuovo progetto eseguiamo il seguente comando:

`$ python manage.py startapp lists`

A questo punto creiamo il nostro primo test errato per verificare che la nostra macchina funzioni correttamente:

`cd lists`

`vim tests.py`

e inseriamo questo codice nel file appena aperto:

```py

from django.test import TestCase

class SmokeTest(TestCase):

    def test_bad_maths(self):

    	self.assertEqual(1 + 1, 3)

```

Ora eseguiamo il programma:

`$ python manage.py test`

Ora modifichiamo nuovamente il file _tests.py_ in modo tale che, una volta eseguito, possa verificare l'esistenza della home page. Il codice è il seguente:

```py

from django.urls import resolve
from django.test import TestCase
from lists.views import home_page  

class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
    	found = resolve('/')  
    	self.assertEqual(found.func, home_page)

```


### Creazione della prima _view_
Per creare la nostra prima _view_ apriamo il file `lists/views.py` e inseriamo le seguenti righe di codice:

```py

from django.shortcuts import render

# Create your views here.
home_page = None

```
Lanciando nuovamente il comando `$ python manage.py test` ci verrà segnalato che abbiamo bisogno di un _URL mapping_.

Per far ciò dovremo modificare il file `superlists/urls.py` inserendo il seguente codice:

```py

from django.conf.urls import url
from lists import views

urlpatterns = [
    url(r'^$', views.home_page, name='home'),
]

```

Eseguendo nuovamente `$ python manage.py test` visualizzeremo il seguente errore:

```

[...]
TypeError: view must be a callable or a list/tuple in the case of include().

```

A questo punto possiamo modificare nuovamente il file _lists/views.py_ come segue:

```py

from django.shortcuts import render

# Create your views here.
def home_page():
    pass

```
ed eseguendo nuovamente `$ python manage.py test` otterremo il seguente risultato soddisfacente:

```

Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...

```


### Test di unità di una _view_
Modifichiamo il file _lists/tests.py_ come segue:

```py

from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest

from lists.views import home_page


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
    	found = resolve('/')
    	self.assertEqual(found.func, home_page)


    def test_home_page_returns_correct_html(self):
    	request = HttpRequest()  
    	response = home_page(request)  
    	html = response.content.decode('utf8')  
    	self.assertTrue(html.startswith('<html>'))  
    	self.assertIn('<title>To-Do lists</title>', html)  
    	self.assertTrue(html.endswith('</html>'))

```

Nel metodo `test_home_page_returns_correct_html(self)` creiamo un oggetto `HttpRequest`, e lo passiamo all'oggetto `response`, ovvero la nostra home page. Nei passaggi seguenti estraiamo il contenuto della pagina ed andiamo a verificare che il titolo della nostra pagina sia _To-Do_ e che il codice della pagina inizi con `<html>` e finisca con `</html>`.

Eseguiamo nuovamente il test di unità e otterremo il seguente errore:

`TypeError: home_page() takes 0 positional arguments but 1 was given`

Procedendo ciclicamente in modo tale da eseguire un test per ogni singola istruzione che scriviamo possiamo giustificare ogni istruzione presente nel nostro codice.
Alla fine del ciclo di programmazione e test della nostra prima view, il file `lists/tests.py` avrà il seguente aspetto:

```py

from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def home_page(request):
    return HttpResponse('<html><title>To-Do lists</title></html>')

```

ed eseguendo nuovamente il test otterremo il nostro primo successo!

Invece, avviando il server di Django ed eseguendo il nostro test funzionale precedentemente creato otterremo il seguente errore:

```

F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_test2.py", line 19, in test_can_start_a_list_and_retrieve_it_later
    self.fail('Finish the test!')
AssertionError: Finish the test!

----------------------------------------------------------------------
Ran 1 test in 2.910s

FAILED (failures=1)

```


## Chapter 4: What Are We Doing with All These Tests? (And, Refactoring)

Creando un file _functional_tests.py_ come segue:

```py

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
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
        header_text = self.browser.find_element_by_tag_name('h1').text  
        self.assertIn('To-Do', header_text)

        # She is invited to enter a to-do item straight away
        inputbox = self.browser.find_element_by_id('id_new_item')  
        self.assertEqual(
            inputbox.get_attribute('placeholder'),
            'Enter a to-do item'
        )

        # She types "Buy peacock feathers" into a text box (Edith's hobby
        # is tying fly-fishing lures)
        inputbox.send_keys('Buy peacock feathers')  

        # When she hits enter, the page updates, and now the page lists
        # "1: Buy peacock feathers" as an item in a to-do list table
        inputbox.send_keys(Keys.ENTER)  
        time.sleep(1)  

        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')  
        self.assertTrue(
            any(row.text == '1: Buy peacock feathers' for row in rows)
        )

        # There is still a text box inviting her to add another item. She
        # enters "Use peacock feathers to make a fly" (Edith is very
        # methodical)
        self.fail('Finish the test!')

if __name__ == '__main__':  
    unittest.main(warnings='ignore')

```

ed eseguendolo dopo aver avviato il server django, otteniamo il seguente messaggio di errore:

```

selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: h1

```

in quanto non è stato possibile trovare un elemento `<h1>` all'interno della pagina web.



