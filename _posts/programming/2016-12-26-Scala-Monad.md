---
layout: post
date: 2016-12-26 21:32
status: public
tags: 'Scala, Monad, Functor,Applicative, Monoid, 教程, 函数式编程'
title: 'Monad 简明教程 -- Scala 表述'
categories: [Programming]
---

# 算数，代数到代数结构
最早我们学习算数，$1+1 = 2$。不久我们发现 $1 + 2 = 2 + 1$, $2 + 5 = 5 + 2$。于是我们提出了「代数」，用字母来表示任意的数字。如此便得到了交换律 $ a + b = b + a$。

之后我们发现 $a + b = b + a$, $ a * b = b *  a $。既然他们都有相同的性质，那么也可以按照类似的思路，将相同的部分抽象出来，得到一个称之为代数结构的东西。我们可以用它来表示任何具有相同性质，比如交换律或者更复杂的的运算。

这样，我们便不再受限于已有的运算，而可以去定义任何合理的代数结构来满足我们的需求。下文说到的 Monoid，Functor，Applicative， Monad，都是根据我们的需要发明出来的东西。

难怪有人说「Monid不过是自函子范畴上的一个幺半群而已」。而所谓的「幺半群」，也就是代数结构的一种了。

# Monoid
从最简单的Monid开始。一个 Monoid 可以定义为
```scala
trait Moioid[T] {
    def zero: T
    def op(l: T, r: T): T
}
```
其中 `zero` 是“零元”， 而 `op` 是满足结合律的二元运算，也就是说
```
(a op b) op c == a op (b op c) 
x op zero == x
zero op x == x
```
这里的`op`不一定满足交换律，所以零元要求满足后面的两个式子。
显然实数和矩阵的加法，乘法都满足这个代数结构。此外如果把`op`定义为连接字符串，`zero`定义为空字符串，那么也是满足这个代数结构的。
# Functor
Functor 的定义为
```scala
trait Functor[C[_]]{
    def op[P,  Q](x: C[P])(f: P => Q): C[Q]
}
```
首先注意到，Functor 的定义中牵扯到了三个类型：`P`， `Q` 和泛型 `C` 。通常也把`C`称为环境(Context)。它可以是 `List[_]`或者`Option[_]`等。而函数`op` 接受两个参数：`x`, `f`。这里的`f`不是一个值，而是一个`P=>Q`的函数。而`op`的作用，是利用`f`把`C[P]` 变成 `C[Q]`。

对 `functor`，有这些限制：
1.  同一性: `op(x)( v => v) = x`
2.  结合律: `op( op( x, f1 ), f2) = op(x, f2(f1))`

很多集合都是一个 functor，因为我们在上面都定义了 `map` 函数，而它完全的符合这里的需要。例如对于 `List[T]` 有
```
List[Int](1, 2, 3).map(v=>v.toString)
```
就完成了 `List[Int]` 到 `List[String]`的一个变变换。 为了不至于混淆，在下面我们把 `functor` 里面的  `op` 称为 `map`。
# Applicative Functor 
Applicative 的定义为
```scala
trait Applicative[C[_]] extends Functor[C]{
    def pure[P](x:P):C[P]
    def apply[P, Q](x:C[P])(C[P=>Q]):C[Q]   
}
```

比起 `functor`，`applicative functor 多了两个函数： `pure` 和`apply`。

`pure`的含义非常简单，它接受一个值，并把它放在环境之中。

`apply`函数则和 `functor`中的`map`函数比较相似，不同之处在于它接受的函数`f`也是在环境`C`中的。

同样，`applicative functor`也需要满足一些定律

1. 同一性`apply(x)(pure(x=>x)) = x`

2. 一致性`apply(pure(x))(pure(f)) = pure(f(x))`

3. 交换律`apply(pure(x))(f) = apply(f)(pure(f=>f(x))` 

4. 结合律`apply(apply(x)(f1))(f2) = apply(x)(apply(f1)(f2)) `


# Monad

Monad 的定义为:

```scala
trait Monad[C[_]] extends Applicative[C]{
    def flatMap[P, Q](x:C[P])(f:P=>C[Q]):C[Q]
}
```

它在`Applicative Functor`的基础上增加了一个函数`flatMap`, 满足的限制为：

1. 左一致性 `flatMap(pure(x))(f) = f(x`
2. 右一致性`flatMap(x)(pure)= x`
3. 结合性 `flatMap(flatMap(x)(f))(g) = flatMap(x)(v=>flatMap(f(x))(g))`

# Functor, Applicative 和 Monad 的关系

* Functor: 已知`P => Q `，得到 `C[P] => C[Q]`
* Applicative: 已知 `C[P => Q]` ，得到 `C[P] => C[Q]`
* Monad: 已知`P => C[Q]`得到`C[P] => C[Q]`