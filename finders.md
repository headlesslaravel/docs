# Finders

Control a model's index results from the url querystring.

### Table of contents
- [Search](https://github.com/headlesslaravel/docs/blob/main/finders.md#search)
- [Sort](https://github.com/headlesslaravel/docs/blob/main/finders.md#sort)
- [Filters](https://github.com/headlesslaravel/docs/blob/main/finders.md#filters)
    - [General](https://github.com/headlesslaravel/docs/blob/main/finders.md#general-filters)
    - [Relationships](https://github.com/headlesslaravel/docs/blob/main/finders.md#relationship-filters)
    - [Location](https://github.com/headlesslaravel/docs/blob/main/finders.md#location-filters)
    - [Scopes](https://github.com/headlesslaravel/docs/blob/main/finders.md#scope-filters)
    - [Soft Delete](https://github.com/headlesslaravel/docs/blob/main/finders.md#soft-delete-filters)
    - [Helpers](https://github.com/headlesslaravel/docs/blob/main/finders.md#filter-helpers)
- [Standalone](https://github.com/headlesslaravel/docs/blob/main/finders.md#standalone)

# Search

Use the key as the `search` query parameter to search columns & relationship columns:

```php
public function search(): array
{
    return [
        Search::make('title'),
        Search::make('comments.body'),
    ];
}
```

# Sort

Use the key in `sort` and `sort-desc` query parameters:

```php
public function sort(): array
{
    return [
        Sort::make('title'),
        Sort::make('comment_body', 'comments.body'),
        Sort::make('comments'),
    ];
}
```



# Filters

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

## Standalone

The above examples are assuming the use of Formations but this package was designed to work with Eloquent models and enables using the same sort, search and filters without any other packages installed. The following example will use url's query string parameters from the request:

```
/posts?search=Laravel&sort-desc=upvotes&category=1
```
```php
Post::query()
    ->search([
        Search::make('title'),
        Search::make('comments.body'),
    ])->sort([
        Sort::make('title'),
        Sort::make('comments'),
        Sort::make('upvotes', 'comments.upvotes'),
    ])->filters([
        Filter::make('category')->relation(),
        Filter::make('author')->relation(),
        Filter::make('comments')->count(),
    ])->paginate()
