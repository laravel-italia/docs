# Eloquent: Collection

- [Introduzione](#introduzione)
- [Metodi Disponibili](#metodi-disponibili)
- [Collection Personalizzate](#collection-personalizzate)

<a name="introduzione"></a>
## Introduzione

Tutti i set di risultati (non le singole istanze, ma i set) sono a loro volta istanze della classe `Illuminate\Database\Eloquent\Collection`. Tale classe estende a sua volta la [base collection](/collection). Molti dei metodi ereditati, quindi, sono quelli della classe base, adattati però a lavorare bene con le istanze dei model Eloquent.

Ovviamente, ogni collection serve anche da iteratore, permettendoti di "loopare" al suo interno come fosse un semplice array:

	$users = App\User::where('active', 1)->get();

	foreach ($users as $user) {
		echo $user->name;
	}

Tuttavia, le collection sono molto più potenti di quello che sembrano. Tra le varie cose interessanti, ad esempio, espongono alcuni metodi dedicati alle operazioni di map/reduce sugli elementi.

Facciamo una prova: rimuoviamo tutti i model degli utenti inattivi e raccogliamo i nomi dei rimanenti.

	$users = App\User::where('active', 1)->get();

	$names = $users->reject(function ($user) {
		return $user->active === false;
	})
	->map(function ($user) {
		return $user->name;
	});

<a name="metodi-disponibili"></a>
## Metodi Disponibili

### La Collection Base

Tutte le Eloquent collection estendono la [base collection](/collection) di Laravel. Ad ogni modo, ereditano tutti i gli utili metodi della classe base:

* [all](/collection#metodo-all)
* [chunk](/collection#metodo-chunk)
* [collapse](/collection#metodo-collapse)
* [contains](/collection#metodo-contains)
* [count](/collection#metodo-count)
* [diff](/collection#metodo-diff)
* [each](/collection#metodo-each)
* [filter](/collection#metodo-filter)
* [first](/collection#metodo-first)
* [flatten](/collection#metodo-flatten)
* [flip](/collection#metodo-flip)
* [forget](/collection#metodo-forget)
* [forPage](/collection#metodo-forpage)
* [get](/collection#metodo-get)
* [groupBy](/collection#metodo-groupby)
* [has](/collection#metodo-has)
* [implode](/collection#metodo-implode)
* [intersect](/collection#metodo-intersect)
* [isEmpty](/collection#metodo-isempty)
* [keyBy](/collection#metodo-keyby)
* [keys](/collection#metodo-keys)
* [last](/collection#metodo-last)
* [map](/collection#metodo-map)
* [merge](/collection#metodo-merge)
* [pluck](/collection#metodo-pluck)
* [pop](/collection#metodo-pop)
* [prepend](/collection#metodo-prepend)
* [pull](/collection#metodo-pull)
* [push](/collection#metodo-push)
* [put](/collection#metodo-put)
* [random](/collection#metodo-random)
* [reduce](/collection#metodo-reduce)
* [reject](/collection#metodo-reject)
* [reverse](/collection#metodo-reverse)
* [search](/collection#metodo-search)
* [shift](/collection#metodo-shift)
* [shuffle](/collection#metodo-shuffle)
* [slice](/collection#metodo-slice)
* [sort](/collection#metodo-sort)
* [sortBy](/collection#metodo-sortby)
* [sortByDesc](/collection#metodo-sortbydesc)
* [splice](/collection#metodo-splice)
* [sum](/collection#metodo-sum)
* [take](/collection#metodo-take)
* [toArray](/collection#metodo-toarray)
* [toJson](/collection#metodo-tojson)
* [transform](/collection#metodo-transform)
* [unique](/collection#metodo-unique)
* [values](/collection#metodo-values)
* [where](/collection#metodo-where)
* [whereLoose](/collection#metodo-whereloose)
* [zip](/collection#metodo-zip)

<a name="collection-personalizzate"></a>
## Collection Personalizzate

Potresti aver bisogno di usare, per un tuo model in particolare, una _Collection_ personalizzata da estendere con i tuoi metodi. Nessun problema, effettua l'override del metodo _newCollection_ nel tuo model.

	<?php namespace App;

	use App\CustomCollection;
	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Create a new Eloquent Collection instance.
		 *
		 * @param  array  $models
		 * @return \Illuminate\Database\Eloquent\Collection
		 */
		public function newCollection(array $models = [])
		{
			return new CustomCollection($models);
		}
	}

Una volta definito il metodo _newCollection_, riceverai un'istanza della collection personalizzata ogni volta che Eloquent ritornerà un'istanza di una collection per tale model. Per intenderci, verrà ritornata la tua collection personalizzata ogni volta che userai i metodi _get_, _all_ e così via.
