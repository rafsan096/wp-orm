# Working with Custom Post Types

WordPress ORM makes it easy to work with custom post types using either the generic `Post` model or by creating dedicated custom models.

## Using the Generic Post Model

### Basic Custom Post Type Queries

```php
use Dbout\WpOrm\Models\Post;

// Get all products
$products = Post::query()
    ->where('post_type', 'product')
    ->where('post_status', 'publish')
    ->get();

// Create a new product
$product = new Post();
$product->setPostType('product');
$product->setPostTitle('New Product');
$product->setPostContent('Product description');
$product->setPostStatus('publish');
$product->save();

// Set product meta
$product->setMeta('price', 99.99);
$product->setMeta('sku', 'PROD-001');
$product->setMeta('category', 'electronics');
```

### Filtering by Custom Post Type Meta

```php
// Find products by price range
$products = Post::query()
    ->where('post_type', 'product')
    ->addMetaToFilter('price', 50, '>=')
    ->addMetaToFilter('price', 200, '<=')
    ->get();

// Find featured products
$featuredProducts = Post::query()
    ->where('post_type', 'product')
    ->addMetaToFilter('featured', 'yes')
    ->get();

// Include meta in results
$products = Post::query()
    ->where('post_type', 'product')
    ->addMetasToSelect([
        'product_price' => 'price',
        'product_sku' => 'sku',
        'product_category' => 'category'
    ])
    ->get();
```

## Creating Custom Post Type Models

For better organization and type safety, you can create dedicated models for your custom post types.

### Creating a Product Model

```php
<?php

namespace App\Models;

use Dbout\WpOrm\Models\CustomPost;

class Product extends CustomPost
{
    /**
     * @var string
     */
    protected static string $postType = 'product';

    /**
     * Meta casting for type safety
     *
     * @var array
     */
    protected array $metaCasts = [
        'price' => 'float',
        'featured' => 'boolean',
        'tags' => 'array',
        'created_date' => 'datetime',
    ];

    /**
     * Get the product price
     *
     * @return float|null
     */
    public function getPrice(): ?float
    {
        return $this->getMetaValue('price');
    }

    /**
     * Set the product price
     *
     * @param float $price
     * @return void
     */
    public function setPrice(float $price): void
    {
        $this->setMeta('price', $price);
    }

    /**
     * Check if product is featured
     *
     * @return bool
     */
    public function isFeatured(): bool
    {
        return (bool) $this->getMetaValue('featured');
    }

    /**
     * Set featured status
     *
     * @param bool $featured
     * @return void
     */
    public function setFeatured(bool $featured): void
    {
        $this->setMeta('featured', $featured);
    }

    /**
     * Get product SKU
     *
     * @return string|null
     */
    public function getSku(): ?string
    {
        return $this->getMetaValue('sku');
    }

    /**
     * Set product SKU
     *
     * @param string $sku
     * @return void
     */
    public function setSku(string $sku): void
    {
        $this->setMeta('sku', $sku);
    }

    /**
     * Scope for featured products
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeFeatured($query)
    {
        return $query->addMetaToFilter('featured', true);
    }

    /**
     * Scope for products in price range
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     * @param float $min
     * @param float $max
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopePriceRange($query, float $min, float $max)
    {
        return $query->addMetaToFilter('price', $min, '>=')
                    ->addMetaToFilter('price', $max, '<=');
    }
}
```

### Using the Custom Model

```php
use App\Models\Product;

// Get all products (automatically filtered by post_type)
$products = Product::all();

// Use custom methods
$product = new Product();
$product->setPostTitle('Amazing Widget');
$product->setPrice(99.99);
$product->setSku('WIDGET-001');
$product->setFeatured(true);
$product->save();

// Use scopes
$featuredProducts = Product::featured()->get();
$affordableProducts = Product::priceRange(10, 50)->get();

// Type-safe meta access
$price = $product->getPrice(); // Returns float|null
$isFeatured = $product->isFeatured(); // Returns bool
```

## Advanced Custom Post Type Patterns

### Creating an Event Model

```php
<?php

namespace App\Models;

use Dbout\WpOrm\Models\CustomPost;
use Carbon\Carbon;

class Event extends CustomPost
{
    protected static string $postType = 'event';

    protected array $metaCasts = [
        'event_date' => 'datetime',
        'end_date' => 'datetime',
        'max_attendees' => 'integer',
        'price' => 'float',
        'is_virtual' => 'boolean',
    ];

    public function getEventDate(): ?Carbon
    {
        return $this->getMetaValue('event_date');
    }

    public function setEventDate(Carbon $date): void
    {
        $this->setMeta('event_date', $date);
    }

    public function getEndDate(): ?Carbon
    {
        return $this->getMetaValue('end_date');
    }

    public function setEndDate(Carbon $date): void
    {
        $this->setMeta('end_date', $date);
    }

    public function getMaxAttendees(): ?int
    {
        return $this->getMetaValue('max_attendees');
    }

    public function setMaxAttendees(int $max): void
    {
        $this->setMeta('max_attendees', $max);
    }

    public function isVirtual(): bool
    {
        return (bool) $this->getMetaValue('is_virtual');
    }

    public function setVirtual(bool $virtual): void
    {
        $this->setMeta('is_virtual', $virtual);
    }

    // Scopes
    public function scopeUpcoming($query)
    {
        return $query->addMetaToFilter('event_date', Carbon::now(), '>=');
    }

    public function scopeVirtual($query)
    {
        return $query->addMetaToFilter('is_virtual', true);
    }

    public function scopeInDateRange($query, Carbon $start, Carbon $end)
    {
        return $query->addMetaToFilter('event_date', $start, '>=')
                    ->addMetaToFilter('event_date', $end, '<=');
    }
}
```

