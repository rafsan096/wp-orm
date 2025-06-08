# Troubleshooting Guide

This guide covers common issues you might encounter when using WordPress ORM and how to resolve them.

## Installation Issues

### Composer Installation Problems

**Problem:** Composer installation fails with dependency conflicts.

**Solution:**

```bash
# Update composer first
composer self-update

# Clear composer cache
composer clear-cache

# Try installing with specific flags
composer require dbout/wp-orm --prefer-dist --optimize-autoloader

# If still having issues, install with no scripts
composer require dbout/wp-orm --no-scripts
```

### Autoloader Not Found

**Problem:** `Class 'Dbout\WpOrm\Models\Post' not found`

**Solution:**

```php
// Make sure autoloader is included in wp-config.php
require_once __DIR__ . '/vendor/autoload.php';

// Or in your theme/plugin
require_once ABSPATH . '/vendor/autoload.php';
```

### PHP Version Issues

**Problem:** Fatal error related to PHP version compatibility.

**Solution:**

- Ensure you're running PHP 8.2 or higher
- Check your PHP version: `php -v`
- Update your hosting PHP version if needed
- Verify composer.json requirements

## Database Connection Issues

### Connection Failed

**Problem:** Cannot connect to WordPress database.

**Solution:**

```php
// Test WordPress database connection
global $wpdb;
if ($wpdb->last_error) {
    error_log('Database error: ' . $wpdb->last_error);
}

// Check if WordPress is properly loaded
if (!defined('ABSPATH')) {
    die('WordPress not loaded');
}

// Verify wp-config.php database settings
if (!defined('DB_NAME') || !defined('DB_USER')) {
    die('Database constants not defined');
}
```

### Table Prefix Issues

**Problem:** Queries fail due to incorrect table prefix.

**Solution:**

```php
// Check your table prefix
global $wpdb;
echo 'Table prefix: ' . $wpdb->prefix;

// Verify tables exist
$tables = $wpdb->get_results("SHOW TABLES LIKE '{$wpdb->prefix}posts'");
if (empty($tables)) {
    die('WordPress tables not found');
}
```

## Query Issues

### Meta Queries Not Working

**Problem:** Meta queries return empty results or fail.

**Solution:**

```php
try {
    // Enable query logging for debugging
    \Illuminate\Support\Facades\DB::enableQueryLog();

    $posts = Post::query()
        ->addMetaToFilter('featured', 'yes')
        ->get();

    // Check executed queries
    $queries = \Illuminate\Support\Facades\DB::getQueryLog();
    foreach ($queries as $query) {
        error_log($query['query']);
    }
} catch (\Exception $e) {
    error_log('Query error: ' . $e->getMessage());
}
```

**Common meta query issues:**

```php
// Wrong: Meta value type mismatch
Post::query()->addMetaToFilter('price', '100'); // String instead of number

// Correct: Use proper types
Post::query()->addMetaToFilter('price', 100); // Integer
Post::query()->addMetaToFilter('featured', true); // Boolean
```

### Relationship Loading Issues

**Problem:** N+1 query problems or missing relationships.

**Solution:**

```php
// Bad: Causes N+1 queries
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name; // N+1 problem
}

// Good: Eager load relationships
$posts = Post::query()
    ->with('author')
    ->get();

foreach ($posts as $post) {
    echo $post->author->name; // No additional queries
}
```

### Custom Post Type Issues

**Problem:** Custom post types not appearing in queries.

**Solution:**

```php
// Make sure to filter by post_type
$products = Post::query()
    ->where('post_type', 'product') // Don't forget this!
    ->where('post_status', 'publish')
    ->get();

// Or use custom model with automatic filtering
class Product extends CustomPost
{
    protected static string $postType = 'product';
}

$products = Product::all(); // Automatically filtered
```

## Performance Issues

### Slow Queries

**Problem:** Queries are taking too long to execute.

**Solution:**

```php
// Profile your queries
$start = microtime(true);

$posts = Post::query()
    ->with('author', 'metas')
    ->where('post_status', 'publish')
    ->get();

$end = microtime(true);
error_log('Query took: ' . ($end - $start) . ' seconds');

// Optimize with selective loading
$posts = Post::query()
    ->select(['ID', 'post_title', 'post_date', 'post_author'])
    ->with(['author:ID,user_login,display_name'])
    ->where('post_status', 'publish')
    ->limit(10)
    ->get();
```

**Performance optimization tips:**

```php
// Use chunks for large datasets
Post::query()
    ->where('post_type', 'post')
    ->chunk(100, function ($posts) {
        foreach ($posts as $post) {
            // Process each post
        }
    });

// Limit meta loading
$posts = Post::query()
    ->with(['metas' => function ($query) {
        $query->whereIn('meta_key', ['featured', 'price']);
    }])
    ->get();

// Use exists() instead of count() > 0
if (Post::query()->where('post_author', 1)->exists()) {
    // More efficient than count() > 0
}
```

### Memory Issues

**Problem:** PHP memory limit exceeded.

**Solution:**

```php
// Increase memory limit temporarily
ini_set('memory_limit', '512M');

// Use cursor for very large datasets
foreach (Post::query()->cursor() as $post) {
    // Process one post at a time
}

// Use chunks instead of get() for large results
Post::query()
    ->chunk(50, function ($posts) {
        foreach ($posts as $post) {
            // Process posts in batches
        }
    });
```

## Model Issues

### Meta Casting Problems

**Problem:** Meta values not casting correctly.

**Solution:**

