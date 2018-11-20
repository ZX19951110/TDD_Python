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
	email = models.EmailField(primary_key=True)
	
	REQUIRED_FIELDS = []
	USERNAME_FIELD = 'email'
	is_anonymous = False
	is_authenticated = True

```

ed eseguiamo il comando:

`$ python manage.py makemigrations`

A questo punto il test d'unità per `accounts` dovrebbe passare correttamente!


### Tests as Documentation

Scriviamo un nuovo test in `accounts/tests/test_models.py`:

```py

def test_email_is_primary_key(self):
	user = User(email='a@b.com')
	self.assertEqual(user.pk, 'a@b.com')

```

Tale test ci permette di evidenziare che user non è al momento chiave primaria; perciò, in `accounts/models.py` effettuiamo la seguente modifica:

```py

class User(models.Model):
	email = models.EmailField(primary_key=True)
	
	REQUIRED_FIELDS = []
	USERNAME_FIELD = 'email'
	is_anonymous = False
	is_authenticated = True

```

cancelliamo l'ultima `migrations`:

`rm accounts/migrations/0001_initial.py`

e rilanciamo il comando:

`python manage.py makemigrations`

Ora il nostro test d'unità dovrebbe passare correttamente.


### A Token Model to Link Emails with a Unique ID

In `accounts/tests/test_models.py` creiamo una nuova classe di test per il modello dei _token_:

```py

from django.test import TestCase
from django.contrib.auth import get_user_model
from accounts.models import Token

User = get_user_model()


class UserModelTest(TestCase):
	[...]


class TokenModelTest(TestCase):

	def test_links_user_with_auto_generated_uid(self):
		token1 = Token.objects.create(email='a@b.com')
		token2 = Token.objects.create(email='a@b.com')
		self.assertNotEqual(token1.uid, token2.uid)

```

Creiamo la classe `Token` in `accounts/models.py`:

```py

from django.db import models
import uuid



class User(models.Model):
	[...]


class Token(models.Model):
	email = models.EmailField()
	uid = models.CharField(default=uuid.uuid4, max_length=40)

```

ed eseguiamo:

`$ python manage.py makemigrations`
`$ python manage.py migrate`

Abbiamo importato il modulo `uuid` al fine di generare codici univoci con una lunghezza prefissata.

Ora il nostro test d'unità dovrebbe passare correttamente.



## Chapter 19: Using Mocks to Test External Dependencies or Reduce Duplication

### Before We Start: Getting the Basic Plumbing In

Scriviamo un test in `accounts/tests/test_views.py` per testare che il nuovo URL per mandare la mail di login ci reindirizzi alla _home page_:

```py

from django.test import TestCase


class SendLoginEmailViewTest(TestCase):

	def test_redirects_to_home_page(self):
		response = self.client.post('/accounts/send_login_email', data={
			'email': 'edith@example.com'
		})
		self.assertRedirects(response, '/')

```

e creiamo la nostra _view_ in `accounts/views.py`:

```py

from django.shortcuts import render
from django.core.mail import send_mail
from django.shortcuts import redirect



def send_login_email(request):
	return redirect('/')

```

Modifichiamo `superlists/urls.py`:

```py

from django.conf.urls import include, url
from accounts import urls as accounts_urls
from lists import views as list_views
from lists import urls as list_urls



urlpatterns = [
	url(r'^$', list_views.home_page, name='home'),
	url(r'^lists/', include(list_urls)),
	url(r'^accounts/', include(accounts_urls)),
]

```

e `accounts/urls.py`:

```py

from django.conf.urls import url
from accounts import views



urlpatterns = [
	url(r'^send_login_email$', views.send_login_email, name='send_login_email'),
]

```

Ora il test d'unità di `accounts` dovrebbe passare correttamente.


### Mocking Manually, aka Monkeypatching

Utilizziamo il _mockup_ in quanto vogliamo testare il funzionamento della chiamata alla funzione `send_mail` senza chiamare effettivamente tale metodo. Per questo motivo aggiungiamo dei nuovi metodi nel file `accounts/tests/test_views.py`:

```py

from django.test import TestCase
import accounts.views


