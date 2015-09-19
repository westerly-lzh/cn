---
layout: post
title:  Python杂记
categories:
- python
tags:
- 

---

由于长时间的不用，Python的很多特性都已经开始遗忘了，这里对Python'的一些特性作记录以方便以后的复习使用。

## 函数的默认值

默认值只计算一次。这使得默认值是列表、字典或大部分类的实例时会有所不同。例如，下面的函数在后续调用过程中会累积传给它的参数：

	def f(a, L=[]):
	    L.append(a)
	    return L

	print f(1)
	print f(2)
	print f(3)
	
	####OUTPUT：
	[1]
	[1, 2]
	[1, 2, 3]

如果你不想默认值在随后的调用中共享，可以像这样编写函数：

	def f(a, L=None):
	    if L is None:
	        L = []
	    L.append(a)
	    return L

## 星号参数

在Python中，函数的参数中单个‘*’表示元组参数，‘**’表示字典参数，其中字典参数必须放在元组参数后面
	
	def ff(kind, *arguments, **keywords):
		pass;
		
	## 调用方式:
	ff(1,2,3,typ1=4,type2=5)

上面的调用中，2，3将会以元组参数的形式赋值给arguments=(2,3);type1,type2将会以字典参数的形式赋值给keywords={type1=4,type2=4}

## 列表推导式

