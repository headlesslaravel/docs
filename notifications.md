# Headless Notifications

Notification Endpoints for Laravel Applications

---

### Install
Add the composer package:
```
composer require headlesslaravel/notifications
```

Call the Laravel command to add a migration.
```
php artisan notifications:table
```
For the full Laravel docs for notifications: [read more](https://laravel.com/docs/notifications)


### Available Endpoints

| Method | Endpoint | Action |
| ------ | -------- | ------ |
| get | /notifications | all |
| get | /notifications/unread | unread |
| get | /notifications/read | read |
| get | /notifications/count | count |
| post | /notifications/clear | clear |
| post | /notifications/{notification}/mark-as-read | markAsRead |
| delete | /notifications/{notification} | destroy |
