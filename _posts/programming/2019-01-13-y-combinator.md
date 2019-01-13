---
layout: post
date: 2019-01-13 16:52
status: public
tags: [函数式编程, C#, Y组合子, Y Combinator, 教程, 变换, 算法]
title: '从零开始理解 Y 组合子'
categories: [Programming]
---

Y Combinator (Y 组合子) 是函数式编程中最重要且优美的概念之一，但也是出了名的抽象和难以理解。《黑客与画家》的作者，硅谷创业教父 Paul Graham 甚至用此命名他的投资公司，以显示对它的喜爱。

本文将从一个最简单的例子开始，逐步地构造出这种优雅的数学结构。

## 从 Lambda 演算开始

Lambda 演算只有 3 条最近本的规则：

1. 变量定义: `x`
2. 函数定义: `lambda x, y. x + y`
3. 函数求值: `(lambda x, y. x + y)(1 2)`

令人惊讶的是，从这些简单的规则出发，我们可以得到和图灵机完全等价的计算模型。为了达到这个目的，首先要解决的一个问题就是递归。

下面的推到过程中，将全部使用 C# 来说明。原因是：
1. 比起 Lisp, Haskell 等语言，语法上更容易理解。
2. 强类型约束，更能体现之后的结构。



## 第一次尝试

先来看看图灵演算下的递归函数

```c#
int fact(int n)
{
	if(n == 0)
	{
		return 1;
	}
	
	return n * fact(n-1);
}
```

在函数体内部，又使用了 `fact` 这个函数的名字。但是，Lambda 演算中只有匿名的函数。那么，一个直观的想法就是把“自己”也当成一个参数传进去，也就是

```c#
int selfFact(Func<int,int> self, int n)
{
	if(n == 0)
	{
		return 1;
	}
	
	return n * self(n-1);
}
```

其中 `Func<In, Out>` 表示一个参数为 `In`  类型，返回值为 `Out` 类型的函数。

但这里又引入了新的问题：`self` 这个函数是什么？



## 转换形式

从含义上说，`self` 就应该是递归函数本身，如果能也写成一样的形式就好了。于是就有

```c#
delegate int Self(Self self, int n);

int fact2(Self self, int n)
{
	if(n == 0)
	{
		return 1;
	}
	
	return n * self(self, n-1);
}

int result = fact2(fact2, 5); //120
```



这里定义了一个新的函数类型 `Self`, 它的参数是 `Self` 和 `int`，返回值是 `int`.

于是，就可以把自身传给自身，从而间接的得到了递归的效果。

只是这个 `fact2(fact2, 5)` 过于的丑陋，难道就不能得到只需要一个参数的函数吗？



## 自由变量外提

为了实现递归的效果，需要把函数作为参数传递给自身。但是函数原本的参数应该是和这一个过程没有关系的。于是就可以把它（在这个例子里是 `int n`）提取出来，得到：

```c#
delegate T P<T>(P<T> p);

Func<int, int> fact3(P<Func<int, int>> self)
{
	return (n) =>
	{
		if(n == 0)
		{
			return 1;
		}
		
		return n * self(self)(n-1);
	};
}

Func<int, int> fac = fact3(fact3);
fac(5); //120
```

至此，第一不目标已经达成。注意：

1. 在 `fact3` 函数中，并没有使用 `fact3` 这个名字。
2. 最后得到的 `fac` 函数的类型是 `Func<int, int>`. 也就是说只需要一个参数 `n`.



## 更加通用

显然，这个方法不只局限于求阶乘。如果我们能把阶乘的逻辑从里面剥离出来，就可以实现一个通用的变化方法。这也就是说，把`self(self)` 从中间拿出来，变成 `f` 的一个参数。

 ```c#
Func<int, int> fact4(P<Func<int, int>> self)
{
	return (n) =>
	{
		Func<Func<int,int>, Func<int, int>> f = (q) =>
		{
			return (m) =>
			{
				if(m == 0)
				{
					return 1;
				}
				
				return m * q(m-1);
			};
		};
		
		return f(self(self))(n);
	};
}
 ```

仔细观察一下中间用到的临时函数 `f`, 完全就是之前不成功的 `selfact` 嘛。那干脆直接用之前定义好的。

```c#
Func<int, int> fact5(P<Func<int, int>> self)
{
	return (n) => selfFact(self(self), n);
}
```



## Y 组合子

从 `fact5` 函数，我们就得到了一个把 “不成功” 递归的函数变成递归的方法。再加上 `fact3`里提到的封装方式，就可以得到一个函数签名也满足需求的函数了，也就是

```c#
Func<int, int> Y(Func<Func<int, int>, int, int> f)
{
	P<Func<int, int>> g = (self) =>
	{
         // fact5 里的调用方法
		return (n) => f(self(self), n);
	};
	
    // fact3 中的封装
	return n => g(g)(n);
}
```

或者更一般的

```c#
Func<In, Out> Y<In, Out>(Func<Func<In, Out>, In, Out> f)
{
	P<Func<In, Out>> g = (self) =>
	{
		return (n) => f(self(self), n);
	};
	
	return n => g(g)(n);
}
```

然后这样使用

```c#
Y<int,int>((self, n) => n == 0 ? 1 : n * self(n - 1))(5); //120
```

这样，就实现了在匿名函数里进行递归的方法。由此扩展到更多数量的参数，或者别的语言，也是显而易见的事情了。