class SendLoginEmailViewTest(TestCase):

	def test_redirects_to_home_page(self):
		[...]

	
	def test_sends_mail_to_address_from_post(self):
		self.send_mail_called = False


	def fake_send_mail(subject, body, from_email, to_list):  
		self.send_mail_called = True
		self.subject = subject
		self.body = body
		self.from_email = from_email
		self.to_list = to_list

		accounts.views.send_mail = fake_send_mail  

		self.client.post('/accounts/send_login_email', data={
			'email': 'edith@example.com'
		})


		self.assertTrue(self.send_mail_called)
		self.assertEqual(self.subject, 'Your login link for Superlists')
		self.assertEqual(self.from_email, 'noreply@superlists')
		self.assertEqual(self.to_list, ['edith@example.com'])

```

In `accounts/views.py` modifichiamo il metodo `send_login_email`:

```py

def send_login_email(request):
	email = request.POST['email']
	send_mail(
		'Your login link for Superlists',
		'body text tbc',
		'noreply@superlists',
		[email]
	)
	return redirect('/')

```


### Using unittest.patch

Modifichiamo la funzione `test_sends_mail_to_address_from_post` in `accounts/tests/test_views.py`:

```py

from django.test import TestCase
from unittest.mock import patch
import accounts.views


class SendLoginEmailViewTest(TestCase):

	def test_redirects_to_home_page(self):
		[...]

	
	@patch('accounts.views.send_mail')
	def test_sends_mail_to_address_from_post(self, mock_send_mail):
		self.client.post('/accounts/send_login_email', data={
			'email': 'edith@example.com'
		})

		self.assertEqual(mock_send_mail.called, True)
		(subject, body, from_email, to_list), kwargs = mock_send_mail.call_args
		self.assertEqual(subject, 'Your login link for Superlists')
		self.assertEqual(from_email, 'noreply@superlists')
		self.assertEqual(to_list, ['edith@example.com'])


	def fake_send_mail(subject, body, from_email, to_list):  
		[...]

```


### Getting the FT a Little Further Along

Modifichiamo la `navbar`in `lists/templates/base.html` in modo tale che possa inviare l'input alla nosta _view_:

```html

<form class="navbar-form navbar-right"
    method="POST"
    action="{% url 'send_login_email' %}">
    <span>Enter email to log in:</span>
    <input class="form-control" name="email" type="text" />
    {% csrf_token %}
</form>

```


### Testing the Django Messages Framework

Scriviamo un test per controllare che la nostra applicazione notifichi l'invio della mail con il link per l'autenticazione:

```py

def test_adds_success_message(self):
	response = self.client.post('/accounts/send_login_email', data={
		'email': 'edith@example.com'
	}, follow=True)

	message = list(response.context['messages'])[0]
	self.assertEqual(
		message.message,
		"Check your email, we've sent you a link you can use to log in."
	)
	self.assertEqual(message.tags, "success")

```

e modifichiamo il metodo `send_login_email` in `accounts/views.py`:

```py

from django.core.mail import send_mail
from django.shortcuts import redirect
from django.contrib import messages



def send_login_email(request):
	email = request.POST['email']
	send_mail(
		'Your login link for Superlists',
		'body text tbc',
		'noreply@superlists',
		[email]
	)
	
	messages.success(
		request,
		"Check your email, we've sent you a link you can use to log in."
	)

	return redirect('/')

```

A questo punto il test d'unità dovrebbe terminare con successo.


### Adding Messages to Our HTML

Aggiungiamo il messaggio precedentemente discusso in `lists/templates/base.html `:

```html

[...]
</nav>

{% if messages %}
    <div class="row">
        <div class="col-md-8">
            {% for message in messages %}
                {% if message.level_tag == 'success' %}
                    <div class="alert alert-success">{{ message }}</div>
                {% else %}
                    <div class="alert alert-warning">{{ message }}</div>
                {% endif %}
            {% endfor %}
        </div>
    </div>
{% endif %}

```

Il test d'unità otterrà un successo.

Ora modifichiamo il metodo `send_login_mail` in `accounts/views.py`:

```py

def send_login_email(request):
	email = request.POST['email']
	send_mail(
		'Your login link for Superlists',
		'Use this link to log in',
		'noreply@superlists',
		[email]
	)
	
	messages.success(
		request,
		"Check your email, we've sent you a link you can use to log in."
	)

	return redirect('/')

