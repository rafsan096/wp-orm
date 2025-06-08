# Quick Start Guide

This guide will help you get up and running with WordPress ORM quickly.

## Prerequisites

- WordPress installed and configured
- PHP >= 8.2
- Composer installed

## Installation

```bash
composer require dbout/wp-orm
```

In your `wp-config.php`, include the autoloader:

```php
require __DIR__ . '/vendor/autoload.php';
```

## Basic Usage

### Working with Posts

```php
use Dbout\WpOrm\Models\Post;

// Get all published posts
$posts = Post::query()
    ->where('post_status', 'publish')
    ->where('post_type', 'post')
    ->get();

// Find a specific post by ID
$post = Post::find(123);

// Find post by slug
$post = Post::findOneByName('my-post-slug');

// Create a new post
$post = new Post();
$post->setPostTitle('My New Post');
$post->setPostContent('This is the content of my post');
$post->setPostType('post');
$post->setPostStatus('publish');
$post->save();

// Update a post
$post = Post::find(123);
$post->setPostTitle('Updated Title');
$post->save();
```

### Working with Users

```php
use Dbout\WpOrm\Models\User;

// Get all users
$users = User::all();

// Find user by email
$user = User::query()
    ->where('user_email', 'user@example.com')
    ->first();

// Find user by login
$user = User::findOneByLogin('username');

// Create new user
$user = new User();
$user->setUserLogin('newuser');
$user->setUserEmail('newuser@example.com');
$user->setUserPass(wp_hash_password('password123'));
$user->save();
```

### Working with Comments

```php
use Dbout\WpOrm\Models\Comment;

// Get approved comments for a post
$comments = Comment::query()
    ->where('comment_post_ID', 123)
    ->where('comment_approved', '1')
    ->get();

// Create a new comment
$comment = new Comment();
$comment->setCommentPostId(123);
$comment->setCommentAuthor('John Doe');
$comment->setCommentAuthorEmail('john@example.com');
$comment->setCommentContent('Great post!');
$comment->setCommentApproved('1');
$comment->save();
```

### Working with Meta Data

```php
// Get post meta
$value = $post->getMetaValue('custom_field_key');

// Set post meta
$post->setMeta('custom_field_key', 'custom_value');

// Check if meta exists
if ($post->hasMeta('custom_field_key')) {
    // Meta exists
}

// Delete meta
$post->deleteMeta('custom_field_key');
```

### Relationships

```php
// Get post author
$post = Post::find(123);
$author = $post->author;

// Get post comments
$comments = $post->comments;

// Get user posts
$user = User::find(1);
$posts = $user->posts;
```

### Query Building

```php
// Complex queries
$posts = Post::query()
    ->where('post_type', 'post')
    ->where('post_status', 'publish')
    ->where('post_date', '>=', '2024-01-01')
    ->orderBy('post_date', 'desc')
    ->limit(10)
    ->get();

// Using scopes
$publishedPosts = Post::query()
    ->published()
    ->ofType('post')
    ->get();
```

## Next Steps

- [Working with Meta Data](Attribute-&-meta-casting.md)
- [Creating Custom Models](Create-custom-model.md)
- [Advanced Query Building](Advanced-Query-Building.md)
- [Working with Custom Post Types](Custom-Post-Types.md)
