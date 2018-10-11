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
[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: h1

```

in quanto non è stato possibile trovare un elemento `<h1>` all'interno della pagina web.


### Template

Creiamo il nostro primo template _home.html_ in `lists/templates` con il seguente codice HTML:

```html

<html>
    <title>To-Do lists</title>
</html>

```

e modifichiamo la nostra view `lists/views.py` come segue:

```py

from django.shortcuts import render

def home_page(request):
    return render(request, 'home.html')

```

Eseguendo:

`$ python manage.py test`

Vedremo apparire il seguente messaggio di errore:

```

E.
======================================================================
ERROR: test_home_page_returns_correct_html (lists.tests.HomePageTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/home/nicolas/Nextcloud/Documents/UNIPV/triennale/tesi/django/lists/tests.py", line 17, in test_home_page_returns_correct_html
    response = home_page(request)
  File "/home/nicolas/Nextcloud/Documents/UNIPV/triennale/tesi/django/lists/views.py", line 5, in home_page
    return render(request, 'home.html')
  File "/home/nicolas/.pyenv/versions/3.7.0/lib/python3.7/site-packages/django/shortcuts.py", line 36, in render
    content = loader.render_to_string(template_name, context, request, using=using)
  File "/home/nicolas/.pyenv/versions/3.7.0/lib/python3.7/site-packages/django/template/loader.py", line 61, in render_to_string
    template = get_template(template_name, using=using)
  File "/home/nicolas/.pyenv/versions/3.7.0/lib/python3.7/site-packages/django/template/loader.py", line 19, in get_template
    raise TemplateDoesNotExist(template_name, chain=chain)
django.template.exceptions.TemplateDoesNotExist: home.html

----------------------------------------------------------------------
Ran 2 tests in 0.004s

```

in quanto non abbiamo aggiunto `lists` alla variabile `INSTALLED _APPS` presente nel file `superlists/settings.py`. Effettuando questa aggiunta ed eseguendo nuovamente il test otterremo il seguente messaggio d'errore:

```

[...]
self.assertTrue(html.endswith('</html>'))
AssertionError: False is not true

```

Per risolvere il problema causato dall'ultima riga del nostro codice HTML è sufficiente sostituire la riga 

`self.assertTrue(html.endswith('</html>'))`

del file `lists/tests.py` con la seguente:

`self.assertTrue(html.strip().endswith('</html>'))`

A questo punto, rilanciando il test, otterremo un successo!


### Django Test Client

Ora utilizziamo il Django Test Client per controllare quali _templates_ sono usati. Perciò dobbiamo modificare il file `lists/tests.py` come segue:

```py

from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest
from lists.views import home_page
from django.template.loader import render_to_string


class HomePageTest(TestCase):

	def test_root_url_resolves_to_home_page_view(self):
		found = resolve('/')
		self.assertEqual(found.func, home_page)


	def test_home_page_returns_correct_html(self):
        	response = self.client.get('/')  

        	html = response.content.decode('utf8')  
        	self.assertTrue(html.startswith('<html>'))
        	self.assertIn('<title>To-Do lists</title>', html)
        	self.assertTrue(html.strip().endswith('</html>'))

        	self.assertTemplateUsed(response, 'home.html')

```

Utilizzando il metodo `response = self.client.get('/')` abbiamo evitato di creare manualmente un oggetto `HttpRequest`.
Il metodo `self.assertTemplateUsed(response, 'home.html')` invece ci permette di verificare se il _template_ `home.html` è effettivamente utilizzato.
Ora, se eseguiamo il test, dovremmo ottenere un successo.

### Front page

Modifichiamo il file `lists/templates/home.html` come segue:

```html

<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>Your To-Do list</h1>
	    <input id="id_new_item" placeholder="Enter a to-do item" />
	    <table id="id_list_table">
            </table>
    </body>
</html>

```

e il file `functional_tests.py`:

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
        	any(row.text == '1: Buy peacock feathers' for row in rows),
        	"New to-do item did not appear in table"
    	)

        # There is still a text box inviting her to add another item. She
        # enters "Use peacock feathers to make a fly" (Edith is very
        # methodical)
        self.fail('Finish the test!')

if __name__ == '__main__':  
    unittest.main(warnings='ignore')

```

A questo punto, eseguendo il test, otterremo il seguente errore:

```

F
======================================================================
FAIL: test_can_start_a_list_and_retrieve_it_later (__main__.NewVisitorTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "functional_tests.py", line 44, in test_can_start_a_list_and_retrieve_it_later
    "New to-do item did not appear in table"
AssertionError: False is not true : New to-do item did not appear in table

----------------------------------------------------------------------
Ran 1 test in 4.929s

FAILED (failures=1)

```

## Chapter 5: Saving User Input: Testing the Database

### POST Request

Ora modifichiamo il file `lists/templates/home.html` come segue:

```html

<html>
    <head>
        <title>To-Do lists</title>
    </head>
    
    <body>
        <h1>Your To-Do list</h1>
        <form method="POST">
	    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
	    {% csrf_token %}
	</form>	

	<table id="id_list_table">
        </table>
    </body>
</html>

```

ed eseguiamo il test funzionale. Riceveremo un messaggio di errore del seguente tipo:

```

[...]
selenium.common.exceptions.NoSuchElementException: Message: Unable to locate element: [id="id_list_table"]

```

Per avere il tempo di visualizzare il messaggio di errore apparso nella scheda del browser possiamo modificare l'istruzione `time.sleep(1)` presente in `functional tests.py` in `time.sleep(10)`, dove il parametro passato al metodo indica il tempo che dovrà intercorrere prima che l'esecuzione del programma possa continuare.

Ora utilizziamo un _template tag_ per inserire un _token CSRF_ all'interno della nostra pagina html:

```html

<form method="POST">
    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
    {% csrf_token %}
</form>

```

Eseguendo di nuovo il test funzionale otterremo il seguente errore:

`AssertionError: False is not true : New to-do item did not appear in table`


### Processare una richiesta POST sul server
Aggiungendo i seguenti metodi al file `lists/tests.py`:

```py

def test_uses_home_template(self):
    response = self.client.get('/')
    self.assertTemplateUsed(response, 'home.html')


def test_can_save_a_POST_request(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})
    self.assertIn('A new list item', response.content.decode())

```

ed eseguendo il comando:

`$ python manage.py test`

otteniamo il seguente messaggio di errore:

```

self.assertIn('A new list item', response.content.decode())
AssertionError: 'A new list item' not found in '<html>\n    <head>\n        <title>To-Do lists</title>\n    </head>\n\n    <body>\n        <h1>Your To-Do list</h1>\n        <form method="POST">\n\t    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />\n\t    <input type="hidden" name="csrfmiddlewaretoken" value="t6h7RCCdD0xN4g1goOcrfsSoHVB7f6jtTpisL9p63BpvF3bajw7sBJFkXbnqlM7z">\n\t</form>\n\n\t<table id="id_list_table">\n        </table>\n    </body>\n</html>\n'

```

Ora modifichiamo il corpo del file `lists/templates/home.html` come segue:

```html

<body>
    <h1>Your To-Do list</h1>
    <form method="POST">
        <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
	    {% csrf_token %}
    </form>

    <table id="id_list_table">
        <tr><td>{{ new_item_text }}</td></tr>
    </table>
</body>

```

`new_item_text` è il nome della variabile dell'input immesso dall'utente che viene visualizzato nel _template_.

Modifichiamo il metodo `test_can_save_a_POST_request(self)` contenuto nel file `lists/tests.py` nel seguente modo:

```py

def test_can_save_a_POST_request(self):
        response = self.client.post('/', data={'item_text': 'A new list item'})
        self.assertIn('A new list item', response.content.decode())
        self.assertTemplateUsed(response, 'home.html')

```

ed eseguiamo il test. Otterremo un messaggio d'errrore.

Ora modifichiamo il metodo `home_page(request)` presente in `lists/views.py`:

```py

def home_page(request):
	return render(request, 'home.html', {
		'new_item_text': request.POST['item_text'],
	})

```

ed otterremo un altro messaggio di errore in quanto abbiamo commesso una svista nel metodo `home_page(request)` presente in `lists/views.py`. Ora modifichiamolo in questo modo:

```py

def home_page(request):
	return render(request, 'home.html', {
		'new_item_text': request.POST.get('item_text', ''),
	})

```

Ora il nostro test dovrebbe passare correttamente, tuttavia la stessa cosa non si può dire per il test fuzionale. Per rimediare all'errore iniziamo ad effettuare la seguente modifica nel file `lists/templates/home.html`:

`<tr><td>1: {{ new_item_text }}</td></tr>`

Tuttavia ciò non è sufficiente in quanto eseguendo il test funzionale otteniamo nuovamente un errore:

`AssertionError: Finish the test!`


### Refactor

Ora facciamo un po' di refactor al codice di `functional_tests.py` e modifichiamolo come segue:

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

    def check_for_row_in_list_table(self, row_text):
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')
        self.assertIn(row_text, [row.text for row in rows])

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

        inputbox.send_keys('Buy peacock feathers')  

        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)
        self.check_for_row_in_list_table('1: Buy peacock feathers')

        inputbox = self.browser.find_element_by_id('id_new_item')
        inputbox.send_keys('Use peacock feathers to make a fly')
        inputbox.send_keys(Keys.ENTER)
        time.sleep(1)

        self.check_for_row_in_list_table('1: Buy peacock feathers')
        self.check_for_row_in_list_table('2: Use peacock feathers to make a fly')
        
        self.fail('Finish the test!')

if __name__ == '__main__':  
    unittest.main(warnings='ignore')

```

Eseguendo il test funzionale otteniamo il seguente messaggio di errore:

`AssertionError: '1: Buy peacock feathers' not found in ['1: Use peacock feathers to make a fly']`


### The Django ORM

Un ORM (Object-Relational Mapper) è uno strato di astrazione per lo _storage_ dei dati in un database con tabelle, righe e colonne.

Ora inseriamo la seguente istruzione di _import_:

```py

from lists.models import Item

```

e creiamo una nuova classe nel file `lists/tests.py`:

```py

class ItemModelTest(TestCase):

    def test_saving_and_retrieving_items(self):
        first_item = Item()
        first_item.text = 'The first (ever) list item'
        first_item.save()

        second_item = Item()
        second_item.text = 'Item the second'
        second_item.save()

        saved_items = Item.objects.all()
        self.assertEqual(saved_items.count(), 2)

        first_saved_item = saved_items[0]
        second_saved_item = saved_items[1]
        self.assertEqual(first_saved_item.text, 'The first (ever) list item')
        self.assertEqual(second_saved_item.text, 'Item the second')

```
Da queste righe di codice possiamo intuire che per salvare un nuovo _Item_ nel database è sufficiente utilizzare il metodo `save()`.
Per interrogare il database utilizziamo il metodo `objects()`, mentre per contare gli elementi restituiti dalla query possiamo utilizzare il metodo `count()`. Il metodo `all()`, invece, ci restituisce tutti gli _Item_ contenuti in una determinata _table_.

Creiamo la classe `Item` all'interno del file `lists/models.py`:

```py

from django.db import models

# Create your models here.
class Item(models.Model):
    pass
    
```


### Database migration
_migrations_ è il sistema che ci permette di costruire il database e di creare e rimuovere tabelle e colonne.

Ora effettuiamo la nostra prima _database migration_:

`$ python manage.py makemigrations`

Modifichiamo la classe che abbiamo appena  creato in `lists/models.py`:

```py

class Item(models.Model):
    text = models.TextField()

```

Eseguendo nuovamente il test otterremo il seguente errore:

`django.db.utils.OperationalError: no such column: lists_item.text`

in quanto abbiamo aggiunto un nuovo campo al nostro database e abbiamo bisogno una nuova _migration_:

```

$ python manage.py makemigrations
You are trying to add a non-nullable field 'text' to item without a default; we can't do that (the database needs something to populate existing rows).
Please select a fix:
1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
2) Quit, and let me add a default in models.py
Select an option: 2

