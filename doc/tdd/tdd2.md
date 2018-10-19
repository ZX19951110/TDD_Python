# Parte 2

## Chapter 7: Prettification: Layout and Styling, and What to Test About It

È arrivato il momento di abbellire la grafica e di testarla. Iniziamo a scrivere un nuovo metodo di test nella classe `NewVisitorTest` del file `functional_tests/tests.py`:

```py

def test_layout_and_styling(self):
	# Edith goes to the home page
	self.browser.get(self.live_server_url)
	self.browser.set_window_size(1024, 768)

	# She notices the input box is nicely centered
	inputbox = self.browser.find_element_by_id('id_new_item')
	self.assertAlmostEqual(
		inputbox.location['x'] + inputbox.size['width'] / 2,
		512,
		delta=10
	)

```

Se eseguiamo il test funzionale otteniamo un errore in quanto la nostra _textbox_ per l'inserimento di nuovi item non è centrata. Modifichiamo il test funzionale per controllare che la _textbox_ sia centrata anche per l'inserimento di nuove liste:

```py

def test_layout_and_styling(self):
	# Edith goes to the home page
	self.browser.get(self.live_server_url)
	self.browser.set_window_size(1024, 768)

	# She notices the input box is nicely centered
	inputbox = self.browser.find_element_by_id('id_new_item')
	self.assertAlmostEqual(
		inputbox.location['x'] + inputbox.size['width'] / 2,
		512,
		delta=10
	)

	# She starts a new list and sees the input is nicely
	# centered there too
    	inputbox.send_keys('testing')
	inputbox.send_keys(Keys.ENTER)
	self.wait_for_row_in_list_table('1: testing')
	inputbox = self.browser.find_element_by_id('id_new_item')
	self.assertAlmostEqual(
		inputbox.location['x'] + inputbox.size['width'] / 2,
		512,
		delta=10
	)
``` 

Eseguendo nuovamente il test funzionale otteniamo un altro errore.


### Prettification: Using a CSS Framework

