# Formations

Formations are the combination of many concepts to form fully featured API endpoints.

### Table of contents

- [Routes](https://github.com/headlesslaravel/docs/blob/main/formations.md#routes)
    - [Resource](https://github.com/headlesslaravel/docs/blob/main/formations.md#resource-routes)
    - [Nested Resource](https://github.com/headlesslaravel/docs/blob/main/formations.md#nested-resource-routes)
    - [Pivot](https://github.com/headlesslaravel/docs/blob/main/formations.md#pivot-resource-routes)
- Fields (TODO)
- [Filters](https://github.com/headlesslaravel/docs/blob/main/formations.md#filters)
    - [Search](https://github.com/headlesslaravel/docs/blob/main/formations.md#search)
    - [Sort](https://github.com/headlesslaravel/docs/blob/main/formations.md#sort)
    - [General Filters](https://github.com/headlesslaravel/docs/blob/main/formations.md#general-filters)
    - [Relationship Filters](https://github.com/headlesslaravel/docs/blob/main/formations.md#relationship-filters)
    - [Location Filters](https://github.com/headlesslaravel/docs/blob/main/formations.md#location-filters)
    - [Scope Filters](https://github.com/headlesslaravel/docs/blob/main/formations.md#scope-filters)
    - [Soft Delete Filters](https://github.com/headlesslaravel/docs/blob/main/formations.md#soft-delete-filters)
    - [Filter Helpers](https://github.com/headlesslaravel/docs/blob/main/formations.md#filter-helpers)
- [Global Search](https://github.com/headlesslaravel/docs/blob/main/formations.md#global-search)
- [Imports](https://github.com/headlesslaravel/docs/blob/main/formations.md#imports)
- [Exports](https://github.com/headlesslaravel/docs/blob/main/formations.md#exports)

### Usage

Lets create a formation class using artisan.

```
php artisan make:formation ArticleFormation
```

> The model will be guessed based on the formation name or pass an option `--model=App\\Models\\Article`:

And then route the formation to create the fully functioning resource: [view routes](https://github.com/headlesslaravel/formations/wiki/Routes)

```php
Route::formation(ArticleFormation::class);
```
Here is the formation located in `App\Http\Formations` with example [search, sort & filters](https://github.com/headlesslaravel/formations/wiki/Filters)

```php
<?php

namespace App\Http\Formations;

use HeadlessLaravel\Formation\Filter;
use HeadlessLaravel\Formation\Formation;

class ArticleFormation extends Formation
{
    /**
     * The model class,
     *
     * @var string
     */
    public $model = \App\Models\Article::class;
    
    /**
     * The column to use for select options,
     *
     * @var string
     */
    public $display = 'title';
    
    /**
     * The searchable columns,
     *
     * @var array
     */
    public $search = ['title', 'comments.body'];

    /**
     * The sortable columns.
     *
     * @var array
     */
    public $sort = ['created_at', 'comments'];

    /**
     * Define the filters.
     *
     * @return array
     */
    public function filters():array
    {
        return [
            Filter::make('author')->related(),
            Filter::make('published_at')->dateRange(),
            Filter::make('comments')->countRange(),
        ];
    }
}
```

# Routes


### Resource Routes

Full resourceful controllers are available including soft delete endpoints.

```php
Route::formation(AuthorFormation::class)
    ->resource('articles');
```
| Method | Action | Endpoint | Name |
| --------|--------|----------|-------|
| GET | index | /authors/ | authors.index |
| GET | show | /authors/1 | authors.show |
| GET | create | /authors/new | authors.create | 
| POST | store | /authors/new | authors.store | 
| GET | edit | /authors/1/edit | authors.edit | 
| PUT | edit | /authors/1/edit | authors.update | 
| DELETE | destroy | /authors/1/ | authors.destory | 
| PUT | restore | /authors/1/restore | authors.restore | 
| DELETE | force-delete | /authors/1/force-delete | authors.force-delete |

### Nested Resource Routes

```php
Route::formation(ArticleFormation::class)
    ->resource('authors.articles');
```
| Method | Action | Endpoint | Name |
| --------|--------|----------|-------|
| GET | index | /authors/1/articles | authors.articles.index |
| GET | show | /authors/1/articles/1 | authors.articles.show |
| GET | create | /authors/1/articles/new | authors.articles.create |
| POST | store | /authors/1/articles/new | authors.articles.store |
| GET | edit | /authors/1/articles/1/edit | authors.articles.edit |
| PUT | edit | /authors/1/articles/1/edit | authors.articles.update |
| DELETE | destroy | /authors/1/articles/1/ | authors.articles.destory |
| PUT | restore | /authors/1/articles/1/restore | authors.articles.restore |
| DELETE | force-delete | /authors/1/articles/1/force-delete | authors.articles.force-delete |

### Pivot Resource Routes

```php
Route::formation(ArticleFormation::class)
    ->resource('authors.articles')
    ->asPivot();
```

| Method | Action | Endpoint | Name | Description |
| --------|--------|----------|-------|-------------|
| GET | index | /posts/1/tags | posts.tags.index | List all tags of a post |
| GET | show | /posts/1/tags/1 | posts.tags.show | Show single tag of a post |
| POST | sync | /posts/1/tags/sync | posts.tags.sync | Accepts `selected` array of tags to sync  |
| POST | toggle | /posts/1/tags/toggle | posts.tags.toggle | Accepts `selected` array of tags to toggle |
| POST | attach | /posts/1/tags/attach | posts.tags.attach | Accepts `selected` array of tags to attach |
| DELETE | detach | /posts/1/tags/detach | posts.tags.detach | Accepts `selected` array of tags to detach |

### Specific Routes

To choose only specific routes, call `->only()` with the `action` name

Only `index` will be defined based on this example:

```php
Route::formation(ArticleFormation::class)->only('index');
```

---

# Helpers


### Adding hardcoded filters

Useful for defining routes with filters: `/active`

```php
public function active(ArticleFormation $formation)
{
    $formation->merge(['status' => 'active']);

    return $formation->results();
}
```

### Adding query conditionals

For hard coding conditions within controllers:

```php
public function index($author_id, ArticleFormation $formation)
{
    $formation->where('author_id', $author->id);

    return $formation->results();
}
```

# Global Search

To add global search to your app, define a route with what formations should be searched.

```php
Route::seeker('search', [
    PostFormation::class,
    AuthorFormation::class
]);
```

This will use the following formation details to populate the response

- `$model` for what model to query
- `$display` for what model value to display in results
- `$detailRouteName` to make a result link (guessed by default based on formation)
- `$search` for what columns are searchable

```
/search?term=hi
```
```json
{
   "data":[
      {
         "data":[
            {
               "value":1,
               "display":"Hi post"
            }
         ],
         "meta":{
            "route":"posts.show",
            "group":"posts"
         }
      },
      {
         "data":[
            {
               "value":2,
               "display":"Hi author"
            }
         ],
         "meta":{
            "route":"authors.show",
            "group":"authors"
         }
      }
   ]
}
```

# Imports

Importing models is very easy with formations

### Routing

```php
Route::formation(ArticleFormation::class)
    ->resource('articles')
    ->asImport()
```

| Method | Action | Endpoint | Name | Description |
| --------|--------|----------|-------|-----------|
| GET | create | /imports/{resource}| posts.imports.create | Download a import template file of headers |
| POST | store | /imports/{resource} | posts.imports.store | Accepts `file`, an upload of what to import  |


### Fields

Lastly, define the accepted fields within your formation:

```php
public function import(): array
{
    return [
        Field::make('title')->rules(['required', 'min:2']),
        Field::make('body')->rules(['required', 'min:2']),
        Field::make('author.name')->rules(['required', 'exists:users,name']),
        Field::make('category_title', 'category.title')->rules(['required', 'categories,title']),
    ];
}
```

# Exports

Exporting models is very easy with formations

### Routing

```php
Route::formation(ArticleFormation::class)
    ->resource('articles')
    ->asExport()
```

| Method | Action | Endpoint | Name | Description |
| --------|--------|----------|-------|-----------|
| GET | create | /exports/{resource}| posts.exports.create | Download a export of a formation |


### Fields

Lastly, define the accepted fields within your formation:

```php
public function export(): array
{
    return [
        Field::make('title'),
        Field::make('body'),
        Field::make('author', 'author.name'),
        Field::make('category', 'category.title'),
    ];
}
```

### Parameters

Adding a `?columns=title` will exclude the other columns and only include `title`

Adding any filters defined on the formation will also work:

```php
Filter::make('author')->relation()
```
```
/exports/posts?columns=title&author=33
```