```

Selezioniamo la seconda opzione  e modifichiamo la classe Item contenuta nelò file `lists/models.pt`:

```py

class Item(models.Model):
    text = models.TextField(default='')

```

Eseguiamo di nuovo la migrazione. Se è andato tutto a buon fine, eseguendo il seguente test:

`$ python manage.py test lists`

dovremmo ottenere un successo.


### Salvare la richiesta POST nel database

Iniziamo a modificare il metodo `test_can_save_a_POST_request(self)` presente nel file `lists/tests.py` nel seguente modo:

```py

def test_can_save_a_POST_request(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})

    self.assertEqual(Item.objects.count(), 1)  
    new_item = Item.objects.first()  
    self.assertEqual(new_item.text, 'A new list item')  

    self.assertIn('A new list item', response.content.decode())
    self.assertTemplateUsed(response, 'home.html')

```

Modifichiamo `lists/views.py`:

```py

from django.shortcuts import render
from lists.models import Item

# Create your views here.
def home_page(request):
    item = Item()
    item.text = request.POST.get('item_text', '')
    item.save()

    return render(request, 'home.html', {
        'new_item_text': item.text
    })

```

Aggiungiamo un nuovo metodo di test al file `lists/tests.py`:

```py

def test_only_saves_items_when_necessary(self):
    self.client.get('/')
    self.assertEqual(Item.objects.count(), 0)

