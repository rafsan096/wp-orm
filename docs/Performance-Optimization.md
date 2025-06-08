# Performance Optimization

This guide covers performance optimization techniques for WordPress ORM, including eager loading, avoiding N+1 queries, query optimization, and caching strategies.

## Understanding N+1 Query Problem

The N+1 query problem occurs when you fetch a collection of models and then access their relationships, causing additional queries for each model.

### Example of N+1 Problem

```php
// BAD: This creates N+1 queries
$posts = Post::query()
    ->where('post_status', 'publish')
    ->limit(10)
    ->get(); // 1 query

foreach ($posts as $post) {
    echo $post->author->display_name;     // +N queries (1 per post)
    echo $post->getMetaValue('featured'); // +N queries (1 per post)

    foreach ($post->comments as $comment) { // +N queries (1 per post)
        echo $comment->user->display_name;  // +N*M queries (1 per comment)
    }
}
// Total: 1 + N + N + N + (N*M) queries
```

### Solution with Eager Loading

```php
// GOOD: This uses only a few queries
$posts = Post::query()
    ->where('post_status', 'publish')
    ->with([
        'author',
        'metas' => function ($query) {
            $query->where('meta_key', 'featured');
        },
        'comments.user'
    ])
    ->limit(10)
    ->get(); // 4 queries total (posts, authors, metas, comments with users)

foreach ($posts as $post) {
    echo $post->author->display_name;     // No additional query
    echo $post->getMetaValue('featured'); // No additional query

    foreach ($post->comments as $comment) {
        echo $comment->user->display_name; // No additional query
    }
}
```

## Eager Loading Strategies

### Basic Eager Loading

```php
use Dbout\WpOrm\Models\Post;
use Dbout\WpOrm\Models\User;
use Dbout\WpOrm\Models\Comment;

// Load posts with authors
$posts = Post::query()
    ->with('author')
    ->where('post_status', 'publish')
    ->get();

// Load users with their posts and comments
$users = User::query()
    ->with(['posts', 'comments'])
    ->get();

// Load posts with nested relationships
$posts = Post::query()
    ->with([
        'author',
        'comments',
        'comments.user',
        'parent'
    ])
    ->get();
```

### Conditional Eager Loading

```php
// Load different relationships based on conditions
$posts = Post::query()
    ->when($includeAuthor, function ($query) {
        return $query->with('author');
    })
    ->when($includeMeta, function ($query) {
        return $query->with('metas');
    })
    ->get();

// Load relationships with constraints
$posts = Post::query()
    ->with([
        'author' => function ($query) {
            $query->select(['ID', 'display_name', 'user_email']);
        },
        'comments' => function ($query) {
            $query->where('comment_approved', '1')
                  ->orderBy('comment_date', 'desc')
                  ->limit(5);
        },
        'metas' => function ($query) {
            $query->whereIn('meta_key', ['featured', 'price', 'category']);
        }
    ])
    ->get();
```

### Lazy Eager Loading

```php
// Load relationships after the initial query
$posts = Post::query()
    ->where('post_status', 'publish')
    ->get();

// Later, when you need the relationships
$posts->load(['author', 'comments']);

// Load with constraints
$posts->load([
    'metas' => function ($query) {
        $query->whereIn('meta_key', ['price', 'featured']);
    }
]);
```

## Meta Data Optimization

### Efficient Meta Loading

```php
// BAD: N+1 queries for meta
$products = Post::query()
    ->where('post_type', 'product')
    ->get();

foreach ($products as $product) {
    $price = $product->getMetaValue('price');     // +N queries
    $featured = $product->getMetaValue('featured'); // +N queries
}

// GOOD: Eager load specific meta
$products = Post::query()
    ->where('post_type', 'product')
    ->with(['metas' => function ($query) {
        $query->whereIn('meta_key', ['price', 'featured', 'sku']);
    }])
    ->get();

foreach ($products as $product) {
    $price = $product->getMetaValue('price');     // No additional query
    $featured = $product->getMetaValue('featured'); // No additional query
}
```

### Meta Joins for Better Performance

