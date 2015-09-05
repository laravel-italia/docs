# Autorizzazioni

- [Introduzione](#introduzione)
- [Definire le "Abilità"](#definire-abilita)
- [Controllo delle Abilità](#controllo-abilita)
	- [Tramite la Facade Gate](#tramite-gate-facade)
	- [Tramite il Model User](#tramite-model-user)
	- [Nei Template Blade](#nei-template-blade)
    - [Nelle Form Request](#nelle-form-request)
- [Policy](#policy)
	- [Creare una Policy](#creare-policy)
	- [Scrivere una Policy](#scrivere-policy)
	- [Controllare una Policy](#controllare-policy)
- [Autorizzazione nei Controller](#autorizzazione-controller)

<a name="introduzione"></a>
## Introduzione

In aggiunta all'ottimo meccanismo di [autenticazione](/autenticazione) già presente nel framework, Larael offre un comodo sistema di organizzazione delle logiche di autorizzazione per accedere alle risorse della tua applicazione. Sono disponibili, infatti, svariati metodi ed helper dedicati a tale scopo ed in questo capitolo della guida vedremo come usarli.

> **Nota:** il sistema di Authorization è stato aggiunto a partire da Laravel 5.1.11. Per maggiori informazioni fai riferimento alla [guida all'aggiornamento](/aggiornamento).

<a name="definire-abilita"></a>
## Definire le "Abilità"

Il modo più semplice di determinare se un certo utente può effettivamente effettuare un'azione è definire una cosiddetta "abilità", tramite la classe `Illuminate\Auth\Access\Gate`. Il posto migliore dove inserire questa definizione può essere la classe `AuthServiceProvider` già presente, di default, in una qualsiasi installazione di Laravel.

Facciamo un esempio: definiamo un'abilità `update-post` che riceve in input l'utente `User` attualmente autenticato ed un'istanza di un model `Post`. All'interno della nostra "abilità" definiremo le condizioni per cui un certo utente può modificare un post.

	<?php

	namespace App\Providers;

	use Illuminate\Contracts\Auth\Access\Gate as GateContract;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * Register any application authentication / authorization services.
	     *
	     * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
	     * @return void
	     */
	    public function boot(GateContract $gate)
	    {
	        parent::registerPolicies($gate);

	        $gate->define('update-post', function ($user, $post) {
	        	return $user->id === $post->user_id;
	        });
	    }
	}

Se ci fai caso, non abbiamo controllato se l'utente corrente è uguale a `NULL`. Il `Gate`, infatti, ritornerà automaticamente `false` nel caso in cui non esista un utente autenticato (o comunque nessun utente selezionato con il metodo `forUser` che vedremo a breve).

#### Abilità Specificate in una Classe

In aggiunta alla registrazione di `Closure` come quella appena vista, puoi registrare dei metodi usando una convenzione che già conosci: `NomeClasse@nomeMetodo`. Il resto del lavoro verrà svolto dal [service container](/container):

    $gate->define('update-post', 'Class@method');

<a name="controllo-abilita"></a>
## Controllo delle Abilità

<a name="tramite-gate-facade"></a>
### Tramite la Facade Gate

Una volta definita un'abilità, effettuare il controllo è molto semplice e può essere svolto in molti modi. Innanzitutto, puoi usare i metodi `allows` o `denies` della [facade](/facade) `Gate`. Non ci sarà bisogno di passare anche l'istanza dell'utente attuale come parametro, in quanto verrà preso in considerazione quello "corrente" del sistema di autenticazione. L'utente "già loggato", insomma.

Tutto quello che dovremo fare quindi sarà passare l'istanza della classe `Post`.

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update-post', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

Come puoi ben immaginare, il metodo `allows` è l'esatto contrario del metodo `denies`. Ritornerà `true` solo nel caso in cui l'azione verrà autorizzata.

#### Controllare l'Autorizzazione per un Determinato Utente

A volte si potrebbe voler controllare un permesso, non tanto per "noi stessi" (l'utente loggato) ma per un altro utente del nostro sistema. In tal caso, il metodo `forUser` è quello che fa al caso nostro.

	if (Gate::forUser($user)->allows('update-post', $post)) {
		//
	}

#### Uso di Più Parametri

Potresti avere la necessità di passare ad una callback più di un parametro. Una cosa del genere.

	Gate::define('delete-comment', function ($user, $post, $comment) {
		//
	});

In tal caso, dovrai passare un array di parametri strutturati in questo modo.

	if (Gate::allows('delete-comment', [$post, $comment])) {
		//
	}

<a name="tramite-model-user"></a>
### Tramite il Model `User`

In alternativa ai metodi visti puoi sempre usare l'istanza di un model `User` per effettuare i controlli sulle autorizzazioni. Di default, infatti, il model `User` si avvale del trait `Authorizable` che presenta due metodi: `can` e `cannot`. Il loro uso è molto simile a `allows` e `denies`.

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  int  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
        	$post = Post::findOrFail($id);

        	if ($request->user()->cannot('update-post', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

Ovviamente, il metodo `can` è semplicemente l'inverso di `cannot`.

	if ($request->user()->can('update-post', $post)) {
		// Update Post...
	}

<a name="nei-template-blade"></a>
### Nei Template Blade

Per una maggiore convenienza e velocità, Laravel offre una nuova direttiva: `@can`. Anche qui la sintassi da usare è praticamente identica alle precedenti.

	<a href="/post/{{ $post->id }}">View Post</a>

	@can('update-post', $post)
		<a href="/post/{{ $post->id }}/edit">Edit Post</a>
	@endcan

Puoi inoltre combinare un `@can` con un `@else`.

	@can('update-post', $post)
		<!-- The Current User Can Update The Post -->
	@else
		<!-- The Current User Can't Update The Post -->
	@endcan

<a name="nelle-form-request"></a>
### Nelle Form Request

Puoi anche scegliere di usare la facade `Gate` nel metodo `authorize` di una [form request](/validazione#validazione-form-request). Guarda l'esempio seguente, che fa uso del risultato ritornato dal metodo `Gate::allows()`.

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $postId = $this->route('post');

        return Gate::allows('update', Post::findOrFail($postId));
    }

<a name="policy"></a>
## Policy

<a name="creare-policy"></a>
### Creare una Policy

Definire le logiche di autorizzazione nella classe `AuthServiceProvider` è comodo, certo, ma come regolarsi in caso di applicazioni che cominciano a diventare molto più grandi del solito? Beh, in tal caso, la cosa migliore da fare è ragionare più "ad oggetti" e dividere le logiche di autorizzazione in vere e proprie `Policy`. Le `Policy` sono delle classi che raggruppano tali logiche in base ad una certa risorsa.

Generare una policy è molto semplice: basta usare il comando `make:policy` di Artisan. Facciamo un esempio di generazione di una policy per la risorsa `Post`.

	php artisan make:policy PostPolicy

#### Registrare una Policy

Una volta creata la policy bisogna registrarla. Nella classe `AuthServiceProvider` puoi trovare una proprietà `policies` nella quale definire le policy per ognuna delle tue entità. Ad esempio, volendo collegare l'entità `Post` alla nuova policy `PostPolicy`, farai così:

	<?php

	namespace App\Providers;

	use App\Post;
	use App\Policies\PostPolicy;
	use Illuminate\Contracts\Auth\Access\Gate as GateContract;
	use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

	class AuthServiceProvider extends ServiceProvider
	{
	    /**
	     * The policy mappings for the application.
	     *
	     * @var array
	     */
	    protected $policies = [
	        Post::class => PostPolicy::class,
	    ];
	}

<a name="scrivere-policy"></a>
### Scrivere una Policy

Una volta che la policy è stata generata e registrata, puoi procedere con l'aggiunta dei metodi in modo tale da definire il meccanismo di autorizzazione per ogni operazione da effettuare. Ad esempio, ritorniamo al caso precedente e definiamo un metodo `update` per la nostra `PostPolicy`. Come puoi facilmente immaginare, servirà a determinare se il nostro utente può effettivamente modificare un post.

	<?php

	namespace App\Policies;

	use App\User;
	use App\Post;

	class PostPolicy
	{
		/**
		 * Determine if the given post can be updated by the user.
		 *
		 * @param  \App\User  $user
		 * @param  \App\Post  $post
		 * @return bool
		 */
	    public function update(User $user, Post $post)
	    {
	    	return $user->id === $post->user_id;
	    }
	}

Nulla ti vieta di continuare con altri metodi, in base alle necessità: prova ad aggiungere dei metodi `show`, `destroy`, `addComment` e così via!

> **Nota:** tutte le policy vengono risolte tramite il [service container](/container), il che vuol dire che puoi effettuare senza problemi il type-hint delle eventuali dipendenze. Che verranno automaticamente iniettate.

<a name="controllare-policy"></a>
### Controllare una Policy

I metodi di una policy vengono richiamati allo stesso identico modo che hai visto, in precedenza, per le `Closure`. Tramite la facade `Gate`, tramite il model `User` o la direttiva Blade `@can`. In aggiunta, puoi usare anche un helper `policy`.

Vediamo un esempio per ognuno di questi quattro metodi.

#### Tramite la Facade `Gate`

La classe `Gate` determinerà automaticamente quale policy usare esaminando i vari parametri passati ai propri metodi. Ad esempio, passando un'istanza di `Post` ad un metodo `Gate::denies`, la facade userà la policy corrispondente `PostPolicy` senza ulteriori istruzioni.

    <?php

    namespace App\Http\Controllers;

    use Gate;
    use App\User;
    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	if (Gate::denies('update', $post)) {
        		abort(403);
        	}

        	// Update Post...
        }
    }

#### Tramite il Model `User`

I metodi `can` e `cannot` del trait `Authorizable` possono usare automaticamente anche le policy. Tutto quello che devi fare è passare i parametri giusti ed il nome del metodo scelto per l'abilità specifica.

Così:

	if ($user->can('update', $post)) {
		//
	}

	if ($user->cannot('update', $post)) {
		//
	}

#### Nei Template Blade

Allo stesso modo, anche la direttiva `@can` di Blade può usare le policy.

	@can('update', $post)
		<!-- The Current User Can Update The Post -->
	@endcan

#### Tramite l'Helper `policy`

Esiste inoltre un helper `policy` che puoi usare per controllare una determinata operazione. Tutto quello che devi fare è passare alla funzione l'istanza necessaria. In questo caso, di `Post`.

	if (policy($post)->update($user, $post)) {
		//
	}

L'istruzione `policy($post)` restituirà un'istanza della `PostPolicy`.

<a name="autorizzazione-controller"></a>
## Autorizzazione nei Controller

Di default, il controller base `App\Http\Controllers\Controller` incluso in Laravel usa il trait `AuthorizesRequests`. Tale trait offre il metodo `authorize`, che può essere usato per un controllo al volo di una determinata abilità da parte di un utente.

Se qualcosa va storto, questo metodo lancia un'eccezione di tipo `HttpException`.

Il metodo `authorize` presenta la stessa segnatura vista nelle altre metodologie finora affrontate, come `Gate::allows` o `$user->can()`. Riprendendo l'esempio precedente, ma da un controller, avremo...

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given post.
         *
         * @param  int  $id
         * @return Response
         */
        public function update($id)
        {
        	$post = Post::findOrFail($id);

        	$this->authorize('update', $post);

        	// Update Post...
        }
    }

Nel caso in cui l'azione venisse autorizzata, il controller proseguirebbe normalmente nell'esecuzione. In caso contrario, l'eccezione lanciata genererebbe una risposta HTTP con status code `403, non Autorizzato`. Il tutto con una singola linea di codice. Tienilo a mente, potrebbe tornarti utile in futuro.

Viene inoltre offerto, dallo stesso trait, il metodo `authorizeForUser` che permette di controllare l'autorizzazione per una certa azione anche su un utente diverso da quello attualmente loggato.

	$this->authorizeForUser($user, 'update', $post);

#### Determinare Automaticamente quale Metodo della Policy Usare

Può capitare (e capita spesso) che un metodo di una policy corrisponda al nome del metodo corrispondente sul controller. Ad esempio, nell'esempio visto prima, controller e policy condividono il nome del metodo. Appunto, `update`.

Per questo motivo Laravel ti offre, come "marcia in più" la possibilità di passare la semplice istanza al metodo `authorize` e lasciare che sia il sistema di autorizzazioni a fare tutto il resto. Il framework sarà capace di ricollegare i due nomi identici e, quindi, di controllare i permessi per la specifica azione.

    /**
     * Update the given post.
     *
     * @param  int  $id
     * @return Response
     */
    public function update($id)
    {
    	$post = Post::findOrFail($id);

    	$this->authorize($post);

    	// Update Post...
    }