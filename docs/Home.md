Welcome to the wp-orm wiki !

This documentation only covers the specific points of this library, if you want to know more about Eloquent, the easiest is to look at [the documentation of Eloquent](https://laravel.com/docs/11.x/eloquent).

**Documentation**

- [Installation](#installation)

### Getting Started

- [Quick Start Guide](Quick-Start-Guide.md) - Get up and running quickly with practical examples

### Core Operations

- [CRUD Operations](CRUD-Operations.md) - Complete guide to Create, Read, Update, Delete operations
- [Meta Data Management](Meta-Management.md) - Comprehensive meta data operations and batch processing
- [Performance Optimization](Performance-Optimization.md) - Eager loading, avoiding N+1 queries, and optimization techniques

### Advanced Topics

- [Advanced Query Building](Advanced-Query-Building.md) - Complex queries and advanced patterns
- [Custom Post Types](Custom-Post-Types.md) - Working with custom post types and dedicated models
- [Relationships](Relationships.md) - Model relationships and advanced relationship patterns
- [Multisite Support](Multisite-Support.md) - Working with WordPress multisite installations

### Reference

- [Models Reference](Models-Reference.md) - Complete API reference for all available models

### Legacy Topics

- [Attribute & meta casting](Attribute-&-meta-casting.md) - Type casting for meta fields
- [Create custom model](Create-custom-model.md) - Building custom models
- [Use Eloquent facade](DB-facade.md) - Using the database facade
- [Events](Events.md) - Model events and hooks
- [Filter data](Filter-data.md) - Filtering and scoping data

### Support

- [Troubleshooting](Troubleshooting.md) - Common issues and solutions
- [Upgrading from v3 to v4](Upgrading-from-v3-to-v4.md) - Migration guide

## Installation

**Requirements**

The server requirements are basically the same as for [WordPress](https://wordpress.org/about/requirements/) with the addition of a few ones :

- PHP >= 8.2
- [Composer](https://getcomposer.org/)

**Installation**

You can use [Composer](https://getcomposer.org/). Follow the [installation instructions](https://getcomposer.org/doc/00-intro.md) if you do not already have composer installed.

```bash
composer require dbout/wp-orm
```

In your PHP script, make sure you include the autoloader:

```php
require __DIR__ . '/vendor/autoload.php';
```

🎉 You have nothing more to do, you can use the library now! Not even need to configure database accesses because it's the `wpdb` connection that is used.

## Contributing

💕 🦄 Contributions are most welcome so feel free to reach out, post issues and / or propose improvements.
