# Python

## Installazione di pyenv

### macOS

_pyenv_ è un semplice gestore di versioni per Python
Per installare _pyenv_ su un sistema operativo macOS è necessario installare Homebrew seguendo [la guida sul sito ufficiale](https://docs.brew.sh/Installation).
Successivamente dobbiamo installare pyenv aprendo il terminale e digitando il seguente comando:

`brew install pyenv`

A questo punto possiamo installare una versione di Python 3 in pyenv utilizzando il seguente comando:

`CFLAGS="-I$(xcrun --show-sdk-path)/usr/include" pyenv install -v 3.7.0`

In questo caso ho installato la versione 3.7.0 di Python in quanto, al momento in cui sto scrivendo è la più recente.

Per rimuovere la versione di Python appena installata invece dovrete utilizzare:

`pyenv uninstall 3.7.0


# Installazione di pytest

Per installare _pytest_ utilizzando pip è sufficiente lanciare il seguente comando da terminale:

`pip install -U pytest`



# Django

## Che cos'è Django
[Django](https://www.djangoproject.com/) è un framework open-source per la creazione di applicazioni in Python.


## Installare Django

### macOS
È possibile installare django utilizzando _pip_; quindi è necessario eseguire il seguente comando da terminale:

`pip install Django`



# SearchSploit

## Cos'è SearchSploit
[_SearchSploit_](https://www.exploit-db.com/) è un tool da riga di comando per cercare exploit di sistema all'interno del database **Exploit-DB**

## Installare SearchSploit

### macOS
Come è descritto sulla guida d'installazione presente sulla pagina ufficiale del progetto, per installare SearchSploit su macOS con Homebrew è sufficiente lanciare il seguente comando da terminale:

`brew install exploitdb`
