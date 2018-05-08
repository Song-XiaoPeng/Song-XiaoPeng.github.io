## 对laravel框架中一些功能实现的研究

### laravel框架自定义异常处理类
1. php对异常的处理
首先，引用手册原话： `自 PHP 7 以来，大多数错误抛出 Error 异常，也能被捕获。 Error 和 Exception 都实现了 Throwable 接口。` 

然后，php提供了一个自定义异常处理类的方法`set_exception_handler`：设置用户自定义的异常处理函数。

> 引用手册上对该类的说明：`设置默认的异常处理程序，用于没有用 try/catch 块来捕获的异常。 在 exception_handler 调用后异常会中止。 `
> callable set_exception_handler ( callable $exception_handler ) 
> 返回之前定义的异常处理程序的名称，或者在错误时返回 NULL。 如果之前没有定义错误处理程序，也会返回 NULL。 


laravel框架就是通过该函数来自定义它的异常处理类的

2. laravel在入口文件中调用了`App\Http\Kernel@handle方法`，

App\Http\Kernel类继承自Illuminate\Foundation\Http\Kernel，然后又继承了Illuminate\Contracts\Http\Kernel

在Illuminate\Foundation\Http\Kernel类中有一个$bootstrappers的数组，`bootstrap`是计算机的一个术语翻译为为:`引导程序`

然后我们看一下该数组的成员
```protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];
```
`\Illuminate\Foundation\Bootstrap\HandleExceptions::class`该类即为laravel框架的异常处理类，即：`如果框架抛出异常，那么PHP就会把异常交该这个类处理。`

3. laravel框架自定义异常处理类的实现 
我们再回到laravel的入口文件中：

```
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
```
首先，通过lravel`服务容器`获得了`App\Http\Kernel`的实例，有关laravel服务容器的概念，可以先看手册学习一下，随后我会再做一篇总结。

紧接着，调用了kernel类的handle方法，该方法在该类的父类中，即Illuminate\Foundation\Http\Kernel类中
```
/**
     * Handle an incoming HTTP request.
        处理一个即将到来的HTTP请求
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();

            $response = $this->sendRequestThroughRouter($request);
        } catch (Exception $e) {
            $this->reportException($e);

            $response = $this->renderException($request, $e);
        } catch (Throwable $e) {
            $this->reportException($e = new FatalThrowableError($e));

            $response = $this->renderException($request, $e);
        }

        $this->app['events']->dispatch(
            new Events\RequestHandled($request, $response)
        );

        return $response;
    }
```    
从注释中，我们可以看到，该方法的作用是：` 处理一个即将到来的HTTP请求`