```php
// Define meta casts in your model
class Product extends Post
{
    protected array $metaCasts = [
        'price' => 'float',
        'featured' => 'boolean',
        'tags' => 'array',
        'release_date' => 'datetime',
    ];
}

// Check if casting is working
$product = Product::find(1);
var_dump($product->getMetaValue('price')); // Should be float
var_dump($product->getMetaValue('featured')); // Should be boolean
```

### Custom Model Issues

**Problem:** Custom models not working as expected.

**Solution:**

```php
// Make sure to extend the correct base class
class Product extends \Dbout\WpOrm\Models\CustomPost // Correct
{
    protected static string $postType = 'product';
}

// Not this
class Product extends \Dbout\WpOrm\Models\Post // Wrong for custom post types
{
    // ...
}

// Implement required interfaces for meta support
class Product extends \Dbout\WpOrm\Models\CustomPost implements \Dbout\WpOrm\Api\WithMetaModelInterface
{
    use \Dbout\WpOrm\Concerns\HasMetas;

    public function getMetaConfigMapping(): \Dbout\WpOrm\MetaMappingConfig
    {
        return new \Dbout\WpOrm\MetaMappingConfig(
            \Dbout\WpOrm\Models\Meta\PostMeta::class,
            \Dbout\WpOrm\Models\Meta\PostMeta::POST_ID
        );
    }
}
```

## WordPress Integration Issues

### Plugin/Theme Conflicts

**Problem:** WordPress ORM conflicts with other plugins or themes.

**Solution:**

```php
// Check if WordPress ORM is loaded properly
if (!class_exists('\Dbout\WpOrm\Models\Post')) {
    error_log('WordPress ORM not loaded');
    return;
}

// Wrap ORM calls in try-catch
try {
    $posts = Post::query()->get();
} catch (\Exception $e) {
    error_log('WordPress ORM error: ' . $e->getMessage());
    // Fallback to WordPress native functions
    $posts = get_posts();
}
```

### Hook Conflicts

**Problem:** WordPress hooks interfering with ORM operations.

**Solution:**

```php
// Temporarily remove problematic hooks
$priority = has_action('save_post', 'your_function');
if ($priority !== false) {
    remove_action('save_post', 'your_function', $priority);
}

// Perform ORM operations
$post = new Post();
$post->setPostTitle('Test');
$post->save();

// Restore hooks
if ($priority !== false) {
    add_action('save_post', 'your_function', $priority);
}
```

## Debugging Tips

### Enable Debug Logging

```php
// Add to wp-config.php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);

// Log WordPress ORM queries
add_action('init', function() {
    if (class_exists('\Illuminate\Database\Capsule\Manager')) {
        \Illuminate\Database\Capsule\Manager::connection()->enableQueryLog();
    }
});
```

### Query Debugging

```php
// Debug specific queries
$query = Post::query()
    ->where('post_type', 'product')
    ->addMetaToFilter('featured', true);

// Get the SQL
echo $query->toSql();

// Get with bindings
dd($query->toSql(), $query->getBindings());

// Execute and debug
try {
    $results = $query->get();
    error_log('Found ' . count($results) . ' results');
} catch (\Exception $e) {
    error_log('Query failed: ' . $e->getMessage());
    error_log('SQL: ' . $query->toSql());
}
```

### Model State Debugging

```php
// Check model state
$post = Post::find(123);

if (!$post) {
    error_log('Post not found');
    return;
}

// Debug model attributes
error_log('Post attributes: ' . print_r($post->getAttributes(), true));

// Debug relationships
error_log('Author: ' . ($post->author ? $post->author->user_login : 'None'));

// Debug meta
error_log('Featured: ' . ($post->getMetaValue('featured') ? 'Yes' : 'No'));
```

## Common Error Messages

### "Class not found" Errors

```
Fatal error: Class 'Dbout\WpOrm\Models\Post' not found
```

**Solution:** Check autoloader inclusion and namespace imports.

### "Table doesn't exist" Errors

```
Table 'database.wp_posts' doesn't exist
```

**Solution:** Verify database connection and table prefix configuration.

### "Method not found" Errors

```
Call to undefined method addMetaToFilter()
```

**Solution:** Ensure you're using the correct builder class and model.

### "Meta not supported" Errors

```
MetaNotSupportedException: Model must implement WithMetaModelInterface
```

**Solution:** Implement the required interface and trait for meta support.

## Getting Help

If you're still experiencing issues:

1. **Check the GitHub Issues**: Look for similar problems
2. **Enable Debug Mode**: Get detailed error messages
3. **Isolate the Problem**: Create minimal reproduction case
4. **Check WordPress Logs**: Look in `/wp-content/debug.log`
5. **Verify Environment**: PHP version, WordPress version, plugin conflicts

### Creating a Debug Script

```php
<?php
// debug-wp-orm.php
require_once __DIR__ . '/wp-config.php';
require_once __DIR__ . '/wp-load.php';

try {
    // Test basic functionality
    echo "Testing WordPress ORM...\n";

    // Test database connection
    global $wpdb;
    echo "Database prefix: " . $wpdb->prefix . "\n";

    // Test model loading
    $postCount = \Dbout\WpOrm\Models\Post::count();
    echo "Total posts: " . $postCount . "\n";

    // Test meta functionality
    $post = \Dbout\WpOrm\Models\Post::first();
    if ($post) {
        echo "First post title: " . $post->getPostTitle() . "\n";
        echo "First post meta count: " . $post->metas()->count() . "\n";
    }

    echo "WordPress ORM is working correctly!\n";

} catch (\Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
    echo "File: " . $e->getFile() . ":" . $e->getLine() . "\n";
    echo "Trace:\n" . $e->getTraceAsString() . "\n";
}
```

Run this script to diagnose basic functionality issues.
