# Models Reference

WordPress ORM provides a comprehensive set of models that correspond to WordPress core entities, each with specific methods and relationships.

## Core Models

### Post Model

The base model for all WordPress posts and custom post types.

#### Class: `Dbout\WpOrm\Models\Post`

**Constants:**

```php
Post::POST_ID           // 'ID'
Post::AUTHOR            // 'post_author'
Post::DATE              // 'post_date'
Post::DATE_GMT          // 'post_date_gmt'
Post::CONTENT           // 'post_content'
Post::TITLE             // 'post_title'
Post::EXCERPT           // 'post_excerpt'
Post::STATUS            // 'post_status'
Post::COMMENT_STATUS    // 'comment_status'
Post::PING_STATUS       // 'ping_status'
Post::PASSWORD          // 'post_password'
Post::POST_NAME         // 'post_name'
Post::TO_PING           // 'to_ping'
Post::PINGED            // 'pinged'
Post::MODIFIED          // 'post_modified'
Post::MODIFIED_GMT      // 'post_modified_gmt'
Post::CONTENT_FILTERED  // 'post_content_filtered'
Post::PARENT            // 'post_parent'
Post::GUID              // 'guid'
Post::MENU_ORDER        // 'menu_order'
Post::TYPE              // 'post_type'
Post::MIME_TYPE         // 'post_mime_type'
Post::COMMENT_COUNT     // 'comment_count'
```

**Methods:**

_Getters/Setters:_

```php
// Date methods
$post->getPostDate(): ?Carbon
$post->setPostDate($date): Post
$post->getPostDateGMT(): ?Carbon
$post->setPostDateGMT($date): Post
$post->getPostModified(): ?Carbon
$post->setPostModified($modified): Post
$post->getPostModifiedGMT(): ?Carbon
$post->setPostModifiedGMT($modified): Post

// Content methods
$post->getPostContent(): ?string
$post->setPostContent(?string $content): Post
$post->getPostTitle(): ?string
$post->setPostTitle(?string $title): Post
$post->getPostExcerpt(): ?string
$post->setPostExcerpt(?string $excerpt): Post

// Status and type methods
$post->getPostStatus(): ?string
$post->setPostStatus(?string $status): Post
$post->getPostType(): ?string
$post->setPostType(string $type): Post

// Other methods
$post->getGuid(): ?string
$post->setGuid(?string $guid): Post
$post->getPostPassword(): ?string
$post->setPostPassword(?string $password): Post
$post->getPostName(): ?string
$post->setPostName(?string $name): Post
$post->getMenuOrder(): ?int
$post->setMenuOrder(?int $order): Post
$post->getPostParent(): ?int
$post->setPostParent(?int $parentId): Post
$post->getPostAuthor(): ?int
$post->setPostAuthor(?int $authorId): Post
```

_Static Methods:_

```php
Post::findOneByName(?string $name): ?Post
Post::findOneByGuid(string $guid): ?Post
```

_Relationships:_

```php
$post->author(): HasOne         // Post author (User)
$post->comments(): HasMany      // Post comments
$post->parent(): HasOne         // Parent post
$post->metas(): HasMany         // Post meta
```

_Casts:_

```php
'post_author' => 'integer'
'post_parent' => 'integer'
'menu_order' => 'integer'
'comment_count' => 'integer'
'post_date' => 'datetime'
'post_modified' => 'datetime'
'post_date_gmt' => 'datetime'
'post_modified_gmt' => 'datetime'
```

### User Model

Represents WordPress users.

#### Class: `Dbout\WpOrm\Models\User`

**Constants:**

```php
User::USER_ID           // 'ID'
User::USER_LOGIN        // 'user_login'
User::USER_PASS         // 'user_pass'
User::USER_NICENAME     // 'user_nicename'
User::USER_EMAIL        // 'user_email'
User::USER_URL          // 'user_url'
User::USER_REGISTERED   // 'user_registered'
User::USER_ACTIVATION_KEY // 'user_activation_key'
User::USER_STATUS       // 'user_status'
User::DISPLAY_NAME      // 'display_name'
```

**Methods:**

_Getters/Setters:_

