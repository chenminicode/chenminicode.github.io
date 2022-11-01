---
layout: post
title: "Python中的几个魔法方法"
categories: Python
tags: Python
author: metseq
---

* content
{:toc}

# 类和对象的`__dict__`属性

## 对象的`__dict__`属性


```
class Point():
    '''Create a Point'''
    my_name = 'A Point'
    def __init__(self, x, y):
        self.x = x
        self.y = y
```


```
p = Point(3, 4)
p.__dict__
```




    {'x': 3, 'y': 4}



对象的`__dict__`属性就是一个字典，这个字典的键是对象的属性，值就是对应属性的值。

当给对象属性复制的时候，就是修改`__dict__`字典。

## 类的`__dict__`属性


```
Point.__dict__
```




    mappingproxy({'__module__': '__main__',
                  '__doc__': 'Create a Point',
                  'my_name': 'A Point',
                  '__init__': <function __main__.Point.__init__(self, x, y)>,
                  '__dict__': <attribute '__dict__' of 'Point' objects>,
                  '__weakref__': <attribute '__weakref__' of 'Point' objects>})



可以看到，`Point`类的`__dict__`属性包括了：
- 类的文档`__doc__`
- 类中定义变量，比如这里的`my_name`
- 类的方法
- 其他一些不知道什么的东西😓

# 类的`__repr__`和`__str__`方法

## `__repr__`方法

直接输出`p`时，会显示一堆没用的东西


```
p
```




    <__main__.Point>



我们想显示关于`Point`的有用信息，这时`__repr__`方法就派上用场了


```
class Point():
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return "Point(x=%r, y=%r)" % (self.x, self.y)
```


```
p = Point(3, 4)
p
```




    Point(x=3, y=4)




```
class Point():
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __setattr__(self, name, value):
        print(f'set attr {name} to float({value})')
        self.__dict__[name] = float(value)
    
    def __repr__(self):
        return "Point(x=%r, y=%r)" % (self.x, self.y)
```

## `__str__`方法

`__str__`方法和`__repr__`方法好像都是用字符表示对象的，有什么区别呢？

[stackoverflow](https://stackoverflow.com/questions/1436703/what-is-the-difference-between-str-and-repr)有一个回答说：
- `__repr__`显示的信息需要避免歧义
- `__str__`显示的信息注重可读性，方便理解
- `__str__`使用了`__repr`


```
str(p)
```




    'Point(x=3, y=4)'



在`Point`中如果没有重写`__str__`方法，会调用`__repr__`输出的结果

如果在`Point`中重写了`__str__`方法，就会输出`__str__`调用的结果。


```
class Point():
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return "Point(x=%r, y=%r)" % (self.x, self.y)
    
    def __str__(self):
        return f'Just a normal 2d point({self.x}, {self.y})'
```


```
p = Point(3, 4)
```


```
p
```




    Point(x=3, y=4)




```
repr(p)
```




    'Point(x=3, y=4)'




```
print(p)
```

    Just a normal 2d point(3, 4)



```
str(p)
```




    'Just a normal 2d point(3, 4)'



# 类的`__setattr__`方法


```
p = Point('3', '4')
p
```




    Point(x='3', y='4')



想在初始化Point的时候，把x，y属性转变成float类型，`__setattr__`方法就能用上了


```
class Point():
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __setattr__(self, name, value):
        self.__dict__[name] = float(value)
    
    def __repr__(self):
        return "Point(x=%r, y=%r)" % (self.x, self.y)
```


```
p = Point('3', '4')
p
```




    Point(x=3.0, y=4.0)




```
p.__dict__
```




    {'x': 3.0, 'y': 4.0}



p的x和y属性都已经是float类型了，这是怎么做到的呢？

在类的`__init__`函数中，`self.x = `x这一句，`self.x`的值设为`x`，

当遇到这种`对象.属性 = 值`的时候，如果重写了`__setattr__`方法，就会调用`__setattr__`方法，

也就是说当遇到`对象.属性 = 值`，相当于调用了`__setattr__(对象, 属性, 值)`

在这里相当于`__setattr__(self, 'x', x)`，在`__setattr__`函数里修改了`self.__dict__['x'] = float(value)`

`__setattr__`函数里面可以做你想做的任何事情，不仅仅是改变默认的属性赋值操作


```
class Point():
    def __init__(self, x, y):
        self._modules = {}
        self.x = x
        self.y = y
    
    def __setattr__(self, name, value):
        if not name.startswith('_'):
            self._modules[name] = value
            print(f'add {name}:{value} to _module')
        print(f'set {name} to {value}')
        super().__setattr__(name, value)
        
    def __repr__(self):
        return "Point(x=%r, y=%r)" % (self.x, self.y)
```


```
p = Point(3, 4)
```

    set _modules to {}
    add x:3 to _module
    set x to 3
    add y:4 to _module
    set y to 4



```
p._modules
```




    {'x': 3, 'y': 4}



