# CRUD Operations

This guide covers comprehensive Create, Read, Update, and Delete operations for all WordPress ORM models with practical examples and best practices.

## Create Operations

### Creating Posts

```php
use Dbout\WpOrm\Models\Post;
use Dbout\WpOrm\Enums\PostStatus;

// Basic post creation
$post = new Post();
$post->setPostTitle('My New Post');
$post->setPostContent('This is the content of my post.');
$post->setPostType('post');
$post->setPostStatus(PostStatus::PUBLISH);
$post->setPostAuthor(1);
$post->save();

// Creation with meta data
$post = new Post([
    'post_title' => 'Product Review',
    'post_content' => 'Detailed review content...',
    'post_type' => 'post',
    'post_status' => 'publish',
    'post_author' => 1
]);
$post->save();

// Add meta after creation
$post->setMeta('rating', 5);
$post->setMeta('category', 'electronics');
$post->setMeta('featured', true);

// Bulk create with meta
$posts = [];
$postData = [
    ['title' => 'Post 1', 'content' => 'Content 1'],
    ['title' => 'Post 2', 'content' => 'Content 2'],
    ['title' => 'Post 3', 'content' => 'Content 3'],
];

foreach ($postData as $data) {
    $post = new Post();
    $post->setPostTitle($data['title']);
    $post->setPostContent($data['content']);
    $post->setPostType('post');
    $post->setPostStatus('publish');
    $post->save();
    $posts[] = $post;
}
```

### Creating Users

```php
use Dbout\WpOrm\Models\User;

// Basic user creation
$user = new User();
$user->setUserLogin('johndoe');
$user->setUserEmail('john@example.com');
$user->setUserPass(wp_hash_password('secure_password'));
$user->setDisplayName('John Doe');
$user->setUserStatus(0);
$user->save();

// User with meta
$user->setMeta('first_name', 'John');
$user->setMeta('last_name', 'Doe');
$user->setMeta('company', 'ACME Corp');
$user->setMeta('wp_capabilities', serialize(['subscriber' => true]));

// Bulk user creation
$userData = [
    ['login' => 'user1', 'email' => 'user1@example.com', 'name' => 'User One'],
    ['login' => 'user2', 'email' => 'user2@example.com', 'name' => 'User Two'],
];

foreach ($userData as $data) {
    $user = new User();
    $user->setUserLogin($data['login']);
    $user->setUserEmail($data['email']);
    $user->setDisplayName($data['name']);
    $user->setUserPass(wp_hash_password('default_password'));
    $user->save();
}
```

### Creating Comments

```php
use Dbout\WpOrm\Models\Comment;

// Basic comment creation
$comment = new Comment();
$comment->setCommentPostID(123);
$comment->setCommentAuthor('Jane Smith');
$comment->setCommentAuthorEmail('jane@example.com');
$comment->setCommentContent('Great post! Very informative.');
$comment->setCommentApproved('1');
$comment->setCommentDate(now());
$comment->save();

// Comment by registered user
$comment = new Comment();
$comment->setCommentPostID(123);
$comment->setUserId(5);
$comment->setCommentContent('Thanks for sharing this!');
$comment->setCommentApproved('1');
$comment->save();

// Reply to another comment
$reply = new Comment();
$reply->setCommentPostID(123);
$reply->setCommentParent(456); // Parent comment ID
$reply->setCommentAuthor('Admin');
$reply->setCommentContent('Thank you for your feedback!');
$reply->setCommentApproved('1');
$reply->save();
```

### Creating Custom Post Types

```php
use Dbout\WpOrm\Models\Post;

// Create product
$product = new Post();
$product->setPostType('product');
$product->setPostTitle('Amazing Widget');
$product->setPostContent('Product description here...');
$product->setPostStatus('publish');
$product->save();

// Add product meta
$product->setMeta('price', 99.99);
$product->setMeta('sku', 'WIDGET-001');
$product->setMeta('inventory', 50);
$product->setMeta('featured', true);

// Create event
$event = new Post();
$event->setPostType('event');
$event->setPostTitle('WordPress Meetup');
$event->setPostContent('Join us for a great WordPress discussion!');
$event->setPostStatus('publish');
$event->save();

$event->setMeta('event_date', '2024-03-15 18:00:00');
$event->setMeta('location', 'Downtown Community Center');
$event->setMeta('max_attendees', 100);
```

## Read Operations

### Reading Single Records

```php
// Find by ID
$post = Post::find(123);
$user = User::find(1);
$comment = Comment::find(456);

// Find with relationships
$post = Post::query()
    ->with(['author', 'comments', 'metas'])
    ->find(123);

// Find by specific field
$post = Post::findOneByName('my-post-slug');
$user = User::findOneByEmail('john@example.com');
$user = User::findOneByLogin('johndoe');

// Find or fail (throws exception if not found)
$post = Post::findOrFail(123);

// Find with conditions
$post = Post::query()
    ->where('post_type', 'product')
    ->where('post_status', 'publish')
    ->first();
```

