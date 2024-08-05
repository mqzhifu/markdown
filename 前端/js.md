函数里：如果不加上var关键字，默认添加到全局对象的属性上去

Arguments：对象包括如下

callee — 指向当前函数的引用

length — 真正传递的参数个数

properties\-indexes \(字符串类型的整数\) 属性的值就是函数的参数值\(按参数列表从左到右排列\)。 properties\-indexes内部元素的个数等于arguments.length. properties\-indexes 的值和实际传递进来的参数之间是共享的。

OBJECT:

属性：

Constructor:对创建对象的函数的引用（指针）。对于Object类，该指针指向原始的object\(\)函数。

Prototype:对该对象的对象原型的引用。对于所有的类，它默认返回Object对象的一个实例。

方法：

hasOwnProperty\(property\):判断对象是否有某个特定的属性。必须用字符串指定该属性（例如，o.hasOwnProperty\(”name”\)）。

isPrototypeOf\(object\):判断该对象是否为另一个对象的原型。

propertyIsEnumerable\(property\):判断给定的属性是否可以用for…in语句进行枚举。

toString\(\):返回对象的原始字符串表示。对于Object类，ECMA－262没有定义这个值，所以不同的ECMAScriipt实现具有不同的值。

valueOf\(\):返回最适合该对象的原值。对于许多类，该方法返回的值都与toString\(\)的返回值相同。

DOM有三种节点类型：元素节点、属性节点、文本节点

property：对象的属性

attribute：属性节点（div.getAttribute\('属性名'\);，div.setAttribute\('属性名'，'值'\);）

基类型：

object

number

string

boolean

null

undefined

如定义的参数列表中有重名参数，该属性（重名的参数名作为属性）值为最右边的参数传入的值。

即使第二个y未传入，结果依然取自最后的y，即：undefined

例：\!function\(x, y, y\){alert\(y\);}\(1, 2\) // alert undefined

\!作用是为了将function…作为函数表达式来解析，而不是函数定义。等同于\(function\(\){}\(\)\)

var arg = 1;

function funcTest\(\) {

var arg; //默认值undefined

alert\(arg\);

arg = 2;

}

arg = 10;

funcTest\(\);

ScopeChain:作用域链，包含在EC类的scope 属性中

Execution Context:运行期上下文，是一个对象，一个函数执行时的环境，函数每次执行时都会创建一个，函数执行完毕，删除

Activation Object:激活对象，EC类的scope 属性第0个元素值

GLOBAL Object:全局对象，EC类的scope 属性第1个元素值

Variable Object:可变对象

定义一个函数，在编译的过程中，实际，我们写的函数，只是构造器，我们可以看成，在这个函数的外面还有一个类，而我

们定义的函数在这个类的里面的一个方法。

这个类中，有一个成员变量：scope，该变量指向一个连接结构\(scope chain\)

链表的第一个元素保存的就是一个大一global数组（windows,this,document）

当开始执行此函数时，就会创建一个Execution Context的内部对象，同样，也有scope属性，但是，元素0是：Activation

Object（激活对象），元素1是，global数组

激活对象：保存了函数中的所有形参，实参，局部变量，this指针等函数执行时函数内部的数据情况。这是个可变对象，随

着执行而变化。

当取变量的时候，会先成 ao中寻找，如果没有，就会去GLOBAL中找

函数结束后销毁EC AO

undefined:变量值未定义

NaN:表达式计算，但不知道是什么值

null:跟对象相关

在进入任何EC前，会创建一个全局的、唯一的全局对象\(Global Object\)，其中包含内置的属性，如：Math, String, Date,

parseInt等等。另外，宿主\(host\)定义的一些属性也会初始化在全局对象中；例如，在HTML DOM中，会添加一个window属性

，值为全局对象本身。

当代码进行执行期时，会创建Execution Context

第一个EC，就是全局EC，接着还会创建函数的EC

也就是EC中包含EC，EC是动态创建的

EC存储在一个LOGIC STACK中，最顶部最是当前的EC

最底部是全局EC

（VO，可变对象，存放当前函数内的一些值，如参数。）

对于函数代码的EC，在VO初始化之前会创建一个激活对象\(Activation Object，缩写为AO\)，该激活对象会被初始化一个

arguments属性。随后，该AO对象被用作VO对象并完成前面的变量初始化阶段。

每个EC都对应一个变量对象\(VO\)

可执行代码分类：

全局代码\\函数代码\\EVAL代码

全局环境，先初始化全局对象\(Global Object\)，再初始化变量对象\(Variable Object\)，然后从全局代码开始执行

对于函数的执行环境，先初始化激活对象（Activation Object），而后该激活对象将被用作变量对象进行变量初始化过程，

最后执行函数代码。

对于Eval环境，执行环境取决于调用环境\(Calling Object\)，若在全局环境中调用eval\(\)，则Eval调用环境为该全局环境；若在函数的执行环境中调用eval\(\)，则Eval调用环境为该函数的执行环境。

