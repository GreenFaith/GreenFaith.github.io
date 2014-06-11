---
layout: post
title: "python Decorators"
date: 2014-06-11 14:23:00

---

# Decorators(装饰器）
Decorators 本身是一个函数，生来就只有一个目的   
` 接受一个函数，返回一个函数`   

看个例子就明白了：

```
def hello(fun):
    def new_fun(*args,**kwargs):
        print fun.__name__ + ' stared' 
        return fun(*args,**kwargs)
    return new_fun

@hello
def fun():
    pass

```

相当于：   

```
def fun():
    pass
fun = hello(fun)
```
 decorate 在原本定义的函数`fun`上悄无声息地添加一些代码。不管是写成
@开头的注释模式，还是直接写成函数,效果都是一样的。
   
</br>   
</br>
## Closures

前面说过 Decorator 是一个接受函数作为参数，并返回函数的东西。   
而Closures 大概就可以认为是一种接受参数(不局限与函数），返回函数的东西。   

先看个简单的例子：   

```
def add_by(a):
    def add(b):
        return a+b
    return add

>>add1 = add_by(1)
>>add1(2)
>>3
```

是不是有点意思，通过 Closures 我们可以生产一堆类似的函数：

```
>>add100 = add_by(100)
>>add100(1)
>>101
```
推广一下这种模式，通过选择不同的输入参数，我们可以动态构造不同的函数。
是不是有点元编程的味道？
</br>
</br>
## Wrappers
Decorate 模式其实存在一个问题：

```
@hello
def fun():
    """this is docstring"""
    pass

>>fun.__name__
>>'new_fun' 

>>fun.__doc__
>>
```
`new_fun` 丢失了 fun 的某些信息。
那么把`__name__` 和 `__doc__`  复制给`new_fun`不就好了?

```
def hello(fun):
    def new_fun(*args,**kwargs):
        print fun.__name__ + ' stared' 
        return fun(*args,**kwargs)
    new_fun.__name__ = fun.__name__
    new_fun.__doc__ = fun.__doc__
    return new_fun

@hello
def fun():
    """fun's docString"""
    pass

>>> fun.__name__ 
>>> fun

>>> fun.__doc__
>>> "fun's docString"
```
但是处女座说这样绝逼不行，如果要复制的东西很多？每次些decorator都得这么复制？
不就是复制函数信息么，哥加个小技能就能搞定：   
</br>

### 带参数的Decorator:
   
```
#: 增加一个 decorator 以解决函数信息丢失问题
def copy_from(src_fun):
    """ put src_fun's info to decorator """
    def copy(fun):
        """ this is the decorator"""
        def new_fun(*args,**kargs):
            return src_fun(*args,**kargs)

        new_fun.__name__ = src_fun.__name__
        new_fun.__doc__ = src_fun.__doc__

        return new_fun

    return copy    
 
def hello(fun):
    @copy_from(fun) #: 相当于 @copy
    def new_fun(*args,**kwargs):
        print fun.__name__ + ' stared' 
        return fun(*args,**kwargs)
    return new_fun

@hello
def fun(num):
    """ fun's docString"""
    return num


>>> fun.__name__ 
>>> fun

>>> fun.__doc__
>>> "fun's docString" 
```
这个例子的亮点在于 `closure` 的使用,把 `src_fucn` 的信息带入 `copy` 装饰器。
而且上面复制函数信息的 `copy_from`,python已经内置了更全面的实现 `functool.wraps`
用法与 `copy_from` 一致。

</br>
</br>

参考文献：   
<< Pro Python >>-- Marty Alchin