```php
// Using joins instead of separate queries for meta filtering
$expensiveProducts = Post::query()
    ->where('post_type', 'product')
    ->join('postmeta', function ($join) {
        $join->on('posts.ID', '=', 'postmeta.post_id')
             ->where('postmeta.meta_key', '=', 'price');
    })
    ->where('postmeta.meta_value', '>', 100)
    ->select('posts.*')
    ->get();

// Multiple meta conditions with joins
$featuredExpensiveProducts = Post::query()
    ->where('post_type', 'product')
    ->join('postmeta as price_meta', function ($join) {
        $join->on('posts.ID', '=', 'price_meta.post_id')
             ->where('price_meta.meta_key', '=', 'price');
    })
    ->join('postmeta as featured_meta', function ($join) {
        $join->on('posts.ID', '=', 'featured_meta.post_id')
             ->where('featured_meta.meta_key', '=', 'featured');
    })
    ->where('price_meta.meta_value', '>', 100)
    ->where('featured_meta.meta_value', '=', '1')
    ->select('posts.*')
    ->distinct()
    ->get();
```

### Using addMetaToSelect for Performance

```php
// Include meta values directly in the result set
$products = Post::query()
    ->where('post_type', 'product')
    ->addMetasToSelect(['price', 'sku', 'category'])
    ->get();

foreach ($products as $product) {
    // Access meta values directly from the model attributes
    echo $product->price_value;    // No additional query
    echo $product->sku_value;      // No additional query
    echo $product->category_value; // No additional query
}
```

## Query Optimization Techniques

### Selective Column Loading

```php
// BAD: Loading all columns
$posts = Post::query()
    ->where('post_status', 'publish')
    ->get(); // Loads all columns

// GOOD: Load only needed columns
$posts = Post::query()
    ->select(['ID', 'post_title', 'post_date', 'post_author'])
    ->where('post_status', 'publish')
    ->get();

// With relationships
$posts = Post::query()
    ->select(['ID', 'post_title', 'post_author'])
    ->with(['author' => function ($query) {
        $query->select(['ID', 'display_name']);
    }])
    ->where('post_status', 'publish')
    ->get();
```

### Chunking Large Datasets

```php
// Process large datasets in chunks to avoid memory issues
Post::query()
    ->where('post_type', 'product')
    ->chunk(100, function ($products) {
        foreach ($products as $product) {
            // Process each product
            $product->setMeta('processed_at', now());
        }
    });

// Chunk with eager loading
Post::query()
    ->where('post_type', 'product')
    ->with('metas')
    ->chunk(100, function ($products) {
        foreach ($products as $product) {
            // Access meta without additional queries
            $price = $product->getMetaValue('price');
        }
    });
```

### Using Cursor Pagination

```php
// For very large datasets, use cursor pagination
function getProductsCursor($cursor = null, $limit = 100) {
    $query = Post::query()
        ->where('post_type', 'product')
        ->orderBy('ID');

    if ($cursor) {
        $query->where('ID', '>', $cursor);
    }

    return $query->limit($limit)->get();
}

// Usage
$cursor = null;
do {
    $products = getProductsCursor($cursor, 100);

    foreach ($products as $product) {
        // Process product
        $cursor = $product->ID;
    }
} while ($products->count() === 100);
```

## Caching Strategies

### Model Caching

```php
// Cache individual models
function getCachedPost($id) {
    $cacheKey = "post_{$id}";

    return cache()->remember($cacheKey, 3600, function () use ($id) {
        return Post::query()
            ->with(['author', 'metas'])
            ->find($id);
    });
}

// Cache collections with relationships
function getCachedFeaturedPosts() {
    return cache()->remember('featured_posts', 1800, function () {
        return Post::query()
            ->where('post_status', 'publish')
            ->addMetaToFilter('featured', true)
            ->with(['author', 'metas'])
            ->orderBy('post_date', 'desc')
            ->limit(10)
            ->get();
    });
}
```

### Query Result Caching

