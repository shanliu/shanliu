# larvale

```
注册路径
添加 APP对象到DI 包详细对象(PackageManifest)到DI
注册服务提供:事件 日志 路由对象到DI
设置常用别名
添加 HTTP 控制台 异常处理 到DI
创建HTTP对象运行:(中途有事件派发,Application属性方式注册回调函数调用省略)
    使用 APP对象[bootstrapWith]运行 bootstrappers
        LoadEnvironmentVariables 基本服务器环境参数
        LoadConfiguration 把环境参数转为 config 对象注册DI
        HandleExceptions 未处理异常处理注册
        RegisterFacades 快捷类调用注册 为Application的注册对象
        RegisterProviders 进行Application[registerConfiguredProviders]配置的服务器提供者注册providers
        BootProviders 运行已注册的服务提供者的boot方法
    执行运行回调 middleware
        $next 运行前调用
            注册request到DI
            派发路由
            根据当前路由查找路由组的 middlewareGroups 和\\\
            路由中 $routeMiddleware(对回调命名) 和 如果存在控制器[middleware()]中getMiddleware() 注册的回调或路由中调用middleware()
            并按 middlewarePriority 排序后进行,并在$next 运行前调用
            运行路由注册回调函数 并将返回包裹成 Response 对象
            完成$next 运行后回调运行
        完成$next 运行后回调运行
    发送Response到客户端
    执行上述运行回调中的 terminate(如果存在) 和 Application->terminatingCallbacks
```

执行回调: handle($request, Closure $next, ...$guards); $request 请求对象 $next($request); 运行后结果对象 $guards 注册时附带数据 如\[throttle:60,1]

快捷调用接口: extends Facade getFacadeAccessor(); 返回为 对应注册的别名

服务提供者: provides();//延时注册:依赖defer when();//某事件注册 ServiceProvider register() bindings=\[]==singletons=\[] =\['key',function(){}]; boot \[已运行后注册]

依赖管理器:DI\[Application] 设置 singleton(); instance(); bindIf(); bind(); 获取 makeWith(); factory(); make(); get();

路由注册 name 在 route() 函数中使用 middleware() 参数可以是类名 回调函数 或在 Kernel->$routeMiddleware 的key get()post()...等 为路径和回调处理 可以是控制器 Route::name("index") ->middleware(\App\Http\Middleware\RedirectIfAuthenticated::class) ->get('/',"MyController@index");
