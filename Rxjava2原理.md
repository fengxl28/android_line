# Rxjava 原理

- 分析一个简单的订阅流程
- 分析操作符实现
- 分析线程切换

#### 一个简单的订阅流程

       核心思想**观察者模式**，被观察者拥有观察者对象，订阅的时候调用观察者的方法。
   
##### 源码流程

```
 Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("1");
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {

            }

            @Override
            public void onNext(String s) {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {

            }
        });
```

1. 创建被观察者（Observable）参数：ObservableOnSubscribe接口 ，只有一个方法 subscribe()参数ObservableEmitter；
2. 创建观察者 (Observer)
3. 订阅方法subscribe，将Observer对象传给Observable，这样Observable就可以调用Observer的方法与ObservableEmitter关联。

##### 创建Observable流程

```
 Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("1");
            }
        })
       
        
        
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
        ObjectHelper.requireNonNull(source, "source is null");
        return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
    }
       
```
 
其实创建的是Observable的子类ObservableCreate，
 
 
##### 关联Observable和Observer，订阅方法subscribe

```
public final class ObservableCreate<T> extends Observable<T> {
    final ObservableOnSubscribe<T> source;

    public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }

    //真实的订阅方法：相当于将观察者对象交给被观察者对象，这样被观察者对象就可以调用观察者对象的方法
    @Override
    protected void subscribeActual(Observer<? super T> observer) {

        //观察者进行装饰者模式，
        CreateEmitter<T> parent = new CreateEmitter<T>(observer);
        //第一个执行的方法：observer 的 onSubscribe
        observer.onSubscribe(parent);
        try {
            //被观察者调用观察者，将ObservableOnSubscribe（源头）与CreateEmitter（Observer，终点）联系起来
            source.subscribe(parent);
        } catch (Throwable ex) {
            source.subscribe(parent);
        } catch (Throwable ex) {
            Exceptions.throwIfFatal(ex);
            parent.onError(ex);
        }
    }
```

抛去一些设计模式的运用，详细的实现：在订阅的时候调用被观察者的抽象真实订阅方法（subscribeActual），参数传入观察者对象，创建的被观察者对象里面实现了真实订阅方法，并在里面调用了观察者对象的方法。

订阅的时候发生的事情：

 1. 订阅其实就是调用被观察者实现的抽象真实订阅方法.
 2. 将观察者对象传给被观察者.
 3. 调用观察者的方法.

#### 操作符实现

最常用的map操作符 ，根据订阅流、数据流分析。

##### 简单流程
```
Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("1");
            }
        }).map(new Function<String, String>() {
            @Override
            public String apply(String s) throws Exception {
                return s;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                
            }
        });
        
```
分析主要代码

```

 public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
        ObjectHelper.requireNonNull(mapper, "mapper is null");
         //返回Observable的子类ObservableMap，参数上游的Observable和Function
        return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
    }
    
 --------------------------------------------------------------------   
    
    public final class ObservableMap<T, U> extends AbstractObservableWithUpstream<T, U> {
    final Function<? super T, ? extends U> function;

    public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends U> function) {
    //super()将上游的Observable保存起来 ，用于subscribeActual()中用。
        super(source);
        this.function = function;
    }

    @Override
    public void subscribeActual(Observer<? super U> t) {
        //实行中间订阅，用MapObserver订阅上游Observable。
        source.subscribe(new MapObserver<T, U>(t, function));
    }

    static final class MapObserver<T, U> extends BasicFuseableObserver<T, U> {
        final Function<? super T, ? extends U> mapper;

        MapObserver(Observer<? super U> actual, Function<? super T, ? extends U> mapper) {
            super(actual);
            this.mapper = mapper;
        }

        @Override
        public void onNext(T t) {
            if (done) {
                return;
            }

            if (sourceMode != NONE) {
                downstream.onNext(null);
                return;
            }

            U v;

            try {
             //这一步执行变换,将上游传过来的T，利用Function转换成下游需要的U。
                v = ObjectHelper.requireNonNull(mapper.apply(t), "The mapper function returned a null value.");
            } catch (Throwable ex) {
                fail(ex);
                return;
            }
            //变换后传递给下游Observer
            downstream.onNext(v);
        }

    
```
订阅的过程，是从下游到上游依次订阅的，订阅后的数据流是从上游到下游。

俗话讲：现在在起点的Observable和终点Observer的中间切了一刀， 在中间新建了一个Observable（ObservableMap）将起点的Observable当作参数，和新建一个Observer（MapObserver）将终点的Observer当作参数，再实现中间订阅。

数据流： 会在中间的MapObserver，对其变换操作后(实际的function在这一步执行)，再调用内部保存的下游Observer的onNext()发送数据给下游。

#### 线程调度
Rxjava 可以用简洁的代码实现线程调度

##### 线程调度subscribeOn

源码