列表推导式中每个for循环的计算顺序是从左向右

	[x**2 for x in range(10)]
	[(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
	###############
	vec = [[1,2,3], [4,5,6], [7,8,9]]
	[num for elem in vec for num in elem]
	###############
	matrix = [
	          [1, 2, 3, 4],
	          [5, 6, 7, 8],
	          [9, 10, 11, 12],
	          ]
	
	print([row[i] for row in matrix for i in range(4)])
	print([[row[i] for row in matrix] for i in range(4)])

	print([row[i] for i in range(4) for row in matrix])
	print([[row[i] for i in range(4)] for row in matrix])
		

## 生成器

	yield_stmt ::=  yield_expression

yield语句只在定义生成器函数时使用，且只在生成器函数的函数体中使用。虽然生成器是一个函数的，但我们不应当把它当做一个函数来理解，可以要把生成器当成一个具有数据结构的变量，如Java中的对象一样。只是这个变量具有一些特殊的“操作”。

	def gen():
	    for x in xrange(0,10):
	        yield x
	        if x%2==0:
	            yield "x is even"
            
	g = gen();

	print(g.next()) ## 0
	print(g.next()) ## x is even
	print(g.next()) ## 1
	print(g.next()) ## 2
	print(g.next()) ## x is even

yield 语句在python的文档中给出的解释如下：

> 当执行一个yield语句时，生成器的状态将冻结起来并且将expression_list的值返回给next()的调用者。“冻结”的意思是所有局部的状态都会被保存起来，包括当前局部变量的绑定、指令指针、内部的计算栈：保存足够的信息以使得下次调用next()时，函数可以准确地继续，就像yield语句只是另外一个外部调用。

在新的yield规范中，增加了对生成器增加了send(),throw(),close()等方法，具体使用参考[PEP 0342](https://www.python.org/dev/peps/pep-0342/)



## 迭代器

迭代器让我们避开了在遍历集合（列表）类型元素使用索引的尴尬，但迭代器不是线程安全的，所以不建议在多线程中使用迭代器。若想一个类型可以具有迭代器功能，只需要在该类型中实现*__iter__()*方法。

	lst = range(4)
	it = iter(lst)
	
	it.next()
	next(it)
	
Python提供了工厂方法从一个具有*iterable*特性的对象中获取迭代器，可以通过迭代器自身的*next()*来进行迭代，也可以使用内建函数*next()*来进行迭代。

但在Python的迭代器中没有判断是否已经到了迭代结束的*hasNext()*方法，所以只能通过异常(StopIteration)来进行迭代结束的判断。

迭代器虽然避开了索引，但是有时我们游需要索引，这时可以通过使用 *enumerate()*方法，对迭代器进行包装，使得调用迭代器的*next()*方法后返回的不光有迭代对象，还有该对象的下标。

	for idx, element in enumerate(lst):
		print(idx,element)


## 上下文管理器

“上下文”的概念表明，在某段程序执行之前，我们需要一些准备工作；在该程序执行结束之后，需要一些收尾工作。这里的“准备工作”和“收尾工作”就是该段程序的上限文。所以上下文管理器在Python中实际的作用就是为代码的开始执行做准备，为结束做善后。这个概念在程序使用计算机资源，比如文件时，特别有用，在使用文件前，需要判断文件是否存在，能否打开，然后打开文件；在使用文件结束后进行文件句柄关闭等资源回收操作。

	with open("new.txt", "w") as f:
	    print(f.closed)
	    f.write("Hello World!")
	print(f.closed)

在Python中任何对象只需要实现*__enter__()*和*__exit__()*操作就可以被用于上下文管理器。这两个方法就像它的名字所表示的那样清晰，*__enter__()*方法需要返回关键字*with*后表达式的返回值，并把该值赋给as后的变量。*__exit__()*方法在代码执行结束后执行，如果代码执行期间抛出了异常，那么该方法其实是可以捕获到异常的。*__exit__()*方法的完整方法签名如下：

	def __exit__(self, exc_type, exc_val, exc_tb)
	
Python中通过contextlib包提供了增强的上下文管理器。

## 装饰器

要了解装饰器，就需要了解面向切面的编程（经常讨论面向对象和面向过程）。在Python中装饰器承担了这样一种功能，在不改变函数的前提下，为该函数增加额外的功能，当然在Python中能用装饰器的不光是函数，其他的对象也可以。

	def decorator(func):
	    def wapper():
	        print "do some before"
	        func()
	        print "do some after"
	    return wapper


	def baseFun():
	    print "the base function"

	fun = decorator(baseFun)
	fun()
	
上面就是给方法baseFun定义了一个装饰器。采用装饰器装饰过的baseFun不光具有原来的功能，还新增加了其他的功能。下面是Python中通过使用*@*符号对函数使用装饰器的方法。和Java中的*注解(Annotation)*很类似。

	def decorator(func):
	    def wapper():
	        print "do some before"
	        func()
	        print "do some after"
	    return wapper

	@decorator
	def baseFun():
	    print "the base function"

	baseFun()
	
上面的装饰器定义，最后通过装饰器返回的函数，并不是我们传入的函数，这在某些编程情境下，比如我想知道该函数的名称时，上面被装饰器作用后的函数是无法获取到原来的函数名的。如果想保留原来的函数基本信息，需要利用functools包中的一个装饰器对来改造之前定义的装饰器。functools包的具体功能，请参考[functools](https://docs.python.org/2/library/functools.html)

	import functools
	def decorator(func):
	    @functools.wraps(func)
	    def wapper():
	        print "do some before"
	        func()
	        print "do some after"
	    return wapper

	@decorator
	def baseFun():
	    print "the base function"

	baseFun()
	print baseFun.__name__


## 闭包

> python中的闭包从表现形式上定义（解释）为：如果在一个内部函数里，对在外部作用域（但不是在全局作用域）的变量进行引用，那么内部函数就被认为是闭包(closure).
> 这里的"外部作用域的变量"是*只读*的。

明白了闭包表现形式上的定义和注意点，就可以很好的理解闭包中的各种匪夷所思的现象了。

	#SECTION 1
	def foo1():
	    a = 1
	    def bar():
	        a = a +1
	        return a
	    return bar

	c = foo1()
	print c()
	#SECTION 2
	def foo2():
	    a = 1
	    def bar():
	        b = a
	        b = b +1
	        return b
	    return bar

	c = foo2()
	print c()
	#SECTION 3
	def foo3():
	    a = 1
	    def bar():
	        b = a
	        a = b +1
	        return a
	    return bar

	c = foo3()
	print c()
	#SECTION 4
	def foo4():
	    a = 1
	    def bar():
	        a = 2
	        a = a +1
	        return a
	    return bar

	c = foo4()
	print c()
	


闭包中需要注意的就是外层变量是不可变的，所以新版的Python中引入了nonlocal关键字，来指定该变量是外部变量。

	def foo5():
	    a = 1
	    def bar():
	        nonlocal a
	        a = a +1
	        return a
	    return bar

	c = foo5()
	print c()
	
装饰器是闭包的一种应用，闭包还经常用于函数配置，配置信息不同，函数的行为结果也会不同：

	def make_adder(addend):
	    def adder(augend):
	        return augend + addend
	    return adder

	p = make_adder(23)
	q = make_adder(44)

	print p(100)
	print q(100)
	
make_adder 方法传入的参数不同，导致q和p函数虽然是同一个函数，且传入参数相同，但结果却不一样。

