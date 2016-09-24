git f327660ac356244c4fe62db527d1b906a968a36d

---

# HTTP-маршрутизация

- [Основы](#basic-routing)
- [Параметры роутов](#route-parameters)
    - [Обязательные параметры](#required-parameters)
    - [Необязательные параметры](#parameters-optional-parameters)
- [Именованные роуты](#named-routes)
- [Группы роутов](#route-groups)
    - [Посредник](#route-group-middleware)
    - [Namespaces](#route-group-namespaces)
    - [Поддоменные роуты](#route-group-sub-domain-routing)
    - [Префикс маршрутизации](#route-group-prefixes)
- [Привязка моделей к роутам](#route-model-binding)
    - [Неявное привязывание](#implicit-binding)
    - [Явное привязывание](#explicit-binding)
- [Подмена метода формы](#form-method-spoofing)
- [Доступ к текущему маршруту](#accessing-the-current-route)

<a name="basic-routing"></a>
## Основы маршутизации

The most basic Laravel routes simply accept a URI and a `Closure`, providing a very simple and expressive method of defining routes:
Самый основной маршрут Laravel просто принимает URI и `Closure`,` обеспечивая очень простой и выразительный метод определения маршрутов:
    Route::get('foo', function () {
        return 'Hello World';
    });

#### The Default Route Files

All Laravel routes are defined in your route files, which are located in the `routes` directory. These files are automatically loaded by the framework. The `routes/web.php` file defines routes that are for your web interface. These routes are assigned the `web` middleware group, which provides features like session state and CSRF protection. The routes in `routes/api.php` are stateless and are assigned the `api` middleware group.
Все маршруты Laravel определены в Ваших файлах, которые расположены в директории `routes`. Эти файлы автоматически загружаются фреймворком. Файл `routes/web.php` определяет маршруты, которые предназначены Вашему web интерфейсу. Этой группе маршрутов присваевается посредник `web`, который обеспечивает такими функциями, как состояние сессии и CSRF защита. Маршруты в `routes/api.php` не сохраняют состояние и этой группе назначен посредник `api`

For most applications, you will begin by defining routes in your `routes/web.php` file.
Для большенства приложения, Вы будете начинать с определения маршрутов в файле `routes/web.php`.

#### Методы доступные маршрутизатору

The router allows you to register routes that respond to any HTTP verb:
Маршрутизатор позволяет Вам регистрировать маршруты, чтобы ответить на любые методы запроса

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the `match` method. Or, you may even register a route that responds to all HTTP verbs using the `any` method:
Иногда Вам может понадобиться зарегестрировать маршрут, который отвечает на несколько методов запроса. Вы можете сделать это, используя метод `match`. Или Вы даже можете зарегестрировать маршрут для ответа на все методы запроса, используя метод `any`.

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF защита

Any HTML forms pointing to `POST`, `PUT`, or `DELETE` routes that are defined in the `web` routes file should include a CSRF token field. Otherwise, the request will be rejected. You can read more about CSRF protection in the [CSRF documentation](/docs/{{version}}/csrf):
Любая HTML форма, указывающая на маршруты `POST`, `PUT` or `DELETE`, которые определены в файле маршрутов `web`, должна включать в себя поле CSRF токена. 
    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="route-parameters"></a>
## Параметры маршрутов

<a name="required-parameters"></a>
### Обязательные параметры

Of course, sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:
Конечно, иногда тебе понадобится завхватить часть URI в пределах вашего маршрута. К примеру, Вам может быть понадобиться захватить ID пользователя из URL. Вы можете это сделать путем определения параметров маршрута:
    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

You may define as many route parameters as required by your route:
Вы можете определить столько параметров маршрута, сколько трубует Ваш маршрут:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Route parameters are always encased within `{}` braces and should consist of alphabetic characters. Route parameters may not contain a `-` character. Use an underscore (`_`) instead.
Параметры маршрута всегда заключены в `{}` скобки и должны состоять из букв алфавита. параметры маршрута не могут содержать символ `-`. Вместо него используйте подчеркивание (`_`).

<a name="parameters-optional-parameters"></a>
### Необязательные параметры

Occasionally you may need to specify a route parameter, but make the presence of that route parameter optional. You may do so by placing a `?` mark after the parameter name. Make sure to give the route's corresponding variable a default value:
Иногда Вам может понадобиться указать параметр маршрута, но сделать присутствие этого параметра не обязательным. Вы можете это сделать поместив знак `?` после имени параметра. Убедитесь в том, что по умолчанию дали маршруту соответствующее значение:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="named-routes"></a>
## Именованные маршруты

Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the `name` method onto the route definition:
Именнованные маршруты позволяют удобное генерирование URL-адресов или перенаправление на определенные машруты. Вы можете указать имя для маршрута путем изменения `name` метода на описание маршрута:

    Route::get('user/profile', function () {
        //
    })->name('profile');

You may also specify route names for controller actions:
Вы также можете указать имена маршрутов для метода контроллера:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### Создание URL-адресов для именованных маршрутов

Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via the global `route` function:
После того, как вы назначили имя для данного маршрута, вы можете использовать имя маршрута при генерации URL-адреса или редиректа через глобальную функции `route`:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

If the named route defines parameters, you may pass the parameters as the second argument to the `route` function. The given parameters will automatically be inserted into the URL in their correct positions:
Если названный маршрут определяет параметры, можно передавать параметры в качестве второго аргумента функции `route`. Данные параметры будут автоматически вставляться в правильном месте URL:  

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## Группы роутов

Route groups allow you to share route attributes, such as middleware or namespaces, across a large number of routes without needing to define those attributes on each individual route. Shared attributes are specified in an array format as the first parameter to the `Route::group` method.
Группы маршрутов позволяют разделять атрибуты маршрута, такие как посредники или пространств имен, через большое количество маршрутов без необходимости определять эти атрибуты на каждом отдельном маршруте. Общие атрибуты задаются в виде массива в качестве первого параметра метода `Route::group`.

<a name="route-group-middleware"></a>
### Посредники

To assign middleware to all routes within a group, you may use the `middleware` key in the group attribute array. Middleware are executed in the order they are listed in the array:
Чтобы назначить посредника для всех маршрутов в группе, вы можете использовать ключ `middleware`  в массиве атрибутов группы. Посредник выполняется в том порядке, в котором они перечислены в массиве:

    Route::group(['middleware' => 'auth'], function () {
        Route::get('/', function ()    {
            // Uses Auth Middleware
        });

        Route::get('user/profile', function () {
            // Uses Auth Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Пространство имен

Another common use-case for route groups is assigning the same PHP namespace to a group of controllers using the `namespace` parameter in the group array:
Другой распространенный вариант использования для группы маршрутов  присваиваивать одно и тоже PHP пространство имен к группе контроллеров с помощью `namespace` параметра в массиве группы:

    Route::group(['namespace' => 'Admin'], function() {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Remember, by default, the `RouteServiceProvider` includes your route files within a namespace group, allowing you to register controller routes without specifying the full `App\Http\Controllers` namespace prefix. So, you only need to specify the portion of the namespace that comes after the base `App\Http\Controllers` namespace.
Запомните, по умолчанию, 

<a name="route-group-sub-domain-routing"></a>
### Поддоменные роуты

Route groups may also be used to handle sub-domain routing. Sub-domains may be assigned route parameters just like route URIs, allowing you to capture a portion of the sub-domain for usage in your route or controller. The sub-domain may be specified using the `domain` key on the group attribute array:

    Route::group(['domain' => '{account}.myapp.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Route Prefixes

The `prefix` group attribute may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

    Route::group(['prefix' => 'admin'], function () {
        Route::get('users', function ()    {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-model-binding"></a>
## Route Model Binding

When injecting a model ID to a route or controller action, you will often query to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

<a name="implicit-binding"></a>
### Implicit Binding

Laravel automatically resolves Eloquent models defined in routes or controller actions whose variable names match a route segment name. For example:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

In this example, since the Eloquent `$user` variable defined on the route matches the `{user}` segment in the route's URI, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

#### Customizing The Key Name

If you would like model binding to use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Explicit Binding

To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings in the `boot` method of the `RouteServiceProvider` class:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Next, define a route that contains a `{user}` parameter:

    $router->get('profile/{user}', function(App\User $user) {
        //
    });

Since we have bound all `{user}` parameters to the `App\User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance from the database which has an ID of `1`.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

#### Customizing The Resolution Logic

If you wish to use your own resolution logic, you may use the `Route::bind` method. The `Closure` you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

    $router->bind('user', function ($value) {
        return App\User::where('name', $value)->first();
    });

<a name="form-method-spoofing"></a>
## Form Method Spoofing

HTML forms do not support `PUT`, `PATCH` or `DELETE` actions. So, when defining `PUT`, `PATCH` or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

You may use the `method_field` helper to generate the `_method` input:

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## Accessing The Current Route

You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Refer to the API documentation for both the [underlying class of the Route facade](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](http://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all accessible methods.