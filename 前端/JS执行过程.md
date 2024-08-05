
# 线程模型

主线程+事件线程

主体模型：event loop 模式，即一个主体线程完成同步的JS处理，如果有异步的扔给 事件线程处理（会通知'事件线程'接收），'事件线程'接收到请求后，开始执行，等执行结束后将结束，输出到队列~主线程会轮询这个队列读取'事件线程'给吐出的结果。主线程拿到执行后的结果，执行回调函数代码。

如：处理AJAX请求的线程、处理DOM事件的线程、定时器线程、读写文件的线程等。

所以，实际上JS还是单线程，一旦遇到大量任务或者遇到一个耗时的任务，网页就会出现"假死"，因为JavaScript停不下来，也就无法响应用户的行为。

# 函数支行
当新定义一个函数后，会在该函数中增加一个成员变量：scope_chain(作用域-链)，指向一个列表：windows、this、document等

当运行这个函数运行的时候，建立一个内部对象（我个人感觉，更像是new class创建实例），称：运行期上下文，且唯一，那么把原函数中的成员变量的s_c内容复制到新的实例中，叫激活对象

所以，就有两种变量，一个是局部的（激活对象），另外一个是原全局变量，实际的执行过程中，也是先在局部的变量中进行搜索，最后再搜索全局，这样，变量最好都要定制成局部

优点过程：可以把全局变量赋值给局部变量

执行过程分为3步：语法分析、预编译、执行

浏览器：先以顺序方式，读出JS代码，<script>标签块的、外部包含的。

# 语法分析

把浏览器加载的代码，进行语法检查，按照<script>块，依次检查。如果发现错误，则抛出异常。结果该块的代码。继续扫描下一块<script>JS代码 。

预编译：如下。

执行：

任何代码执行：都会经历，预编译和执行。而代码加载和语法分析则是一次性的。

JS运行环境

全局环境：，即进入 全局环境

函数环境：函数调用执行时，进入该函数环境，不同的函数则函数环境不同

eval：不建议使用，会有安全，性能等问题

当执行一个函数时，先把该函数的 上下文 信息， 压入 函数栈中，在这个 上下文 中执行 函数代码，执行完毕后，出线，栈底永远是全局上下文 。

全局栈上下亠，Global Execution Context：则在浏览器或者该标签页关闭时出栈。

创建上下文过程及包含信息

变量对象VO：Variable Object ，参数变量、成员变量、内部函数引用等。

作用域链：scopeChai，

this

以上均是在预编译的时候创建，实际上就是一些环境变量及引用等，函数在执行的时候可以用到。

VO只是创建对象，并没有给参数赋值，只有当执行时，VO切成Active Object状态，参数才有值。

DEMO：

var num = 30;

function test() {

var a = 10;

function innerTest() {

var b = 20;

return a + b

}

innerTest()

}

test();

实际执行：test预编译，生成函数栈，然后调出函数线信息，开始执行。执行过程中发现又调用了另外一个函数，于是，innetTest进入预编译，生成新函数线，然后执行。

顺序执行的，作用域链：AO(global )、AO(test)、VO(innerTest)

innerTest 的上下文信息，包含如下：

innerTestEC = {

//变量对象

VO: {b: undefined},

//作用域链

scopeChain: [VO(innerTest), AO(test), AO(global)],

//this指向

this: window

}

作用域链的第一项永远是当前作用域，最后一项永远是全局作用域，这样的话，如下：

作用域链：保证了变量访问顺序，也就ed/freFf是先访问本域的成员变量，最后访问全局变量。实际上也就是闭包了，外层定义成员变量，内部访问。

具体执行过程

JS虽然是单线程执行，但还有其它的线程辅助处理。

JS引擎线程： 也称为JS内核，负责解析执行Javascript脚本程序的主线程

事件触发线程： 归属于浏览器内核进程，不受JS引擎线程控制。主要用于控制事件（例如鼠标，键盘

等事件），当该事件被触发时候，事件触发线程就会把该事件的处理函数推进事件队

列，等待JS引擎线程执行

定时器触发线程：主要控制计时器setInterval和延时器setTimeout，用于定时器的计时，计时完毕，

满足定时器的触发条件，则将定时器的处理函数推进事件队列中，等待JS引擎线程

执行。注：W3C在HTML标准中规定setTimeout低于4ms的时间间隔算为4ms。

HTTP异步请求线程：通过XMLHttpRequest连接后，通过浏览器新开的一个线程，监控readyState状

态变更时，如果设置了该状态的回调函数，则将该状态的处理函数推进事件队

列中，等待JS引擎线程执行。注：浏览器对通一域名请求的并发连接数是有限

制的，Chrome和Firefox限制数为6个，ie8则为10个。

DEMO：

console.log('script start');

setTimeout(function() {

console.log('setTimeout');

}, 0);

Promise.resolve().then(function() {

console.log('promise1');

}).then(function() {

console.log('promise2');

});

console.log('script end');

宏任务（macro-task），宏任务又按执行顺序分为同步任务和异步任务

同步任务

console.log('script start');

console.log('script end');

异步任务

setTimeout(function() {

console.log('setTimeout');

}, 0);

微任务（micro-task）

Promise.resolve().then(function() {

console.log('promise1');

}).then(function() {

console.log('promise2');

});

执行过程：宏任务（同步任务）-》微任务-》宏任务（异步任务）

同步任务：也就是顺序执行的全局代码，非函数的代码。

异步任务：就是该行代码不直接执行，而是扔给 定时器 线程，定时器线程会定时的，往任务队列中推入。

当执行线程，先把 同步任务执行完毕后，会去任务队列中，找寻异步任务。

以上理解了，也就理解了：event-loop

最终执行过程：

var a = 5;

function f(n){

　　alert(a);

}

f();

先是全局预编译：

1、扫描函数声明、扫描变量定义。函数名相同时：会覆盖，变量名相同时，会忽略

2、将扫描到的函数和变量，保存到一个对象中。（windows）

3、a 是undefine，f 指向该函数。

{a : undefined, f : 'function(){alert(a);var a = 5;}'}

预编译结束，开始执行阶段：

a赋值为5，调用f函数，于是又开启该函数的 预编译与执行过程

基本上同全局过程一样。