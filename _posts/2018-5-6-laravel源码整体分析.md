## laravel
>  Illuminate\Foundation\Application 阐明/基础/应用

### 从入口文件开始
```
<?php

/**
 * Laravel - A PHP Framework For Web Artisans
 *  一个为web艺术家创造的php框架
 * @package  Laravel
 * @author   Taylor Otwell <taylor@laravel.com>
 */

define('LARAVEL_START', microtime(true));

/*
|--------------------------------------------------------------------------
| Register The Auto Loader
|--------------------------------------------------------------------------
|
| Composer provides a convenient, automatically generated class loader for
    Composer为我们的应用提供一个方便的，自动生成的类加载器。
| our application. We just need to utilize it! We'll simply require it
    我们仅仅需要好好利用它！
| into the script here so that we don't have to worry about manual
    我们将简单的依赖它到我们的这个脚本里
    因此我们不用担心稍后手动的加载任何的类
| loading any of our classes later on. It feels great to relax.
    这种放松的感觉是棒的.
|
*/

require __DIR__.'/../vendor/autoload.php';

/*
|--------------------------------------------------------------------------
| Turn On The Lights
|--------------------------------------------------------------------------
|
| We need to illuminate PHP development, so let us turn on the lights.
    我们需要阐明PHP开发，所以让我们打开电灯。
| This bootstraps the framework and gets it ready for use, then it
    这个引导框架并让它准备好使用，然后
| will load up this application so that we can run it and send
    会加载这个应用程序，这样我们就可以运行它并发送
| the responses back to the browser and delight our users.
    响应到浏览器，让我们的用户高兴。
|
*/

$app = require_once __DIR__.'/../bootstrap/app.php';

/*
|--------------------------------------------------------------------------
| Run The Application
|--------------------------------------------------------------------------
|
| Once we have the application, we can handle the incoming request
| through the kernel, and send the associated response back to
| the client's browser allowing them to enjoy the creative
| and wonderful application we have prepared for them.
|
*/

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class); //instance of App\Http\Kernel

$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

$response->send();

$kernel->terminate($request, $response);

```

