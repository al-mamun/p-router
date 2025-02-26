# p-router

```
  _____            _____             _
 |  __ \          |  __ \           | |
 | |__)   ______  | |__) |___  _   _| |_ ___ _ __
 |  ___/ |______| |  _  // _ \| | | | __/ _ \ '__|
 | |              | | \ \ (_) | |_| | ||  __/ |
 |_|              |_|  \_\___/ \__,_|\__\___|_|

```

A simple, lightweight, and powerful PHP Router with rich features like Middleware and Controllers. Heavily inspired by the way Laravel handles routing.

## Features:

- Supports `GET` `POST` `PUT` `PATCH` and `DELETE` HTTPS verbs
- The methods that the router supports are - `get()` `post()` `put()` `patch()` `delete()` `match()` `any()`
- Named Routes
- Middleware
- Regular Expression Constraints for parameters
- Fallback method
- Easy way to manage request
- Helper methods
- CSRF protection
- Command line interface(CLI)


## Table of Contents:

- **[Installation](#Installation)**
- **[Examples](#Examples)**
- **[Directories](#Directories)**
- **[Routes](#Routes)**
- **[Middlewares](#middlewares)**
- **[Controllers](#Controllers)**
- **[Request](#Request)**
- **[Handle Html View Content File](#Handle-Html-View-Content-File)**
- **[Helpers](#Helpers)**


## Installation:

Installation is possible using Composer.

```bash
composer require devamirul/p-router
```
Add the following script to `composer.json` file:

```json
"autoload": {
    "psr-4": {
        "App\\": "app/"
    }
},
"scripts": {
    "start": [
        "php -S 127.0.0.1:8000"
    ],
    "middleware": "cd vendor/devamirul/p-router/src/CLI && php createMiddleware.php && cd -",
    "controller": "cd vendor/devamirul/p-router/src/CLI && php createController.php && cd -",
    "app": "cd vendor/devamirul/p-router/src/CLI && php createApp.php && cd -"
}
```

Run the following command:

```bash
composer dump-autoload
```
```bash
composer app
```

## Examples:

#### Basic route:

```php
$router->get('/', function () {
    echo 'welcome to p-route';
})->name('home');
```

or

```php
$router->get('/', [WelcomeController::class, 'index'])->name('home');
```

#### Dynamic Route:

```php
$router->get('/users/:id', function(){
    //
})->where(['id' => '^\d+$'])->name('user');
```

#### Middleware:

```php
$router->get('/users/:id', function(){
    //
})->middleware('auth')->where(['id' => '^\d+$'])->name('user');
```
**You can do method chaining if you want.**


## Directories:

`app/config`: This folder contains system config files. Make your changes only in the config file.

`app/Middlewares`: Create your custom middlewares in this folder.

`app/Controllers`: Create your custom controllers in this folder.


## Routes:

### Available Router Methods:

The router allows you to register routes that respond to any HTTP verb:

```php
$router->get($uri, $callback);
$router->post($uri, $callback);
$router->put($uri, $callback);
$router->patch($uri, $callback);
$router->delete($uri, $callback);
$router->match($uri, $callback);
$router->any($uri, $callback);
```

### Basic Routes:

Routes accept a URI and a closure or a array, providing a very simple and expressive method of defining routes and behavior without complicated routing configuration files.

1. First Create index.php file.
2. Define App root path.
3. Require vendor autoload file.
4. Create `router` singleton instance.
5. Define routes.
6. Run the application via the `run()` method.

```php
<?php
/**
 * Define root directory.
 */
define('APP_ROOT', dirname(__FILE__));

/**
 * Require composer autoloader.
 */
require __DIR__ . '/vendor/autoload.php';

// Get singleton route instance.
$router = Devamirul\PRouter\Router::singleton();

// Define routes.
$router->get('/greeting', function () {
    return 'Hello World';
});

// Resolve and run application.
$router->run();
?>
```

Or create a separate `route.php` file and include that file in the `index.php` file.

First create route.php or name the file according to your choice:

```php
<?php

// Define routes
$router->get('/greeting', function () {
    return 'Hello World';
});

$router->fallback(function () {
    return 'Fallback route';
});

?>
```

Include route.php in the `index.php` file:

```php
/**
 * Require composer autoloader.
 */
require __DIR__ . '/vendor/autoload.php';

// Get singleton route instance.
$router = Devamirul\PRouter\Router::singleton();

// Require route.php
require_once './route.php';

// Resolve and run application.
$router->run();
```

Let's discuss the second parameter. The second parameter accepts a closure or an array of key value pairs. The 'key' of the array will be a class and the value will be a method of the class, the method will be invoked by the class.

```php
use App\Controllers\WelcomeController;

$router->get('/', [WelcomeController::class, 'index'])->name('home');
```

**Use return instead of echo.**

```php
// Right way.
$router->get('/greeting', function () {
    return 'Hello World';
});

// wrong way.
$router->get('/greeting', function () {
    echo 'Hello World';
});
```

<!-- ### Enable case sensitive routes:

By default router case sensitive mode is enabled.
The third parameter of route is set to true by default. You can disable case sensitive mode by setting the second parameter to false:

```php
$router->get('/', [WelcomeController::class, 'index'], false)->name('home');
``` -->

### Named Routes:

Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the name method onto the route definition:

**Route names should always be unique.**

```php
$router->get('/user/profile', function () {
    // ...
})->name('profile');
```

#### Generating URLs To Named Routes

Once you assign a name to a given route, you can redirect via `toRoute()`:

```php
return toRoute('profile');
```

If the named route defines parameters, you may pass the parameters as the second argument to the `toRoute` function. The given parameters will automatically be inserted into the generated URL in their correct positions:

```php
$router->get('/user/:id/profile', function () {
    // ...
})->name('profile');

return toRoute('profile', ['id' => 1]);
```

If you pass additional parameters in the array, those key / value pairs will automatically be added to the generated URL's query string:

```php
$router->get('/user/:id/profile', function () {
    // ...
})->name('profile');

return toRoute('profile', ['id' => 1, 'photos' => 'yes']);
```
**If you use an asterisk (*) in the route, you cannot call it via `toRoute()`.**


### Route Parameters:

#### Required Parameters:

Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You can get it through `$request->getParam()` method.

```php
$router->get('/user/:id', function (Request $request) {
    return 'User ' . $request->getParam('id');
});
```

You may define as many route parameters as required by your route:

```php
$router->get('/posts/:post/comments/:comment', function (Request $request) {
    //Get all parameters array.
    return 'User ' . $request->getParam();

    //Get specific parameter.
    return 'User ' . $request->getParam('post');
});
```

Route parameters will always start with Colon ':' and should contain alphanumeric characters.

#### Parameters:

Automatically get 'Request' instances in your route callback or controller.

In callback:

```php
$router->get('/user/:id', function (Request $request) {
    return 'User ' . $request->getParam('id');
});
```
In controller:

```php
$router->get('/user/:id', [UserController::class, 'index']);
```

```php
namespace App\Http\Controllers;

use Devamirul\PhpMicro\core\Foundation\Application\Request\Request;
use Devamirul\PhpMicro\core\Foundation\Controller\BaseController;

class UserController extends BaseController {
    public function index(Request $request) {
        return 'User ' . $request->getParam('id');
    }
}
```

#### Optional Parameters:

Occasionally you may need to specify a route parameter that may not always be present in the URI. You may do so by placing a question sign  `?` mark after the parameter:

**It is important to note that optional parameter are always placed at the end of the URLs.**

```php
$router->get('/user/:name?', function () {
    //
});
```

#### Regular Expression Constraints:

You can restrict the format of your route parameter by using the `where` method on a route instance.
The `where()` method takes a regular expression as parameter which determines how the parameter should be delimited. The "where()" method will accept the serialized parameters of the router's dynamic parameters:

```php
$router->get('/user/:id', function () {
    // ...
})->where(['id' => '^\d+$']);
```

Regular expression constraints for optional parameter:

```php
$router->get('/user/:name?', function () {
    //
})->where(['name' => '^[a-zA-Z ]*$']);
```

Regular expression constraints for multiple optional parameters:

```php
$router->get('/profile/:id/user/:name?', function () {
    //
})->where(['id' => '^\d+$', 'name' => '^[a-zA-Z ]*$']);
```

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the match method. Or, you may even register a route that responds to all HTTP verbs using the any method:

```php
$router->match(['get', 'post'], '/', function () {
    // ...
});

$router->any('/', function () {
    // ...
});
```

### Route wildcard:

You can use dynamic routes using asterisks:

```php
$router->get('admin/*', function () {
    // ...
});
```

In the example above, you can dynamically use any path after `admin/`. The asterisk is used as a wildcard and matches any combination of characters.

### Redirect Routes:

If you are defining a route that redirects to another URI, you may use the `redirect()` method. This method provides a convenient shortcut so that you do not have to define a full route

```php
$router->redirect('/here', '/there');
```

## CSRF Protection:

Remember, any HTML forms pointing to POST, PUT, PATCH, or DELETE routes that are defined in the web routes file should include a CSRF token field. Otherwise, the request will be rejected:

```php
<form method="POST" action="/profile">
    <?=setCsrf()?>
    ...
</form>
```

## Middlewares:

`app/Middlewares`: Middleware provides a convenient mechanism for inspecting and filtering HTTP requests entering your application.

The predefined middleware files are:- `AuthMiddleware.php` `CsrfMiddleware.php`

<em>**By default you will get a request instance in the handle method.**</em>

### Make middleware:

To create a new middleware, use the `composer middleware` command:

```cli
composer middleware
```

The command line interface will ask you for a middleware name, you enter a name. It will automatically add "Middleware" to the name you provided. For example, you want to create a middleware named "example". Then your middleware class will be `ExampleMiddleware.php`


```php
namespace App\Middlewares;

use Devamirul\PRouter\Interfaces\Middleware;
use Devamirul\PRouter\Request\Request;

class AuthMiddleware implements Middleware {
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request): void {
        //
    }
}
```

For example, This framework includes a middleware that verifies the user of your application is authenticated. If the user is not authenticated, the middleware will redirect the user to your application's login screen. However, if the user is authenticated, the middleware will allow the request to proceed further into the application.

```php
public function handle(Request $request): void {
    if (!isset($_SESSION['user'])) {
        redirect('/login');
    }
    return;
}
```
### Add middleware

After creating the middleware add it to the middleware array in the 'app/config/middleware.php' file.
Add your own middleware to this list and assign it an alias of your choice:

```php
'middleware' => [
    'csrf' => Devamirul\PRouter\Middleware\Middlewares\CsrfMiddleware::class,
    'auth' => App\Middlewares\AuthMiddleware::class
],
```

If you would like to assign middleware to specific routes, you may invoke the middleware method when defining the route.
Once the middleware alias is defined, you use the alias when assigning middleware to routes:

```php
Router::get('/users/:id', function(){
    //
})->middleware('auth');
```

You can assign multiple middleware at once if you want:

```php
Router::put('/users/:id', function(){
    //
})->middleware(['auth','csrf']);
```

<!-- #### Middleware Parameters:

**We can optionally pass parameters to the middleware.**

```php
Router::get('/profile', [AuthenticatedController::class, 'show'])->name('profile')->middleware('auth:parameter');
```

Multiple parameters:

```php
Router::get('/profile', [AuthenticatedController::class, 'show'])->name('profile')->middleware('auth:parameter');
``` -->
<!--
Handle middleware arguments:

```php
public function handle(Request $request, array $guards) {
    if (!empty($guards)) {
        // Handle middleware arguments.
        foreach ($guards as $guard) {
            if (!Auth::guard($guard)->check() && $guard === 'admin') {
                return redirect('/admin/login');
            }
        }
    } elseif (!Auth::check()) {
        return redirect('/login');
    }
    return;
}
``` -->

#### Set default middlewares:

If you want to set some middleware to Https verbs by default, you can do that very easily, The defined middleware will run when that https method request is handled:

Open `app/config/middleware.php`

```php
'get'        => [],
'post'       => [ 'csrf' ],
'put'        => [ 'csrf' ],
'patch'      => [ 'csrf' ],
'delete'     => [ 'csrf', 'auth' ],
```

## Controllers:

`app/Controllers`: Controllers respond to user actions (submitting forms, show users, view data, and any action etc.). Controllers are classes that extend the BaseController class.

<em>**By default you will get request instance in each method.**</em>

### Make controller

To create a new controller, use the `composer controller` command:

```cli
composer controller
```
The command line interface will ask you for a controller name, you enter a name. It will automatically add "Controller" to the name you provided. For example you want to create a controller named "example". Then your controller class will be `ExampleController.php`

```php
namespace App\Controllers;

use Devamirul\PRouter\Request\Request;
use Devamirul\PRouter\Controller\BaseController;

class UserController extends BaseController {
    /**
     * Show user.
     */
    public function show(Request $request) {
        return 'user name -' . $request->input('name');
    }
}
```

## Request:

Framework's Request class provides an object-oriented way to interact with the current HTTP request being handled by your application as well as retrieve the input that were submitted with the request.

#### Accessing The Request:

You can get request instance through the request helper function:

```php
// Get all input data.
request()->all();
```

```php
// Get all input data.
request()->input();

// Get input data specified by key, return default data if key not found.
request()->input('name', 'default');

// Get input data specified by key.
request()->only('name', 'email');

// Get path.
request()->path();

// Get all query.
request()->query();

// Get query data specified by key.
request()->query('name');

// Get current method.
request()->method();

// Get all input data.
request()->all();

// Get dynamic params.
request()->getParam();

// Get specific param.
request()->getParam('id');
```

Also you will get methods.

`isGet()` `isPost()` `isPut()` `isPatch()` `isDelete()`


## Handle Html View Content File:

You can easily view `html content` from a controller or callback function.

Simply `require' the file you want to view:

```php
$router->get('/', function () {
    require_once './home.php';
});
```
In Content file:

```php
<body>
    <h1>Home Page</h1>
</body>
```

You can easily view the data in the `content` file:

```php
$router->get('/', function (Request $request) {
    $name = 'Amirul islam';

    require_once './home.php';
});
```
In Content file:

```php
<body>
    <h1>Welcome Mr <?= $name ?> </h1>

    <?php
        foreach ($request->input() as $value) {
            echo $value;
        }
    ?>
</body>
```

## Helpers:

### Table of Contents

- **[General Helpers](#General-Helpers)**
- **[Form Helpers](#Form-Helpers)**
- **[Request Helpers](#Request-Helpers)**
- **[Response Helpers](#Response-Helpers)**


### General Helpers:

Get config data:
```php
config('app', 'timezone');
```
View the data in details then exit the code:
```php
dd([1,2,3]);
```
View the data in details:
```php
dump();
```

### Form Helpers:

Set new CSRF value:
```php
setCsrf();
```
Example:
```php
<form>
    <?=setCsrf()?>
</form>
```
Check CSRF is valid or not, return bool:
```php
isCsrfValid();
```
Set form method, like put/patch/delete:
```php
setMethod();
```
Example:
```php
<form>
    <?=setMethod('delete')?>
</form>
```

### Request Helpers:

**Get request instance**
```php
request();
```
Example:
```php
request()->input();
```

### Response Helpers:

Redirect link:
```php
return redirect('/redirect-link');
```

Finds route by route name and redirect this route:
```php
return toRoute('users');
```