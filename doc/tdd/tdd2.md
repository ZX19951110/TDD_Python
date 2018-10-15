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

[...]
