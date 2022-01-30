# Metrics

Compose robust Eloquent model metrics for cards, charts, emails etc.

Customize with a robust set of chainable options.

### Table of contents

- [Basic Usage](https://github.com/headlesslaravel/docs/blob/main/metrics.md#basic-usage)
- [Aggregates](https://github.com/headlesslaravel/docs/blob/main/metrics.md#aggregates)
- [Date Ranges](https://github.com/headlesslaravel/docs/blob/main/metrics.md#date-ranges)
- [Date Intervals](https://github.com/headlesslaravel/docs/blob/main/metrics.md#date-intervals)
- [Conditions & filters](https://github.com/headlesslaravel/docs/blob/main/metrics.md#conditions)
- [Comparisons](https://github.com/headlesslaravel/docs/blob/main/metrics.md#comparisons)
- [Display options](https://github.com/headlesslaravel/docs/blob/main/metrics.md#display)

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

## Date Ranges

### Dynamic Date Range

Allow a dropdown to change chart data ranges and refresh the same endpoint:

```php
Metric::make(Order::class)
    ->fromRequestRange()
    ->perDay();
```
Which is shorthand for:

```php
Metric::make(Order::class)
    ->from(Request::input('from', 'month'))
    ->to(Request::input('to', now()))
    ->perDay();
```

Here are a list of periods you can set `from` and `to`

| from | to |
|------|----|
| fromYear() | toYear() |
| fromQuarter() | toQuarter() |
| fromMonth() | toMonth() |
| fromWeek() | toWeek() |
| fromDay() | toDay() |
| fromHour() | toHour() |
| fromMinute() | toMinute() |

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
->fromYear()->perQuarter() // Q1
->fromQuarter()->perMonth() // January
->fromMonth()->perWeek() // Week 1
->fromWeek()->perDay() // Wednesday
->fromWeek()->perDate() // 1/22
->fromDay()->perHour() // 1pm
```


### Dynamic Intervals

Allow a dropdown to change chart data intervals and refresh the same endpoint:

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


### Compare previous period

> Get this month and previous month's order total

```php
Metric::make(Order::class)
    ->fromMonth()
    ->withPrevious()
    ->count('total')
```
```php
[
    'value' => 500,
    'previous' => 400,
    'percent' => 20
]
```
```php
Metric::make(Order::class)
    ->fromYear()
    ->perMonth()
    ->withPrevious()
    ->count('total')
```

```php
[
    [
        'label' => 'January'
        'value' => 500,
        'previous' => null,
        'percent' => null
    ],
    [
        'label' => 'February'
        'value' => 400,
        'previous' => 500,
        'percent' => 20
    ],   
]
```

## Display options

#### asText
#### asJson
#### asCsv
#### asHtml
#### asChart
