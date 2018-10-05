# Python

## Installazione di zlib macOS

Per installare _zlib_ con _Homebrew_ su macOS lanciare il seguente programma da terminale:

`$ brew install zlib`


## Installazione di pyenv

### macOS

_pyenv_ è un semplice gestore di versioni per Python
Per installare _pyenv_ su un sistema operativo macOS è necessario installare Homebrew seguendo [la guida sul sito ufficiale](https://docs.brew.sh/Installation).
Successivamente dobbiamo installare pyenv aprendo il terminale e digitando il seguente comando:

`$ brew install pyenv`

A questo punto possiamo installare una versione di Python 3 in pyenv utilizzando il seguente comando:

`$ CFLAGS="-I$(xcrun --show-sdk-path)/usr/include" pyenv install -v 3.7.0`

`$ pyenv global 3.7.0`

In questo caso ho installato la versione 3.7.0 di Python in quanto, al momento in cui sto scrivendo è la più recente.

Per rimuovere la versione di Python appena installata invece dovrete utilizzare:

`$ pyenv uninstall 3.7.0`

### Linux Ubuntu

È possibile installare _pyenv_ su Linux Ubuntu utilizzando la procedura di installazione automatica:

`$ curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash`

Per potere chiamare _pyenv_ dalla shell con il comando `pyenv` lanciare i seguenti comandi da terminale:

`$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc`

`$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc`

`$ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bashrc`

Quando si arriva al punto in cui bisogna installare una versione di Python in _pyenv_ eseguire il seguente comando:

`$ pyenv install 3.7.0`

`$ pyenv global 3.7.0`

Se si dovesse incontrare un errore in fase d'installazione a causa della mancanza delle librerie di _zlib_ e di _openssl_, eseguire le seguenti operazioni:

```

$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
$ sudo apt-get install build-essential python-dev python-setuptools python-pip python-smbus
$ sudo apt-get install libncursesw5-dev libgdbm-dev libc6-dev
$ sudo apt-get install zlib1g-dev libsqlite3-dev tk-dev
$ sudo apt-get install libssl-dev openssl
$ sudo apt-get install libffi-dev
$ sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev

```

A questo punto è possibile procedere con l'installazione di Python 3.7 in _pyenv_.


## Installazione di pytest

Per installare _pytest_ utilizzando pip è sufficiente lanciare il seguente comando da terminale:

`$ pip install -U pytest`

Il flag `-U` fa in modo che se _pytest_ è gia installato sul sistema ed è disponibile un aggiornamento, effettua l'avanzamento all'ultima versione disponibile.


# Django

## Che cos'è Django

[Django](https://www.djangoproject.com/) è un framework open-source per la creazione di applicazioni in Python.'''


## Installare Django

È possibile installare django utilizzando _pip_; quindi è necessario eseguire il seguente comando da terminale:

`$ pip install django`



# SearchSploit

## Cos'è SearchSploit

[_SearchSploit_](https://www.exploit-db.com/) è un tool da riga di comando per cercare exploit di sistema all'interno del database **Exploit-DB**

## Installare SearchSploit

### macOS

Come è descritto nella guida d'installazione presente sulla pagina ufficiale del progetto, per installare _SearchSploit_ su macOS con Homebrew è sufficiente lanciare il seguente comando da terminale:

`$ brew install exploitdb`


### Linux Ubuntu

Per installare _SearchSploit_ su Linux Ubuntu, come descritto sulla pagina ufficiale del progetto è sufficiente eseguire il _clone_ dal repository ufficiale:

`$ sudo git clone https://github.com/offensive-security/exploitdb.git /opt/exploitdb`

Per poter successivamente lanciare il programma da terminale con il comando `searchsploit`, eseguire il seguente comando:

`$ ln -sf /opt/exploitdb/searchsploit /usr/local/bin/searchsploit`

