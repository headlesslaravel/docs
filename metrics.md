# Metrics

Compose robust Eloquent model metrics for cards, charts, emails etc.

Customize with a robust set of chainable options.

### Table of contents

- [Basic Usage](https://github.com/headlesslaravel/docs/blob/main/metrics.md#basic-usage)
- [Aggregates](https://github.com/headlesslaravel/docs/blob/main/metrics.md#aggregates)
- [Date Intervals](https://github.com/headlesslaravel/docs/blob/main/metrics.md#date-intervals)
- [Date Ranges](https://github.com/headlesslaravel/docs/blob/main/metrics.md#date-ranges)
- [Conditions & filters](https://github.com/headlesslaravel/docs/blob/main/metrics.md#conditions)
    - Manual where clauses
    - Filter class support
- [Comparisons](https://github.com/headlesslaravel/docs/blob/main/metrics.md#comparisons)
- Output options

### Basic Usage

> What are the number of posts for the past 7 days?

```php
Metric::make(Post::class)
    ->from(now()->subDays(7))
    ->count()
```
or add the `HasMetrics` trait to a model and use like so:
```php
Post::metrics()
    ->from(now()->subDays(7))
    ->count()
```


## Aggregates 

#### Count

Get the total count of the results:

> What are the total number of orders?

```php
Metric::make(Order::class)->count()
```

#### Sum

Get the sum of a column in the results:

> What is the total revenue from orders?

```php
Metric::make(Order::class)->sum('total')
```

#### Min

Get the minimum value of a result column:

> What is the lowest order total?

```php
Metric::make(Order::class)->min('total')
```

#### Max

Get the maximum value of a result column:

> What is the highest order total?

```php
Metric::make(Order::class)->max('total')
```

#### Avg

Get the average result column value:

> What is the average order total?

```php
Metric::make(Order::class)->avg('total')
```

## Date Intervals

When an interval is added to a metric, the results will be per that interval.

Metrics can be returned per day, week, month etc.

> What is the average order total per day for the last week?
```php
Metric::make(Order::class)
    ->avg('total')
    ->fromWeek()
    ->perDay();
```

Here are a few common examples:

```php
->fromYear()->perQuarter()
->fromQuarter()->perMonth()
->fromMonth()->perWeek()
->fromWeek()->perDay()
->fromDay()->perHour()
```

### Dynamic Intervals

Allow a dropdown to change chart data intervals and refresh the same

```
/cards/dashboard/total-orders?interval=day
```
```php
Metric::make(Order::class)
    ->fromMonth()
    ->perRequestInterval()
    ->count('total')
    ->asChart()
```

Which is short for:
```php
Metric::make(Order::class)
    ->per(Request::input('interval', 'day'))
    ->count('total')
```

## Date Ranges

## Conditions

Get more specific with metrics by applying conditions.

#### Manual where clauses

Apply a where condition directory to the metric chain:

```php
Metric::make(Order::class)
    ->where('state', 'NY')
    ->count('total')
    
Metric::make(Order::class)
    ->where('total', '>' '5')
    ->count('total')
```

#### Apply Metric Filters

Automatically apply request query parameters using filters:

```php
Metric::make(Order::class)
    ->filters([
        Filter::make('customer')->relation()
    ])
```

## Comparisons


## Output Options

#### asText
#### asJson
#### asCsv
#### asHtml
#### asChart
