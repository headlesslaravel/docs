# Formations

Formations combine Laravel features to form functioning API endpoints.

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


# Filters


## Search
Define columns or relationship columns to search:

```php
public $search = ['title', 'comments.body'];
```

```
/articles?search=laravel
```

---

## Sort

Define which columns are sortable:
```php
public $sort = ['name', 'created_at'];
```

```
/articles?sort=created_at
```

#### Descending Order
Change `sort` to `sort-desc` changes it's direction.
```
/articles?sort-desc=created_at
```

#### Sorting Relationships
Sort relationship counts, relationship columns and or alias them:

```php
public $sort = [
    'comments',
    'comments.upvotes',
    'comments.downvotes as disliked',
];
```
`sort=comments` `sort=upvotes` `sort=disliked`

---

## General Filters

Enable query parameters by adding to `filters()`:
```php
public function filters()
{
    return [
        Filter::make('active')->boolean(),
    ];
}
```
### boolean
Results where a boolean value is true or false:
```php
Filter::make('active')->boolean(),
```
```
/articles?active=true
```

### toggle
Same a boolean() but only allows `true`:
```php
Filter::make('active')->toggle(),
```
```
/articles?active=true
```

### options
Results where a column equals one of the optionst:
```php
Filter::make('status')->options(['published', 'draft']),
```
```
/articles?status=draft
```

### range
Results where a number has a min and max value:
```php
Filter::make('length')->range(),
```
```
/articles?length:min=5&length:max=100
```

### between
Results by a named range set
```php
Filter::make('length')
    ->between('small', [1, 100])
    ->between('medium', [101, 200])
    ->between('large', [201, 300])
```
```
/articles?length=small
```

### date
Results where a specific date is filtered by:
```php
Filter::make('published_at')->date(),
```
```
/articles?published_at=01/01/2021
```

### dateRange
Results where date range is filtered by:
```php
Filter::make('published_at')->dateRange(),
```
```
/articles?published_at:min=01/01/2021&published_at:max=02/01/2021
```

### search
Results where a group of columns are searched
```php
Filter::make('written-by')->search([
    'author_id',
    'author.name',
    'author.username'
]),
```
```
/articles?written-by=Brian
```

## Relationship Filters

Below are a few filters for Laravel's Eloquent relationships.

### relation
Results by a relationship primary key:
```php
Filter::make('author')->relation(),
```
```
/articles?author=1
```

### exists
Results where a relationship exists
```php
Filter::make('like')->exists(),
```
```
/articles?like:exists=true
```

### count
Results by a relationship count
```php
Filter::make('comments')->count(),
```
```
/articles?comments:count=5
```

### countRange
Results by a relationship count min / max range
```php
Filter::make('tags')->countRange(),
```
```
/articles?tags:min=5&tags:max=10
```

## Location Filters

These filters use geo location to filter Laravel models.

### radius
Results within a latitude, longtitude & distance.
```php
Filter::radius(),
```
```
/users?latitude=40.7517&longitude=-73.9755&distance=10
```

### bounds
Results within a set of map boundaries.
```php
Filter::bounds(),
```
```
/users?ne_lat=40.75555971122113&ne_lng=-73.96922446090224&sw_lat=40.74683062112093&sw_lng=-73.98124075728896
```
`ne_lat`, `ne_lng`, `sw_lat`, `sw_lng`


## Scope Filters

These filters use Laravel's model scope.

### scope
Results where a model scope is applied
```php
Filter::make('status')->scope(),
```
```
/articles?status=active
```

```php
public function scopeStatus($query, $value)
{
    $query->where('status', $value);
}
```


### scopeBoolean
Results in scope = `true`, or not in scope = `false`
```php
Filter::make('published')->scopeBoolean(),
```
```
/articles?published=true
```
```
/articles?published=false
```

>  `false` produces the opposite and returns all results that are not in the scope


## Soft Delete Filters

### withTrashed
Results include soft deleted and not deleted
```php
Filter::make('with-deleted')->withTrashed(),
```
```
/articles?with-deleted=true
```

### onlyTrashed
Results only include soft deleted
```php
Filter::make('only-deleted')->onlyTrashed(),
```
```
/articles?deleted=true
```

## Filter Helpers

Here are some scenarios and recipes:

### Auth restricted

Redirects to route('login') if unauthenticated.

```php
Filter::make('like')->exists()->auth(),
```

### Using a different public key

A url key different from the relationship or column name
```php
Filter::make('status', 'status_id'),
Filter::make('author', 'activeAuthor'),
```
```
/articles?status=1
```

### Adding a multiple values

Allow the filter to accept many values for the same key
```php
Filter::make('status')->multiple(),
```
```
/articles?status[]=active&status[]=draft
```

### Adding filter rules
Appends the rules set by the filter types.
```php
Filter::make('status')->withRules('in:active,inactive'),
```

### Adding value conditions
Add conditions based on the parameter value.
```php
Filter::make('status')
    ->when('active', function($query) {
        return $query->where('status', 'active');
    }),
```

### Appending the query
Add to the query to apply additional conditions.
```php
Filter::make('published')
    ->withQuery(function($query) {
        $query->whereNotNull('published_at');
    }),
```

### Converting to cents
When the public value is in dollars and db is in cents
```php
Filter::make('price')->asCents(),
```
```
/products?price=100 // where('price', 10000)
```

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