```
//1、Observable和Observer之间增加了一句线程调度代码
.subscribeOn(Schedulers.io())

--------------------------------------------

public final Observable<T> subscribeOn(Scheduler scheduler) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        //2、还是老套路 ，包装返回子类
        return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
    }
    
--------------------------------------------  

public final class ObservableSubscribeOn<T> extends AbstractObservableWithUpstream<T, T> {
    final Scheduler scheduler;

    public ObservableSubscribeOn(ObservableSource<T> source, Scheduler scheduler) {
        super(source);
        this.scheduler = scheduler;
    }

    @Override
    public void subscribeActual(final Observer<? super T> observer) {
         //3、创建一个包装Observer
        final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(s);
        //4、手动调用 下游（终点）Observer.onSubscribe()方法,所以onSubscribe()方法执行在 订阅处所在的线程
        s.onSubscribe(parent);
        //5、setDisposable()是为了将子线程的操作加入Disposable管理中
        parent.setDisposable(scheduler.scheduleDirect(new Runnable() {
            @Override
            public void run() {
            //6、此时已经运行在相应的Scheduler 的线程中
                source.subscribe(parent);
            }
        }));
    } 
 
```
基本框架还是跟map套路一致，ObservableSubscribeOn自身同样是个包装类
创建了一个SubscribeOnObserver类，实现了Observer、Disposable接口的包装类,
SubscribeOnObserver核心代码

```
static final class SubscribeOnObserver<T> extends AtomicReference<Disposable> implements Observer<T>, Disposable {
        //真正的下游（终点）观察者
        final Observer<? super T> actual;
        //用于保存上游的Disposable，以便在自身dispose时，连同上游一起dispose
        final AtomicReference<Disposable> s;

        SubscribeOnObserver(Observer<? super T> actual) {
            this.actual = actual;
            this.s = new AtomicReference<Disposable>();
        }

        @Override
        public void onSubscribe(Disposable s) {
            //onSubscribe()方法由上游调用，传入Disposable。在本类中赋值给this.s，加入管理。
            DisposableHelper.setOnce(this.s, s);
        }

        //直接调用下游观察者的对应方法
        @Override
        public void onNext(T t) {
            actual.onNext(t);
        }
        @Override
        public void onError(Throwable t) {
            actual.onError(t);
        }
        @Override
        public void onComplete() {
            actual.onComplete();
        }
        //取消订阅时，连同上游Disposable一起取消
        @Override
        public void dispose() {
            DisposableHelper.dispose(s);
            DisposableHelper.dispose(this);
        }

        @Override
        public boolean isDisposed() {
            return DisposableHelper.isDisposed(get());
        }
        //这个方法在subscribeActual()中被手动调用，为了将Schedulers返回的Worker加入管理
        void setDisposable(Disposable d) {
            DisposableHelper.setOnce(this, d);
        }
    }

```
总结过程：

返回一个ObservableSubscribeOn包装类对象
上一步返回的对象被订阅时，回调该类中的subscribeActual()方法，在其中会立刻将线程切换到对应的Schedulers.xxx()线程。
在切换后的线程中，执行source.subscribe(parent);，对上游(终点)Observable订阅
上游(终点)Observable开始发送数据，上游发送数据仅仅是调用下游观察者对应的onXXX()方法而已，所以此时操作是在切换后的线程中进行。

**subscribeOn()多次调用问题**
总是以第一次为准，或者说离源Observable最近的那次为准，并且对其上面的代码生效（这一点对比的ObserveOn()）。

- 订阅流程从下游往上游传递 
- 在subscribeActual()里开启了Scheduler的工作，source.subscribe(parent);,从这一句开始切换了线程，所以在这之上的代码都是在切换后的线程里的了。 
- 但如果连续切换，最上面的切换最晚执行,此时线程变成了最上面的subscribeOn(xxxx)指定的线程， 
- 而数据push时，是从上游到下游的，所以会在离源头最近的那次subscribeOn(xxxx)的线程里push数据（onXXX()）给下游。

##### 线程调度observeOn

Android里面涉及到线程切换的基本用到了handler 机制， 这里配合java语法，要实现线程切换需要用到队列思想。 再加上Rx中间订阅套路。----》 基本思路。

```
 .observeOn(AndroidSchedulers.mainThread())
 
 --------------------------------------------
 
 
 public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        ObjectHelper.verifyPositive(bufferSize, "bufferSize");
        //返回子类
        return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
    }
    
--------------------------------------------
    
    public final class ObservableObserveOn<T> extends AbstractObservableWithUpstream<T, T> {
    final Scheduler scheduler;
    final boolean delayError;
    final int bufferSize;
    public ObservableObserveOn(ObservableSource<T> source, Scheduler scheduler, boolean delayError, int bufferSize) {
        super(source);
        this.scheduler = scheduler;
        this.delayError = delayError;
        this.bufferSize = bufferSize;
    }

    @Override
    protected void subscribeActual(Observer<? super T> observer) {
        if (scheduler instanceof TrampolineScheduler) {
            source.subscribe(observer);
        } else {
        //创建出一个 现场调度的Worker
            Scheduler.Worker w = scheduler.createWorker();
//2 订阅上游数据源，
            source.subscribe(new ObserveOnObserver<T>(observer, w, delayError, bufferSize));
        }
    }
```

