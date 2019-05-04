# lumen-passport-mongodb

[![Latest Stable Version](https://poser.pugx.org/kayrules/lumen-passport-mongodb/v/stable.svg)](https://packagist.org/packages/kayrules/lumen-passport-mongodb)
[![Latest Unstable Version](https://poser.pugx.org/kayrules/lumen-passport-mongodb/v/unstable.svg)](https://packagist.org/packages/kayrules/lumen-passport-mongodb)
[![License](https://poser.pugx.org/kayrules/lumen-passport-mongodb/license.svg)](https://packagist.org/packages/kayrules/lumen-passport-mongodb)


Making Laravel Passport work with Lumen and MongoDB

This repository was forked from [dusterio/lumen-passport](https://github.com/dusterio/lumen-passport) and added  [lucas-cardial/laravel-passport-mongodb](https://github.com/lucca-cardial/laravel-passport-mongodb) package to make Laravel Passport work with Lumen and MongoDB.


## Dependencies

* PHP >= 5.6.3
* Lumen >= 5.3
* jenssegers/mongodb >=3.2
* lucas-cardial/laravel-passport-mongodb >= 1.0

## Installation via Composer

First install Lumen if you don't have it yet:
```bash
$ composer create-project --prefer-dist laravel/lumen lumen-app
```

Then install Lumen Passport (it will fetch Laravel Passport along):

```bash
$ cd lumen-app
$ composer require kayrules/lumen-passport-mongodb
```

Or if you prefer, edit `composer.json` manually:

```json
{
    "require": {
        "kayrules/lumen-passport-mongodb": "^0.3.0"
    }
}
```

### Modify the bootstrap flow (```bootstrap/app.php``` file)

We need to enable both Laravel Passport provider and Lumen-specific provider:

```php
// Enable Facades
$app->withFacades();

// Register jenssegers/mongodb service provider before `$app->withEloquent()`
$app->register(Jenssegers\Mongodb\MongodbServiceProvider::class);

// Enable Eloquent
$app->withEloquent();

// Enable auth middleware (shipped with Lumen)
$app->routeMiddleware([
    'auth' => App\Http\Middleware\Authenticate::class,
]);

// Finally register service providers
$app->register(Kayrules\LumenPassport\PassportServiceProvider::class);
```

### Migrate and install Laravel Passport

```bash
# Create new tables for Passport
php artisan migrate

# Install encryption keys and other necessary stuff for Passport
php artisan passport:install
```

### Installed routes

Adding this service provider, will mount the following routes.

Verb | Path | NamedRoute | Controller | Action | Middleware
--- | --- | --- | --- | --- | ---
POST   | /oauth/token                             |            | AccessTokenController           | issueToken | -
GET    | /oauth/tokens                            |            | AuthorizedAccessTokenController | forUser    | auth
DELETE | /oauth/tokens/{token_id}                 |            | AuthorizedAccessTokenController | destroy    | auth
POST   | /oauth/token/refresh                     |            | TransientTokenController        | refresh    | auth
GET    | /oauth/clients                           |            | ClientController                | forUser    | auth
POST   | /oauth/clients                           |            | ClientController                | store      | auth
PUT    | /oauth/clients/{client_id}               |            | ClientController                | update     | auth
DELETE | /oauth/clients/{client_id}               |            | ClientController                | destroy    | auth
GET    | /oauth/scopes                            |            | ScopeController                 | all        | auth
GET    | /oauth/personal-access-tokens            |            | PersonalAccessTokenController   | forUser    | auth
POST   | /oauth/personal-access-tokens            |            | PersonalAccessTokenController   | store      | auth
DELETE | /oauth/personal-access-tokens/{token_id} |            | PersonalAccessTokenController   | destroy    | auth

Please note that some of the Laravel Passport's routes had to 'go away' because they are web-related and rely on sessions (eg. authorise pages). Lumen is an
API framework so only API-related routes are present.

## Configuration

Edit config/api.php to add prefix to all API endpoints. (Eg: `/api/oauth/token`)

```php
return [
	'prefix' => env('API_PREFIX', 'api')
];
```

Edit config/auth.php to suit your needs. A simple example:

```php
return [
    'defaults' => [
        'guard' => 'api',
        'passwords' => 'users',
    ],

    'guards' => [
        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => \App\User::class
        ]
    ]
];
```

## User model

Make sure your user model uses Passport's ```HasApiTokens``` trait, eg.:

```php
use MoeenBasra\LaravelPassportMongoDB\HasApiTokens;

class User extends Model implements AuthenticatableContract, AuthorizableContract
{
    use HasApiTokens, Authenticatable, Authorizable;

    /* rest of the model */
}
```

## Extra features

There are a couple of extra features that aren't present in Laravel Passport

### Allowing multiple tokens per client

Sometimes it's handy to allow multiple access tokens per password grant client. Eg. user logs in from several browsers
simultaneously. Currently Laravel Passport does not allow that.

```php
use Kayrules\LumenPassport\LumenPassport;

// Somewhere in your application service provider or bootstrap process
LumenPassport::allowMultipleTokens();

```

### Different TTLs for different password clients

Laravel Passport allows to set one global TTL for access tokens, but it may be useful sometimes
to set different TTLs for different clients (eg. mobile users get more time than desktop users).

Simply do the following in your service provider:

```php
// Second parameter is the client Id
LumenPassport::tokensExpireIn(Carbon::now()->addYears(50), 2);
```

If you don't specify client Id, it will simply fall back to Laravel Passport implementation.

### Console command for purging expired tokens

Simply run ```php artisan passport:purge``` to remove expired refresh tokens and their corresponding access tokens from the database.


## Running with Apache httpd

If you are using Apache web server, it may strip Authorization headers and thus break Passport.

Add the following either to your config directly or to ```.htaccess```:

```
RewriteEngine On
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
```

## License

The MIT License (MIT)
Copyright (c) 2016 Denis Mysenko

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
