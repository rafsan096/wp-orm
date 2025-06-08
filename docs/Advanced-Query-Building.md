# Advanced Query Building

WordPress ORM provides powerful query building capabilities through Eloquent's query builder with WordPress-specific enhancements.

## Meta Queries

### Basic Meta Queries

```php
use Dbout\WpOrm\Models\Post;

// Filter posts by meta value
$posts = Post::query()
    ->addMetaToFilter('featured', 'yes')
    ->get();

// Filter with operators
$posts = Post::query()
    ->addMetaToFilter('price', 100, '>')
    ->get();

// Multiple meta filters
$posts = Post::query()
    ->addMetaToFilter('category', 'electronics')
    ->addMetaToFilter('price', 50, '>=')
    ->addMetaToFilter('price', 500, '<=')
    ->get();
```

### Including Meta in Results

```php
// Add meta values to select
$posts = Post::query()
    ->addMetaToSelect('price')
    ->addMetaToSelect('category')
    ->get();

// Add multiple metas with custom aliases
$posts = Post::query()
    ->addMetasToSelect([
        'product_price' => 'price',
        'product_category' => 'category',
        'featured_status' => 'featured'
    ])
    ->get();

// Access meta values
foreach ($posts as $post) {
    echo $post->price_value; // Default alias format
    echo $post->product_price; // Custom alias
}
```

### Advanced Meta Joins

```php
// Custom join types
$posts = Post::query()
    ->joinToMeta('featured', 'left')
    ->get();

// Complex meta queries with raw SQL
$posts = Post::query()
    ->joinToMeta('price')
    ->whereRaw('CAST(price.meta_value AS DECIMAL(10,2)) BETWEEN ? AND ?', [50, 200])
    ->get();
```

## Post Type Specific Queries

### Built-in Post Types

```php
use Dbout\WpOrm\Models\Post;
use Dbout\WpOrm\Models\Article;
use Dbout\WpOrm\Models\Page;
use Dbout\WpOrm\Models\Attachment;

// Articles (post type: 'post')
$articles = Article::query()
    ->published()
    ->orderBy('post_date', 'desc')
    ->get();

// Pages
$pages = Page::query()
    ->where('post_status', 'publish')
    ->get();

// Attachments
$images = Attachment::query()
    ->where('post_mime_type', 'LIKE', 'image/%')
    ->get();
```

### Custom Post Types

```php
// Generic post queries for custom post types
$products = Post::query()
    ->where('post_type', 'product')
    ->where('post_status', 'publish')
    ->get();

// Using ofType scope (if available)
$products = Post::query()
    ->ofType('product')
    ->published()
    ->get();
```

## User Queries

### Basic User Filtering

```php
use Dbout\WpOrm\Models\User;

// Find users by role (using meta)
$editors = User::query()
    ->addMetaToFilter('wp_capabilities', '%editor%', 'LIKE')
    ->get();

// Users with specific meta
$users = User::query()
    ->addMetaToFilter('company', 'ACME Corp')
    ->get();
```

### User Relationships

```php
// Get users with their posts
$users = User::query()
    ->with('posts')
    ->get();

// Users who have published posts
$authors = User::query()
    ->whereHas('posts', function ($query) {
        $query->where('post_status', 'publish')
              ->where('post_type', 'post');
    })
    ->get();
```

## Comment Queries

### Advanced Comment Filtering

```php
use Dbout\WpOrm\Models\Comment;

// Comments with meta filtering
$comments = Comment::query()
    ->addMetaToFilter('rating', 5)
    ->where('comment_approved', '1')
    ->get();

// Comments by post type
$comments = Comment::query()
    ->whereHas('post', function ($query) {
        $query->where('post_type', 'product');
    })
    ->get();
```

## Scopes and Custom Query Methods

### Using Scopes

```php
// Predefined scopes (check individual models for available scopes)
$posts = Post::query()
    ->published()
    ->recent()
    ->get();

// Combining scopes
$articles = Article::query()
    ->published()
    ->featured()
    ->orderBy('post_date', 'desc')
    ->get();
```

### Custom Query Methods

```php
// Find by slug
$post = Post::findOneByName('my-post-slug');

// Find by GUID
$post = Post::findOneByGuid('http://example.com/?p=123');

// Find user by login
$user = User::findOneByLogin('username');
```

## Eager Loading

### Loading Relationships

```php
// Load post with author and comments
$posts = Post::query()
    ->with(['author', 'comments'])
    ->get();

// Load with meta
$posts = Post::query()
    ->with('metas')
    ->get();

// Conditional eager loading
$posts = Post::query()
    ->with(['comments' => function ($query) {
        $query->where('comment_approved', '1');
    }])
    ->get();
```

## Raw Queries and Complex Joins

### Raw SQL Integration

```php
use Dbout\WpOrm\Orm\Database;

// Raw queries
$results = Database::getInstance()
    ->table('posts')
    ->whereRaw('YEAR(post_date) = ?', [2024])
    ->get();

// Complex joins
$posts = Post::query()
    ->join('postmeta', 'posts.ID', '=', 'postmeta.post_id')
    ->where('postmeta.meta_key', 'featured')
    ->where('postmeta.meta_value', 'yes')
    ->select('posts.*')
    ->get();
```

### Subqueries

```php
// Posts with the most comments
$posts = Post::query()
    ->whereIn('ID', function ($query) {
        $query->select('comment_post_ID')
              ->from('comments')
              ->where('comment_approved', '1')
              ->groupBy('comment_post_ID')
              ->havingRaw('COUNT(*) > 10');
    })
    ->get();
```

## Performance Optimization

### Query Optimization

```php
// Select only needed columns
$posts = Post::query()
    ->select(['ID', 'post_title', 'post_date'])
    ->get();

// Limit results
$posts = Post::query()
    ->limit(10)
    ->offset(20)
    ->get();

// Use chunks for large datasets
Post::query()
    ->where('post_type', 'post')
    ->chunk(100, function ($posts) {
        foreach ($posts as $post) {
            // Process each post
        }
    });
```

### Caching Considerations

```php
// Remember query results
$posts = Post::query()
    ->where('post_status', 'publish')
    ->remember(60) // Cache for 60 minutes (if cache driver available)
    ->get();
```

## Working with Dates

### Date Filtering

```php
use Carbon\Carbon;

// Posts from last 30 days
$posts = Post::query()
    ->where('post_date', '>=', Carbon::now()->subDays(30))
    ->get();

// Posts published this year
$posts = Post::query()
    ->whereYear('post_date', date('Y'))
    ->get();

// Date range
$posts = Post::query()
    ->whereBetween('post_date', [
        Carbon::parse('2024-01-01'),
        Carbon::parse('2024-12-31')
    ])
    ->get();
```

## Error Handling

### Query Exception Handling

```php
try {
    $posts = Post::query()
        ->addMetaToFilter('complex_meta', 'value')
        ->get();
} catch (\Dbout\WpOrm\Exceptions\WpOrmException $e) {
    // Handle WP ORM specific exceptions
    error_log('WP ORM Error: ' . $e->getMessage());
} catch (\Exception $e) {
    // Handle general exceptions
    error_log('General Error: ' . $e->getMessage());
}
```

## Best Practices

1. **Use Specific Queries**: Always specify post types, statuses, and other filters
2. **Eager Load Relationships**: Use `with()` to avoid N+1 queries
3. **Limit Results**: Use `limit()` and pagination for large datasets
4. **Index Meta Queries**: Consider database indexes for frequently queried meta keys
5. **Cache Results**: Cache expensive queries when possible
6. **Use Raw Queries Sparingly**: Prefer the query builder over raw SQL when possible
