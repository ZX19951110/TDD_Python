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

from selenium import webdriver`

`browser = webdriver.Firefox()`
`browser.get('http://localhost:8000')`

`assert 'Django' in browser.title

```

Eseguire il programma:

`python functional_tests.py`

A questo punto, se abbiamo fatto tutto correttamente, vedremo aprirsi una finestra del browser che tenterà di aprire una pagina web allocata all'indirizzo `localhost:8000`. Tuttavia visualizzeremo un errore che ci avviserà che la connessione al server non è riuscita. Ciò è accaduto in quanto non abbiamo ancora avviato il framework Django.

Ora creiamo un nuovo progetto Django con il comando:

`django-admin.py startproject superlists .`

ed eseguiamo il server Django digitando:

`python manage.py runserver`

A questo punto se eseguiamo nuovamente il programma _functional_test.py_, si aprirà una finestra del browser e vedremo comparire un messaggio che ci segnalerà che il server Django è stato avviato correttamente.
