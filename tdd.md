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

from selenium import webdriver

browser = webdriver.Firefox()
browser.get('http://localhost:8000')

assert 'Django' in browser.title

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


## Chapter 3: Testing a Simple Home Page with Unit Tests

Ora creiamo la nostra prima applicazione web utilizzando Django. Per creare un nuovo progetto eseguiamo il seguente comando:

`python manage.py startapp lists`

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

`python manage.py test`

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
Lanciando nuovamente il comando `python manage.py test` ci verrà segnalato che abbiamo bisogno di un _URL mapping_.

Per far ciò dovremo modificare il file `superlists/urls.py` inserendo il seguente codice:

```py

from django.conf.urls import url
from lists import views

urlpatterns = [
url(r'^$', views.home_page, name='home'),
]

```

Eseguendo nuovamente `python manage.py test` visualizzeremo il seguente errore:

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
ed eseguendo nuovamente `python manage.py test` otterremo il seguente risultato soddisfacente:

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

Eseguiamo nuovamente il test di unità [...]