```php
$user->getUserLogin(): ?string
$user->setUserLogin(string $login): User
$user->getUserPass(): ?string
$user->setUserPass(string $password): User
$user->getUserNicename(): ?string
$user->setUserNicename(string $nicename): User
$user->getUserEmail(): ?string
$user->setUserEmail(string $email): User
$user->getUserUrl(): ?string
$user->setUserUrl(?string $url): User
$user->getUserRegistered(): ?Carbon
$user->setUserRegistered($registered): User
$user->getUserActivationKey(): ?string
$user->setUserActivationKey(?string $key): User
$user->getUserStatus(): ?int
$user->setUserStatus(int $status): User
$user->getDisplayName(): ?string
$user->setDisplayName(string $name): User
```

_Static Methods:_

```php
User::findOneByLogin(string $login): ?User
User::findOneByEmail(string $email): ?User
```

_Relationships:_

```php
$user->posts(): HasMany         // User's posts
$user->comments(): HasMany      // User's comments
$user->metas(): HasMany         // User meta
```

_Casts:_

```php
'user_status' => 'integer'
'user_registered' => 'datetime'
```

### Comment Model

Represents WordPress comments.

#### Class: `Dbout\WpOrm\Models\Comment`

**Constants:**

```php
Comment::COMMENT_ID         // 'comment_ID'
Comment::POST_ID            // 'comment_post_ID'
Comment::AUTHOR             // 'comment_author'
Comment::AUTHOR_EMAIL       // 'comment_author_email'
Comment::AUTHOR_URL         // 'comment_author_url'
Comment::AUTHOR_IP          // 'comment_author_IP'
Comment::DATE               // 'comment_date'
Comment::DATE_GMT           // 'comment_date_gmt'
Comment::CONTENT            // 'comment_content'
Comment::KARMA              // 'comment_karma'
Comment::APPROVED           // 'comment_approved'
Comment::AGENT              // 'comment_agent'
Comment::TYPE               // 'comment_type'
Comment::PARENT             // 'comment_parent'
Comment::USER_ID            // 'user_id'
```

**Methods:**

_Getters/Setters:_

```php
$comment->getCommentPostId(): ?int
$comment->setCommentPostId(int $postId): Comment
$comment->getCommentAuthor(): ?string
$comment->setCommentAuthor(string $author): Comment
$comment->getCommentAuthorEmail(): ?string
$comment->setCommentAuthorEmail(?string $email): Comment
$comment->getCommentAuthorUrl(): ?string
$comment->setCommentAuthorUrl(?string $url): Comment
$comment->getCommentAuthorIp(): ?string
$comment->setCommentAuthorIp(?string $ip): Comment
$comment->getCommentDate(): ?Carbon
$comment->setCommentDate($date): Comment
$comment->getCommentDateGmt(): ?Carbon
$comment->setCommentDateGmt($date): Comment
$comment->getCommentContent(): ?string
$comment->setCommentContent(?string $content): Comment
$comment->getCommentKarma(): ?int
$comment->setCommentKarma(int $karma): Comment
$comment->getCommentApproved(): ?string
$comment->setCommentApproved(string $approved): Comment
$comment->getCommentAgent(): ?string
$comment->setCommentAgent(?string $agent): Comment
$comment->getCommentType(): ?string
$comment->setCommentType(string $type): Comment
$comment->getCommentParent(): ?int
$comment->setCommentParent(int $parent): Comment
$comment->getUserId(): ?int
$comment->setUserId(?int $userId): Comment
```

_Relationships:_

```php
$comment->post(): BelongsTo      // Comment's post
$comment->author(): BelongsTo    // Comment author (User)
$comment->parent(): BelongsTo    // Parent comment
$comment->children(): HasMany    // Child comments
$comment->metas(): HasMany       // Comment meta
```

_Casts:_

```php
'comment_post_ID' => 'integer'
'comment_karma' => 'integer'
'comment_parent' => 'integer'
'user_id' => 'integer'
'comment_date' => 'datetime'
'comment_date_gmt' => 'datetime'
```

### Option Model

Represents WordPress options.

#### Class: `Dbout\WpOrm\Models\Option`

**Constants:**

```php
Option::OPTION_ID       // 'option_id'
Option::OPTION_NAME     // 'option_name'
Option::OPTION_VALUE    // 'option_value'
Option::AUTOLOAD        // 'autoload'
```