Scarichiamo [bootstrap](http://getbootstrap.com/) e copiamo il contenuto dell archivio in `lists/static/bootstrap`


### Django Template Inheritance

Creiamo il file `lists/templates/base.html`:

```html

<html>
    <head>
        <title>To-Do lists</title>
    </head>

    <body>
        <h1>{% block header_text %}{% endblock %}</h1>
        <form method="POST" action="{% block form_action %}{% endblock %}">
            <input name="item_text" id="id_new_item" placeholder="Enter a to-do item" />
            {% csrf_token %}
        </form>
        {% block table %}
        {% endblock %}
    </body>
</html>

```

Modiichiamo anche i _template_ `home.html` e `list.html` affiché possano sfruttare i blocchi utilizzati in `base.html` in modo tale da evitare la ripetizione del codice.

`lists/templates/home.html`:

```html

{% extends 'base.html' %}

{% block header_text %}Start a new To-Do list{% endblock %}

{% block form_action %}/lists/new{% endblock %}

```

`lists/templates/list.html`:

```html

{% extends 'base.html' %}

{% block header_text %}Your To-Do list{% endblock %}

{% block form_action %}/lists/{{ list.id }}/add_item{% endblock %}

{% block table %}
  <table id="id_list_table">
    {% for item in list.item_set.all %}
      <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
    {% endfor %}
  </table>
{% endblock %}

```


### Integrating Bootstrap

Ora vediamo come integrare il CSS di Bootstrap nel nostro progetto. Modifichiamo il file `lists/templates/base.html`:

```html

<!DOCTYPE html>
<html lang="en">

    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>To-Do lists</title>
        <link href="css/bootstrap.min.css" rel="stylesheet">
    </head>

    

    <body>
        <div class="container">

            <div class="row">
                <div class="col-md-6 col-md-offset-3 style="margin-left:auto;margin-right:auto;"">
                    <div class="text-center">
                        <h1>{% block header_text %}{% endblock %}</h1>
                        <form method="POST" action="{% block form_action %}{% endblock %}">
                            <input name="item_text" id="id_new_item"
                                placeholder="Enter a to-do item" />
                            {% csrf_token %}
                        </form>
                    </div>
                </div>
            </div>

            <div class="row">
                <div class="col-md-6 col-md-offset-3">
                    {% block table %}
                    {% endblock %}
                </div>
            </div>

        </div>
    </body>

</html>

```


### Static Files in Django

Django ci permette di utilizzare un prefisso per segnalare che un URL che inizia con tale prefisso deve essere trattato come una richiesta per un file statico.
Al momento i link al CSS in `base.html` non funzionano in quanto non abbiamo utilizzato il prefisso `/static/` per l'URL. perciò modifichiamo l'`head` del file `lists/templates/base.html` come segue:

```html

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>To-Do lists</title>
    <link href="/static/bootstrap/css/bootstrap.min.css" rel="stylesheet">
</head>

```


### Switching to StaticLiveServerTestCase

Tuttavia, se eseguiamo il test funzionale, otteniamo un altro errore; ciò è dovuto al fatto che, a differenza del _runserver_, la classe `LiveServerTestCase` non è in grado di trovare automaticamente i file statici. Per risolvere il problema dobbiamo utilizzare la classe `StaticLiveServerTestCase`. Modifichiamo quindi il file `functional/tests/tests.py` come segue:

```py

from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
from selenium.common.exceptions import WebDriverException

MAX_WAIT = 10

class NewVisitorTest(StaticLiveServerTestCase):
	[...]

```


### Using Bootstrap Components to Improve the Look of the Site

Utilizziamo la classe `jumbotron` per migliorare l'estetica dei nostri template.
`lists/templates/base.html`:

```html

<div class="col-md-6 col-md-offset-3 jumbotron style="margin-left:auto;margin-right:auto;"">
    [...]
</div>

```

Utilizziamo la classe `form-control` di Bootstrap per allargare la _textbox_ per l'inserimento del testo:

```html

<input name="item_text" id="id_new_item"
    class="form-control input-lg"
    placeholder="Enter a to-do item" />

```

In `lists/templates/list.html` utilizziamo la classe `table` per migliorare il layout della lista delle cose da fare:

```html

<table id="id_list_table" class="table">
    {% for item in list.item_set.all %}
      <tr><td>{{ forloop.counter }}: {{ item.text }}</td></tr>
    {% endfor %}
</table>

```


### Using Our Own CSS

Aggiungiamo un offset al contenuto della textbox creando il file `lists/static/base.css`:

```css

#id_new_item {
	margin-top: 2ex;
}

```

e modificando il template `base.html`:

```html

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>To-Do lists</title>
    <link href="/static/css/bootstrap.min.css" rel="stylesheet">
    <link href="/static/base.css" rel="stylesheet">
</head>

```

Eseguendo il test funzionale dovremmo ottenere un successo.



## Chapter 9: Testing Deployment Using a Staging Site

Modifichiamo il metodo `SetUp` in `functional_tests/tests.py` affinchè possa verificare la presenza di una variabile d'ambiente chimata `STAGING_SERVER`:

```py

def setUp(self):
	self.browser = webdriver.Firefox()
	staging_server = os.environ.get('STAGING_SERVER')  
	if staging_server:
		self.live_server_url = 'http://' + staging_server

```

### Virtualenv

Utilizziamo _virtualenv_ per gestire i pacchetti e le dipendenze quando sul computer possono girare più applicazioni scritte in Python. Nel caso di Python 3, _virtualenv_ è gia installato. Aggiungiamo _virtualenv_ al nostro progetto usando il seguente comando:

`$ python -m venv virtualenv`

A questo punto vedremo i file di _virtualenv_ all'interno della directory `./virtualenv`

Per attivare _virtualenv_:

`$ source virtualenv/bin/activate`

Per disattivarlo:

`$ deactivate`

Inoltre ricordiamoci di installre _Django_ e _Selenium_ in _virtualenv_. Per eseguire il nostro programma in _virtualenv_ dobbiamo utilizzare il comando:

`$ ./virtualenv/bin/python manage.py runserver`

Eseguendo il test d'unità ci accorgiamo di un errore causato dall'istruzione:

`self.assertTrue(html.startswith('<html>'))`

Perciò modifichiamola in:

`self.assertTrue(html.startswith('<!DOCTYPE html>'))`

Ora il nostro test d'unità dovrebbe passare correttamente.



## Chapter 12: Splitting Our Tests into Multiple Files, and a Generic Wait Helper

Ora modifichiamo il nostro test funzionale in modo tale che possa verificare che la nostra _textbox_ non accetti in input stringhe vuote. Quindi aggiungiamo il seguente metodo:

```py

def test_cannot_add_empty_list_items(self):
	self.fail('write me!')

```

### Skipping a test

Utilizzando `@skip`possiamo saltare un determinato metodo di test:

```py

from unittest import skip

@skip
def test_cannot_add_empty_list_items(self):
	self.fail('write me!')

```


### Splitting Functional Tests Out into Many Files

Poniamo un po' d'ordine nel file `functional_tests/tests.py`:

```py

from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
import time
from selenium.common.exceptions import WebDriverException
import os
from unittest import skip

MAX_WAIT = 10

class FunctionalTest(StaticLiveServerTestCase):
	
	def setUp(self):
		self.browser = webdriver.Firefox()
		staging_server = os.environ.get('STAGING_SERVER')  
		if staging_server:
			self.live_server_url = 'http://' + staging_server

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


class NewVisitorTest(FunctionalTest):

	def test_can_start_a_list_for_one_user(self):
		self.browser.get(self.live_server_url)		
		inputbox = self.browser.find_element_by_id('id_new_item')
		inputbox.send_keys('Buy peacock feathers')		
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


class LayoutAndStylingTest(FunctionalTest):
	
	def test_layout_and_styling(self):
		# Edith goes to the home page
		self.browser.get(self.live_server_url)
		self.browser.set_window_size(1024, 768)

		# She notices the input box is nicely centered
		inputbox = self.browser.find_element_by_id('id_new_item')
		self.assertAlmostEqual(
			inputbox.location['x'] + inputbox.size['width'] / 2,
			512,
			delta=10
		)

		# She starts a new list and sees the input is nicely
		# centered there too
		inputbox.send_keys('testing')
		inputbox.send_keys(Keys.ENTER)
		self.wait_for_row_in_list_table('1: testing')
		inputbox = self.browser.find_element_by_id('id_new_item')
		self.assertAlmostEqual(
			inputbox.location['x'] + inputbox.size['width'] / 2,
			512,
			delta=10
		)


class ItemValidationTest(FunctionalTest):

	@skip
	def test_cannot_add_empty_list_items(self):
		self.fail('write me!')

```

ed eseguiamo nuovamente il test funzionale per assicurarci di aver fatto tutto correttamente.

Ora dividiamo il file relativo al file funzionale in più file:

`functional_tests/base.py`:

```py

import os
from django.contrib.staticfiles.testing import StaticLiveServerTestCase
from selenium import webdriver
from selenium.common.exceptions import WebDriverException
import time

MAX_WAIT = 10


class FunctionalTest(StaticLiveServerTestCase):
	
	def setUp(self):
		self.browser = webdriver.Firefox()
		staging_server = os.environ.get('STAGING_SERVER')  
		if staging_server:
			self.live_server_url = 'http://' + staging_server

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

```

`functional_tests/test_simple_list_creation.py`:

```py

from .base import FunctionalTest
from selenium import webdriver
from selenium.webdriver.common.keys import Keys


class NewVisitorTest(FunctionalTest):

	def test_can_start_a_list_for_one_user(self):
		self.browser.get(self.live_server_url)		
		inputbox = self.browser.find_element_by_id('id_new_item')
		inputbox.send_keys('Buy peacock feathers')		
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

`functional_tests/test_layout_and_styling.py`:

```py

from selenium.webdriver.common.keys import Keys
from .base import FunctionalTest


class LayoutAndStylingTest(FunctionalTest):
	
	def test_layout_and_styling(self):
		# Edith goes to the home page
		self.browser.get(self.live_server_url)
		self.browser.set_window_size(1024, 768)

		# She notices the input box is nicely centered
		inputbox = self.browser.find_element_by_id('id_new_item')
		self.assertAlmostEqual(
			inputbox.location['x'] + inputbox.size['width'] / 2,
			512,
			delta=10
		)

		# She starts a new list and sees the input is nicely
		# centered there too
		inputbox.send_keys('testing')
		inputbox.send_keys(Keys.ENTER)
		self.wait_for_row_in_list_table('1: testing')
		inputbox = self.browser.find_element_by_id('id_new_item')
		self.assertAlmostEqual(
			inputbox.location['x'] + inputbox.size['width'] / 2,
			512,
			delta=10
		)

```

`functional_tests/test_list_item_validation.py`:

```py

from selenium.webdriver.common.keys import Keys
from unittest import skip
from .base import FunctionalTest


class ItemValidationTest(FunctionalTest):

	@skip
	def test_cannot_add_empty_list_items(self):
		self.fail('write me!')

```


### A New Functional Test Tool: A Generic Explicit Wait Helper

Ora scriviamo il metodo di test di cui abbiamo parlato ad inizio capitolo:

```py

class ItemValidationTest(FunctionalTest):

	def test_cannot_add_empty_list_items(self):
		# Edith goes to the home page and accidentally tries to submit
		# an empty list item. She hits Enter on the empty input box
		self.browser.get(self.live_server_url)
		self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)

		# The home page refreshes, and there is an error message saying
		# that list items cannot be blank
		self.wait_for(lambda: self.assertEqual(  
			self.browser.find_element_by_css_selector('.has-error').text,
			"You can't have an empty list item"
		))

		# She tries again with some text for the item, which now works
		self.fail('write me!')

