## Attribute casting

In your Eloquent model, you can define the casts property to specify which attributes should be cast to specific data types. The casts property should be an associative array where the keys represent the attribute names, and the values represent the desired data types.

```php
class User extends Model
{
    protected $casts = [        
        'is_admin' => 'boolean',        
        'settings' => 'array',        
        'created_at' => 'datetime',    
    ];
}
```

In this example:

- The `is_admin` attribute will be cast to a boolean type.
- The `settings` attribute will be cast to an array type.
- The `created_at` attribute will be cast to a datetime type.

If you want to know more about casting, look [Eloquent document](https://laravel.com/docs/11.x/eloquent-mutators#attribute-casting).

> [!NOTE]
> Depending on the database driver used, attributes can be casted automatically. However, in most cases, it's mandatory to use the `$casts` property. You can find more information in this issue: [Attributes are not automatically cast to their column types](https://github.com/dimitriBouteille/wp-orm/issues/74).

## Meta casting

Meta casting allow you to transform meta values when you retrieve or set them on model instances. For example, you may want to use the Laravel encrypter to encrypt a value while it is stored in the database, and then automatically decrypt the meta when you access it on an Eloquent model. Or, you may want to convert a JSON string that is stored in your database to an array when it is accessed via your Eloquent model. 

**This system works in the same way as the [cast of attributes](https://laravel.com/docs/11.x/eloquent-mutators#attribute-casting).**

The `metaCasts` method should return an array where the key is the name of the meta being cast and the value is the type you wish to cast the column to. The supported cast types are:

- `array`
- `bool`
- `boolean`
- `collection`
- `date`
- `datetime`
- `double`
- `float`
- `immutable_date`
- `int`
- `integer`
- `json`
- `object`
- `string`
- `timestamp`
- custom `enum`

To demonstrate meta casting, let's cast the `is_admin` meta, which is stored in our database as an integer (0 or 1) to a boolean value:

```php
namespace App\Models;

use Dbout\WpOrm\Models\Post;

class User extends Post
{
    protected function metaCasts(): array
    {
        return [
            'is_admin' => 'boolean',
        ];
    }
}
```

After defining the cast, the `is_admin` meta will always be cast to a boolean when you access it, even if the underlying value is stored in the database as an integer:

```php
$user = App\Models\User::find(1);
$isAdmin = $user->getMetaValue('is_admin');
```

If you want, you can also use `$metaCasts` :

```php
namespace App\Models;

use Dbout\WpOrm\Models\Post;

class User extends Post
{
    protected $casts = [        
        'is_admin' => 'boolean',
    ];
}
```

> [!WARNING]
> Metas that are `null` will not be cast.
