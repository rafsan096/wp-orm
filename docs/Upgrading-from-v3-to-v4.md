The [v4 upgrade](https://github.com/dimitriBouteille/wp-orm/releases/tag/v4.0.0) has Some breaking changes, especially if with the v3 you use deprecated functions. All functions have been removed and replaced in some cases.


## Update to latest v4.x

First update to the latest minor version of v4.x.

```bash
composer require dbout/wp-orm
```

## Removed functions

Here is a detail of the functions removed and their equivalent with the new version : 

| Function | Replaced by |
| --- | --- |
| `\Dbout\WpOrm\Builders\OptionBuilder::findOneByName()` | `\Dbout\WpOrm\Models\Option::findOneByName()` |
| `\Dbout\WpOrm\Builders\PostBuilder::findOneByName()` | `\Dbout\WpOrm\Models\Post::findOneByName()` |
| `\Dbout\WpOrm\Builders\UserBuilder::findOneByEmail()` | `\Dbout\WpOrm\Models\User::findOneByEmail()` |
| `\Dbout\WpOrm\Builders\UserBuilder::findOneByLogin()` | `\Dbout\WpOrm\Models\User::findOneByLogin()` |
| `\Dbout\WpOrm\Builders\Comment::setAuthor()` | `\Dbout\WpOrm\Models\Comment::setCommentAuthor()` |
| `\Dbout\WpOrm\Builders\Comment::getAuthor()` | `\Dbout\WpOrm\Models\Comment::getCommentAuthor()` |
| `\Dbout\WpOrm\Builders\Comment::setAuthorEmail()` | `\Dbout\WpOrm\Models\Comment::setCommentAuthorEmail()` |
| `\Dbout\WpOrm\Builders\Comment::getAuthorEmail()` | `\Dbout\WpOrm\Models\Comment::getCommentAuthorEmail()` |
| `\Dbout\WpOrm\Builders\Comment::setAuthorUrl()` | `\Dbout\WpOrm\Models\Comment::setCommentAuthorUrl()` |
| `\Dbout\WpOrm\Builders\Comment::getAuthorUrl()` | `\Dbout\WpOrm\Models\Comment::getCommentAuthorUrl()` |
| `\Dbout\WpOrm\Builders\Comment::setAuthorIp()` | `\Dbout\WpOrm\Models\Comment::setCommentAuthorIP()` |
| `\Dbout\WpOrm\Builders\Comment::getAuthorIp()` | `\Dbout\WpOrm\Models\Comment::getCommentAuthorIP()` |
| `\Dbout\WpOrm\Builders\Comment::setContent()` | `\Dbout\WpOrm\Models\Comment::setCommentContent()` |
| `\Dbout\WpOrm\Builders\Comment::getContent()` | `\Dbout\WpOrm\Models\Comment::getCommentContent()` |
| `\Dbout\WpOrm\Builders\Comment::setKarma()` | `\Dbout\WpOrm\Models\Comment::setCommentKarma()` |
| `\Dbout\WpOrm\Builders\Comment::getKarma()` | `\Dbout\WpOrm\Models\Comment::getCommentKarma()` |
| `\Dbout\WpOrm\Builders\Comment::setAgent()` | `\Dbout\WpOrm\Models\Comment::setCommentAgent()` |
| `\Dbout\WpOrm\Builders\Comment::getAgent()` | `\Dbout\WpOrm\Models\Comment::getCommentAgent()` |
| `\Dbout\WpOrm\Builders\Comment::setType()` | `\Dbout\WpOrm\Models\Comment::setCommentType()` |
| `\Dbout\WpOrm\Orm\AbstractModel::getTable()` | _No equivalent_ |