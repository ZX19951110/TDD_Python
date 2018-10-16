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
                <div class="col-md-6 col-md-offset-3">
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

[...]
