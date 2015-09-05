# Contratti

- [Introduzione](#introduzione)
- [Perchè I Contratti?](#perche-contratti)
- [Reference Dei Contratti](#reference-contratti)
- [Come Usare I Contratti](#come-usare-contratti)

<a name="introduzione"></a>
## Introduzione

I "Contratti" di Laravel sono una serie di interfacce che definiscono tutti i servizi core del framework. Per esempio, il contratto `Illuminate\Contracts\Queue\Queue` definisce i metodi necessari per lavorare con le code, mentre il contratto `Illuminate\Contracts\Mail\Mailer` definisce i metodi necessari per l'invio di e-mail.

Ogni contratto ha una sua corrispondente implementazione fornita dal framework. Per esempio, Laravel forniesce un'implemetazione delle code con svariati driver, mentre per Mailer c'è quella che coinvolge anche [SwiftMailer](http://swiftmailer.org/).

Tutti i contratti di Laravel possono essere analizzati facilmente sull'apposito [repository GitHub](https://github.com/illuminate/contracts). Molto utile se si ha bisogno di una reference veloce per capire meglio il concetto.

### Contratti Vs. Facade

Le [facade](/facade) di Laravel offrono un semplice modo di utilizzare i servizi di Laravel senza importare e risolvere i contratti attraverso il service container. Comunque, l'uso  dei contratti ti permette di definire delle dipendenze esplicite per le tue classi. Per molte applicazioni, l'uso della facade e ancora ben visto. Tuttavia se hai veramente bisogno di loose coupling che i contratti offrono, continua a leggere!

<a name="perche-contratti"></a>
## Perchè I Contratti?

Probabilmente ti sarai fatto già qualche domanda sul perché dell'implementazione di questi contratti. Anzi, perché mai usare le interfacce? Sono complicate! Fondamentalmente le ragioni della scelta sono due: loose coupling e semplicità.

### Loose Coupling

Vediamo, innanzitutto, un codice che presenta un'implementazione "fortemente accoppiata" (tightly coupled) alla cache. 

	<?php namespace App\Orders;

	class Repository
	{
		/**
		 * The cache.
		 */
		protected $cache;

		/**
		 * Create a new repository instance.
		 *
		 * @param  \SomePackage\Cache\Memcached  $cache
		 * @return void
		 */
		public function __construct(\SomePackage\Cache\Memcached $cache)
		{
			$this->cache = $cache;
		}

		/**
		 * Retrieve an Order by ID.
		 *
		 * @param  int  $id
		 * @return Order
		 */
		public function find($id)
		{
			if ($this->cache->has($id))	{
				//
			}
		}
	}

In questa classe il codice è fortemente dipendente dall'implementazione della cache. E' fortemente dipendente perchè abbiamo fatto in modo che la cache sia dipendente da una classe concreta Cache da qualche nostro package. Se l'API di Memcached, in questo caso, cambiasse, il programmatore si ritroverebbe nei guai. 

Oppure, supponiamo di dover cambiare la tecnologia sottostante (Memcached) con un'altra (Redis). Come comportarci? Dovremmo modificare, ancora una volta, il nostro repository. Insomma: il nostro repository non dovrebbe avere conoscenza di ciò che c'è sotto.

**Ecco quindi la soluzione. Dipendere non da una classe specifica, ma da un'interfaccia "agnostica" rispetto al vendor.**

	<?php namespace App\Orders;

	use Illuminate\Contracts\Cache\Repository as Cache;

	class Repository
	{
		/**
		 * Create a new repository instance.
		 *
		 * @param  Cache  $cache
		 * @return void
		 */
		public function __construct(Cache $cache)
		{
			$this->cache = $cache;
		}
	}

Da questo momento in poi il tuo codice non è più legato ad uno specifico vendor (e neanche a Laravel!). Il package dei contract, infatti, non contiene implementazioni e neanche dipendenze. Se dovessi decidere di scriverti da solo i tuoi componenti, potresti farlo tranquillamente dovendo seguire l'unica regola di implementare i contratti necessari.

### Semplicità

Se tutti i servizi di Laravel sono definiti implementando semplici interfacce, è ancora più semplice determinare le funzionalità offerte da ogni singolo servizio. **I contratti, insomma, fanno anche da "riassunto" della documentazione delle feature del framework.**

Inoltre, quando dipendi da interfacce semplici, il tuo codice è più semplice da comprendere e mantenere nel tempo. Al posto di doverti cercare quali metodi usare in una grande classe complicata, basta dare uno sguardo all'interfaccia! Semplice.

<a name="reference-contratti"></a>
## Reference Dei Contratti

Ecco una reference di tutti i Contract di Laravel, con la loro Facade corrispondente:

* [Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/master/Broadcasting/Broadcaster.php)  | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="come-usare-contratti"></a>
## Come Usare I Contratti

Ok, come ottenere un'implementazione di un contract? Piuttosto semplicemente, in realtà. 

Molti tipi di classe in Laravel sono risolte tramite il [service container](/container), inclusi controller, event listener, middleware, code ed addirittura le Closure. Tutto quello che devi fare per ottenere l'implementazione corrispondente di una certa interfaccia (o contratto) è specificare il contratto tramite type-hinting.

Per esempio, dai uno sguardo a questo event listener:

	<?php namespace App\Listeners;

	use App\User;
	use App\Events\NewUserRegistered;
	use Illuminate\Contracts\Redis\Database;

	class CacheUserInformation
	{
		/**
		 * The Redis database implementation.
		 */
		protected $redis;

		/**
		 * Create a new event handler instance.
		 *
		 * @param  Database  $redis
		 * @return void
		 */
		public function __construct(Database $redis)
		{
			$this->redis = $redis;
		}

		/**
		 * Handle the event.
		 *
		 * @param  NewUserRegistered  $event
		 * @return void
		 */
		public function handle(NewUserRegistered $event)
		{
			//
		}
	}

Nel momento in cui l'event listener viene risolto, il service container legge il type-hint desiderato dal costruttore della classe, iniettando l'istanza corrispondente. Se il meccanismo non ti è ancora del tutto chiaro, dai uno sguardo [alla documentazione](/container).
