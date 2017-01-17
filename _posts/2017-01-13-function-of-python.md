---
title: Python function prohibition
layout: post
date: Sun Jan 15 23:08:10 EST 2017 
categroy: Python
tags: Python
---
* content
{:toc}
# 高阶函数
** 返回函数或者参数是函数的函数 
** Python中函数是一等对象(first class)，函数也是对象，并且它可以像普通对象一样赋值，作为参数，作为返回值；
相关情况：
* 函数作为返回值： 通常是用于闭包的场景， 需要封装一些变量
```python
def counter(i):
    base = i
    def inc(x=1):
        nonlocal base
        base += x
        return base
    return inc
```
```python
inc = counter(3)
```
```python
inc(3)
```
    6
* 函数作为参数：通常用于大多数逻辑固定，少部分逻辑不固定的场景
```python
def sort(it, cmp=lambda a, b: a < b):    
    ret = []
    for x in it:
        for i, e in enumerate(ret):
            if cmp(x, e):
                ret.insert(i, x)
                break
        else:
            ret.append(x)
    return ret
            
```
```python
sort([1, 3, 2, 4, 6, 8, 5], lambda a, b: a>b)
```
    [8, 6, 5, 4, 3, 2, 1]
* 函数作为参数，返回值也是函数： 通常用于作为参数函数执行前后需要一些额外操作
```python
import datetime
def logger(fn): # 函数作为返回值： 封装了fn
    def wrap(*args, **kwargs):
        start = datetime.datetime.now()
        ret = fn(*args, **kwargs)
        end = datetime.datetime.now()
        print('call {} took {}'.format(fn.__name__, end-start))
        return ret
    return wrap
```
```python
def add(x, y):
    return x + y
```
```python
loged_add = logger(add)
```
```python
loged_add(3, y=5)
```
    call add took 0:00:00.000030
    8
```python
import time
def sleep(x):
    time.sleep(x)
```
```python
logged_sleep = logger(sleep)
```
```python
logged_sleep(3)
```
    call sleep took 0:00:03.005698
```python
sleep = logger(sleep)
```
```python
sleep(3)
```
    call sleep took 0:00:03.003675
# 装饰器
参数是**一个函数**， 返回值是**一个函数**的**函数**，就可以作为装饰器
* 装饰器的使用
```python
import datetime
def logger(fn): # 函数作为返回值： 封装了fn
    def wrap(*args, **kwargs):
        start = datetime.datetime.now()
        ret = fn(*args, **kwargs)
        end = datetime.datetime.now()
        print('call {} took {}'.format(fn.__name__, end-start))
        return ret
    return wrap
```
```python
@logger
def sleep(x):
    time.sleep(x)
```
```python
sleep(3)
```
    call sleep took 0:00:03.003233
相当于：
```python
import datetime
def logger(fn): # 函数作为返回值： 封装了fn
    def wrap(*args, **kwargs):
        start = datetime.datetime.now()
        ret = fn(*args, **kwargs)
        end = datetime.datetime.now()
        print('call {} took {}'.format(fn.__name__, end-start))
        return ret
    return wrap

import time
def sleep(x):
   time.sleep(x)
sleep=logger(sleep)
```

```python
help(list.insert)
```
    Help on method_descriptor:
    
    insert(...)
        L.insert(index, object) -- insert object before index
    
```python
help(add)
```
    Help on function add in module __main__:
    
    add(x, y)
## 柯里化    
函数可以有它的文档：可以使用fn.__doc__或help(fn)，函数的其他属性可以使用dir(fn)得到
```python
def fn():
    '''this is fn''' 
```
```python
help(fn)
```
    Help on function fn in module __main__:
    
    fn()
        this is fn
    
```python
fn.__doc__
```
    'this is fn'
```python
fn.__name__
```
    'fn'
```python
sleep.__name__  # sleep函数的信息，在装饰后改变了，--没有柯里化
```
    'wrap'
```python
import datetime  ##  引入函数专门做柯里化
def logger(fn): # 函数作为返回值： 封装了fn
    def wrap(*args, **kwargs):
        start = datetime.datetime.now()
        ret = fn(*args, **kwargs)
        end = datetime.datetime.now()
        print('call {} took {}'.format(fn.__name__, end-start))
        return ret
    wrap.__name__ = fn.__name__
    wrap.__doc__ = fn.__doc__
    return wrap
```
```python
@logger
def sleep(x):
    time.sleep(x)
```
```python
sleep.__name__
```
    'sleep'   # 实现了，不过这部分逻辑不需要重复，写个函数先替代它...
