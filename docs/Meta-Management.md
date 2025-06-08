# Meta Data Management

This guide covers comprehensive meta data management in WordPress ORM, including batch operations, performance optimization, and advanced patterns.

## Meta Basics

### Understanding Meta Models

WordPress ORM provides dedicated meta models for different entity types:

```php
use Dbout\WpOrm\Models\Meta\PostMeta;
use Dbout\WpOrm\Models\Meta\UserMeta;
use Dbout\WpOrm\Models\Meta\CommentMeta;

// Working with meta models directly
$postMeta = PostMeta::query()
    ->where('post_id', 123)
    ->where('meta_key', 'featured')
    ->first();

$userMeta = UserMeta::query()
    ->where('user_id', 1)
    ->where('meta_key', 'first_name')
    ->first();
```

### Meta through Parent Models

```php
use Dbout\WpOrm\Models\Post;
use Dbout\WpOrm\Models\User;

// Working with meta through parent models (recommended)
$post = Post::find(123);
$featuredValue = $post->getMetaValue('featured');
$post->setMeta('featured', true);

$user = User::find(1);
$firstName = $user->getMetaValue('first_name');
$user->setMeta('first_name', 'John');
```

## Single Meta Operations

### Reading Meta Values

```php
$post = Post::find(123);

// Get single meta value
$value = $post->getMetaValue('price');
$featured = $post->getMetaValue('featured');
$tags = $post->getMetaValue('tags');

// Check if meta exists
if ($post->hasMeta('featured')) {
    echo "Post is featured";
}

// Get meta with default value
$rating = $post->getMetaValue('rating') ?? 0;

// Get all meta
$allMeta = $post->metas;
foreach ($allMeta as $meta) {
    echo "{$meta->getKey()}: {$meta->getValue()}";
}
```

### Setting Meta Values

```php
$post = Post::find(123);

// Set single meta
$post->setMeta('price', 99.99);
$post->setMeta('featured', true);
$post->setMeta('tags', ['electronics', 'gadgets']);

// Set meta with objects
$post->setMeta('product_data', [
    'manufacturer' => 'ACME Corp',
    'model' => 'Widget Pro',
    'warranty' => '2 years'
]);

// Set meta with dates
$post->setMeta('sale_start', now());
$post->setMeta('sale_end', now()->addWeeks(2));
```

### Deleting Meta

```php
$post = Post::find(123);

// Delete single meta
$post->deleteMeta('old_field');

// Delete multiple meta keys
$keysToDelete = ['temp_data', 'cache_field', 'deprecated_setting'];
foreach ($keysToDelete as $key) {
    $post->deleteMeta($key);
}

// Conditional delete
if ($post->hasMeta('temporary_flag')) {
    $post->deleteMeta('temporary_flag');
}
```

## Batch Meta Operations

### Batch Setting Meta

```php
// Method 1: Loop through models
$posts = Post::query()
    ->where('post_type', 'product')
    ->get();

foreach ($posts as $post) {
    $post->setMeta('updated_at', now());
    $post->setMeta('status', 'active');
}

// Method 2: Batch function
function batchSetMeta($models, $metaData) {
    foreach ($models as $model) {
        foreach ($metaData as $key => $value) {
            $model->setMeta($key, $value);
        }
    }
}

$products = Post::query()
    ->where('post_type', 'product')
    ->get();

batchSetMeta($products, [
    'updated_by' => 'admin',
    'batch_updated' => now(),
    'version' => '2.0'
]);
```

### Efficient Batch Meta Operations

```php
// Using chunks for large datasets
function batchUpdateProductMeta($updates) {
    Post::query()
        ->where('post_type', 'product')
        ->chunk(100, function ($products) use ($updates) {
            foreach ($products as $product) {
                foreach ($updates as $key => $value) {
                    $product->setMeta($key, $value);
                }
            }
        });
}

// Usage
batchUpdateProductMeta([
    'last_sync' => now(),
    'api_version' => '2.1',
    'status' => 'synchronized'
]);
```

### Conditional Batch Updates

```php
// Update meta based on conditions
function updateExpiredProducts() {
    $expiredDate = now()->subDays(30);

    Post::query()
        ->where('post_type', 'product')
        ->addMetaToFilter('expiry_date', $expiredDate, '<')
        ->chunk(50, function ($products) {
            foreach ($products as $product) {
                $product->setMeta('status', 'expired');
                $product->setMeta('expired_at', now());
                $product->setPostStatus('draft');
                $product->save();
            }
        });
}

// Update featured products
function updateFeaturedProducts($isFeatured) {
    Post::query()
        ->where('post_type', 'product')
        ->addMetaToFilter('featured', !$isFeatured)
        ->chunk(100, function ($products) use ($isFeatured) {
            foreach ($products as $product) {
                $product->setMeta('featured', $isFeatured);
                $product->setMeta('featured_updated', now());
            }
        });
}
```

## Bulk Meta Deletion

### Delete Specific Meta Keys