```

Modifichiamo il metodo `home_page(request)` in `lists/views` come segue:

```py

def home_page(request):
    if request.method == 'POST':
        new_item_text = request.POST['item_text']  
        Item.objects.create(text=new_item_text)  
    else:
        new_item_text = ''  
    
    return render(request, 'home.html', {
        'new_item_text': new_item_text,  
    })

```

A questo punto il test dovrebbe andare a buon fine!


### Reindirizzamento dopo una richiesta POST

Ora modifichiamo il file `lists/tests.py` come segue:

```py

def test_can_save_a_POST_request(self):
	self.client.post('/', data={'item_text': 'A new list item'})

	self.assertEqual(Item.objects.count(), 1)
	new_item = Item.objects.first()
	self.assertEqual(new_item.text, 'A new list item')


def test_redirects_after_POST(self):
	response = self.client.post('/', data={'item_text': 'A new list item'})
	self.assertEqual(response.status_code, 302)
	self.assertEqual(response['location'], '/')

```

Modifichiamo anche `lists/views.py` in modo tale che assuma questo aspetto:

```py

from django.shortcuts import redirect, render
from lists.models import Item

# Create your views here.
def home_page(request):
	if request.method == 'POST':
		new_item_text = request.POST['item_text']
		Item.objects.create(text=new_item_text)
		return redirect('/')
	else:
		new_item_text = ''
		return render(request, 'home.html', {
			'new_item_text': new_item_text,
		})