1. app.php
```
/*
|--------------------------------------------------------------------------
| Create The Application 创建一个应用
|--------------------------------------------------------------------------
|
| The first thing we will do is create a new Laravel application instance
    首先要做的事情是创建一个laravel应用实例
| which serves as the "glue" for all the components of Laravel, and is
    它将为所有laravel组件提供类似于粘合剂的东西
| the IoC container for the system binding all of the various parts.
    它是一个IoC容器，为系统绑定所有的不同的部件
|
*/
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../') //这个路径为项目根目录
);


/*
|--------------------------------------------------------------------------
| Bind Important Interfaces 绑定重要的接口
|--------------------------------------------------------------------------
|
| Next, we need to bind some important interfaces into the container so
    下一步，我们需要为这个容器绑定一些重要的接口
| we will be able to resolve them when needed. The kernels serve the
    我们将能够在需要的时候解决他们。
| incoming requests to this application from both the web and CLI.
    这个核心为 通过web或者cli方式的到达这个应用的请求 提供服务。
|
*/
阐明、说明\合约、契约\Http\内核、核心
$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);
//该方法是Application类继承自Container类的

```
2. Illuminate\Foundation\Application 继承 Illuminate\Container\Container
```
/**
     * Register a shared binding in the container.
     *  在容器中注册一个共享的绑定实例（单例：共享一个实例）
     * @param  string  $abstract                    //抽象的
     * @param  \Closure|string|null  $concrete      //具体的
     * @return void                                 //返回空值
     */
    public function singleton($abstract, $concrete = null)
    {
        $this->bind($abstract, $concrete, true); //该方法也在Container类中，Application直接继承它
    }

    /**
     * Register a binding with the container.
     *  对容器注册一个绑定。
     * @param  string  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        // If no concrete type was given, we will simply set the concrete type to the
            如果没有给出具体的类型，我们只需要将具体类型设置为抽象类型
        // abstract type. After that, the concrete type to be registered as shared
            在那之后，将被注册为共享的具体类型
        // without being forced to state their classes in both of the parameters.
            而不必被迫在两个参数中声明它们的类。
        $this->dropStaleInstances($abstract);

        if (is_null($concrete)) { 
            $concrete = $abstract;
        }

        // If the factory is not a Closure, it means it is just a class name which is
            如果工厂不是一个闭包，它意味着它只是一个类名作为抽象类型被绑定到这个容器中
        // bound into this container to the abstract type and we will just wrap it
            我们将把它封装在它自己的闭包中，以便在扩展时给我们更多的方便。
        // up inside its own Closure to give us more convenience when extending.
        if (! $concrete instanceof Closure) {
            $concrete = $this->getClosure($abstract, $concrete);
        }

        $this->bindings[$abstract] = compact('concrete', 'shared');

        // If the abstract type was already resolved in this container we'll fire the
            如果抽象类型已经在容器里被解决了  我们将解除rebound监听
        // rebound listener so that any objects which have already gotten resolved
            任何已经被解决的对象都可以通过侦听器回调来更新对象的副本。
        // can have their copy of the object updated via the listener callbacks.
        if ($this->resolved($abstract)) {
            $this->rebound($abstract);
        }
    }

       /**
     * Drop all of the stale instances and aliases.
     *  删除所有过期的实例和别名。
     * @param  string  $abstract
     * @return void
     */
    protected function dropStaleInstances($abstract)
    {
        unset($this->instances[$abstract], $this->aliases[$abstract]);
    }

    /**
     * Get the Closure to be used when building a type.
        获得当构建类型时被使用的闭包
     *
     * @param  string  $abstract
     * @param  string  $concrete
     * @return \Closure
     */
    protected function getClosure($abstract, $concrete)
    {
        return function ($container, $parameters = []) use ($abstract, $concrete) {
            if ($abstract == $concrete) {
                return $container->build($concrete);
            }

            return $container->make($concrete, $parameters);
        };
    }

    /**
     * Determine if the given abstract type has been resolved.
     *
     * @param  string  $abstract
     * @return bool
     */
    public function resolved($abstract)
    {
        if ($this->isAlias($abstract)) {
            $abstract = $this->getAlias($abstract);
        }

        return isset($this->resolved[$abstract]) ||
               isset($this->instances[$abstract]);
    }
```

3. 























































1. .env [关于Lumen / Laravel .env 文件中的环境变量加载原理](http://www.bcty365.com/content-153-5874-1.html)
```
composer require vlucas/phpdotenv
getenv()
putenv()
Str::startsWith()
value() return $value instanceof Closure ? $value() : $value;
```

2. config()
```
app('config') return Container::getInstance()->make($abstract, $parameters);
```

3. laravel如何工作？ 以及工作原理
```
1. Laravel 的第一个动作就是创建一个自身应用实例 / 服务容器。
    $app = require_once __DIR__.'/../bootstrap/app.php';
    传入的请求会被发送给 HTTP 内核或者 console 内核，这根据进入应用的请求的类型而定
    HTTP内核：
    继承链：    （App\Http\Kernel::class）-（Illuminate\Foundation\Http\Kernel）-（Illuminate\Contracts\Http\Kernel）
    （定义中间件）-定义了一个 bootstrappers 数组）-（）
    HTTP 内核继承自 Illuminate\Foundation\Http\Kernel 类
    它定义了一个 bootstrappers 数组，数组中的类在请求真正执行前进行前置执行。
    这些引导程序配置了错误处理，日志记录，检测应用程序环境，以及其他在请求被处理前需要完成的工作。
    定义了 HTTP 中间件 列表
    HTTP 内核的标志性handle 方法：接收一个 Request 并返回一个 Response

    在内核引导启动的过程中最重要的动作之一就是载入 服务提供者 到你的应用
  
```

- bootstrappers 数组

```
protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class, //加载配置
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class, //错误处理
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];
```

- 服务容器
> Laravel 服务容器是管理类依赖和运行依赖注入的有力工具。
> 类的依赖通过构造器或在某些情况下通过「setter」方法进行「注入」。 
> 该容器提供了整个框架中需要的一系列服务

- 服务提供者注册至服务容器