```

Utilizzando l'istruzione

```py

self.wait_for(lambda: self.assertEqual(  
	self.browser.find_element_by_css_selector('.has-error').text,
	"You can't have an empty list item"
))

```

il programma rimane in attesa fino a che l'asserzione passa.
Ora dobbiamo aggiungere il metodo `wait_for` alla nostra classe:

```py

def wait_for(self, fn):
		start_time = time.time()
		while True:
			try:
				return fn()  
			except (AssertionError, WebDriverException) as e:
				if time.time() - start_time > MAX_WAIT:
					raise e
				time.sleep(0.5)

```

Eseguendo il test funzionale sul file appena modificato utilizzando il comando:

`$ python manage.py test functional_tests.test_list_item_validation`

otteniamo il seguente errore:

`ERROR: test_cannot_add_empty_list_items (functional_tests.test_list_item_validation.ItemValidationTest)`

### Finishing Off the FT

Completiamo il test funzionale `test_list_item_validation.py`:

```py

def test_cannot_add_empty_list_items(self):
		# Edith goes to the home page and accidentally tries to submit
		# an empty list item. She hits Enter on the empty input box
		self.browser.get(self.live_server_url)
		self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)

		# The home page refreshes, and there is an error message saying
		# that list items cannot be blank
		self.wait_for(lambda: self.assertEqual(  
			self.browser.find_element_by_css_selector('.has-error').text,
			"You can't have an empty list item"
		))

		# She tries again with some text for the item, which now works
		self.browser.find_element_by_id('id_new_item').send_keys('Buy milk')
		self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)
		self.wait_for_row_in_list_table('1: Buy milk')

		# Perversely, she now decides to submit a second blank list item
		self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)

		# She receives a similar warning on the list page
		self.wait_for(lambda: self.assertEqual(
		self.browser.find_element_by_css_selector('.has-error').text,
			"You can't have an empty list item"
		))

		# And she can correct it by filling some text in
		self.browser.find_element_by_id('id_new_item').send_keys('Make tea')
		self.browser.find_element_by_id('id_new_item').send_keys(Keys.ENTER)
		self.wait_for_row_in_list_table('1: Buy milk')
		self.wait_for_row_in_list_table('2: Make tea')