```

Ora eseguendo il test dovremmo ottenere un successo.


### Rendering Items in the Templates

Ora modifichiamo la classe `HomePageTest()` del nostro test `lists/tests.py` in modo tale che possa verificare che il _template_ possa mostrare più item. Perciò aggiungiamo il seguente metodo alla classe:

```py

def test_displays_all_list_items(self):
        Item.objects.create(text='itemey 1')
        Item.objects.create(text='itemey 2')

        response = self.client.get('/')

        self.assertIn('itemey 1', response.content.decode())
        self.assertIn('itemey 2', response.content.decode())

```

La sintassi del template di Django ha un tag per iterare attraverso le liste:

`{% for .. in .. %}`

Possiamo usarlo nel seguente modo aggiungendolo al file `lists/templates/home.html`:

```html

<table id="id_list_table">
    {% for item in items %}
        <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
    {% endfor %}
</table>

```

Ora dobbiamo fare il modo di passare gli item al nostro template modificando `lists/views.py`:

```py

def home_page(request):
	if request.method == 'POST':
		Item.objects.create(text=request.POST['item_text'])
		return redirect('/')

	items = Item.objects.all()
	return render(request, 'home.html', {'items': items})

```

Eseguiamo il comando:

`$ python manage.py migrate`


## Chapter 6: Improving Functional Tests: Ensuring Isolation and Removing Voodoo Sleeps

Creiamo una directory `functional_tests` contenete i seguenti file:
* `__init__.py`;
* `tests.py` ovvero la copia del nostro test funzionale `functional_tests.py`.
In questo modo potremo eseguire il nostro test funzionale utilizzando il seguente comando:

`$ python manage.py test functional_tests`

Modifichiamo il file `functional_tests/tests.py` nel seguente modo:

```py