```php
// Cache query results
class ProductService
{
    public function getExpensiveProducts($minPrice = 100)
    {
        $cacheKey = "expensive_products_{$minPrice}";

        return cache()->remember($cacheKey, 3600, function () use ($minPrice) {
            return Post::query()
                ->where('post_type', 'product')
                ->addMetaToFilter('price', $minPrice, '>=')
                ->with(['metas' => function ($query) {
                    $query->whereIn('meta_key', ['price', 'sku', 'category']);
                }])
                ->get();
        });
    }

    public function invalidateProductCache()
    {
        // Clear related cache when products are updated
        cache()->forget('expensive_products_100');
        cache()->forget('featured_products');
    }
}
```

### Meta Value Caching

```php
// Cache frequently accessed meta values
function getCachedMetaValue($model, $key, $ttl = 3600)
{
    $cacheKey = sprintf(
        'meta_%s_%d_%s',
        get_class($model),
        $model->getKey(),
        $key
    );

    return cache()->remember($cacheKey, $ttl, function () use ($model, $key) {
        return $model->getMetaValue($key);
    });
}

// Batch cache meta values
function cacheMetaValues($models, $metaKeys, $ttl = 3600)
{
    foreach ($models as $model) {
        foreach ($metaKeys as $key) {
            getCachedMetaValue($model, $key, $ttl);
        }
    }
}
```

## Database Indexing

### Recommended Indexes

```sql
-- Indexes for better performance (add these to your database)

-- Posts table indexes
ALTER TABLE wp_posts ADD INDEX idx_post_type_status (post_type, post_status);
ALTER TABLE wp_posts ADD INDEX idx_post_date (post_date);
ALTER TABLE wp_posts ADD INDEX idx_post_author (post_author);
ALTER TABLE wp_posts ADD INDEX idx_post_name (post_name);

-- Meta table indexes
ALTER TABLE wp_postmeta ADD INDEX idx_meta_key_value (meta_key, meta_value(100));
ALTER TABLE wp_postmeta ADD INDEX idx_post_id_meta_key (post_id, meta_key);

ALTER TABLE wp_usermeta ADD INDEX idx_meta_key_value (meta_key, meta_value(100));
ALTER TABLE wp_usermeta ADD INDEX idx_user_id_meta_key (user_id, meta_key);

-- Comments table indexes
ALTER TABLE wp_comments ADD INDEX idx_comment_post_approved (comment_post_ID, comment_approved);
ALTER TABLE wp_comments ADD INDEX idx_comment_date (comment_date);
```

### Query-Specific Indexes

```php
// For frequently executed queries, consider specific indexes

// If you often query products by price range:
// ALTER TABLE wp_postmeta ADD INDEX idx_price_range (meta_key, CAST(meta_value AS DECIMAL(10,2)))
// WHERE meta_key = 'price';

// Query that benefits from the index
$products = Post::query()
    ->where('post_type', 'product')
    ->join('postmeta', function ($join) {
        $join->on('posts.ID', '=', 'postmeta.post_id')
             ->where('postmeta.meta_key', '=', 'price');
    })
    ->whereBetween('postmeta.meta_value', [50, 200])
    ->select('posts.*')
    ->get();
```

## Monitoring and Profiling

### Query Logging

```php
// Enable query logging for development
use Illuminate\Support\Facades\DB;

// Log all queries
DB::enableQueryLog();

// Execute your code
$posts = Post::query()
    ->with(['author', 'comments'])
    ->where('post_status', 'publish')
    ->get();

// Review executed queries
$queries = DB::getQueryLog();
foreach ($queries as $query) {
    echo $query['query'] . "\n";
    echo "Time: " . $query['time'] . "ms\n";
    echo "Bindings: " . implode(', ', $query['bindings']) . "\n\n";
}
```

### Performance Monitoring

```php
// Monitor query performance
class QueryMonitor
{
    private static $slowQueries = [];

    public static function trackQuery($callback, $description = '')
    {
        $start = microtime(true);

        $result = $callback();

        $duration = microtime(true) - $start;

        if ($duration > 0.5) { // Log queries taking more than 500ms
            self::$slowQueries[] = [
                'description' => $description,
                'duration' => $duration,
                'timestamp' => now()
            ];

            error_log("Slow query detected: {$description} ({$duration}s)");
        }

        return $result;
    }

    public static function getSlowQueries()
    {
        return self::$slowQueries;
    }
}

// Usage
$posts = QueryMonitor::trackQuery(function () {
    return Post::query()
        ->with(['author', 'comments.user'])
        ->where('post_status', 'publish')
        ->get();
}, 'Loading posts with relationships');
```