### Reading Multiple Records

```php
// Get all records
$posts = Post::all();
$users = User::all();

// Get with conditions
$publishedPosts = Post::query()
    ->where('post_status', 'publish')
    ->where('post_type', 'post')
    ->get();

// Get with relationships
$posts = Post::query()
    ->with(['author', 'comments'])
    ->where('post_status', 'publish')
    ->get();

// Paginated results
$posts = Post::query()
    ->where('post_status', 'publish')
    ->orderBy('post_date', 'desc')
    ->paginate(10);

// Get specific columns
$posts = Post::query()
    ->select(['ID', 'post_title', 'post_date'])
    ->where('post_status', 'publish')
    ->get();
```

### Reading with Meta Data

```php
// Get model with all meta
$post = Post::query()
    ->with('metas')
    ->find(123);

// Access meta values
$featuredValue = $post->getMetaValue('featured');
$price = $post->getMetaValue('price');

// Get models by meta value
$featuredPosts = Post::query()
    ->addMetaToFilter('featured', true)
    ->get();

// Include meta in results
$products = Post::query()
    ->where('post_type', 'product')
    ->addMetasToSelect(['price', 'sku', 'category'])
    ->get();

foreach ($products as $product) {
    echo "Price: " . $product->price_value;
    echo "SKU: " . $product->sku_value;
}
```

### Complex Queries

```php
// Multiple conditions
$posts = Post::query()
    ->where('post_type', 'post')
    ->where('post_status', 'publish')
    ->where('post_date', '>=', '2024-01-01')
    ->whereNotNull('post_excerpt')
    ->get();

// OR conditions
$posts = Post::query()
    ->where('post_type', 'post')
    ->where(function ($query) {
        $query->where('post_status', 'publish')
              ->orWhere('post_status', 'future');
    })
    ->get();

// Join with other tables
$posts = Post::query()
    ->join('postmeta', 'posts.ID', '=', 'postmeta.post_id')
    ->where('postmeta.meta_key', 'featured')
    ->where('postmeta.meta_value', 'yes')
    ->select('posts.*')
    ->distinct()
    ->get();
```

## Update Operations

### Updating Single Records

```php
// Find and update
$post = Post::find(123);
$post->setPostTitle('Updated Title');
$post->setPostContent('Updated content...');
$post->save();

// Update meta
$post->setMeta('featured', false);
$post->setMeta('updated_by', 'admin');

// Mass assignment update
$post->fill([
    'post_title' => 'New Title',
    'post_content' => 'New content',
    'post_status' => 'draft'
]);
$post->save();

// Update with validation
$user = User::find(1);
if ($user) {
    $user->setUserEmail('newemail@example.com');
    $user->setDisplayName('New Display Name');
    $user->save();
}
```

### Bulk Updates

```php
// Update multiple records
Post::query()
    ->where('post_type', 'product')
    ->where('post_status', 'draft')
    ->update(['post_status' => 'publish']);

// Update with conditions
Post::query()
    ->where('post_author', 1)
    ->where('post_type', 'post')
    ->update([
        'post_status' => 'publish',
        'post_modified' => now()
    ]);

// Conditional bulk update
$postIds = [1, 2, 3, 4, 5];
Post::query()
    ->whereIn('ID', $postIds)
    ->update(['post_status' => 'trash']);
```

### Updating with Relationships

```php
// Update post and its meta
$post = Post::find(123);
$post->setPostTitle('Updated Post');
$post->setMeta('price', 149.99);
$post->setMeta('sale_price', 129.99);
$post->save();

// Update user and capabilities
$user = User::find(1);
$user->setDisplayName('Administrator');
$user->setMeta('wp_capabilities', serialize(['administrator' => true]));
$user->save();
```

## Delete Operations

### Soft Delete (WordPress Trash)

```php
// Move to trash (WordPress way)
$post = Post::find(123);
$post->setPostStatus('trash');
$post->save();

// Restore from trash
$post = Post::find(123);
$post->setPostStatus('publish');
$post->save();
```

### Hard Delete

```php
// Permanently delete post
$post = Post::find(123);
$post->delete(); // This will also delete related meta

// Delete with relationships
$post = Post::query()
    ->with('metas')
    ->find(123);
$post->delete(); // Deletes post and meta

// Delete user (careful with this!)
$user = User::find(5);
$user->delete(); // This will also delete user meta

// Delete comment
$comment = Comment::find(456);
$comment->delete();
```

### Bulk Delete

```php
// Delete multiple records
Post::query()
    ->where('post_status', 'trash')
    ->where('post_modified', '<', now()->subMonths(3))
    ->delete();

// Delete by IDs
$postIds = [1, 2, 3, 4, 5];
Post::destroy($postIds);

// Delete with conditions
Comment::query()
    ->where('comment_approved', 'spam')
    ->delete();
```

### Deleting Meta Data