from django.test import LiveServerTestCase
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
from selenium.common.exceptions import WebDriverException

MAX_WAIT = 10

class NewVisitorTest(LiveServerTestCase):

    def setUp(self):
        self.browser = webdriver.Firefox()

    def tearDown(self):
        self.browser.quit()

    def wait_for_row_in_list_table(self, row_text):
        start_time = time.time()
        while True:  
            try:
                table = self.browser.find_element_by_id('id_list_table')
                rows = table.find_elements_by_tag_name('tr')
                self.assertIn(row_text, [row.text for row in rows])
                return  
            except (AssertionError, WebDriverException) as e:
                if time.time() - start_time > MAX_WAIT:
                    raise e
                time.sleep(0.5)

    def test_can_start_a_list_and_retrieve_it_later(self):
        # Edith has heard about a cool new online to-do app. She goes
        # to check out its homepage
        self.browser.get(self.live_server_url)

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

        inputbox.send_keys('Buy peacock feathers')  

        # When she hits enter, the page updates, and now the page lists
        # "1: Buy peacock feathers" as an item in a to-do list table
        inputbox.send_keys(Keys.ENTER)
        self.wait_for_row_in_list_table('1: Buy peacock feathers')

        # There is still a text box inviting her to add another item. She
        # enters "Use peacock feathers to make a fly" (Edith is very
        # methodical)
        inputbox = self.browser.find_element_by_id('id_new_item')
        inputbox.send_keys('Use peacock feathers to make a fly')
        inputbox.send_keys(Keys.ENTER)

        # The page updates again, and now shows both items on her list
        self.wait_for_row_in_list_table('2: Use peacock feathers to make a fly')
        self.wait_for_row_in_list_table('1: Buy peacock feathers')

        self.fail('Finish the test!')

```

## Chapter 7: Working Incrementally

Ogni lista avrà un proprio URL:

`/lists/<list identifier>/`

Per creare una nuova lista avremo a disposizione uno speciale URL che accetta le richieste POST:

`lists/new`

Per aggiungere un nuovo item ad una lista esistente avremo a disposizione un'apposito URL al quale potremo inviare richieste POST:

`/lists/<list identifier>/add_item`


### Regression Test

`asserRegex` è una funzione di `unittest` che controlla se una stringa corrisponde ad un'espressione regolare.

Aggiungiamo i seguenti due metodi al file `functional_tests/tests.py`:

```py

def test_can_start_a_list_for_one_user(self):
    # Edith has heard about a cool new online to-do app. She goes
    #[...]
    # The page updates again, and now shows both items on her list
    self.wait_for_row_in_list_table('2: Use peacock feathers to make a fly')
    self.wait_for_row_in_list_table('1: Buy peacock feathers')

    # Satisfied, she goes back to sleep


