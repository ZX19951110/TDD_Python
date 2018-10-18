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

[...]