```

Tuttavia il nostro test funzionale ci mostrerà ancora un errore.


### Starting on the Login URL

In `accounts/tests/test_views.py ` creiamo una nuova classe di test per la _view_:

```py

class LoginViewTest(TestCase):

	def test_redirects_to_home_page(self):
		response = self.client.get('/accounts/login?token=abcd123')
		self.assertRedirects(response, '/')

```

Modifichiamo il file `accounts/urls.py`:

```py

urlpatterns = [
	url(r'^send_login_email$', views.send_login_email, name='send_login_email'),
	url(r'^login$', views.login, name='login'),
]

```

In `accounts/views.py` definiamo il metodo `login`:

```py

def login(request):
	#incomplete
	return redirect('/')

```

in modo tale da far passare il test funzionale che abbiamo appena scritto.


### Checking That We Send the User a Link with a Token

Scriviamo due nuovi metodi di test in `accounts/tests/test_views.py`:

```py

from django.test import TestCase
from unittest.mock import patch
import accounts.views
from accounts.models import Token


class SendLoginEmailViewTest(TestCase):

	[...]

	def test_creates_token_associated_with_email(self):
		self.client.post('/accounts/send_login_email', data={
			'email': 'edith@example.com'
		})
	  	token = Token.objects.first()
		self.assertEqual(token.email, 'edith@example.com')


	@patch('accounts.views.send_mail')
	def test_sends_link_to_login_using_token_uid(self, mock_send_mail):
		self.client.post('/accounts/send_login_email', data={
			'email': 'edith@example.com'
		})

		token = Token.objects.first()
		expected_url = f'http://testserver/accounts/login?token={token.uid}'
		(subject, body, from_email, to_list), kwargs = mock_send_mail.call_args
		self.assertIn(expected_url, body)

[...]

```

Modifichiamo il file `accounts/views.py` i due test:

```py

from django.core.mail import send_mail
from django.shortcuts import redirect
from django.contrib import messages
from accounts.models import Token
from django.urls import reverse



def send_login_email(request):
	email = request.POST['email']
	token = Token.objects.create(email=email)
	url = request.build_absolute_uri(  
		reverse('login') + '?token=' + str(token.uid)
	)
	message_body = f'Use this link to log in:\n\n{url}'
	send_mail(
		'Your login link for Superlists',
		message_body,
		'noreply@superlists',
		[email]
	)
	
	messages.success(
		request,
		"Check your email, we've sent you a link you can use to log in."
	)

	return redirect('/')

[...]

```


### De-spiking Our Custom Authentication Backend

Creiamo un nuovo file `accounts/tests/test_authentication.py` che servirà per testare il file `accounts/authentication.py` che andremo a creare successivamente:

```py

from django.test import TestCase
from django.contrib.auth import get_user_model
from accounts.authentication import PasswordlessAuthenticationBackend
from accounts.models import Token
User = get_user_model()



class AuthenticateTest(TestCase):

	def test_returns_None_if_no_such_token(self):
		result = PasswordlessAuthenticationBackend().authenticate(
			'no-such-token'
		)
		self.assertIsNone(result)


	def test_returns_new_user_with_correct_email_if_token_exists(self):
		email = 'edith@example.com'
		token = Token.objects.create(email=email)
		user = PasswordlessAuthenticationBackend().authenticate(token.uid)
		new_user = User.objects.get(email=email)
		self.assertEqual(user, new_user)


	def test_returns_existing_user_with_correct_email_if_token_exists(self):
		email = 'edith@example.com'
		existing_user = User.objects.create(email=email)
		token = Token.objects.create(email=email)
		user = PasswordlessAuthenticationBackend().authenticate(token.uid)
		self.assertEqual(user, existing_user)

```

In `accounts/authentication.py`:

```py

from accounts.models import User, Token



class PasswordlessAuthenticationBackend(object):

	def authenticate(self, uid):
		try:
			token = Token.objects.get(uid=uid)
			return User.objects.get(email=token.email)
		except User.DoesNotExist:
			return User.objects.create(email=token.email)
		except Token.DoesNotExist:
			return None

```

A questo punto il nostro test d'unità dovrebbe passare correttamente!


### The get_user Method

È il momento di creare una nuova classe in `accounts/tests/test_authentication.py` per testare il metodo `get_user`:

```py