def test_multiple_users_can_start_lists_at_different_urls(self):
    # Edith starts a new to-do list
    self.browser.get(self.live_server_url)
    inputbox = self.browser.find_element_by_id('id_new_item')
    inputbox.send_keys('Buy peacock feathers')
    inputbox.send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: Buy peacock feathers')

    # She notices that her list has a unique URL
    edith_list_url = self.browser.current_url
    self.assertRegex(edith_list_url, '/lists/.+')

    # Now a new user, Francis, comes along to the site.

    ## We use a new browser session to make sure that no information
    ## of Edith's is coming through from cookies etc
    self.browser.quit()
    self.browser = webdriver.Firefox()

    # Francis visits the home page.  There is no sign of Edith's
    # list
    self.browser.get(self.live_server_url)
    page_text = self.browser.find_element_by_tag_name('body').text
    self.assertNotIn('Buy peacock feathers', page_text)
    self.assertNotIn('make a fly', page_text)

    # Francis starts a new list by entering a new item. He
    # is less interesting than Edith...
    inputbox = self.browser.find_element_by_id('id_new_item')
    inputbox.send_keys('Buy milk')
    inputbox.send_keys(Keys.ENTER)
    self.wait_for_row_in_list_table('1: Buy milk')

    # Francis gets his own unique URL
    francis_list_url = self.browser.current_url
    self.assertRegex(francis_list_url, '/lists/.+')
    self.assertNotEqual(francis_list_url, edith_list_url)

    # Again, there is no trace of Edith's list
    page_text = self.browser.find_element_by_tag_name('body').text
    self.assertNotIn('Buy peacock feathers', page_text)
    self.assertIn('Buy milk', page_text)

    # Satisfied, they both go back to sleep

```

Modifichiamo il metodo `test_redirects_after_POST(self)` nel file `lists/tests.py`:

```py

def test_redirects_after_POST(self):
    response = self.client.post('/', data={'item_text': 'A new list item'})
    self.assertEqual(response.status_code, 302)
    self.assertEqual(response['location'], '/lists/the-only-list-in-the-world/')

```
Eseguendo il test di unità in `lists` otteniamo il seguente errore:

`AssertionError: '/' != '/lists/the-only-list-in-the-world/'`

Ora modifichiamo la _view_ della nostra `home_page` nel file `lists/views`:

```py

def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/lists/the-only-list-in-the-world/')

    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})

```

Così facendo abbiamo introdotto una regressione in quanto, non solo il nuovo test produce un errore, ma produce un errore anche il vecchio test.


### Taking a First, Self-Contained Step: One New URL

Creaimo una nuova classe in `lists/tests.py`:

```py

class ListViewTest(TestCase):

    def test_displays_all_items(self):
        Item.objects.create(text='itemey 1')
        Item.objects.create(text='itemey 2')

        response = self.client.get('/lists/the-only-list-in-the-world/')

        self.assertContains(response, 'itemey 1')  
        self.assertContains(response, 'itemey 2')

```

Lanciando il nuovo test otteniamo il seguente errore:

`AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404 (expected 200)`

in quanto non è stato possibile trovare la pagina richiesta.


### A new URL

Aggiungiamo l'URL alla nostra nuova lista nel file `superlists/urls.py`:

```py

urlpatterns = [
    url(r'^$', views.home_page, name='home'),
    url(r'^lists/the-only-list-in-the-world/$', views.view_list, name='view_list'),
]

