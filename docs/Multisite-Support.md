# Multisite Support

WordPress ORM provides comprehensive support for WordPress multisite installations, allowing you to work with sites, blogs, and cross-site data efficiently.

## Multisite Models

### Site Model

The Site model represents individual sites in a multisite network.

#### Class: `Dbout\WpOrm\Models\Multisite\Site`

```php
use Dbout\WpOrm\Models\Multisite\Site;

// Get all sites in the network
$sites = Site::all();

// Find a specific site
$site = Site::find(1);

// Get site by domain
$site = Site::query()
    ->where('domain', 'example.com')
    ->where('path', '/')
    ->first();

// Create a new site
$site = new Site();
$site->setDomain('newsite.example.com');
$site->setPath('/');
$site->setRegistered(now());
$site->setPublic(true);
$site->save();
```

### Blog Model

The Blog model is an alias for the Site model, provided for backwards compatibility.

#### Class: `Dbout\WpOrm\Models\Multisite\Blog`

```php
use Dbout\WpOrm\Models\Multisite\Blog;

// Same functionality as Site model
$blogs = Blog::all();
```

## Working with Sites

### Site Properties

```php
$site = Site::find(1);

// Basic properties
echo $site->getBlogId();      // Site ID
echo $site->getSiteId();      // Network ID
echo $site->getDomain();      // Domain name
echo $site->getPath();        // Site path
echo $site->getRegistered();  // Registration date
echo $site->getLastUpdated(); // Last update timestamp

// Status properties
echo $site->getPublic();      // Public status
echo $site->getArchived();    // Archived status
echo $site->getMature();      // Mature content flag
echo $site->getSpam();        // Spam status
echo $site->getDeleted();     // Deleted status
echo $site->getLangId();      // Language ID
```

### Site Status Management

```php
$site = Site::find(2);

// Check site status
if ($site->getPublic()) {
    echo "Site is public";
}

if ($site->getArchived()) {
    echo "Site is archived";
}

if ($site->getSpam()) {
    echo "Site is marked as spam";
}

// Update site status
$site->setPublic(false);
$site->setArchived(true);
$site->save();
```

### Querying Sites

```php
// Active public sites only
$activeSites = Site::query()
    ->where('public', 1)
    ->where('archived', 0)
    ->where('spam', 0)
    ->where('deleted', 0)
    ->get();

// Sites registered in the last month
$recentSites = Site::query()
    ->where('registered', '>=', now()->subMonth())
    ->orderBy('registered', 'desc')
    ->get();

// Sites by domain pattern
$subdomainSites = Site::query()
    ->where('domain', 'like', '%.example.com')
    ->get();

// Sites with specific path
$subsites = Site::query()
    ->where('path', 'like', '/subsite/%')
    ->get();
```

## Cross-Site Data Access

### Switching Between Sites

```php
// Get posts from a specific site
$currentSiteId = get_current_blog_id();
$targetSiteId = 2;

// Switch to target site
switch_to_blog($targetSiteId);

// Now queries will run against the target site
$posts = Post::query()
    ->where('post_status', 'publish')
    ->get();

// Switch back to original site
switch_to_blog($currentSiteId);
// or restore_current_blog();
```

### Working with Site-Specific Data

```php
use Dbout\WpOrm\Models\Post;
use Dbout\WpOrm\Models\User;

// Function to get posts from multiple sites
function getPostsFromSites(array $siteIds): array
{
    $allPosts = [];
    $currentSiteId = get_current_blog_id();

    foreach ($siteIds as $siteId) {
        switch_to_blog($siteId);

        $posts = Post::query()
            ->where('post_status', 'publish')
            ->where('post_type', 'post')
            ->get();

        foreach ($posts as $post) {
            $post->site_id = $siteId; // Add site context
        }

        $allPosts = array_merge($allPosts, $posts->toArray());
    }

    switch_to_blog($currentSiteId);
    return $allPosts;
}

// Usage
$networkPosts = getPostsFromSites([1, 2, 3]);
```

## Network-Wide Queries

### Users Across the Network

Users are shared across the entire network, so you don't need to switch sites:

```php
// Get all network users
$networkUsers = User::all();

// Get users with specific role on any site
$networkAdmins = User::query()
    ->whereHas('metas', function ($query) {
        $query->where('meta_key', 'like', '%capabilities%')
              ->where('meta_value', 'like', '%administrator%');
    })
    ->get();

// Get users registered in the last week
$recentUsers = User::query()
    ->where('user_registered', '>=', now()->subWeek())
    ->get();
```

### Site-Specific User Capabilities

```php
// Get users with capabilities on a specific site
function getUsersForSite(int $siteId): Collection
{
    $prefix = "wp_{$siteId}_";
    if ($siteId === 1) {
        $prefix = 'wp_'; // Main site uses wp_ prefix
    }

    return User::query()
        ->whereHas('metas', function ($query) use ($prefix) {
            $query->where('meta_key', $prefix . 'capabilities');
        })
        ->get();
}

// Usage
$siteUsers = getUsersForSite(2);
```

## Network Options

### Site-Specific Options

```php
use Dbout\WpOrm\Models\Option;

// Get options for current site
$siteOptions = Option::all();

// Get option from specific site
function getOptionFromSite(int $siteId, string $optionName): ?string
{
    $currentSiteId = get_current_blog_id();
    switch_to_blog($siteId);

    $option = Option::findOneByName($optionName);
    $value = $option ? $option->getOptionValue() : null;

    switch_to_blog($currentSiteId);
    return $value;
}

// Usage
$siteName = getOptionFromSite(2, 'blogname');
```

### Network-Wide Settings