```php
// Delete specific meta
$post = Post::find(123);
$post->deleteMeta('old_field');

// Delete multiple meta fields
$metaKeys = ['temp_field1', 'temp_field2', 'cache_data'];
foreach ($metaKeys as $key) {
    $post->deleteMeta($key);
}

// Clear all meta for a model
$post = Post::find(123);
$post->metas()->delete();
```

## Advanced CRUD Patterns

### Transactions

```php
use Illuminate\Support\Facades\DB;

// Using transactions for complex operations
DB::transaction(function () {
    $post = new Post();
    $post->setPostTitle('Transactional Post');
    $post->setPostContent('Content...');
    $post->setPostType('post');
    $post->setPostStatus('publish');
    $post->save();

    // Add meta data
    $post->setMeta('price', 99.99);
    $post->setMeta('category', 'electronics');

    // Create related comment
    $comment = new Comment();
    $comment->setCommentPostID($post->getKey());
    $comment->setCommentAuthor('System');
    $comment->setCommentContent('Auto-generated comment');
    $comment->setCommentApproved('1');
    $comment->save();
});
```

### Batch Operations

```php
// Batch insert
$posts = collect([
    ['title' => 'Post 1', 'content' => 'Content 1'],
    ['title' => 'Post 2', 'content' => 'Content 2'],
    ['title' => 'Post 3', 'content' => 'Content 3'],
]);

$posts->each(function ($data) {
    $post = new Post();
    $post->setPostTitle($data['title']);
    $post->setPostContent($data['content']);
    $post->setPostType('post');
    $post->setPostStatus('publish');
    $post->save();
});

// Batch update with chunk
Post::query()
    ->where('post_type', 'product')
    ->chunk(100, function ($posts) {
        foreach ($posts as $post) {
            $post->setMeta('updated_at', now());
        }
    });
```

### Upsert Operations (Update or Insert)

```php
// Update if exists, create if not
function upsertPost($slug, $data) {
    $post = Post::findOneByName($slug);

    if (!$post) {
        $post = new Post();
        $post->setPostName($slug);
        $post->setPostType($data['type'] ?? 'post');
    }

    $post->setPostTitle($data['title']);
    $post->setPostContent($data['content']);
    $post->setPostStatus($data['status'] ?? 'publish');
    $post->save();

    // Update meta
    foreach ($data['meta'] ?? [] as $key => $value) {
        $post->setMeta($key, $value);
    }

    return $post;
}

// Usage
$post = upsertPost('my-product-slug', [
    'title' => 'My Product',
    'content' => 'Product description',
    'type' => 'product',
    'meta' => [
        'price' => 99.99,
        'sku' => 'PROD-001'
    ]
]);
```

## Best Practices for CRUD Operations

### 1. Always Validate Data

```php
function createValidatedPost($data) {
    // Validate required fields
    if (empty($data['title'])) {
        throw new \InvalidArgumentException('Title is required');
    }

    if (empty($data['content'])) {
        throw new \InvalidArgumentException('Content is required');
    }

    // Sanitize data
    $data['title'] = sanitize_text_field($data['title']);
    $data['content'] = wp_kses_post($data['content']);

    $post = new Post();
    $post->setPostTitle($data['title']);
    $post->setPostContent($data['content']);
    $post->setPostType($data['type'] ?? 'post');
    $post->setPostStatus($data['status'] ?? 'draft');
    $post->save();

    return $post;
}
```

### 2. Use Eager Loading

```php
// Good: Eager load relationships
$posts = Post::query()
    ->with(['author', 'comments'])
    ->where('post_status', 'publish')
    ->get();

// Bad: N+1 query problem
$posts = Post::query()
    ->where('post_status', 'publish')
    ->get();

foreach ($posts as $post) {
    echo $post->author->display_name; // This creates N+1 queries
}
```

### 3. Handle Errors Gracefully

```php
function safeUpdatePost($id, $data) {
    try {
        $post = Post::findOrFail($id);

        if (isset($data['title'])) {
            $post->setPostTitle($data['title']);
        }

        if (isset($data['content'])) {
            $post->setPostContent($data['content']);
        }

        $post->save();

        return ['success' => true, 'post' => $post];

    } catch (\Exception $e) {
        return ['success' => false, 'error' => $e->getMessage()];
    }
}
```

### 4. Use Transactions for Complex Operations

```php
function createPostWithRelations($postData, $metaData = [], $commentData = []) {
    return DB::transaction(function () use ($postData, $metaData, $commentData) {
        // Create post
        $post = new Post();
        $post->fill($postData);
        $post->save();

        // Add meta data
        foreach ($metaData as $key => $value) {
            $post->setMeta($key, $value);
        }

        // Add comments
        foreach ($commentData as $commentItem) {
            $comment = new Comment();
            $comment->setCommentPostID($post->getKey());
            $comment->fill($commentItem);
            $comment->save();
        }

        return $post;
    });
}
```

This comprehensive CRUD guide covers all the essential operations you'll need when working with WordPress ORM models.
