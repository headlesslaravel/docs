# Headless Cards

The card component class for dashboards and such.

## Installation

You can install the package via composer:

```bash
composer require headlesslaravel/cards
```
Make a card class to manage multiple cards of one  type:
```bash
php artisan make:cards Dashboard
```
Then add the class to an endpoint:
```php
Route::cards('api/dashboard', Dashboard::class);
```

```php
<?php

namespace App\Http\Cards;

use HeadlessLaravel\Cards\Cards;

class Dashboard extends Cards
{
    public function cards(): array
    {
        return [
            Card::make('Total Users')
                ->link('/users')
                ->component('number-card')
                ->value(function() {
                    return User::count();
                }),
        ];
    }
}
```
```bash
/api/dashboard
```
```json
{
    "data": [
        {
            "key": "total_users",
            "title": "Total Users",
            "value": 5,
            "component": "number-card",
            "link": "/users",
            "endpoint": "api/dashboard/total-users"
        }
    ]
}
```
You can also reference the single method using the `key` in slug format.

This is useful when you want your ui to update / filter a single card. 
```
/api/dashboard/total-users
```

```json
{
    "key": "total_users",
    "title": "Total Users",
    "value": 5,
    "component": "number-card",
    "link": "/users",
    "endpoint": "api/dashboard/total-users"
}
```

This is only a basic example. The real power comes in the filtering multiple cards using one query string and validating that the query string is accurate.

```php
<?php

namespace App\Http\Cards;

use HeadlessLaravel\Cards\Cards;

class Dashboard extends Cards
{
    public function rules()
    {
        return [
            'from' => ['nullable', 'date', 'before_or_equal:to'],
            'to' => ['nullable', 'date', 'after_or_equal:from'],
        ];
    }
    
    public function cards(): array
    {
        return [
        
            Card::make('Total Users')
                ->link('/users')
                ->component('number-card')
                ->value(function() {
                    return User::whereBetween('created_at', [
                        Request::input('from', now()),  
                        Request::input('to', now())
                    ])->count();
                }),
            
            Card::make('Total Orders', 'total_orders')
                ->link('/orders')
                ->component('number-card')
                ->value(function() {
                    return Order::whereBetween('created_at', [
                        Request::input('from', now()),  
                        Request::input('to', now())
                    ])->count();
                }),
        ];
    }
}
```
Which results in both models  being filtered by the same query string.
```bash
/dashboard?from=...&to=...
```
```json
{
    "data": [
        {
            "key": "total_users",
            "title": "Total Users",
            "value": 5,
            "component": "number-card",
            "link": "/users",
            "endpoint": "api/dashboard/total-users"
        },
        {
            "key": "total_orders",
            "title": "Total Orders",
            "value": 50,
            "component": "number-card",
            "link": "/orders",
            "endpoint": "api/dashboard/total-orders"
        }
    ]
}
```
The filters also work on a single card request:
```bash
/dashboard/total-users?from=...&to=...
```
```bash
/dashboard/total-orders?from=...&to=...
```

You can pass a number of things as values:

### Views
```php
Card::make('Welcome')->view('cards.welcome');
```
```json
{
    "key": "welcome",
    "title": "Welcome",
    "value": "<h1>Welcome!</h1>",
    "component": null,
    "link": null,
    "endpoint": "api/dashboard/welcome"
}
```
### Http 
```php
Card::make('Weather')->http('api.com/weather', 'data.results.0');
```
```json
{
    "key": "weather",
    "title": "Weather",
    "value": {
        "today": "90 degrees",
        "tomorrow": "50 degrees"
    },
    "component": null,
    "link": null,
    "endpoint": "api/dashboard/weather"
}
```
Which is just shorthand for:
```php
Card::make('Weather')
    ->value(function() {
        return Http::get('api.com/weather')->json('data.results.0');
    }),
```

### Cache
Any values in a callable can be cached: (seconds)

```php
Card::make('Weather')
    ->cache(60)
    ->value(function() {
        return Http::get('api.com/weather')->json('data.today');
    }),
```

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