class GetUserTest(TestCase):

	def test_gets_user_by_email(self):
		User.objects.create(email='another@example.com')
		desired_user = User.objects.create(email='edith@example.com')
		found_user = PasswordlessAuthenticationBackend().get_user(
			'edith@example.com'
		)
		self.assertEqual(found_user, desired_user)


	def test_returns_None_if_no_user_with_that_email(self):
		self.assertIsNone(
			PasswordlessAuthenticationBackend().get_user('edith@example.com')
		)

```

Creiamo il metodo `get_user` nella classe `PasswordlessAuthenticationBackend` del file `accounts/authentication.py`:

```py

def get_user(self, email):
	try:
		return User.objects.get(email=email)
	except User.DoesNotExist:
		return None

```

in modo tale da far passare correttamente il nostro test d'unità.


### Using Our Auth Backend in the Login View

Modifichiamo il file `superlists/settings.py` in modo tale da utilizzare l'autenticazione _backend_ che abbiamo appena sviluppato nella nostra _view_ di login:

```py

AUTH_USER_MODEL = 'accounts.User'
AUTHENTICATION_BACKENDS = [
	'accounts.authentication.PasswordlessAuthenticationBackend',
]

[...]

```

### An Alternative Reason to Use Mocks: Reducing Duplication

Modifichiamo il file `accounts/tests/test_views.py`:

```py

from django.test import TestCase
from unittest.mock import patch
import accounts.views
from accounts.models import Token
from unittest.mock import patch, call


class SendLoginEmailViewTest(TestCase):
	[...]



class LoginViewTest(TestCase):

	def test_redirects_to_home_page(self):
		response = self.client.get('/accounts/login?token=abcd123')
		self.assertRedirects(response, '/')


	@patch('accounts.views.auth')  
	def test_calls_authenticate_with_uid_from_get_request(self, mock_auth):  
		self.client.get('/accounts/login?token=abcd123')
		self.assertEqual(
			mock_auth.authenticate.call_args,  
			call(uid='abcd123')  
		)

```

In questo test stiamo facendo il _mocking_ di un intero modulo.
Infine completiamo il metodo `login` in `accounts/views.py`:

```py

from django.core.mail import send_mail
from django.shortcuts import redirect
from django.contrib import messages
from accounts.models import Token
from django.urls import reverse
from django.contrib import auth, messages



def send_login_email(request):
	[...]


def login(request):
	auth.authenticate(uid=request.GET.get('token'))
	return redirect('/')

```

Ora il nostro test funzionale passerà senza alcun problema!


### Using mock.return_value

Ora creiamo un nuovo metodo in `accounts/tests/test_views.py` per testare che se la funzione di autenticazione ci restituisce un utente, essa lo passi come parametro al metodo `auth.login`:

```py

class LoginViewTest(TestCase):

	[...]


	@patch('accounts.views.auth')  
	def test_calls_auth_login_with_user_if_there_is_one(self, mock_auth):
		response = self.client.get('/accounts/login?token=abcd123')
		self.assertEqual(
			mock_auth.login.call_args,  
			call(response.wsgi_request, mock_auth.authenticate.return_value)  
		)

```


### Patching at the Class Level

In `accounts/tests/test_views.py` spostiamo il _tag_ `@patch` dal livello metodo al livello classe (prestando attenzione a modificare i parametri passati al metodo `test_redirects_to_home_page`) e aggiungiamo un nuovo metodo in tale classe:

```py

@patch('accounts.views.auth')  
class LoginViewTest(TestCase):

	def test_redirects_to_home_page(self, mock_auth):
		[...]


	def test_calls_authenticate_with_uid_from_get_request(self, mock_auth):  
		[...]


	def test_calls_auth_login_with_user_if_there_is_one(self, mock_auth):
		[...]


	def test_does_not_login_if_user_is_not_authenticated(self, mock_auth):
		mock_auth.authenticate.return_value = None  
		self.client.get('/accounts/login?token=abcd123')
		self.assertEqual(mock_auth.login.called, False)

```

Ora completiamo definitivamente la funzione di login per far passare il test d'unità:

```py



```


### The Moment of Truth: Will the FT Pass?

Modifichiamo il file `lists/templates/base.html` affinché la _nav bar_ mostri il pulsante di log out nel caso in cui l'utente sia già loggato:

```html



```