```php
// Network options are stored in the main site's options table
function getNetworkOption(string $optionName): ?string
{
    $currentSiteId = get_current_blog_id();
    switch_to_blog(1); // Switch to main site

    $option = Option::findOneByName($optionName);
    $value = $option ? $option->getOptionValue() : null;

    switch_to_blog($currentSiteId);
    return $value;
}

// Usage
$networkName = getNetworkOption('site_name');
```

## Advanced Multisite Patterns

### Site Collection Helper

```php
class SiteCollection
{
    protected Collection $sites;

    public function __construct(Collection $sites)
    {
        $this->sites = $sites;
    }

    public static function active(): self
    {
        $sites = Site::query()
            ->where('public', 1)
            ->where('archived', 0)
            ->where('spam', 0)
            ->where('deleted', 0)
            ->get();

        return new self($sites);
    }

    public function getPostCounts(): array
    {
        $counts = [];
        $currentSiteId = get_current_blog_id();

        foreach ($this->sites as $site) {
            switch_to_blog($site->getBlogId());
            $counts[$site->getBlogId()] = Post::query()
                ->where('post_status', 'publish')
                ->where('post_type', 'post')
                ->count();
        }

        switch_to_blog($currentSiteId);
        return $counts;
    }

    public function getLatestPosts(int $limit = 5): array
    {
        $allPosts = [];
        $currentSiteId = get_current_blog_id();

        foreach ($this->sites as $site) {
            switch_to_blog($site->getBlogId());

            $posts = Post::query()
                ->where('post_status', 'publish')
                ->where('post_type', 'post')
                ->orderBy('post_date', 'desc')
                ->limit($limit)
                ->get();

            foreach ($posts as $post) {
                $post->site_id = $site->getBlogId();
                $post->site_domain = $site->getDomain();
                $allPosts[] = $post;
            }
        }

        switch_to_blog($currentSiteId);

        // Sort all posts by date
        usort($allPosts, function ($a, $b) {
            return $b->getPostDate() <=> $a->getPostDate();
        });

        return array_slice($allPosts, 0, $limit);
    }
}

// Usage
$activeSites = SiteCollection::active();
$postCounts = $activeSites->getPostCounts();
$latestPosts = $activeSites->getLatestPosts(10);
```

### Network Dashboard Helper

```php
class NetworkDashboard
{
    public static function getNetworkStats(): array
    {
        return [
            'total_sites' => Site::count(),
            'active_sites' => Site::query()
                ->where('public', 1)
                ->where('archived', 0)
                ->where('spam', 0)
                ->where('deleted', 0)
                ->count(),
            'total_users' => User::count(),
            'recent_users' => User::query()
                ->where('user_registered', '>=', now()->subMonth())
                ->count(),
            'spam_sites' => Site::query()->where('spam', 1)->count(),
            'archived_sites' => Site::query()->where('archived', 1)->count(),
        ];
    }

    public static function getRecentActivity(): array
    {
        return [
            'new_sites' => Site::query()
                ->where('registered', '>=', now()->subWeek())
                ->orderBy('registered', 'desc')
                ->limit(10)
                ->get(),
            'new_users' => User::query()
                ->where('user_registered', '>=', now()->subWeek())
                ->orderBy('user_registered', 'desc')
                ->limit(10)
                ->get(),
        ];
    }
}

// Usage
$stats = NetworkDashboard::getNetworkStats();
$activity = NetworkDashboard::getRecentActivity();
```

## Best Practices for Multisite

### 1. Always Switch Context Properly

```php
// Good
$currentSiteId = get_current_blog_id();
switch_to_blog($targetSiteId);
// ... perform operations
switch_to_blog($currentSiteId);

// Better - with exception handling
function withSiteContext(int $siteId, callable $callback): mixed
{
    $currentSiteId = get_current_blog_id();

    try {
        switch_to_blog($siteId);
        return $callback();
    } finally {
        switch_to_blog($currentSiteId);
    }
}

// Usage
$posts = withSiteContext(2, function() {
    return Post::query()->where('post_status', 'publish')->get();
});
```

### 2. Cache Cross-Site Queries

```php
function getCachedSitePosts(int $siteId): Collection
{
    $cacheKey = "site_{$siteId}_posts";

    return cache()->remember($cacheKey, 3600, function() use ($siteId) {
        return withSiteContext($siteId, function() {
            return Post::query()
                ->where('post_status', 'publish')
                ->orderBy('post_date', 'desc')
                ->limit(10)
                ->get();
        });
    });
}
```

### 3. Use Bulk Operations

```php
// Bulk site operations
function performBulkSiteOperation(array $siteIds, callable $operation): array
{
    $results = [];
    $currentSiteId = get_current_blog_id();

    foreach ($siteIds as $siteId) {
        try {
            switch_to_blog($siteId);
            $results[$siteId] = $operation($siteId);
        } catch (\Exception $e) {
            $results[$siteId] = ['error' => $e->getMessage()];
        }
    }

    switch_to_blog($currentSiteId);
    return $results;
}

// Usage
$results = performBulkSiteOperation([1, 2, 3], function($siteId) {
    return Post::query()->where('post_status', 'publish')->count();
});
```

### 4. Monitor Performance

```php
// Monitor site switching performance
function timedSiteOperation(int $siteId, callable $operation): array
{
    $start = microtime(true);
    $currentSiteId = get_current_blog_id();

    switch_to_blog($siteId);
    $result = $operation();
    switch_to_blog($currentSiteId);

    $end = microtime(true);

    return [
        'result' => $result,
        'execution_time' => $end - $start,
        'site_id' => $siteId
    ];
}
```

This multisite support allows you to build powerful network-wide applications while maintaining clean, readable code.
