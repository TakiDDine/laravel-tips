# Laravel Tips

Awesome Laravel tips. Based on the of Povilas Korop's idea from [Laravel Daily](http://www.laraveldaily.com/). PR are welcome!

## Summary

- [Controllers](#controllers)
- [Models](#models)
- [Models Relations](#models-relations)
- [Migrations](#migrations)
- [Views](#views)
- [Routing](#routing)
- [Validation](#validation)
- [Mails](#mails)
- [Artisan](#artisan)
- [Factories](#factories)
- [Logging](#logging)

## Controllers

⬆️ [Go to top](#summary) ➡️ [Next (Models)](#models)

- [Single Action Controllers](#single-action-controllers)

### Single Action Controllers

If you want to create a controller with just one action, you can use `__invoke()` method and even create "invokable" controller.

Route: `Route::get('user/{id}', 'ShowProfile');`
Artisan: `php artisan make:controller ShowProfile --invokable` 

```php
class ShowProfile extends Controller
{
    public function __invoke($id)
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

## Models

⬆️ [Go to top](#summary) ⬅️ [Previous (Controllers)](#controllers) ➡️ [Next (Models Relations)](#models-relations)

- [Eloquent where date methods](#eloquent-where-date-methods)
- [Increments and decrements](#increments-and-decrements)
- [No timestamp columns](#no-timestamp-columns)
- [Set logged in user with Observers](#set-logged-in-user-with-observers)
- [Soft-deletes: multiple restore](#soft-deletes-multiple-restore)
- [Model all: columns](#model-all-columns)
- [To Fail or not to Fail](#to-fail-or-not-to-fail)
- [Column name change](#column-name-change)

### Eloquent where date methods

In Eloquent, check the date with functions `whereDay()`, `whereMonth()`, `whereYear()`, `whereDate()` and `whereTime()`.

```php
$products = Product::whereDate('created_at', '2018-01-31')->get();
$products = Product::whereMonth('created_at', '12')->get();
$products = Product::whereDay('created_at', '31')->get();
$products = Product::whereYear('created_at', date('Y'))->get();
$products = Product::whereTime('created_at', '=', '14:13:58')->get();
```

### Increments and decrements

If you want to increment some DB column in some table, just use `increment()` function. Oh, and you can increment not only by 1, but also by some number, like 50.

```php
Post::find($post_id)->increment('view_count');
User::find($user_id)->increment('points', 50);
```

### No timestamp columns

If your DB table doesn't contain timestamp fields `created_at` and `updated_at`, you can specify that Eloquent model wouldn't use them, with `$timestamps = false` property.

```php
class Company extends Model
{
    public $timestamps = false;
}
```

### Set logged in user with Observers

Use `make:observer` and fill in `creating()` method to automatically set up `user_id` field for current logged in user.

```php
class PostObserver
{
    public function creating(Post $post)
    {
        $post->user_id = auth()->id();
    }
}
```

### Soft-deletes: multiple restore

When using soft-deletes, you can restore multiple rows in one sentence.

```php
Post::withTrashed()->where('author_id', 1)->restore();
```

### Model all: columns

When calling Eloquent's `Model::all()`, you can specify which columns to return.

```php
$users = User::all(['id', 'name', 'email']);
```

### To Fail or not to Fail

In addition to `findOrFail()`, there's also Eloquent method `firstOrFail()` which will return 404 page if no records for query are found.

```php
$user = User::where('email', 'povilas@laraveldaily.com')->firstOrFail();
```

### Column name change

In Eloquent Query Builder, you can specify "as" to return any column with a different name, just like in plain SQL query.

```php
$users = DB::table('users')->select('name', 'email as user_email')->get();
```

## Models Relations

⬆️ [Go to top](#summary) ⬅️ [Previous (Models)](#models) ➡️ [Next (Migrations)](#migrations)

- [OrderBy on Eloquent relationships](#orderby-on-eloquent-relationships)
- [Raw DB Queries: havingRaw()](#raw-db-queries-havingraw)
- [Eloquent has() deeper](#eloquent-has-deeper)
- [Has Many. How many exactly?](#has-many-how-many-exactly)
- [Default model](#default-model)
- [Use hasMany to create Many](#use-hasmany-to-create-many)

### OrderBy on Eloquent relationships

You can specify orderBy() directly on your Eloquent relationships.

```php
public function products()
{
    return $this->hasMany(Product::class);
}

public function productsByName()
{
    return $this->hasMany(Product::class)->orderBy('name');
}
```

### Raw DB Queries: havingRaw()

You can use RAW DB queries in various places, including `havingRaw()` function after `groupBy()`.

```php
Product::groupBy('category_id')->havingRaw('COUNT(*) > 1')->get();
```

### Eloquent has() deeper

You can use Eloquent `has()` function to query relationships even two layers deep!

```php
// Author -> hasMany(Book::class);
// Book -> hasMany(Rating::class);
$authors = Author::has('books.ratings')->get();
```

### Has Many. How many exactly?

In Eloquent `hasMany()` relationships, you can filter out records that have X amount of children records.

```php
// Author -> hasMany(Book::class)
$authors = Author::has('books', '>', 5)->get();
```

### Default model

You can assign a default model in `belongsTo` relationship, to avoid fatal errors when calling it like `{{ $post->user->name }}` if $post->user doesn't exist.

```php
public function user()
{
    return $this->belongsTo('App\User')->withDefault();
}
```

### Use hasMany to create Many

If you have `hasMany()` relationship, you can use `saveMany()` to save multiple "child" entries from your "parent" object, all in one sentence.

```php
$post = Post::find(1);
$post->comments()->saveMany([
    new Comment(['message' => 'First comment']),
    new Comment(['message' => 'Second comment']),
]);
```

## Migrations

⬆️ [Go to top](#summary) ⬅️ [Previous (Models Relations)](#models-relations) ➡️ [Next (Views)](#views)

- [Order of Migrations](#order-of-migrations)
- [Migration fields with timezones](#migration-fields-with-timezones)
- [Database migrations column types](#database-migrations-column-types)
- [Default Timestamp](#default-timestamp)

### Order of Migrations

If you want to change the order of DB migrations, just rename the file's timestamp, like from `2018_08_04_070443_create_posts_table.php` to`2018_07_04_070443_create_posts_table.php` (changed from `2018_08_04` to `2018_07_04`).

They run in alphabetical order.

### Migration fields with timezones

Did you know that in migrations there's not only `timestamps()` but also `timestampsTz()`, for the timezone?

```php
Schema::create('employees', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('email');
    $table->timestampsTz();
});
```

Also, there are columns `dateTimeTz()`, `timeTz()`, `timestampTz()`, `softDeletesTz()`.

### Database migrations column types

There are interesting column types for migrations, here are a few examples.

```php
$table->geometry('positions');
$table->ipAddress('visitor');
$table->macAddress('device');
$table->point('position');
$table->uuid('id');
```

See all column types on the [official documentation](https://laravel.com/docs/master/migrations#creating-columns).

### Default Timestamp

While creating migrations, you can use `timestamp()` column type with option
`useCurrent()`, it will set `CURRENT_TIMESTAMP` as default value.

```php
$table->timestamp('created_at')->useCurrent();
$table->timestamp('updated_at')->useCurrent();
```

## Views

⬆️ [Go to top](#summary) ⬅️ [Previous (Views)](#views) ➡️ [Next (Routing)](#routing)

- [$loop variable in foreach](#loop-variable-in-foreach)
- [Does view file exist?](#does-view-file-exist)
- [Error code Blade pages](#error-code-blade-pages)
- [View without controllers](#view-without-controllers)
- [Blade @auth](#blade-auth)

### $loop variable in foreach

Inside of foreach loop, check if current entry is first/last by just using `$loop` variable.

```blade
@foreach ($users as $user)
     @if ($loop->first)
        This is the first iteration.
     @endif

     @if ($loop->last)
        This is the last iteration.
     @endif

     <p>This is user {{ $user->id }}</p>
@endforeach
```

There are also other properties like `$loop->iteration` or `$loop->count`.
Learn more on the [official documentation](https://laravel.com/docs/master/blade#the-loop-variable).

### Does view file exist?

You can check if View file exists before actually loading it.

```php
if (view()->exists('custom.page')) {
 // Load the view
}
```

You can even load an array of views and only the first existing will be actually loaded.

```php
return view()->first(['custom.dashboard', 'dashboard'], $data);
```

### Error code Blade pages

If you want to create a specific error page for some HTTP code, like 500 - just create a blade file with this code as filename, in `resources/views/errors/500.blade.php`, or `403.blade.php` etc, and it will automatically be loaded in case of that error code.

### View without controllers

If you want route to just show a certain view, don't create a Controller method, just use `Route::view()` function.

```php
// Instead of this
Route::get('about', 'TextsController@about');
// And this
class TextsController extends Controller
{
    public function about()
    {
        return view('texts.about');
    }
}
// Do this
Route::view('about', 'texts.about');
```

### Blade @auth

Instead of if-statement to check logged in user, use `@auth` directive.

Typical way:
```blade
@if(auth()->user())
    // The user is authenticated.
@endif
```

Shorter:
```blade
@auth
    // The user is authenticated.
@endauth
```

The opposite is `@guest` directive:
```blade
@guest
    // The user is not authenticated.
@endguest
```

## Routing

⬆️ [Go to top](#summary) ⬅️ [Previous (Routing)](#routing) ➡️ [Next (Validation)](#validation)

- [Route group within a group](#route-group-within-a-group)
- [Wildcard subdomains](#wildcard-subdomains)
- [What's behind the routes?](#whats-behind-the-routes)

### Route group within a group

In Routes, you can create a group within a group, assigning a certain middleware only to some URLs in the "parent" group.

```php
Route::group(['prefix' => 'account', 'as' => 'account.'], function() {
    Route::get('login', 'AccountController@login');
    Route::get('register', 'AccountController@register');
    
    Route::group(['middleware' => 'auth'], function() {
        Route::get('edit', 'AccountController@edit');
    });
});
```

### Wildcard subdomains

You can create route group by dynamic subdomain name, and pass its value to every route.

```php
Route::domain('{username}.workspace.com')->group(function () {
    Route::get('user/{id}', function ($username, $id) {
        //
    });
});
```

### What's behind the routes?

Want to know what routes are actually behind `Auth::routes()`?
From Laravel 7, it’s in a separate package, so check the file `/vendor/laravel/ui/src/AuthRouteMethods.php`.

```php
public function auth()
{
    return function ($options = []) {
        // Authentication Routes...
        $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
        $this->post('login', 'Auth\LoginController@login');
        $this->post('logout', 'Auth\LoginController@logout')->name('logout');
        // Registration Routes...
        if ($options['register'] ?? true) {
            $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
            $this->post('register', 'Auth\RegisterController@register');
        }
        // Password Reset Routes...
        if ($options['reset'] ?? true) {
            $this->resetPassword();
        }
        // Password Confirmation Routes...
        if ($options['confirm'] ?? class_exists($this->prependGroupNamespace('Auth\ConfirmPasswordController'))) {
            $this->confirmPassword();
        }
        // Email Verification Routes...
        if ($options['verify'] ?? false) {
            $this->emailVerification();
        }
    };
}
```

Before Laravel 7, check the file `/vendor/laravel/framework/src/illuminate/Routing/Router.php`.

## Validation

⬆️ [Go to top](#summary) ⬅️ [Previous (Routing)](#routing) ➡️ [Next (Mails)](#mails)

### Image validation

While validating uploaded images, you can specify the dimensions you require.

```php
['photo' => 'dimensions:max_width=4096,max_height=4096']
```

## Mails

⬆️ [Go to top](#summary) ⬅️ [Previous (Validation)](#validation) ➡️ [Next (Artisan)](#artisan)

- [Testing email into laravel.log](#testing-email-into-laravellog)
- [Preview Mailables](#preview-mailables)

### Testing email into laravel.log

If you want to test email contents in your app but unable or unwilling to set up something like Mailgun, use `.env` parameter `MAIL_DRIVER=log` and all the email will be saved into `storage/logs/laravel.log` file, instead of actually being sent.

### Preview Mailables

If you use Mailables to send email, you can preview the result without sending, directly in your browser. Just return a Mailable as route result:

```php
Route::get('/mailable', function () {
    $invoice = App\Invoice::find(1);
    return new App\Mail\InvoicePaid($invoice);
});
```

## Artisan

⬆️ [Go to top](#summary) ⬅️ [Previous (Mails)](#mails) ➡️ [Next (Factories)](#factories)

- [Artisan command parameters](#artisan-command-parameters)

### Artisan command parameters

When creating Artisan command, you can ask the input in variety of ways: `$this->confirm()`, `$this->anticipate()`, `$this->choice()`.

```php
// Yes or no?
if ($this->confirm('Do you wish to continue?')) {
    //
}

// Open question with auto-complete options
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

// One of the listed options with default index
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
```

## Factories

⬆️ [Go to top](#summary) ⬅️ [Previous (Artisan)](#artisan) ➡️ [Next (Logging)](#logging)

- [Factory callbacks](#factory-callbacks)

### Factory callbacks

While using factories for seeding data, you can provide Factory Callback functions to perform some action after record is inserted.

```php
$factory->afterCreating(App\User::class, function ($user, $faker) {
    $user->accounts()->save(factory(App\Account::class)->make());
});
```

## Logging

⬆️⬆️⬆️ [Go to top](#summary) ⬆️⬆️⬆️

- [Logging with parameters](#logging-with-parameters)

### Logging with parameters

You can write `Log::info()`, or shorter `info()` message with additional parameters, for more context about what happened.

```php
Log::info('User failed to login.', ['id' => $user->id]);
```