```python
def copy_proprities(src, dst):
    dst.__name__ =  src.__name__
    dst.__doc__ = src.__doc__
```
```python
import datetime
def logger(fn):
    def wrap(*args, **kwargs):
        start = datetime.datetime.now()
        ret = fn(*args, **kwargs)
        end = datetime.datetime.now()
        print('call {} took {}'.format(fn.__name__, end-start))
        return ret
    copy_proprities(fn, wrap)
    return wrap
```
```python
def copy_proprities(src):  # 既然里面有了src的变量，为什么不写一个目标参数就可以的函数，等等这里输入是一个函数，返回也是一个函数，那么应该是装饰器！不过它要求输入两个参数：dst需要在装饰器中传递进去。
    def _copy(dst):
        dst.__name__ =  src.__name__
        dst.__doc__ = src.__doc__
        return dst
    return _copy
        
```
```python
import datetime
def logger(fn): # 函数作为返回值： 封装了fn
    @copy_proprities(fn)
    def wrap(*args, **kwargs):
        start = datetime.datetime.now()
        ret = fn(*args, **kwargs)
        end = datetime.datetime.now()
        print('call {} took {}'.format(fn.__name__, end-start))
        return ret
    return wrap
```
```python
@logger
def sleep(x):
    time.sleep(x)
```
```python
sleep.__name__
```
    'sleep'
```python
import functools
```
```python
import datetime
def logger(fn): # 函数作为返回值： 封装了fn
    @functools.wraps(fn)
    def wrap(*args, **kwargs):
        start = datetime.datetime.now()
        ret = fn(*args, **kwargs)
        end = datetime.datetime.now()
        print('call {} took {}'.format(fn.__name__, end-start))
        return ret
    return wrap
```
```python
@logger
def sleep(x):
    time.sleep(x)
```
```python
sleep.__name__
```
    'sleep'
```python
help(functools.wraps)  # 这里有个函数专职做柯里化，调用它就行了
```
    Help on function wraps in module functools:
    
    wraps(wrapped, assigned=('__module__', '__name__', '__qualname__', '__doc__', '__annotations__'), updated=('__dict__',))
        Decorator factory to apply update_wrapper() to a wrapper function
        
        Returns a decorator that invokes update_wrapper() with the decorated
        function as the wrapper argument and the arguments to wraps() as the
        remaining arguments. Default arguments are as for update_wrapper().
        This is a convenience function to simplify applying partial() to
        update_wrapper().
## 带参数的装饰器： 一个函数， 返回一个不带参数的装饰器
```python
s = datetime.datetime.now()
```
```python
e = datetime.datetime.now()
```
```python
delta = e - s
```
```python
delta.total_seconds()
```
    13.524782
```python
def logger(s):
    def _logger(fn):
        @functools.wraps(fn)
        def wrap(*args, **kwargs):
            start = datetime.datetime.now()
            ret = fn(*args, **kwargs)
            end = datetime.datetime.now(ii)
            if (end-start).total_seconds() > s:
                print('call {} took {}'.format(fn.__name__, end-start))
            return ret
        return wrap
    return _logger
    
```
```python
@logger(2)
def sleep(x):
    time.sleep(x)
```
```python
sleep(3)
```
    call sleep took 0:00:03.004279
```python
sleep(1)
```
```python
def logger(s, p=lambda name, t: print('call {} took {}'.format(name, t))):
    def _logger(fn):
        
        @functools.wraps(fn)
        def wrap(*args, **kwargs):
            
            start = datetime.datetime.now()
            ret = fn(*args, **kwargs)
            end = datetime.datetime.now()
            if (end-start).total_seconds() > s:
                p(fn.__name__, end-start)
            return ret
        return wrap
    return _logger
    
```
```python
@logger(2, p=lambda name, t: None)
def sleep(x):
    time.sleep(x)
```
```python
sleep(3)
```
```python
def logger(s):
    def _logger(p=lambda name, t: print('call {} took {}'.format(name, t))):
        def __logger(fn):
            @functools.wraps(fn)
            def wrap(*args, **kwargs):
                start = datetime.datetime.now()
                ret = fn(*args, **kwargs)
                end = datetime.datetime.now()
                if (end-start).total_seconds() > s:
                    p(fn.__name__, end-start)
                return ret
            return wrap
        return __logger
    return _logger
    
```
```python
logger2s = logger(2)
```
```python
@logger2s()
def sleep(x):
    time.sleep(x)
```
```python
sleep(3)
```
    call sleep took 0:00:03.003878
```python
sleep(1)
```
```python
sleep = logger2s()(sleep)
```

