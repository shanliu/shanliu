# rxjs

> 概念

```
Observable (可观察对象): 表示一个概念，这个概念是一个可调用的未来值或事件的集合。(操作流) 
Observer (观察者): 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。(在Observable回调函数的参数) 
Subscription (订阅): 表示 Observable 的执行，主要用于取消 Observable 的执行。(回调处理) 
Operators (操作符): 采用函数式编程风格的纯函数 (pure function)，使用像 map、filter、concat、flatMap 等这样的操作符来处理集合。 
Subject (主体): 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。(多个监听者) 
Schedulers (调度器): 用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 setTimeout 或 requestAnimationFrame 或其他。(什么时候执行)
```

> 示例

```
var observable1111 = Rx.Observable.ajax({
  url: 'http://localhost:8080/'
});
var observable2 = Rx.Observable.ajax({
  url: 'http://localhost:8080/'
});
observable1111.merge(observable2)
.subscribe(x => console.log(x));
//Rx.Observable.merge()
```

1. Subject 继承 Observable ,同时又实现 observer
2. 所以可以 subscribe 可以 next 等.
3. 因为实现 observer 所以可以继承到其他 Observable,即可以当做其他Observable 的 subscribe 的回调函数


> 默认单播,实现多播如下

```
    var subject = new Rx.Subject();
    subject.subscribe({
      next: (v) => console.log('observerA: ' + v)
    });
    subject.subscribe({
      next: (v) => console.log('observerB: ' + v)
    });
    var observable = Rx.Observable.from([1, 2, 3]);
    observable.subscribe(subject);//一般传函数,这里传Subject
    //如果要传递,需要传两个对象
var source = Rx.Observable.from([1, 2, 3]);
var subject = new Rx.Subject();
var multicasted = source.multicast(subject);
//var multicasted = source.multicast(subject).refCount(); //有订阅自动执行,无订阅自动关闭,不需要手动调用 connect
multicasted.subscribe({// 此函数内部调用 subject.subscribe({...})
  next: (v) => console.log('observerA: ' + v)
});
multicasted.subscribe({// 此函数内部调用 subject.subscribe({...})
  next: (v) => console.log('observerB: ' + v)
});
multicasted.connect();//此函数内部调用 source.subscribe(subject)
//multicasted等于实现连接[ConnectableObservable] Observable+Subject
```

> Subject 变体

1. ReplaySubject 带历史的Subject
2. AsyncSubject 执行完成时(执行 complete())发送给观察者
3. BehaviorSubject 保存了发送给消费者的最新值 类似ReplaySubject(1)

> Scheduler 调度器[如何发送给订阅者]

> observeOn() 更改调度器

创建数据流：

	单值：of, empty, never
	多值：from
	定时：interval, timer
	从事件创建：fromEvent
	从Promise创建：fromPromise
	自定义创建：create
	
转换操作：

	改变数据形态：map, mapTo, pluck
	过滤一些值：filter, skip, first, last, take
	时间轴上的操作：delay, timeout, throttle, debounce, audit, bufferTime
	累加：reduce, scan
	异常处理：throw, catch, retry, finally
	条件执行：takeUntil, delayWhen, retryWhen, subscribeOn, ObserveOn
	转接：switch
	
数据流进行组合：

	concat，保持原来的序列顺序连接两个数据流
	merge，合并序列
	race，预设条件为其中一个数据流完成
	forkJoin，预设条件为所有数据流都完成
	zip，取各来源数据流最后一个值合并为对象
	combineLatest，取各来源数据流最后一个值合并为数组