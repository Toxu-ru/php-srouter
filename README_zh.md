# php simple router

非常轻量级的单一文件的路由器。自定义性强

> referrer the project **[noahbuscher\macaw](https://github.com/noahbuscher/Macaw)** , but add some feature.

- 支持请求方法: `GET` `POST` `PUT` `DELETE` `HEAD` `OPTIONS`
- 支持事件: `found` `notFound`. 你可以做一些事情当触发事件时(比如记录日志等)
- 支持设置匹配路由的解析器: `SRoute::setMatchedRouteParser()`. 你可以自定义如何调用匹配的路由处理程序.
- 支持自动匹配路由到控制器就像 yii 一样, 请参看配置项 `autoRoute`. 
- 支持手动调度一个路由通过方法 `SRoute::dispatchTo()`
- more interesting config, please see `SRoute::$_config`
- 你也可以不需要配置什么,它也能很好的工作

## 安装

```json
{
    "require": {
        "inhere/sroute": "dev-master"
    }
}
```

## 使用

first, import the class

```php
use inhere\sroute\SRoute;
```

## 添加路由

```php
// match GET. handler use Closure
SRoute::get('/', function() {
    echo 'hello';
});

// access 'test/john'
SRoute::get('/test/(\w+)', function($arg) {
    echo $arg; // 'john'
});

// match POST
SRoute::post('/user/login', function() {
    var_dump($_POST);
});

// match GET or POST
SRoute::map(['get', 'post'], '/user/login', function() {
    var_dump($_GET, $_POST);
});

// match any method
SRoute::any('/home', function() {
    echo 'hello, you request page is /home';
});
```

### 使用控制器方法

```php
// if you config 'ignoreLastSep' => true, '/index' is equals to '/index/'
SRoute::get('/index', 'app\controllers\Home@index');
```

### 动态匹配控制器方法

动态匹配控制器方法, 需配置 `'dynamicAction' => true`

> NOTICE: 使用动态匹配控制器方法, 应当使用 `any()` 添加路由. 即此时无法限定请求方法 `REQUEST_METHOD`

```php
// access '/home/test' will call 'app\controllers\Home::test()'
SRoute::any('/home/(\w+)', app\controllers\Home::class);

// can match '/home', '/home/test'
SRoute::any('/home(/\w+)?', app\controllers\Home::class);
```

### 使用方法执行器

配置 actionExecutor 为你需要的方法名，例如配置为 `'actionExecutor' => 'run'`，那所有的方法请求都会提交各此方法。
会将真实的action名作为参数传入`run()`,需要你在此方法中调度来执行真正的请求方法。

```php
// access '/user', will call app\controllers\User::run('')
// access '/user/profile', will call app\controllers\User::run('profile')
SRoute::get('/user', 'app\controllers\User');
SRoute::get('/user/profile', 'app\controllers\User');

// if config 'actionExecutor' => 'run' and 'dynamicAction' => true,
// access '/user', will call app\controllers\User::run('')
// access '/user/profile', will call app\controllers\User::run('profile')
SRoute::get('/user(/\w+)?', 'app\controllers\User');
```

## 自动匹配路由到控制器

支持自动匹配路由到控制器就像 yii 一样, 需配置 `autoRoute`. 

```php 
    'autoRoute' => [
        'enable' => 1, // 启用
        'controllerNamespace' => 'examples\\controllers', // 控制器类所在命名空间
        'controllerSuffix' => 'Controller', // 控制器类后缀
    ],
```

## 匹配所有

配置 'matchAll' 可用于拦截所有请求。 （例如网站维护时）

可允许配置 'matchAll' 的值为 

- 路由path

```php
    'matchAll' => '/about', // a route path
```

将会直接执行此路由。

- 回调

```php 
    'matchAll' => function () {
        echo 'System Maintaining ... ...';
    },
```

将会直接执行此回调


## 设置事件处理(if you need)

```php
SRoute::any('/404', function() {
    echo "Sorry,This page {$_GET['path']} not found.";
});
```

```php
// 成功匹配路由
SRoute::on(SRoute::FOUND, function ($uri, $cb) use ($app) {
    $app->logger->debug("Matched uri path: $uri, setting callback is: " . is_string($cb) ? $cb : get_class($cb));
});

// 当匹配失败, 重定向到 '/404'
SRoute::on('notFound', '/404');

// 或者, 当匹配失败, 输出消息...
SRoute::on('notFound', function ($uri) {
    echo "the page $uri not found!";
});
```

## 设置配置(if you need)

```php
// set config
SRoute::config([
    'stopOnMatch' => true,
    'ignoreLastSep' => true,
    'dynamicAction' => true,
    
//    'matchAll' => '/', // a route path
//    'matchAll' => function () {
//        echo 'System Maintaining ... ...';
//    },
    
    // enable autoRoute, work like yii framework
    // you can access '/demo' '/admin/user/info', Don't need to configure any route
    'autoRoute' => [
        'enable' => 1,
        'controllerNamespace' => 'examples\\controllers',
        'controllerSuffix' => 'Controller',
    ],
]);
```

- 默认配置如下

```php
// there are default config.
[
    // stop on matched. only match one
    'stopOnMatch' => true,
    // Filter the `/favicon.ico` request.
    'filterFavicon' => false,
    // ignore last '/' char. If is True, will clear last '/', so '/home' equals to '/home/'
    'ignoreLastSep' => false,

    // match all request.
    // 1. If is a valid URI path, will match all request uri to the path.
    // 2. If is a callable, will match all request then call it
    'matchAll' => '', // eg: '/site/maintenance' or `function () { echo 'System Maintaining ... ...'; }`

    // auto route match @like yii framework
    'autoRoute' => [
        // If is True, will auto find the handler controller file.
        'enable' => false,
        // The default controllers namespace, is valid when `'enable' = true`
        'controllerNamespace' => '', // eg: 'app\\controllers'
        // controller suffix, is valid when `'enable' = true`
        'controllerSuffix' => '',    // eg: 'Controller'
    ],

    // default action method name
    'defaultAction' => 'index',

    // enable dynamic action.
    // e.g
    // if set True;
    //  SRoute::any('/demo/(\w+)', app\controllers\Demo::class);
    //  you access '/demo/test' will call 'app\controllers\Demo::test()'
    'dynamicAction' => false,

    // action executor. will auto call controller's executor method to run all action.
    // e.g
    //  `run($action)`
    //  SRoute::any('/demo/(:act)', app\controllers\Demo::class);
    //  you access `/demo/test` will call `app\controllers\Demo::run('test')`
    'actionExecutor' => '', // 'run'
]
```

> NOTICE: 必须在调用 `SRoute::dispatch()` 之前使用 `SRoute::config()` 来进行一些配置

## 开始路由分发

```php
SRoute::dispatch();
```

## examples

please the `examples` folder's codes.

你可以通过 `bash ./php_server` 来运行一个测试服务器, 现在你可以访问 http://127.0.0.1:5670

## License 

MIT