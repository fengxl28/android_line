# Android中的插件化

#### 区分插件化和热修复：
技术|目的|难易度
---|:--:|---:
插件化|实现功能模块独立出来加载|相对简单
热修复|修复bug|实现复杂


#### 含义：
不必再像原来一样把所有的内容都放在一个apk中，分为宿主apk、插件apk。

- 宿主： 就是当前运行的APP
- 插件： 相对于插件化技术来说，就是要加载运行的apk类文件

![hahah](./image/android/插件化1.png)

#### 优点：
1. 热部署：需要什么功能加载什么功能
2. 模块化
3. 不发版本上新功能
4. 宿主小
5. 独立开发独立模块
6. bug修复（不建议）～～～～


#### 方案：
- 业务上：应用在运行的时候动态的加载一些可替换的执行文件实现一些特定的功能，
- 代码上：通过Android的ClassLoader类加载器加载外部dex文件（即插件）来实现一些特定的功能。
- 难点：1、加载Android四大组件，2、加载资源文件


#### 核心内容
- 类加载器：
虚拟机通过ClassLoader加载Jar文件并执行里面的代码，Android应用类似于Java程序，虚拟机换成了Dalvik/ART，而Jar换成了Dex，Android中大体两类

1. DexClassLoader可以加载jar/apk/dex，可以从SD卡中加载未安装的apk；
2. PathClassLoader只能加载系统中已经安装过的apk；

它们都继承与BaseDexClassLoader，里面核心加载类的过程代码

```
//加载类到内存流程上，先去当前的加载器找是否加载过，再去父加载器查找，最后再到子加载器找。

protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {
        Class<?> clazz = findLoadedClass(className);

        if (clazz == null) {
            ClassNotFoundException suppressed = null;
            try {
                clazz = parent.loadClass(className, false);
            } catch (ClassNotFoundException e) {
                suppressed = e;
            }

            if (clazz == null) {
                try {
                    clazz = findClass(className);
                } catch (ClassNotFoundException e) {
                    e.addSuppressed(suppressed);
                    throw e;
                }
            }
        }

        return clazz;
    }
```

- 特点：如果一个类被位于树根的ClassLoader加载过，那么在以后整个系统的生命周期内，这个类永远不会被重新加载。（双亲代理）
作用：1、共享，不用多次加载。 2、隔离，防止自定义的类覆盖系统的类。

- 动态修复Bug的原理：加载一个新版本的dex文件，使用里面的新类替换原有的旧类，从而修复原有类的BUG，那么你必须保证在加载新类的时候，旧类还没有被加载。

- 加载成功后调用：
   1、反射， 2、接口（维护一套公共接口，类型转换）

- 难点

1、Android中许多组件类（如Activity、Service等）是需要在Manifest文件里面注册后才能工作的（系统会检查该组件有没有注册），所以即使动态加载了一个新的组件类进来，没有注册的话还是无法工作；

2、Res资源是Android开发中经常用到的，而Android是把这些资源用对应的R.id注册好，运行时通过这些ID从Resource实例中获取对应的资源。如果是运行时动态加载进来的新类，那类里面用到R.id的地方将会抛出找不到资源或者用错资源的异常，因为新类的资源ID根本和现有的Resource实例中保存的资源ID对不上；

#### 扩展
加载Android组件
绕行方案：Fragment代替Activity，这样可以最大限度得避开“无法注册新组件的限制”。

代理Activity ：https://segmentfault.com/a/1190000004062972
动态创建Activity ： https://segmentfault.com/a/1190000004077469

ClassLoader的动态加载技术（除了加载SO库外），使用ClassLoader的一个特点就是，如果程序不重新启动，加载过一次的类就无法重新加载。因此，如果使用ClassLoader来动态升级APP或者动态修复BUG，都需要重新启动APP才能生效。

除了使用ClassLoader外，还可以使用jni hook的方式修改程序的执行代码。前者是在虚拟机上操作的，而后者做的已经是Native层级的工作了，直接修改应用运行时的内存地址，所以使用jni hook的方式时，不用重新应用就能生效。

加载so库

#### 应用：
项目里面的实际应用


#### 开源框架学习：
Dynamic-load-apk
DroidPlugin


#### 积累点

1、加载的可执行文件一定要存放在内部存储

2、一个Android程序和标准的Java程序最大的区别就在于他们的上下文环境（Context）不同。Android中，这个环境可以给程序提供组件需要用到的功能，也可以提供一些主题、Res等资源


#### 参考：

https://www.jianshu.com/p/704cac3eb13d


http://lruheng.com/2017/07/01/Android%E6%8F%92%E4%BB%B6%E5%8C%96%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97/

https://tech.meituan.com/android_hydra.html

https://segmentfault.com/a/1190000004062866