Welcome to the wp-orm wiki ! 

This documentation only covers the specific points of this library, if you want to know more about Eloquent, the easiest is to look at [the documentation of Eloquent](https://laravel.com/docs/11.x/eloquent).

**Documentation** 

- [Installation](#installation)
- [Attribute & meta casting](https://github.com/dimitriBouteille/wp-orm/wiki/Attribute-&-meta-casting)
- [Create custom model](https://github.com/dimitriBouteille/wp-orm/wiki/Create-custom-model)
- [Use Eloquent facade](https://github.com/dimitriBouteille/wp-orm/wiki/DB-facade)
- [Events](https://github.com/dimitriBouteille/wp-orm/wiki/Events)
- [Filter data](https://github.com/dimitriBouteille/wp-orm/wiki/Filter-data)

## Installation

**Requirements**

The server requirements are basically the same as for [WordPress](https://wordpress.org/about/requirements/) with the addition of a few ones :

- PHP >= 8.2
- [Composer](https://getcomposer.org/)

**Installation**

You can use [Composer](https://getcomposer.org/). Follow the [installation instructions](https://getcomposer.org/doc/00-intro.md) if you do not already have composer installed.

~~~bash
composer require dbout/wp-orm
~~~

In your PHP script, make sure you include the autoloader:

~~~php
require __DIR__ . '/vendor/autoload.php';
~~~

🎉 You have nothing more to do, you can use the library now! Not even need to configure database accesses because it's the `wpdb` connection that is used.

## Contributing

💕 🦄 Contributions are most welcome so feel free to reach out, post issues and / or propose improvements.