```php
// Delete meta from all posts of a type
function deleteMetaFromPostType($postType, $metaKeys) {
    Post::query()
        ->where('post_type', $postType)
        ->chunk(100, function ($posts) use ($metaKeys) {
            foreach ($posts as $post) {
                foreach ($metaKeys as $key) {
                    $post->deleteMeta($key);
                }
            }
        });
}

// Usage
deleteMetaFromPostType('product', [
    'old_price_field',
    'deprecated_setting',
    'temp_cache'
]);
```

### Clean Up Orphaned Meta

```php
// Clean up meta for deleted posts
function cleanupOrphanedMeta() {
    // Using raw queries for efficiency
    $orphanedMeta = PostMeta::query()
        ->leftJoin('posts', 'postmeta.post_id', '=', 'posts.ID')
        ->whereNull('posts.ID')
        ->get();

    foreach ($orphanedMeta as $meta) {
        $meta->delete();
    }

    return count($orphanedMeta);
}

// Clean up specific meta keys
function cleanupMetaByKey($metaKey) {
    PostMeta::query()
        ->where('meta_key', $metaKey)
        ->delete();

    UserMeta::query()
        ->where('meta_key', $metaKey)
        ->delete();
}
```

## Advanced Meta Patterns

### Meta Synchronization

```php
// Sync meta between different models
function syncUserToPostMeta($userId, $postId, $metaKeys) {
    $user = User::find($userId);
    $post = Post::find($postId);

    if (!$user || !$post) {
        return false;
    }

    foreach ($metaKeys as $key) {
        $value = $user->getMetaValue($key);
        if ($value !== null) {
            $post->setMeta("author_{$key}", $value);
        }
    }

    return true;
}

// Usage
syncUserToPostMeta(1, 123, ['company', 'bio', 'social_links']);
```

### Meta Aggregation

```php
// Calculate aggregated meta values
function calculateAverageRating($postId) {
    $comments = Comment::query()
        ->where('comment_post_ID', $postId)
        ->where('comment_approved', '1')
        ->with('metas')
        ->get();

    $ratings = [];
    foreach ($comments as $comment) {
        $rating = $comment->getMetaValue('rating');
        if ($rating !== null) {
            $ratings[] = (float) $rating;
        }
    }

    if (empty($ratings)) {
        return null;
    }

    $average = array_sum($ratings) / count($ratings);

    // Store the calculated average
    $post = Post::find($postId);
    $post->setMeta('average_rating', round($average, 2));
    $post->setMeta('rating_count', count($ratings));

    return $average;
}
```

### Meta Versioning

```php
// Version control for meta values
function setVersionedMeta($model, $key, $value) {
    // Store current value as previous version
    $currentValue = $model->getMetaValue($key);
    if ($currentValue !== null) {
        $model->setMeta("{$key}_previous", $currentValue);
        $model->setMeta("{$key}_updated_at", now());
    }

    // Set new value
    $model->setMeta($key, $value);
    $model->setMeta("{$key}_version", time());
}

// Usage
$post = Post::find(123);
setVersionedMeta($post, 'price', 149.99);
```

## Meta Performance Optimization

### Avoiding N+1 Queries

```php
// Bad: N+1 query problem
$posts = Post::query()
    ->where('post_type', 'product')
    ->get();

foreach ($posts as $post) {
    $price = $post->getMetaValue('price'); // N+1 queries
}

// Good: Eager load meta
$posts = Post::query()
    ->where('post_type', 'product')
    ->with(['metas' => function ($query) {
        $query->whereIn('meta_key', ['price', 'featured', 'category']);
    }])
    ->get();

foreach ($posts as $post) {
    $price = $post->getMetaValue('price'); // No additional queries
}
```

### Bulk Meta Loading

```php
// Load specific meta for multiple models
function loadMetaForModels($models, $metaKeys) {
    $modelIds = $models->pluck('ID')->toArray();

    // Pre-load all needed meta
    $metaData = PostMeta::query()
        ->whereIn('post_id', $modelIds)
        ->whereIn('meta_key', $metaKeys)
        ->get()
        ->groupBy('post_id');

    // Attach meta to models
    foreach ($models as $model) {
        $modelMeta = $metaData->get($model->getKey(), collect());
        $model->setRelation('metas', $modelMeta);
    }

    return $models;
}

// Usage
$posts = Post::query()
    ->where('post_type', 'product')
    ->get();

$postsWithMeta = loadMetaForModels($posts, ['price', 'sku', 'category']);
```

### Meta Caching

```php
// Cache frequently accessed meta
function getCachedMeta($model, $key, $ttl = 3600) {
    $cacheKey = sprintf('meta_%s_%d_%s',
        get_class($model),
        $model->getKey(),
        $key
    );

    return cache()->remember($cacheKey, $ttl, function () use ($model, $key) {
        return $model->getMetaValue($key);
    });
}

// Cache meta for multiple models
function cacheMetaForModels($models, $metaKeys, $ttl = 3600) {
    foreach ($models as $model) {
        foreach ($metaKeys as $key) {
            getCachedMeta($model, $key, $ttl);
        }
    }
}
```