**Methods:**

_Getters/Setters:_

```php
$option->getOptionName(): ?string
$option->setOptionName(string $name): Option
$option->getOptionValue(): mixed
$option->setOptionValue(mixed $value): Option
$option->getAutoload(): ?string
$option->setAutoload(string $autoload): Option
```

_Static Methods:_

```php
Option::findOneByName(string $name): ?Option
Option::getValueByName(string $name, mixed $default = null): mixed
```

### Term Model

Represents WordPress terms.

#### Class: `Dbout\WpOrm\Models\Term`

**Constants:**

```php
Term::TERM_ID       // 'term_id'
Term::NAME          // 'name'
Term::SLUG          // 'slug'
Term::TERM_GROUP    // 'term_group'
```

**Methods:**

_Getters/Setters:_

```php
$term->getTermId(): ?int
$term->getName(): ?string
$term->setName(string $name): Term
$term->getSlug(): ?string
$term->setSlug(string $slug): Term
$term->getTermGroup(): ?int
$term->setTermGroup(int $group): Term
```

_Relationships:_

```php
$term->taxonomies(): HasMany     // Term taxonomies
```

### TermTaxonomy Model

Represents WordPress term taxonomies.

#### Class: `Dbout\WpOrm\Models\TermTaxonomy`

**Constants:**

```php
TermTaxonomy::TERM_TAXONOMY_ID  // 'term_taxonomy_id'
TermTaxonomy::TERM_ID           // 'term_id'
TermTaxonomy::TAXONOMY          // 'taxonomy'
TermTaxonomy::DESCRIPTION       // 'description'
TermTaxonomy::PARENT            // 'parent'
TermTaxonomy::COUNT             // 'count'
```

**Methods:**

_Getters/Setters:_

```php
$taxonomy->getTermTaxonomyId(): ?int
$taxonomy->getTermId(): ?int
$taxonomy->setTermId(int $termId): TermTaxonomy
$taxonomy->getTaxonomy(): ?string
$taxonomy->setTaxonomy(string $taxonomy): TermTaxonomy
$taxonomy->getDescription(): ?string
$taxonomy->setDescription(?string $description): TermTaxonomy
$taxonomy->getParent(): ?int
$taxonomy->setParent(int $parent): TermTaxonomy
$taxonomy->getCount(): ?int
$taxonomy->setCount(int $count): TermTaxonomy
```

_Relationships:_

```php
$taxonomy->term(): BelongsTo     // Associated term
```

## Specialized Post Models

### Article Model

Represents standard blog posts (post_type = 'post').

#### Class: `Dbout\WpOrm\Models\Article`

Extends `Post` model with automatic post_type filtering.

### Page Model

Represents WordPress pages (post_type = 'page').

#### Class: `Dbout\WpOrm\Models\Page`

Extends `Post` model with automatic post_type filtering.

### Attachment Model

Represents WordPress media attachments (post_type = 'attachment').

#### Class: `Dbout\WpOrm\Models\Attachment`

Extends `Post` model with automatic post_type filtering.

## Custom Models

### CustomPost Model

Base class for creating custom post type models.

#### Class: `Dbout\WpOrm\Models\CustomPost`

**Usage:**

```php
class Product extends CustomPost
{
    protected static string $postType = 'product';

    // Your custom methods and properties
}
```

### CustomComment Model

Base class for creating custom comment type models.

#### Class: `Dbout\WpOrm\Models\CustomComment`

**Usage:**

```php
class Review extends CustomComment
{
    protected static string $commentType = 'review';

    // Your custom methods and properties
}
```

## Meta Models

### PostMeta Model

Represents WordPress post meta.

#### Class: `Dbout\WpOrm\Models\Meta\PostMeta`

**Constants:**

```php
PostMeta::META_ID       // 'meta_id'
PostMeta::POST_ID       // 'post_id'
PostMeta::META_KEY      // 'meta_key'
PostMeta::META_VALUE    // 'meta_value'
```

**Methods:**

_Getters/Setters:_

```php
$meta->getPostId(): ?int
$meta->setPostId(int $postId): PostMeta
$meta->getKey(): ?string
$meta->setKey(string $key): PostMeta
$meta->getValue(): mixed
$meta->setValue(mixed $value): PostMeta
```

