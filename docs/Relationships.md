# Relationships

WordPress ORM provides comprehensive relationship support between WordPress entities, leveraging Eloquent's relationship system with WordPress-specific enhancements.

## Basic Relationships

### Post Relationships

#### Post Author

```php
use Dbout\WpOrm\Models\Post;
use Dbout\WpOrm\Models\User;

$post = Post::find(123);
$author = $post->author; // Get the post author

// Eager load author
$posts = Post::query()
    ->with('author')
    ->get();

// Filter posts by author
$posts = Post::query()
    ->whereHas('author', function ($query) {
        $query->where('user_login', 'john_doe');
    })
    ->get();
```

#### Post Comments

```php
use Dbout\WpOrm\Models\Comment;

$post = Post::find(123);
$comments = $post->comments; // Get all comments

// Eager load comments
$posts = Post::query()
    ->with(['comments' => function ($query) {
        $query->where('comment_approved', '1');
    }])
    ->get();

// Count comments
$posts = Post::query()
    ->withCount('comments')
    ->get();

foreach ($posts as $post) {
    echo "Post has {$post->comments_count} comments";
}
```

#### Post Parent

```php
// Get post parent
$post = Post::find(123);
$parent = $post->parent;

// Get child posts
$parent = Post::find(456);
$children = $parent->children; // This would need to be defined in the model

// Find posts with specific parent
$childPosts = Post::query()
    ->where('post_parent', 456)
    ->get();
```

### User Relationships

#### User Posts

```php
use Dbout\WpOrm\Models\User;

$user = User::find(1);
$posts = $user->posts; // Get all user posts

// Eager load posts
$users = User::query()
    ->with('posts')
    ->get();

// Filter users by post count
$activeAuthors = User::query()
    ->whereHas('posts', function ($query) {
        $query->where('post_status', 'publish')
              ->where('post_type', 'post');
    })
    ->get();
```

#### User Comments

```php
$user = User::find(1);
$comments = $user->comments; // Get all user comments

// Recent comments by user
$recentComments = Comment::query()
    ->where('user_id', 1)
    ->where('comment_approved', '1')
    ->orderBy('comment_date', 'desc')
    ->limit(10)
    ->get();
```

### Comment Relationships

#### Comment Post

```php
use Dbout\WpOrm\Models\Comment;

$comment = Comment::find(123);
$post = $comment->post; // Get the commented post

// Eager load posts
$comments = Comment::query()
    ->with('post')
    ->get();
```

#### Comment Author

```php
$comment = Comment::find(123);
$author = $comment->author; // Get comment author (if registered user)

// Comments by registered users only
$comments = Comment::query()
    ->whereNotNull('user_id')
    ->with('author')
    ->get();
```

## Meta Relationships

### Post Meta

```php
use Dbout\WpOrm\Models\Meta\PostMeta;

$post = Post::find(123);
$metas = $post->metas; // Get all post meta

// Eager load specific meta
$posts = Post::query()
    ->with(['metas' => function ($query) {
        $query->whereIn('meta_key', ['featured', 'price', 'category']);
    }])
    ->get();

// Direct meta access
$post = Post::with('metas')->find(123);
foreach ($post->metas as $meta) {
    echo "{$meta->meta_key}: {$meta->meta_value}";
}
```

### User Meta

```php
use Dbout\WpOrm\Models\Meta\UserMeta;

$user = User::find(1);
$metas = $user->metas; // Get all user meta

// Load specific user meta
$users = User::query()
    ->with(['metas' => function ($query) {
        $query->whereIn('meta_key', ['first_name', 'last_name', 'company']);
    }])
    ->get();
```

## Custom Relationships

### Defining Custom Relationships

You can define custom relationships in your models:

```php
<?php

namespace App\Models;

use Dbout\WpOrm\Models\Post;
use Dbout\WpOrm\Models\User;
use Dbout\WpOrm\Models\Comment;

class Product extends Post
{
    /**
     * Get product reviews (comments with specific meta)
     */
    public function reviews()
    {
        return $this->hasMany(Comment::class, 'comment_post_ID')
                   ->where('comment_approved', '1')
                   ->whereHas('metas', function ($query) {
                       $query->where('meta_key', 'review_rating');
                   });
    }

    /**
     * Get product categories (assuming custom taxonomy)
     */
    public function categories()
    {
        return $this->belongsToMany(
            Term::class,
            'term_relationships',
            'object_id',
            'term_taxonomy_id'
        )->whereHas('taxonomy', function ($query) {
            $query->where('taxonomy', 'product_category');
        });
    }

    /**
     * Get related products
     */
    public function relatedProducts()
    {
        $categoryIds = $this->categories()->pluck('term_id');

        return $this->belongsToMany(
            self::class,
            'term_relationships',
            'term_taxonomy_id',
            'object_id'
        )->whereIn('term_id', $categoryIds)
         ->where('object_id', '!=', $this->getKey());
    }

    /**
     * Get product vendor (custom relationship via meta)
     */
    public function vendor()
    {
        return $this->belongsTo(User::class, 'post_author')
                   ->whereHas('metas', function ($query) {
                       $query->where('meta_key', 'user_role')
                             ->where('meta_value', 'vendor');
                   });
    }
}
```