```
Eseguendo nuovamente il test otteniamo l'errore:

`AttributeError: module 'lists.views' has no attribute 'view_list'`


### A New View Function

Creiamo il metodo `view_list(request` in `lists/views.py`:

```py

def view_list(request):
    items = Item.objects.all()
    return render(request, 'home.html', {'items': items})

```

eseguendo il test d'unità su `lists` otteniamo un successo, mentre se eseguiamo il test funzionale otteniamo un errore.

Modifichiamo il file `lists/templates/home.html` in modo tale che la richiesta POST venga reindirizzata alla directory radice al posto che alla stessa pagina che ha effettuato la richiesta:

```html

<form method="POST" action="/">
    <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
    {% csrf_token %}
</form>

```

Avviando nuovamente il test funzionale otterremo il seguente errore:

`AssertionError: 'Buy peacock feathers' unexpectedly found in 'Your To-Do list\n1: Buy peacock feathers'`


### Green? Refactor

Possiamo eliminare il metodo per il test unitario `test_displays_all_list_items`.


### Another Small Step: A Separate Template for Viewing Lists

Aggiungiamo il seguente metodo:

```py

def test_uses_list_template(self):
    response = self.client.get('/lists/the-only-list-in-the-world/')
    self.assertTemplateUsed(response, 'list.html')

```

alla classe `ListViewTest`, presente nel file `lists/tests.py`, in modo tale da testare l'effettivo utilizzo del _template_ `list.html`.

Modifichiamo anche il metodo `view_list(request)` in `lists/views`:

```py

def view_list(request):
    items = Item.objects.all()
    return render(request, 'list.html', {'items': items})

```
Facciamo una copia di `lists/templates/home.html` e rinominiamola `lists/templates/list.html`
Semplifichiamo il _body_ del file `lists/templates/home.html` in quanto abbiamo integrato alcune funzionalità nel _template_ `list.html`:

```html

<body>
    <h1>Start a new To-Do list</h1>
    <form method="POST" action="/">
        <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
        {% csrf_token %}
    </form>
</body>

```
e modifichiamo di conseguenza anche il metodo `home_page(request)` presente in `lists/views.py`:

```py

def home_page(request):
    if request.method == 'POST':
        Item.objects.create(text=request.POST['item_text'])
        return redirect('/lists/the-only-list-in-the-world/')
    return render(request, 'home.html')

```


### A Test Class for New List Creation

Spostiamo i metodi `test_can_save_a_POST_request` e `test_redirects_after_POST`, presenti in `lists/tests.py` in una nuova classe chiamata `NewListTest(TestCase)` e modifichiamoli nel seguente modo:

```py

class NewListTest(TestCase):
    def test_can_save_a_POST_request(self):
        self.client.post('/lists/new', data={'item_text': 'A new list item'})
        self.assertEqual(Item.objects.count(), 1)
        new_item = Item.objects.first()
        self.assertEqual(new_item.text, 'A new list item')


    def test_redirects_after_POST(self):
        response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
        self.assertRedirects(response, '/lists/the-only-list-in-the-world/')

```

Eseguendo il test di unità su `lists` otteniamo i seguenti errori:

```

[...]
self.assertEqual(Item.objects.count(), 1)
AssertionError: 0 != 1
[...]
AssertionError: 404 != 302 : Response didn't redirect as expected: Response code was 404 (expected 302)

```

Il primo errore è dovuto al fatto che non stiamo salvando il nuovo item all'interno del database. Il secondo, _errore 404_, invece si presenta dal momento che non abbiamo ancora creato un URL per `lists/new` Procediamo alla correzione del secondo errore modificando il file `superlists/urls.py`:

```py

urlpatterns = [
	url(r'^$', views.home_page, name='home'),
	url(r'^lists/new$', views.new_list, name='new_list'),
	url(r'^lists/the-only-list-in-the-world/$', views.view_list, name='view_list'),
]

```

e definiamo il metodo `new_list(request)` in `lists/views.py`:

```py

def new_list(request):
	Item.objects.create(text=request.POST['item_text'])
	return redirect('/lists/the-only-list-in-the-world/')

```

A questo punto il test d'unità dovrebbe passare, mentre quello di funzionalità dovrebbe restituire ancora due errori


### Removing Now-Redundant Code and Tests

Facciamo un po' di refactor in `lists/views.py`:

```py

def home_page(request):
	return render(request, 'home.html')

```

e in `lists/tests.py` rimuovendo il metodo `test_only_saves_​items_when_necessary`.


### A Regression! Pointing Our Forms at the New URL

Effettuiamo la seguente modifica sia in `lists/templates/home.html` che in `lists/templates/list.html`:

```html

<form method="POST" action="/lists/new">


### Biting the Bullet: Adjusting Our Models

Ora modifichiamo `lists/tests.py` come segue:

```py

from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest
from lists.views import home_page
from django.template.loader import render_to_string
from django.db import models
from lists.models import Item, List

class HomePageTest(TestCase):
	def test_root_url_resolves_to_home_page_view(self):
		found = resolve('/')
		self.assertEqual(found.func, home_page)

	def test_home_page_returns_correct_html(self):
		response = self.client.get('/')
		html = response.content.decode('utf8')
		self.assertTrue(html.startswith('<html>'))
		self.assertIn('<title>To-Do lists</title>', html)
		self.assertTrue(html.strip().endswith('</html>'))
		self.assertTemplateUsed(response, 'home.html')

	def test_uses_home_template(self):
		response = self.client.get('/')
		self.assertTemplateUsed(response, 'home.html')


class ListAndItemModelsTest(TestCase):
    
	def test_saving_and_retrieving_items(self):
		list_ = List()
		list_.save()

		first_item = Item()
		first_item.text = 'The first (ever) list item'
		first_item.list = list_
		first_item.save()

		second_item = Item()
		second_item.text = 'Item the second'
		second_item.list = list_
		second_item.save()

		saved_list = List.objects.first()
		self.assertEqual(saved_list, list_)

		saved_items = Item.objects.all()
		self.assertEqual(saved_items.count(), 2)

		first_saved_item = saved_items[0]
		second_saved_item = saved_items[1]
		self.assertEqual(first_saved_item.text, 'The first (ever) list item')
		self.assertEqual(first_saved_item.list, list_)
		self.assertEqual(second_saved_item.text, 'Item the second')
		self.assertEqual(second_saved_item.list, list_)


class ListViewTest(TestCase):

	def test_uses_list_template(self):
		response = self.client.get('/lists/the-only-list-in-the-world/')
		self.assertTemplateUsed(response, 'list.html')

	def test_displays_all_items(self):
		Item.objects.create(text='itemey 1')
		Item.objects.create(text='itemey 2')

		response = self.client.get('/lists/the-only-list-in-the-world/')

		self.assertContains(response, 'itemey 1')  
		self.assertContains(response, 'itemey 2')

class NewListTest(TestCase):
	def test_can_save_a_POST_request(self):
		self.client.post('/lists/new', data={'item_text': 'A new list item'})
		self.assertEqual(Item.objects.count(), 1)
		new_item = Item.objects.first()
		self.assertEqual(new_item.text, 'A new list item')


	def test_redirects_after_POST(self):
		response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
		self.assertRedirects(response, '/lists/the-only-list-in-the-world/')

```

Per correggere l'errore `ImportError: cannot import name 'List'` dobbiamo creare una nuova classe `List`

```py

class List(models.Model):
    pass

```

Eseguiamo:

`$ python manage.py makemigrations`

e rilanciamo il test di unità. Visualizzeremo il seguente errore:

```

self.assertEqual(first_saved_item.list, list_)
AttributeError: 'Item' object has no attribute 'list'

```


### A Foreign Key Relationship

Modifichiamo la classe `Item` in `lists/models.py`:

 ```py

class Item(models.Model):
	text = models.TextField(default='')
	list = models.ForeignKey(List, default=None, on_delete=models.DO_NOTHING)

```

Eseguire i seguenti comandi da riga di comando:

```

$ rm lists/migrations/0004_item_list.py
$ python manage.py makemigrations

```


### Adjusting the Rest of the World to Our New Models

Se ora eseguiamo il test di unità su `lists` otteniamo tre errori. Per risolvere il problema modifichiamo il file `lists/tests.py` in quanto abbiamo appena introdotto una nuova relazione fra gli oggetti di tipo `Item` e quelli di tipo `List` che richiede che ogni item abbia una lista genitore:

```py

class ListViewTest(TestCase):

	[...]

	def test_displays_all_items(self):
		list_ = List.objects.create()
		Item.objects.create(text='itemey 1', list=list_)
		Item.objects.create(text='itemey 2', list=list_)

```

Modifichiamo anche `lists/views.py`:

```py

from django.shortcuts import redirect, render
from lists.models import Item, List

# Create your views here.
def home_page(request):
	return render(request, 'home.html')

def view_list(request):
    items = Item.objects.all()
    return render(request, 'list.html', {'items': items})


def new_list(request):
	list_ = List.objects.create()	
	Item.objects.create(text=request.POST['item_text'], list=list_)
	return redirect('/lists/the-only-list-in-the-world/')

```

Se abbiamo fato tutto correttamente, ora il nostro test d'unità dovrebbe passare.

### Each List Should Have Its Own URL

[...]




