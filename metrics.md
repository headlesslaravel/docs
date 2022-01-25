# Headless Metrics

Compose metrics for cards and such.

### Usage

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
