# .net 框架

## 概览

.net = dot net

> 对.net的这4个字符的定义，我是没查到太官方的文档，都是些小道的新闻吧。
> 
> 
> 具说当时 互联网/网络 是下一个风口吧，所以微软在定义自家的很多技术的时候，统一就归结到了：net 这个词，至于前面为啥又加了个dot 或 . ，估计是跟域名结合有关吧

## .net

1. 它并不是具体的某一个技术，也不包含代码，它更像是一种协议、一种对固定技术群体的描述
2. 它更像是一个全家桶，微软的所有技术都包含在它的里面

## .net framework

是对.net思想的实现，1.0版本：发行于2002年左右

优点：稳定

缺点：与微软过于紧密，跨平台不好

## .net core

也是对.net思想的实现，1.0版本：发行于2016年左右

> 但毕发行之初竟新东西，之前那一整套的：ADO ASP winForm WPF WCF 根本不可能全重写，它是先移殖的asp 再加可执行c\#，东西较少，所以就叫：.net core

优点 跨平台更好

缺点 我没查着

> net core 是个新项目，与.net framework 没有代码关系

## .net5

它并不是新的东西，算是.net core 的别称吧，之所以是5.0，也是为了区别.net framework的版本号。

好像是把.net framework 和 .net core 给合并了

2020年.net core 差不多全移殖了.net farmework，于是将两个合并统一叫：.net5

最终看整个.net平台就包含了：.net framework , .net core , xamarin，这是完全包含了所有平台了

.net framework:对比core是旧版，但是开发WIN的程序非常好

.core :对于有跨平台的需求，用这个即可

xamarin:做安卓IOS端的开发

web开发：ASP 被包含在.net framework .core 里

桌面开发： winForm

## .net standard

对 .net frame 和 .netframework 的约束

## 为毛搞出这么我的术语名词？

好像是：鲍尔默 当值期间，与比尔思想吻合，闭源才是王道。大家为了用你的framework而必须购买微软的相关产品及服务，但随着个linux java这种东西的出现并爆火，好像，从商业上看有点失败。 之后，萨提亚·纳德拉上任后，提出 可跨平台方案。但，是不是开源了我不确定，但至少能让用户可以选择微软相关技术了，可以跨平台了，于是就有了.net core\(.net5\)

> 但我比较奇怪的是：苹果这些年一直也不开源，而且各种捆绑、且对3方的程序支持不够，那为毛苹果还是如日中天？

## mono

因为前期的不开源/不跨平台，导致开源社区的程序员，一直觉得c\#这么好的语言，不能通用性，有点可惜，于是他们自己搞了个:MONO，几乎等于重新实现了:.net framework

早期，这个mono发展的还不错，毕竟.net core 还不成气候。

在这个基础上又搞了自己的touch ，后期改名为 xamarin ，看名字也能猜出是在手机移动端的一个框架，不过好像被微软收购了，被包含在.net5 里了。

不过，随着.net core的出现并且完善后，mono就略尴尬，它原本的特性：跨平台，就没有了，在原.net framework 上自研的touch 又被收购了...

## 总结

结合上面总体上看：整个.net好大，还有开源分支mono的狗血剧情，整个历史20来年。

所以，有趣的事情就是：

如果一个人说我是.net开发？或者我是写C\#的？那如何界定它是在哪个领域的？mono开发？

桌面应用？WEB开发？游戏开发？ 哈哈

> .net framwork mono .net5 .net core xamarin asp winFrom 感觉真TMD乱，

## c\#

编译 IL =\> .exe .dll =\>CLR\(runtime\) =\> cpu

CLR

1. 翻译机器指令
2. GC

不同CPU ，用不同的CLR