### Using Custom Relationships

```php
$product = Product::find(123);

// Get product reviews
$reviews = $product->reviews;

// Get average rating
$averageRating = $product->reviews()
    ->join('commentmeta', 'comments.comment_ID', '=', 'commentmeta.comment_id')
    ->where('commentmeta.meta_key', 'review_rating')
    ->avg('commentmeta.meta_value');

// Get product categories
$categories = $product->categories;

// Get related products
$relatedProducts = $product->relatedProducts;

// Eager load all relationships
$product = Product::query()
    ->with(['reviews', 'categories', 'vendor'])
    ->find(123);
```

## Advanced Relationship Patterns

### Many-to-Many with Pivot Data

```php
class User extends \Dbout\WpOrm\Models\User
{
    /**
     * User's enrolled courses with enrollment data
     */
    public function courses()
    {
        return $this->belongsToMany(
            Course::class,
            'course_enrollments',
            'user_id',
            'course_id'
        )->withPivot(['enrolled_at', 'status', 'progress'])
         ->withTimestamps();
    }
}

// Usage
$user = User::find(1);
foreach ($user->courses as $course) {
    echo "Enrolled: {$course->pivot->enrolled_at}";
    echo "Progress: {$course->pivot->progress}%";
}
```

### Polymorphic Relationships

```php
class Attachment extends \Dbout\WpOrm\Models\Attachment
{
    /**
     * Get the owning attachable model (post, user, etc.)
     */
    public function attachable()
    {
        // This would require custom implementation
        // based on your specific attachment system
        return $this->morphTo();
    }
}

class Post extends \Dbout\WpOrm\Models\Post
{
    /**
     * Get all attachments for this post
     */
    public function attachments()
    {
        return $this->morphMany(Attachment::class, 'attachable');
    }
}
```

### Has Many Through

```php
class User extends \Dbout\WpOrm\Models\User
{
    /**
     * Get all comments through user's posts
     */
    public function postComments()
    {
        return $this->hasManyThrough(
            Comment::class,
            Post::class,
            'post_author', // Foreign key on posts table
            'comment_post_ID', // Foreign key on comments table
            'ID', // Local key on users table
            'ID' // Local key on posts table
        );
    }
}

// Usage
$user = User::find(1);
$allComments = $user->postComments; // Comments on all user's posts
```

## Relationship Querying

### Conditional Relationships

```php
// Posts with approved comments only
$posts = Post::query()
    ->with(['comments' => function ($query) {
        $query->where('comment_approved', '1')
              ->orderBy('comment_date', 'desc');
    }])
    ->get();

// Users with published posts only
$users = User::query()
    ->with(['posts' => function ($query) {
        $query->where('post_status', 'publish')
              ->where('post_type', 'post');
    }])
    ->get();
```

### Relationship Existence

```php
// Posts that have comments
$postsWithComments = Post::query()
    ->has('comments')
    ->get();

// Posts with at least 5 comments
$popularPosts = Post::query()
    ->has('comments', '>=', 5)
    ->get();

// Posts without comments
$postsWithoutComments = Post::query()
    ->doesntHave('comments')
    ->get();
```

### Complex Relationship Queries

```php
// Posts by users who have specific meta
$posts = Post::query()
    ->whereHas('author', function ($query) {
        $query->whereHas('metas', function ($metaQuery) {
            $metaQuery->where('meta_key', 'verified')
                      ->where('meta_value', 'yes');
        });
    })
    ->get();

// Products in specific categories with reviews
$products = Product::query()
    ->whereHas('categories', function ($query) {
        $query->whereIn('slug', ['electronics', 'computers']);
    })
    ->whereHas('reviews', function ($query) {
        $query->where('comment_approved', '1');
    })
    ->with(['categories', 'reviews'])
    ->get();
```

## Performance Considerations

### Eager Loading

```php
// Good: Eager load to avoid N+1 queries
$posts = Post::query()
    ->with(['author', 'comments.author', 'metas'])
    ->get();

// Bad: Will cause N+1 queries
$posts = Post::all();
foreach ($posts as $post) {
    echo $post->author->name; // N+1 query problem
}
```

### Selective Loading

```php
// Load only specific columns
$posts = Post::query()
    ->with(['author:id,user_login,user_email'])
    ->get();

// Count relationships without loading
$posts = Post::query()
    ->withCount(['comments', 'metas'])
    ->get();
```

### Chunking with Relationships

```php
// Process large datasets efficiently
Post::query()
    ->with('author')
    ->chunk(100, function ($posts) {
        foreach ($posts as $post) {
            // Process post with author data
        }
    });
```

## Best Practices

1. **Always Use Eager Loading**: Prevent N+1 query problems
2. **Define Inverse Relationships**: Define both sides of relationships when applicable
3. **Use Relationship Constraints**: Add where clauses to relationship definitions
4. **Consider Database Indexes**: Index foreign keys for better performance
5. **Use Relationship Counts**: Use `withCount()` instead of loading full relationships when you only need counts
6. **Optimize Queries**: Select only needed columns in relationships
7. **Cache Relationship Data**: Cache expensive relationship queries when appropriate