_Relationships:_

```php
$meta->post(): BelongsTo         // Associated post
```

### UserMeta Model

Represents WordPress user meta.

#### Class: `Dbout\WpOrm\Models\Meta\UserMeta`

Similar structure to PostMeta but for users.

**Constants:**

```php
UserMeta::UMETA_ID      // 'umeta_id'
UserMeta::USER_ID       // 'user_id'
UserMeta::META_KEY      // 'meta_key'
UserMeta::META_VALUE    // 'meta_value'
```

### CommentMeta Model

Represents WordPress comment meta.

#### Class: `Dbout\WpOrm\Models\Meta\CommentMeta`

Similar structure to PostMeta but for comments.

**Constants:**

```php
CommentMeta::META_ID        // 'meta_id'
CommentMeta::COMMENT_ID     // 'comment_id'
CommentMeta::META_KEY       // 'meta_key'
CommentMeta::META_VALUE     // 'meta_value'
```

## Multisite Models

### Site Model

Represents WordPress multisite sites.

#### Class: `Dbout\WpOrm\Models\Multisite\Site`

**Constants:**

```php
Site::BLOG_ID       // 'blog_id'
Site::SITE_ID       // 'site_id'
Site::DOMAIN        // 'domain'
Site::PATH          // 'path'
Site::REGISTERED    // 'registered'
Site::LAST_UPDATED  // 'last_updated'
Site::PUBLIC        // 'public'
Site::ARCHIVED      // 'archived'
Site::MATURE        // 'mature'
Site::SPAM          // 'spam'
Site::DELETED       // 'deleted'
Site::LANG_ID       // 'lang_id'
```

### Blog Model

Alias for Site model for backwards compatibility.

#### Class: `Dbout\WpOrm\Models\Multisite\Blog`

## Enums

### PostStatus Enum

#### Class: `Dbout\WpOrm\Enums\PostStatus`

**Values:**

```php
PostStatus::PUBLISH     // 'publish'
PostStatus::FUTURE      // 'future'
PostStatus::DRAFT       // 'draft'
PostStatus::PENDING     // 'pending'
PostStatus::PRIVATE     // 'private'
PostStatus::TRASH       // 'trash'
PostStatus::AUTO_DRAFT  // 'auto-draft'
PostStatus::INHERIT     // 'inherit'
```

### PingStatus Enum

#### Class: `Dbout\WpOrm\Enums\PingStatus`

**Values:**

```php
PingStatus::OPEN        // 'open'
PingStatus::CLOSED      // 'closed'
```

### YesNo Enum

#### Class: `Dbout\WpOrm\Enums\YesNo`

**Values:**

```php
YesNo::YES      // 'yes'
YesNo::NO       // 'no'
```

## Model Features

### Meta Support

Models implementing `WithMetaModelInterface` support meta operations:

```php
// Check if model supports meta
if ($model instanceof WithMetaModelInterface) {
    $value = $model->getMetaValue('key');
    $model->setMeta('key', 'value');
    $exists = $model->hasMeta('key');
    $model->deleteMeta('key');
}
```

### Type Casting

All models support automatic type casting:

```php
// In your model
protected $casts = [
    'price' => 'float',
    'is_featured' => 'boolean',
    'created_at' => 'datetime',
];

// Meta casting
protected array $metaCasts = [
    'price' => 'float',
    'tags' => 'array',
    'featured' => 'boolean',
];
```

### Timestamps

WordPress models use WordPress-specific timestamp columns:

```php
// For Post model
const CREATED_AT = 'post_date';
const UPDATED_AT = 'post_modified';

// For Comment model
const CREATED_AT = 'comment_date';
const UPDATED_AT = null; // Comments don't have updated_at
```

### Custom Builders

Each model can have a custom query builder:

```php
// Post model uses PostBuilder
$posts = Post::query(); // Returns PostBuilder instance

// Available builder methods
$posts->addMetaToFilter('featured', true);
$posts->addMetaToSelect('price');
$posts->joinToMeta('category');
```

This reference covers all the main models and their capabilities. Each model provides WordPress-specific functionality while maintaining full Eloquent compatibility.
