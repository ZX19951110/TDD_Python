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


