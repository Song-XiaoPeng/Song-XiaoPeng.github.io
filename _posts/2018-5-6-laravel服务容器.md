服务提供者注册顺序
Laravel\Passport\PassportServiceProvider

App\Providers\AuthServiceProvider

其中AuthServiceProvider中使用到了Passport，因此需要PassportServiceProvider先注册

那么它们又都是在什么时机注册的呢？

1. 小思考
观察AuthServiceProvider类，发现其中使用到了Passport，那么是不是就需要PassportServiceProvider先注册呢？

2. 进一步观察
进一步的观察，发现，PassportServiceProvider和AuthServiceProvider都存在于`config/app.php`中的`providers数组`中,但是PassportServiceProvider顺序靠前

3. 验证猜想
那么我们就来做一个实验，将PassportServiceProvider放到AuthServiceProvider的后面，再运行框架，看看会发生什么？

发现仍然是Laravel\Passport\PassportServiceProvider先注册的，刚才的猜想是错误的，那么laravel的服务提供者的注册到底是什么机制呢？有待我们进一步的探讨。



1. Application
function boot()
