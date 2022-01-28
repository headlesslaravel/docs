# Metrics

Compose robust Eloquent model metrics for cards, charts, emails etc.

Customize with a robust set of chainable options.

### Table of contents

- [Basic Usage](https://github.com/headlesslaravel/docs/blob/main/metrics.md#basic-usage)
- [Aggregates](https://github.com/headlesslaravel/docs/blob/main/metrics.md#aggregates)
    - [count](https://github.com/headlesslaravel/docs/blob/main/metrics.md#count)
    - [sum](https://github.com/headlesslaravel/docs/blob/main/metrics.md#sum)
    - [min](https://github.com/headlesslaravel/docs/blob/main/metrics.md#min)
    - [max](https://github.com/headlesslaravel/docs/blob/main/metrics.md#max)
    - [avg](https://github.com/headlesslaravel/docs/blob/main/metrics.md#avg)
- Date Ranges
    - from
    - to  
- Conditions
    - Manual where clauses
    - Filter class support
- Comparisons
    -  Period over period
    -  Percentage
- Output options
    - asText
    - asJson
    - asCsv
    - asHtml
    - asChart
- Things I don't know about yet
    - percentages
    - 

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
Metric::make(Order::class)->count('total')
```