```

Tuttavia il nostro test funzionale fallirà nuovamente.


### Refactoring Unit Tests into Several Files

Facciamo un po' di refactoring al nostro test d'unità:

```

$ mkdir lists/tests
$ touch lists/tests/__init__.py
$ git mv lists/tests.py lists/tests/test_all.py
$ mv lists/tests/test_all.py lists/tests/test_views.py
$ cp lists/tests/test_views.py lists/tests/test_models.py

```
Modifichiamo il file `lists/tests/test_models.py`:

```py

from django.test import TestCase
from lists.models import Item, List



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

```

e poi modifichiamo anche `lists/tests/test_views.py`:

```py

from django.urls import resolve
from django.test import TestCase
from django.http import HttpRequest
from lists.views import home_page
from django.template.loader import render_to_string
from django.db import models	#utile?
from lists.models import Item, List

class HomePageTest(TestCase):
	def test_root_url_resolves_to_home_page_view(self):
		found = resolve('/')
		self.assertEqual(found.func, home_page)

	def test_home_page_returns_correct_html(self):
		response = self.client.get('/')
		html = response.content.decode('utf8')
		self.assertTrue(html.startswith('<!DOCTYPE html>'))
		self.assertIn('<title>To-Do lists</title>', html)
		self.assertTrue(html.strip().endswith('</html>'))
		self.assertTemplateUsed(response, 'home.html')

	def test_uses_home_template(self):
		response = self.client.get('/')
		self.assertTemplateUsed(response, 'home.html')