## Meta Type Casting

### Automatic Type Casting

```php
class Product extends Post
{
    protected array $metaCasts = [
        'price' => 'float',
        'featured' => 'boolean',
        'tags' => 'array',
        'created_date' => 'datetime',
        'specifications' => 'json',
    ];
}

$product = Product::find(123);

// Values are automatically cast to correct types
$price = $product->getMetaValue('price');        // float
$featured = $product->getMetaValue('featured');  // boolean
$tags = $product->getMetaValue('tags');          // array
$date = $product->getMetaValue('created_date');  // Carbon instance
```

### Custom Type Casting

```php
class Product extends Post
{
    protected array $metaCasts = [
        'price' => 'float',
        'currency' => 'string',
    ];

    // Custom getter with formatting
    public function getFormattedPrice(): string
    {
        $price = $this->getMetaValue('price');
        $currency = $this->getMetaValue('currency') ?? 'USD';

        return number_format($price, 2) . ' ' . $currency;
    }

    // Custom setter with validation
    public function setPrice(float $price): void
    {
        if ($price < 0) {
            throw new \InvalidArgumentException('Price cannot be negative');
        }

        $this->setMeta('price', $price);
        $this->setMeta('price_updated', now());
    }
}
```

## Meta Query Optimization

### Indexed Meta Queries

```php
// Optimize queries for frequently searched meta
function findProductsByPriceRange($minPrice, $maxPrice) {
    return Post::query()
        ->where('post_type', 'product')
        ->where('post_status', 'publish')
        ->addMetaToFilter('price', $minPrice, '>=')
        ->addMetaToFilter('price', $maxPrice, '<=')
        ->get();
}

// Use joins for better performance with multiple meta conditions
function findFeaturedProductsInCategory($category) {
    return Post::query()
        ->where('post_type', 'product')
        ->where('post_status', 'publish')
        ->join('postmeta as pm1', function ($join) {
            $join->on('posts.ID', '=', 'pm1.post_id')
                 ->where('pm1.meta_key', '=', 'featured')
                 ->where('pm1.meta_value', '=', '1');
        })
        ->join('postmeta as pm2', function ($join) use ($category) {
            $join->on('posts.ID', '=', 'pm2.post_id')
                 ->where('pm2.meta_key', '=', 'category')
                 ->where('pm2.meta_value', '=', $category);
        })
        ->select('posts.*')
        ->distinct()
        ->get();
}
```

## Best Practices for Meta Management

### 1. Use Consistent Naming

```php
// Good: Consistent prefix and naming
$product->setMeta('product_price', 99.99);
$product->setMeta('product_sku', 'PROD-001');
$product->setMeta('product_category', 'electronics');

// Bad: Inconsistent naming
$product->setMeta('price', 99.99);
$product->setMeta('SKU', 'PROD-001');
$product->setMeta('cat', 'electronics');
```

### 2. Validate Meta Data

```php
function setValidatedMeta($model, $key, $value, $rules = []) {
    // Basic validation
    if (isset($rules['required']) && $rules['required'] && empty($value)) {
        throw new \InvalidArgumentException("Meta key '{$key}' is required");
    }

    if (isset($rules['type'])) {
        $type = $rules['type'];
        if ($type === 'numeric' && !is_numeric($value)) {
            throw new \InvalidArgumentException("Meta key '{$key}' must be numeric");
        }
        if ($type === 'email' && !filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("Meta key '{$key}' must be a valid email");
        }
    }

    if (isset($rules['max_length']) && strlen($value) > $rules['max_length']) {
        throw new \InvalidArgumentException("Meta key '{$key}' exceeds maximum length");
    }

    $model->setMeta($key, $value);
}

// Usage
setValidatedMeta($user, 'email', 'john@example.com', [
    'required' => true,
    'type' => 'email'
]);
```

### 3. Clean Up Unused Meta

```php
// Regular cleanup of temporary meta
function cleanupTemporaryMeta($olderThan = '7 days') {
    $cutoffDate = now()->sub($olderThan);

    PostMeta::query()
        ->where('meta_key', 'like', 'temp_%')
        ->where('created_at', '<', $cutoffDate)
        ->delete();

    UserMeta::query()
        ->where('meta_key', 'like', 'cache_%')
        ->where('created_at', '<', $cutoffDate)
        ->delete();
}
```

### 4. Monitor Meta Performance

```php
// Track meta query performance
function trackMetaQuery($callback, $description = '') {
    $start = microtime(true);

    $result = $callback();

    $end = microtime(true);
    $duration = $end - $start;

    if ($duration > 1.0) { // Log slow queries
        error_log("Slow meta query ({$duration}s): {$description}");
    }

    return $result;
}

// Usage
$products = trackMetaQuery(function () {
    return Post::query()
        ->where('post_type', 'product')
        ->addMetaToFilter('featured', true)
        ->get();
}, 'Featured products query');
```

This comprehensive meta management guide covers all aspects of working with meta data efficiently in WordPress ORM.
