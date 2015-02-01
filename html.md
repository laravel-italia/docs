# Forms & HTML

- [Aprire Un Form](#aprire-un-form)
- [Protezione CSRF](#protezione-csrf)
- [Legare Form e Model](#legare-form-e-model)
- [Label](#label)
- [Text, Text Area, Password e Hidden Field](#text)
- [Checkbox e Radio Button](#checkboxe-e-radio-buttons)
- [File Input](#file-input)
- [Input Numerico](#input-numerico)
- [Drop-Down](#drop-down)
- [Button](#button)
- [Macro Personalizzate](#macro-personalizzate)
- [Generare URL](#generare-url)

<a name="aprire-un-form"></a>
## Aprire Un Form

#### Aprire Un Form

	{{ Form::open(array('url' => 'foo/bar')) }}
		//
	{{ Form::close() }}

Di default viene utilizzato il metodo `POST` ma hai la possibilità di specificare il metodo che ti serve:

	echo Form::open(array('url' => 'foo/bar', 'method' => 'put'))

> **Nota:** I form HTML supportano solo i metodi `POST` e `GET`, i metodi `PUT` e `DELETE` saranno inseriti all'interno del form aggiungendo `_method` come campo nascosto.

Puoi anche specificare dei form che puntano al nome di una route o all'action di un controller:

	echo Form::open(array('route' => 'route.name'))

	echo Form::open(array('action' => 'Controller@method'))

Puoi anche passare dei parametri alle route:

	echo Form::open(array('route' => array('route.name', $user->id)))

	echo Form::open(array('action' => array('Controller@method', $user->id)))

Se il form che stai creando conterrà un campo per l'upload di file, assicurati di aggiungere l'opzione `files` quando apri il form:

	echo Form::open(array('url' => 'foo/bar', 'files' => true))

<a name="protezione-csrf"></a>
## Protezione CSRF

#### Aggiungere il token CSRF Al Form

Laravel fornisce un modo per proteggere la tua aplicazione dagli attacchi di tipo CSRF (Cross-site request forgery). Per prima cosa, all'interno della sessione dell'utente, viene salvato un token casuale. Se utilizzi il metodo `Form::open` con una richiesta `POST`, `PUT` o `DELETE` il token CSRF viene aggiunto automaticamente come campo nascosto all'interno del tuo form. In alternativa, se vuoi creare manualmente il token, lo puoi fare utlizzando il metodo `token`:

	echo Form::token();

#### Attaccare il filtro CSRF A Una Route

	Route::post('profile', array('before' => 'csrf', function()
	{
		//
	}));

<a name="legare-form-e-model"></a>
## Legare Form E Model

#### Aprire Un Form Model

Spesso ti capiterà di dover popolare un form con dei dati recuperati dal database attraverso il tuo Model. Per far in modo che i dati vengano ripopolati automaticamente puoi utilizzare il metodo `Form::model`:

	echo Form::model($user, array('route' => array('user.update', $user->id)))

In questo modo quando creerai un elemento del form, come ad esempio un campo di testo, il valore del campo sarà ripopolato se l'attributo `name` corrisponderà ad un valore del Model. Per esempio, se il Model recupera un campo `email` e nel form trova un campo di input con il `name` impostato a `email` si occuperà automaticamente di impostarne il valore. Ma non è finita qui! Se nella sessione esiste un valore che corrisponde al name di un campo input tale valore sarà utilizzato per ripopolare quel campo. In questo caso la sessione ha priorità maggiore rispetto al valore recuperato dal Model. Ricapitolando le priorità sono:

1. Session Flash Data (Input Precedente)
2. Valori passati in modo esplicito
3. Dati recuperati dal Model

Questo ti permette non solo di creare dei form ripopolati automaticamente, ma anche form ripopolati in seguito a qualche errore di validazione lato server!

> **Nota:** Quando usi `Form::model` assicurati di usare anche `Form::close` per chiudere il form!

<a name="label"></a>
## Label

#### Generare Un Elemento Label

	echo Form::label('email', 'Indirizzo E-Mail');

#### Specificare Attributi HTML Extra

	echo Form::label('email', 'Indirizzo E-Mail', array('class' => 'fantastico'));

> **Nota:** Dopo che crei una label, qualsiasi elemento del form che crei che possiede un name che combacia il name della label, riceverà un ID che combacerà con tale name.

<a name="text"></a>
## Text, Text Area, Password E Hidden Field

#### Generare Un Input Di Testo

	echo Form::text('username');

#### Specificare Un Valore Di Default

	echo Form::text('email', 'example@gmail.com');

> **Nota:** I metodi *hidden* e *textarea* hanno la stessa sintassi del metodo *text*.

#### Generare Un Input Per Una Password

	echo Form::password('password');

#### Generare Altri Input

	echo Form::email($name, $value = null, $attributes = array());
	echo Form::file($name, $attributes = array());

<a name="checkboxe-e-radio-button"></a>
## Checkbox E Radio Button

#### Generare Una Checkbox O Un Radio Input

	echo Form::checkbox('name', 'value');

	echo Form::radio('name', 'value');

#### Generare Una Checkbox O Un Radio Input Pre-Selezionato

	echo Form::checkbox('name', 'value', true);

	echo Form::radio('name', 'value', true);

<a name="input-numerico"></a>
## Input Numerico

#### Generare Un Input Numerico

	echo Form::number('name', 'value');

<a name="file-input"></a>
## File Input

#### Generare Un File Input

	echo Form::file('image');

> **Nota:** Il Form deve essere aperto con l'opzione `files` impostata a `true`.

<a name="drop-down"></a>
## Drop-Down

#### Generare Una Drop-Down

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'));

#### Generare Una Drop-Down Con Un Elemento Pre-Selezionato

	echo Form::select('size', array('L' => 'Large', 'S' => 'Small'), 'S');

#### Generare Una Lista Raggruppata

	echo Form::select('animal', array(
		'Cats' => array('leopard' => 'Leopard'),
		'Dogs' => array('spaniel' => 'Spaniel'),
	));

#### Generare Una Drop-Down List Con Un Intervallo Di Valori

    echo Form::selectRange('number', 10, 20);

#### Generare Una Lista Contenente I Nomi Dei Mesi

    echo Form::selectMonth('month');

<a name="button"></a>
## Button

#### Generare Un Submit Button

	echo Form::submit('Cliccami!');

> **Nota:** Hai bisogno di creare un elemento button? Puoi usare il metodo *button*, ha la stessa sintassi del metodo *submit*.

<a name="macro-personalizzate"></a>
## Macro Personalizzate

#### Registrare Una Macro Per Un Form

E' molto facile definire degli helper, chiamati "macro", per la classe Form. Ecco come puoi fare. Per prima cosa registra semplicemente la macro e assegnale un nome e una Closure:

	Form::macro('mioCampo', function()
	{
		return '<input type="fantastico">';
	});

Ora puoi richiamare la macro utilizzando il suo nome:

#### Richiamare Una Macro Personalizzata In Un Form

	echo Form::mioCampo();

<a name="generare-url"></a>
## Generare URL

Per avere maggiori informazioni su come generare gli URL, dai una occhiata alla documentazione dedicata agli [helper](/docs/helpers#urls).