### Portfolio Item Model

```php
<?php

namespace App\Models;

use Dbout\WpOrm\Models\CustomPost;

class PortfolioItem extends CustomPost
{
    protected static string $postType = 'portfolio';

    protected array $metaCasts = [
        'project_url' => 'string',
        'client_name' => 'string',
        'technologies' => 'array',
        'project_date' => 'date',
        'featured' => 'boolean',
    ];

    public function getProjectUrl(): ?string
    {
        return $this->getMetaValue('project_url');
    }

    public function setProjectUrl(string $url): void
    {
        $this->setMeta('project_url', $url);
    }

    public function getClientName(): ?string
    {
        return $this->getMetaValue('client_name');
    }

    public function setClientName(string $name): void
    {
        $this->setMeta('client_name', $name);
    }

    public function getTechnologies(): array
    {
        return $this->getMetaValue('technologies') ?? [];
    }

    public function setTechnologies(array $technologies): void
    {
        $this->setMeta('technologies', $technologies);
    }

    public function addTechnology(string $technology): void
    {
        $technologies = $this->getTechnologies();
        $technologies[] = $technology;
        $this->setTechnologies(array_unique($technologies));
    }

    // Scopes
    public function scopeFeatured($query)
    {
        return $query->addMetaToFilter('featured', true);
    }

    public function scopeByTechnology($query, string $technology)
    {
        return $query->addMetaToFilter('technologies', "%{$technology}%", 'LIKE');
    }
}
```

## Working with Taxonomies

### Associating Custom Post Types with Taxonomies

```php
use Dbout\WpOrm\Models\Term;
use Dbout\WpOrm\Models\TermTaxonomy;

// Get products in a specific category
$categoryTerm = Term::query()
    ->where('slug', 'electronics')
    ->first();

if ($categoryTerm) {
    $taxonomy = TermTaxonomy::query()
        ->where('term_id', $categoryTerm->getTermId())
        ->where('taxonomy', 'product_category')
        ->first();

    if ($taxonomy) {
        $products = Product::query()
            ->whereHas('termRelationships', function ($query) use ($taxonomy) {
                $query->where('term_taxonomy_id', $taxonomy->getTermTaxonomyId());
            })
            ->get();
    }
}
```

## Custom Fields and Meta Management

### Complex Meta Structures

```php
class Product extends CustomPost
{
    // ... other code ...

    public function getVariations(): array
    {
        return $this->getMetaValue('variations') ?? [];
    }

    public function addVariation(array $variation): void
    {
        $variations = $this->getVariations();
        $variations[] = $variation;
        $this->setMeta('variations', $variations);
    }

    public function getGalleryImages(): array
    {
        $imageIds = $this->getMetaValue('gallery_images') ?? [];

        if (empty($imageIds)) {
            return [];
        }

        return Attachment::query()
            ->whereIn('ID', $imageIds)
            ->get()
            ->toArray();
    }

    public function setGalleryImages(array $imageIds): void
    {
        $this->setMeta('gallery_images', $imageIds);
    }
}
```

## Best Practices

### 1. Use Dedicated Models

Create dedicated models for your custom post types for better organization and type safety.

### 2. Define Meta Casts

Always define meta casts for proper type handling:

```php
protected array $metaCasts = [
    'price' => 'float',
    'is_featured' => 'boolean',
    'tags' => 'array',
    'created_at' => 'datetime',
];
```

### 3. Create Helper Methods

Add convenient methods for common operations:

```php
public function getFormattedPrice(): string
{
    $price = $this->getPrice();
    return $price ? '$' . number_format($price, 2) : 'N/A';
}
```

### 4. Use Scopes for Common Queries

Define scopes for frequently used queries:

```php
public function scopeInStock($query)
{
    return $query->addMetaToFilter('stock_quantity', 0, '>');
}
```

### 5. Validate Data

Implement validation in your setter methods:

```php
public function setPrice(float $price): void
{
    if ($price < 0) {
        throw new \InvalidArgumentException('Price cannot be negative');
    }

    $this->setMeta('price', $price);
}
```

### 6. Handle Relationships

Define relationships for better data organization:

```php
public function category()
{
    return $this->belongsToMany(
        Term::class,
        'term_relationships',
        'object_id',
        'term_taxonomy_id'
    );
}
```

This approach provides a clean, type-safe way to work with custom post types while leveraging all the power of WordPress ORM and Eloquent.
