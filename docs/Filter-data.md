You can filter data in several ways:

- With `findOneBy` functions in place on some models
- With taps
- With Eloquent query builder. If your model has metas, you can use custom filter methods.

## With findOneBy

By default, Eloquent does not offer a magic feature [findOneBy*](https://github.com/laravel/ideas/issues/107), however you can use this feature on some models like `User`, `Option` or `Post`.

- `User::findOneByEmail()`
- `User::findOneByLogin()`
- `Option::findOneByName()`
- `Post::findOneByName()`
- `Post::findOneByGuid()`

## With taps

The `tap` method passes the query builder to the given callback, allowing you to "tap" into the query builder at a specific point and do something with the query:

You can easily filter data via the `tap` function :

```php
use Dbout\WpOrm\Taps\Post\IsAuthorTap;
use Dbout\WpOrm\Taps\Post\IsStatusTap;
use Dbout\WpOrm\Enums\PostStatus;
use Dbout\WpOrm\Models\Post;

$posts = Post::query()
    ->tap(new IsAuthorTap(1))
    ->get();
```

This query, returns all user posts with ID 1.

If you want to apply multiple filters, nothing complicated :

```php
use Dbout\WpOrm\Taps\Post\IsAuthorTap;
use Dbout\WpOrm\Taps\Post\IsStatusTap;
use Dbout\WpOrm\Enums\PostStatus;
use Dbout\WpOrm\Models\Post;

$posts = Post::query()
    ->tap(new IsAuthorTap(1))
    ->tap(new IsStatusTap(PostStatus::Publish))
    ->get();
```

You can find all the available filters here: 

| Type | Class |
| --- | --- |
| Post | `Dbout\WpOrm\Taps\Post\IsAuthorTap` |
| | `Dbout\WpOrm\Taps\Post\IsPingStatusTap` |
| | `Dbout\WpOrm\Taps\Post\IsPostTypeTap` | 
| | `Dbout\WpOrm\Taps\Post\IsStatusTap` |
| Attachment | `Dbout\WpOrm\Taps\Attachment\IsMimeTypeTap` |
| Comment | `Dbout\WpOrm\Taps\Comment\IsApprovedTap` |
| | `Dbout\WpOrm\Taps\Comment\IsCommentTypeTap` |
| | `Dbout\WpOrm\Taps\Comment\IsUserTap` |
| Option | `Dbout\WpOrm\Taps\Option\IsAutoloadTap`

## With query builder

The Eloquent `all` method will return all of the results in the model's table. However, since each Eloquent model serves as a [query builder](https://laravel.com/docs/10.x/queries), you may add additional constraints to queries and then invoke the `get` method to retrieve the results:

```php
use Dbout\WpOrm\Models\Post;

$posts = Post::query()
    ->where('ping_status', 'closed')
    ->get();
```

> [!INFO]
> More information here: [Eloquent query builder](https://laravel.com/docs/queries).

### Model with meta relation

For models that may have metas (e.g. `Post`, `User`, ...), you can filter with `addMetaToFilter`, here is an example that speaks for itself:)

```php
$products = Post::query()
    ->addMetaToFilter('product_type', 'simple')
    ->get();
```
