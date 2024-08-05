# unity 高级语法

## delegate（委托）

```
//声明一个委托 XXXX
public delegate void XXXX();
```

```
//定义 XXXX 的一个变量
public XXXX delegateObj1;

//定义一个真实的函数
public void func1(){}

//将真实函数赋值给变量
delegateObj1 = func1;
```

```
//调用委托对象等同于调用它代表的函数
delegateObj1();//等价于func1();
```

## event

```
//声明一个：<事件委托>的变量
public event ImDelegate  AAA;
```

```
//给这个变量添加N个函数
AAA += func1; //添加
AAA += func2; //添加
```

```
执行这个成员变量
AAA();//此时：函数 func1 和 func2 都会调用
```

> 挺神奇的一个东西...... ，一个接口的实现的N个函数，可以叠加，统一触发

## Action/Func

Action:用于无返还值的委托类型

Func\<T\>:用于有返回值的委托类型，最后一个类型参数 T 代表返还值类型。

不用再单独去申请一个委托了，直接在定义成员变量时，可以直接使用。有点泛型函数的意思。

```
//int 返回值，string double 为参数
public event Func<string,double,int> myObj;
```

感觉是挺简单，一行代码，即有委托的定义，也有变量的定义，同时把 event也加进去了

## Lambda

委托的变量名 = \(参数类型 参数变量\) =\> {具体函数内容}

> 感觉它这个有点像匿名函数

## sleep

Thread.Sleep\(1000\);//阻塞线程

Task.Delay\(1000\);//异步非阻塞

## 协程

使用协程的前置条件：

1. 调用者必须继承：monobehaviour
2. 调用类必须使用函数 startcoroutine 触发协程函数
3. 协程函数必须返回 IEnumerator 类型
4. 最终靠 yield 来实现异步
5. 不能在代码中直接使用 try\-catch
6. 不能有返回值

从中看一下缺点：

1. 不能 try catch
2. 必须继承 monobehaviour
3. 没有返回值，就得无限回调了
4. 并不是真正的异步，而是伪异步

大体执行过程：

monobehaviour:startcoroutine\-\>IEnumerator\-\>自己的函数\-\>yeild控制流程\<\-monobehaviour调度

#### 核心1：IEnumerable\(MoveNext\)

IEnumerable:迭代器，foreach 可以直接遍历它

IEnumerator:迭代器,或者叫对象容器也行。它才是核心。上面更倾向是一个接口，而这个更倾向于管理元素。它把一个函数拆分成\(yeild\)若干个小函数，放到若干的元素里，遍历执行。\(MoveNext\)

```
public interface IEnumerator
{
     bool MoveNext();//下一段代码(函数)的入口
     void Reset();
     Object Current{get;}//上一次执行返回的结果
}
```

IEnumerator是个迭代器，不停的调用 MoveNext 函数，再把用户自己的函数进行分段拆分，如 MoveNext 函数内，用一个select ，把代码拆成若干段，执行每段后，返回执行的位置。下次再执行 MoveNext 函数，就直接跳到下一个段的函数。

所以，严格来说它并不是纯异步，依然还是在主线程上执行\(每次update之后lateupdate之前\)。

### 核心2 yield

如何自己去实现 IEnumerator 类，重复代码太多，且有危险。如果某个函数，想在某个点做拆分，直接在前面加个 yield 就行了。

> yield 是C\# 的。 monobehaviour 这个鸟东西是 UNITY的

|代码                                   |描述                                                                                                                     ||
|---------------------------------------|-------------------------------------------------------------------------------------------------------------------------||
|yield return null                      |下一帧再执行后续代码                                                                                                     ||
|yield break                            |直接结束该协程的后续操作                                                                                                 ||
|yield return asyncOperation            |等异步操作结束后再执行后续代码                                                                                           ||
|yield return StartCoroution            |等待某个协程执行完毕后再执行后续代码                                                                                     ||
|yield return WWW\(\)                   |等待WWW操作完成后再执行后续代码                                                                                          ||
|yield return new WaitForEndOfFrame\(\) |等待帧结束,等待直到所有的摄像机和GUI被渲染完成后，在该帧显示在屏幕之前执行                                               ||
|yield return new WaitForSeconds\(0.3f\)|等待0.3秒，一段指定的时间延迟之后继续执行，在所有的Update函数完成调用的那一帧之后（这里的时间会受到Time.timeScale的影响）||
|                                       |                                                                                                                         ||

> yield return XXXX

通过 上面的表达式，也就是关键字：yield return any\-Object，实现了异步

any\-Object：可以是任意值，如标量的整形、字符串等。也可以是个对象，还可以是个执行函数

这里重点说一个：

> yield return webRequest.SendWebRequest\(\);

webRequest.SendWebRequest\(\)函数中有 isDone

## async/await

前置条件：

1. 异步的函数，前面要加上关键字：async
2. 函数内有异步的代码时，在前面加 await
3. await 后面的代码，返回值必须得是 Task.GetAwaiter

缺点：await 后面的代码必须得实现 Task.GetAwaiter

缺点：可以有返回值。可以使用 try catch

> 如果函数名中包含 async，但函数体内没有 await 就跟正常函数一样。换个角度理解：async和await 必须成对出现才是异步

它的内部也是个 MoveNext 函数，与协程比，它没有 迭代器的概念，是加入了：状态机的概念。但总得看大同小异，都差不多。

## async/await 与 协程对比

相同：

1. 都是单线程，伪异步
2. 都比较简单，几个关键字，不像线程 得做变量安全、异常处理等
3. 如果有非常耗时的代码，依然会阻塞主线程

区别：

1. 协程的限制更多，且是 UNITY 自己实现的
2. async/await 是在C\#层面实现的，且有返回值有try\-catch

最好的使用建议：在 await yield 后面最后是能接异步函数，保证不会阻塞。

> 异步函数中需要有 isDone 和 GetAwaiter

另外，感觉协程这东西有点鸡肋，或者说跟我理解的真正的协程区别有点大。并不真正的异步，且限制比较多。async\+await 可能略好点

不推荐使用的场景：较长占用执行时间的代码

总结：这两个东西更适合一些小场景，不占用执行时间的场景。大点复杂的应用，可能多线程更好一些。

## TASK

它是C\#实现的，有点像任务调度系统。底层并不是：每个执行体就开一个线程。它是有个线程池。把需要执行的函数代码调度性的分配到池子里的线程来执行。但至少是真的异步执行。

感觉C\#这个功能挺方便的：

1. 减少了开发者自己处理线程。真要是开发自己管理线程很麻烦的。
2. 语言自带了个任务调度功能，不用中间件和自己实现了。确实方便
3. 也确实是真的实现了异步

几个经常用的函数：

run：创建一个任务并立即执行

start：将已创建的任务，运行起来

wait：等待某个任务执行完毕后，再执行后面的代码

waitAll：等待某些任务执行完毕后，再执行后面的代码

ContinueWith：某个任务执行完后，可以再注册个回调函数，继续执行

Result：

异常处理：一个线程无法捕获另一个线程的异常（调用任务函数的外部加try\-catch不是起使用的），另外，执行函数中，使用TRY\-CATCH也是没用的，但可以直接抛出异常，该异常会在 wait/waitALL 返回的时候，判断状态获取到。也可以给wait/waitAll 加 try\-catch捕获。也可以用 ContinueWith \+ Task.Exception 获取

取消任务

var tokenSource2 = new CancellationTokenSource\(\);

CancellationToken ct = tokenSource2.Token;

tokenSource2.Cancel\(\);

Created

WaitingToRun

RanToCompletion