class ListViewTest(TestCase):

	def test_uses_list_template(self):
		list_ = List.objects.create()
		response = self.client.get(f'/lists/{list_.id}/')
		self.assertTemplateUsed(response, 'list.html')

	def test_displays_only_items_for_that_list(self):
		correct_list = List.objects.create()
		Item.objects.create(text='itemey 1', list=correct_list)
		Item.objects.create(text='itemey 2', list=correct_list)
		other_list = List.objects.create()
		Item.objects.create(text='other list item 1', list=other_list)
		Item.objects.create(text='other list item 2', list=other_list)

		response = self.client.get(f'/lists/{correct_list.id}/')

		self.assertContains(response, 'itemey 1')
		self.assertContains(response, 'itemey 2')
		self.assertNotContains(response, 'other list item 1')
		self.assertNotContains(response, 'other list item 2')

class NewListTest(TestCase):
	def test_can_save_a_POST_request(self):
		self.client.post('/lists/new', data={'item_text': 'A new list item'})
		self.assertEqual(Item.objects.count(), 1)
		new_item = Item.objects.first()
		self.assertEqual(new_item.text, 'A new list item')


	def test_redirects_after_POST(self):
		response = self.client.post('/lists/new', data={'item_text': 'A new list item'})
		new_list = List.objects.first()
		self.assertRedirects(response, f'/lists/{new_list.id}/')

	def test_passes_correct_list_to_template(self):
		other_list = List.objects.create()
		correct_list = List.objects.create()
		response = self.client.get(f'/lists/{correct_list.id}/')
		self.assertEqual(response.context['list'], correct_list)

class NewItemTest(TestCase):

	def test_can_save_a_POST_request_to_an_existing_list(self):
		other_list = List.objects.create()
		correct_list = List.objects.create()

		self.client.post(
		f'/lists/{correct_list.id}/add_item',
			data={'item_text': 'A new item for an existing list'}
		)

		self.assertEqual(Item.objects.count(), 1)
		new_item = Item.objects.first()
		self.assertEqual(new_item.text, 'A new item for an existing list')
		self.assertEqual(new_item.list, correct_list)

	def test_redirects_to_list_view(self):
		other_list = List.objects.create()
		correct_list = List.objects.create()

		response = self.client.post(
			f'/lists/{correct_list.id}/add_item',
			data={'item_text': 'A new item for an existing list'}
		)

		self.assertRedirects(response, f'/lists/{correct_list.id}/')

```

Se abbiamo fatto tutto correttamente il nostro test d'unità dovrebbe passare correttamente.



## Chapter 13: Validation at the Database Layer

[...]