## Advanced Optimization Patterns

### Repository Pattern with Caching

```php
class PostRepository
{
    public function findFeaturedPosts($limit = 10)
    {
        return cache()->remember(
            "featured_posts_{$limit}",
            1800,
            fn() => Post::query()
                ->where('post_status', 'publish')
                ->addMetaToFilter('featured', true)
                ->with(['author'])
                ->orderBy('post_date', 'desc')
                ->limit($limit)
                ->get()
        );
    }

    public function findPostsByAuthor($authorId, $limit = 10)
    {
        return cache()->remember(
            "author_posts_{$authorId}_{$limit}",
            3600,
            fn() => Post::query()
                ->where('post_author', $authorId)
                ->where('post_status', 'publish')
                ->orderBy('post_date', 'desc')
                ->limit($limit)
                ->get()
        );
    }

    public function invalidateCache($postId = null)
    {
        if ($postId) {
            $post = Post::find($postId);
            cache()->forget("author_posts_{$post->post_author}_10");
        }

        cache()->forget('featured_posts_10');
        cache()->tags(['posts'])->flush();
    }
}
```

### Batch Loading Pattern

```php
class BatchLoader
{
    public static function loadPostsWithMeta($postIds, $metaKeys = [])
    {
        // Load posts
        $posts = Post::query()
            ->whereIn('ID', $postIds)
            ->get()
            ->keyBy('ID');

        if (!empty($metaKeys)) {
            // Batch load meta
            $metaData = PostMeta::query()
                ->whereIn('post_id', $postIds)
                ->whereIn('meta_key', $metaKeys)
                ->get()
                ->groupBy('post_id');

            // Attach meta to posts
            foreach ($posts as $post) {
                $postMeta = $metaData->get($post->ID, collect());
                $post->setRelation('metas', $postMeta);
            }
        }

        return $posts;
    }
}

// Usage
$postIds = [1, 2, 3, 4, 5];
$posts = BatchLoader::loadPostsWithMeta($postIds, ['price', 'featured']);
```

## Best Practices Summary

### 1. Always Use Eager Loading

```php
// Good
$posts = Post::query()
    ->with(['author', 'comments'])
    ->where('post_status', 'publish')
    ->get();

// Bad
$posts = Post::query()
    ->where('post_status', 'publish')
    ->get();
// Then accessing relationships in a loop
```

### 2. Load Only What You Need

```php
// Good: Specific columns and constrained relationships
$posts = Post::query()
    ->select(['ID', 'post_title', 'post_author'])
    ->with(['author' => function ($query) {
        $query->select(['ID', 'display_name']);
    }])
    ->get();

// Bad: Loading everything
$posts = Post::query()->with('author')->get();
```

### 3. Use Chunking for Large Datasets

```php
// Good: Process in chunks
Post::query()
    ->where('post_type', 'product')
    ->chunk(100, function ($products) {
        // Process products
    });

// Bad: Load everything at once
$products = Post::query()
    ->where('post_type', 'product')
    ->get(); // Could use too much memory
```

### 4. Cache Frequently Accessed Data

```php
// Cache expensive queries
$featuredPosts = cache()->remember('featured_posts', 1800, function () {
    return Post::query()
        ->where('post_status', 'publish')
        ->addMetaToFilter('featured', true)
        ->with('author')
        ->get();
});
```

### 5. Monitor and Profile

```php
// Always monitor performance in production
DB::listen(function ($query) {
    if ($query->time > 1000) { // Log queries over 1 second
        Log::warning('Slow query detected', [
            'sql' => $query->sql,
            'time' => $query->time,
            'bindings' => $query->bindings
        ]);
    }
});
```

This comprehensive performance guide will help you build efficient, scalable WordPress applications using the ORM.
