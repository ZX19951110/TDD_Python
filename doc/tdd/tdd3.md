# Part 3: More Advanced Topics

## Chapter 18: User Authentication, Spiking and De-Spiking

Scriviamo un metodo per testare il login in `functional_tests/test_login.py`:

```py

from django.core import mail
from selenium.webdriver.common.keys import Keys
import re

from .base import FunctionalTest

TEST_EMAIL = 'edith@example.com'
SUBJECT = 'Your login link for Superlists'



class LoginTest(FunctionalTest):

	def test_can_get_email_link_to_log_in(self):
		# Edith goes to the awesome superlists site
		# and notices a "Log in" section in the navbar for the first time
		# It's telling her to enter her email address, so she does
		self.browser.get(self.live_server_url)
		self.browser.find_element_by_name('email').send_keys(TEST_EMAIL)
		self.browser.find_element_by_name('email').send_keys(Keys.ENTER)

		# A message appears telling her an email has been sent
		self.wait_for(lambda: self.assertIn(
			'Check your email',
			self.browser.find_element_by_tag_name('body').text
		))

		# She checks her email and finds a message
		email = mail.outbox[0]  
		self.assertIn(TEST_EMAIL, email.to)
		self.assertEqual(email.subject, SUBJECT)

		# It has a url link in it
		self.assertIn('Use this link to log in', email.body)
		url_search = re.search(r'http://.+/.+$', email.body)
		if not url_search:
			self.fail(f'Could not find url in email body:\n{email.body}')
		url = url_search.group(0)
		self.assertIn(self.live_server_url, url)

		# she clicks it
		self.browser.get(url)

		# she is logged in!
		self.wait_for(
			lambda: self.browser.find_element_by_link_text('Log out')
        	)
		navbar = self.browser.find_element_by_css_selector('.navbar')
		self.assertIn(TEST_EMAIL, navbar.text)

```

Modifichiamo il file `lists/templates/base.html` in modo tale da inserire una barra di navigazione:

```html

<div class="container">

    <nav class="navbar navbar-default" role="navigation">
        <div class="container-fluid">
            <a class="navbar-brand" href="/">Superlists</a>
            <form class="navbar-form navbar-right" method="POST" action="#">
                <span>Enter email to log in:</span>
                <input class="form-control" name="email" type="text" />
                {% csrf_token %}
            </form>
        </div>
    </nav>

    <div class="row">

```

Creiamo una nuova app `accounts` contenente tutti i file relativi alla funzione di login:

`$ python manage.py startapp accounts`


### A Minimal Custom User Model

Eliminiamo `accounts/tests.py` e creiamo la directory `accounts/tests` con all'interno i file `__init__.py` e `test_models.py`. Scriviamo una nuova classe in quest'ultimo:

```py

from django.test import TestCase
from django.contrib.auth import get_user_model

User = get_user_model()


class UserModelTest(TestCase):

	def test_user_is_valid_with_email_only(self):
		user = User(email='a@b.com')
		user.full_clean()  # should not raise


```

Creiamo un modello per l'utente in `accounts/models.py`:

```py

from django.db import models



class User(models.Model):
	email = models.EmailField()

```

e in `superlists/settings.py` aggiungiamo `accounts` in `INSTALLED_APPS` e una nuova variabile chiamata `AUTH_USER_MODEL`:

```py

INSTALLED_APPS = [
    #'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'lists',
    'accounts',
]

AUTH_USER_MODEL = 'accounts.User'

```

Eseguendo il test:

`$ python manage.py test accounts/`

otteniamo un errore generato dal database; perciò modifichiamo la classe `User` in `accounts/models.py`:

```py

class User(models.Model):
	email = models.EmailField(unique=True)
	
	REQUIRED_FIELDS = []
	USERNAME_FIELD = 'email'
	is_anonymous = False
	is_authenticated = True

```

ed eseguiamo il comando:

`$ python manage.py makemigrations`

A questo punto il test funzionale per `accounts` dovrebbe passare correttamente!
