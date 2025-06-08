[Facades](https://laravel.com/docs/facades) provide a "static" interface to classes that are available in the application's [service container](https://laravel.com/docs/container). Laravel contains several facades including `DB` which is used to easily access the database.

If you want to use the facades, you must initialize the container yourself. The simplest solution for is to create a `mu-plugin`, here’s an example :

```php
use Dbout\WpOrm\Orm\Database;
use Illuminate\Container\Container;
use Illuminate\Support\Facades\Facade;

$container = new Container();
$container->instance('db', Database::getInstance());
Facade::setFacadeApplication($container);
```

You can now use the `DB` facade without any problems :

```php
use \Illuminate\Support\Facades\DB;

$count = DB::raw('count + 1');
```