创建一个AndroidSchedulers.mainThread()对应的Worker
用ObserveOnObserver订阅上游数据源。这样当数据从上游push下来，会由ObserveOnObserver对应的onXXX()处理。所以主要看ObserveOnObserver的onNext
核心源码。

```
static final class ObserveOnObserver<T> extends BasicIntQueueDisposable<T>
    implements Observer<T>, Runnable {
    //下游的观察者
        final Observer<? super T> actual;
        //对应Scheduler里的Worker
        final Scheduler.Worker worker;
        //上游被观察者 push 过来的数据都存在这里
        SimpleQueue<T> queue;
        Disposable s;
        //如果onError了，保存对应的异常
        Throwable error;
        //是否完成
        volatile boolean done;
        //是否取消
        volatile boolean cancelled;
        // 代表同步发送 异步发送 
        int sourceMode;

        ObserveOnObserver(Observer<? super T> actual, Scheduler.Worker worker, boolean delayError, int bufferSize) {
            this.downstream = actual;
            this.worker = worker;
            this.delayError = delayError;
            this.bufferSize = bufferSize;
        }

        @Override
        public void onSubscribe(Disposable d) {
            if (DisposableHelper.validate(this.upstream, d)) {
                this.upstream = d;
                
                //创建一个queue 用于保存上游 onNext() push的数据
                queue = new SpscLinkedArrayQueue<T>(bufferSize);
                //回调下游观察者onSubscribe方法
                actual.onSubscribe(this);
          }
        }

        @Override
        public void onNext(T t) {
            //1 执行过error / complete 会是true
            if (done) {
                return;
            }
            //2 如果数据源类型不是异步的， 默认不是
            if (sourceMode != QueueDisposable.ASYNC) {
                //3 将上游push过来的数据 加入 queue里
                queue.offer(t);
            }
            //4 开始进入对应Workder线程，在线程里 将queue里的t 取出 发送给下游Observer
            schedule();
        }

       void schedule() {
            if (getAndIncrement() == 0) {
                //5 该方法需要传入一个线程， 注意看本类实现了Runnable的接口，所以查看对应的run()方法
                worker.schedule(this);
            }
        }
        //从这里开始，这个方法已经是在Workder对应的线程里执行的了
        @Override
        public void run() {
            //默认是false
            if (outputFused) {
                drainFused();
            } else {
                //取出queue里的数据 发送
                drainNormal();
            }
        }
        
        void drainNormal() {
            int missed = 1;

            final SimpleQueue<T> q = queue;
            final Observer<? super T> a = actual;

            for (;;) {
                // 1 如果已经 终止 或者queue空，则跳出函数，
                if (checkTerminated(done, q.isEmpty(), a)) {
                    return;
                }

                for (;;) {
                    boolean d = done;
                    T v;

                    try {
                        //2 从queue里取出一个值
                        v = q.poll();
                    } catch (Throwable ex) {
                        //3 异常处理 并跳出函数
                        Exceptions.throwIfFatal(ex);
                        s.dispose();
                        q.clear();
                        a.onError(ex);
                        return;
                    }
                    boolean empty = v == null;
                    //4 再次检查 是否 终止  如果满足条件 跳出函数
                    if (checkTerminated(d, empty, a)) {
                        return;
                    }
                    //5 上游还没结束数据发送，但是这边处理的队列已经是空的，不会push给下游 Observer
                    if (empty) {
                        //仅仅是结束这次循环，不发送这个数据而已，并不会跳出函数
                        break;
                    }
                    //6 发送给下游了
                    a.onNext(v);
                }          
          
            }
        }
}
```
1. ObserveOnObserver实现了Observer和Runnable接口。
2. 在onNext()里，先不切换线程，将数据加入队列queue。然后开始切换线程，在另一线程中，从queue里取出数据，push给下游Observer
3. 所以observeOn()影响的是其下游的代码，且多次调用仍然生效。
4. 因为其切换线程代码是在Observer里onXXX()做的，这是一个主动的push行为（影响下游）。

**observeOn()多次调用问题**
关于多次调用生效问题。对比subscribeOn()切换线程是在subscribeActual()里做的，只是主动切换了上游的订阅线程，从而影响其发射数据时所在的线程。而直到真正发射数据之前，任何改变线程的行为，都会生效（影响发射数据的线程）。所以subscribeOn()只生效一次。observeOn()是一个主动的行为，并且切换线程后会立刻发送数据，所以会生效多次.

https://github.com/ReactiveX/RxJava