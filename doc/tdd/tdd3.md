# Part 3: More Advanced Topics

## Chapter 18: User Authentication, Spiking and De-Spiking

### Starting a Branch for the Spike

Creiamo un nuovo branch su GitHub per sviluppare una nuova funzionalità per la nostra _To-do List_:

`$ git checkout -b passwordless-spike`


### Frontend Log in UI

Modifichiamo il file `lists/templates/base.html` in modo tale che sia possibile inserire il proprio indirizzo email per autenticarsi e che sia possibile cliccare su un link per effettuare il logout:

```html

<body>
    <div class="container">

        <div class="navbar">
            {% if user.is_authenticated %}
                <p>Logged in as {{ user.email }}</p>
                <p><a id="id_logout" href="{% url 'logout' %}">Log out</a></p>
            {% else %}
                 <form method="POST" action ="{% url 'send_login_email' %}">
                     Enter email to log in: <input name="email" type="text" />
                     {% csrf_token %}
                 </form>
            {% endif %}
        </div>

        <div class="row">
        [...]

```


### Sending Emails from Django

Ecco come funziona il nostro metodo di login:
* Quando un utente si vuole loggare, la nostra web app dovrà generare un _token_ segreto ed univoco che dovrà essere salvato nel database ed inviato all'utente per email.
* L'utente cliccherà sul link ricevuto per email.
* Quando l'utente cliccherà sul link, la web app dovrà controllare se il token esiste nel database e, in caso positivo, effettuare il login per l'utente.

Creiamo un'app per permettere l'autenticazione:

`$ python manage.py startapp accounts`

Modifichiamo il file `superlists/urls.py`:

```py

from django.conf.urls import include, url
from lists import views as list_views
from lists import urls as list_urls
from accounts import urls as accounts_urls

urlpatterns = [
	url(r'^$', list_views.home_page, name='home'),
	url(r'^lists/', include(list_urls)),
	url(r'^accounts/', include(accounts_urls)),
]

```

e creiamo il file `accounts/urls.py`:

```py

from django.conf.urls import url
from accounts import views

urlpatterns = [
	url(r'^send_email$', views.send_login_email, name='send_login_email'),
]

```

Ora scriviamo in `accounts/views.py` il metodo che ci permetterà di creare un _token_ associato all'indirizzo email inserito dall'utente in fase di login:

```py

import uuid
import sys
from django.shortcuts import render
from django.core.mail import send_mail

from accounts.models import Token



def send_login_email(request):
	email = request.POST['email']
	uid = str(uuid.uuid4())
	Token.objects.create(email=email, uid=uid)
	print('saving uid', uid, 'for email', email, file=sys.stderr)
	url = request.build_absolute_uri(f'/accounts/login?uid={uid}')
	send_mail(
		'Your login link for Superlists',
		f'Use this link to log in:\n\n{url}',
		'noreply@superlists',
		[email],
	)
	return render(request, 'login_email_sent.html')

```

Creiamo un nuovo _template_ in `accounts/templates/login_email_sent.html` per avvisare l'utente che gli è stata appena inviata una mail con il link da utilizzare per il login:

```html

[...]

# Send email for login
# to edit with server mail credentials

EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'obeythetestinggoat@gmail.com'
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_PASSWORD')
EMAIL_PORT = 587
EMAIL_USE_TLS = True

```

### Another Secret, Another Environment Variable

[...]
