### 0. 前言

列表推导式 & lambda表达式会碰撞出什么样的火花呢，请看。

### 1. 正文

事情由一段代码引发：

![](.\img\list-and-lambda-01.jpg)

上述的列表推导式+lambda表达式+for循环，他们碰撞出来的结果搞的人晕头转向，咱们逐步来分析一下他们到底是个什么鬼。

lambda表达式可以表示成：

```python
def func(x):
    return x*i
```

加上for循环：

```python
for i in range(10):
    def func(x):
        return x*i
```

加上列表推导式： # *注释一*

```python
li = []
 
def funcs():
    for i in range(10):
        def func(x):
            return x*i
        li.append(func)
    return li
funcs()
```

到此，就可以看清楚他的结构了，funcs函数里面形成了一个 **闭包**
闭包的两个特性：

> ①、内部函数使用外部函数的变量（不对外部函数的变量进行重新赋值）
>
> ②、外部函数 return 内部函数的函数名（其实是返回内部函数的内存地址）
>

对上面的函数进行简化：去掉for循环，使用单个变量代替，去掉 列表 li 直接返回内部函数名，它就变成了下面这个样子：

```python
def funcs():
    i=9  # 定义了变量，也称为内部函数的环境变量，它不会改变，不被外界影响
 
    def func(x):
        return x*i  # 使用了外部函数的变量
 
    return func  # 返回了内部函数
```

**简单理解：闭包 = 内部函数 + 环境变量**

闭包的特性：当内部函数返回的时候，会把连同内部函数定义时候的环境变量一同返回。

了解了闭包，在来看结果为什么全都是18，而不是预期结果：

①区分内部函数的定义和调用：

- 当我们只到是定义了外部和内部函数，调用了外部函数，并未调用内部函数，也就是到 *注释一*

- for循环的作用就是创建10个函数，最后 i 的取值是9


所以我们获得了类似下面的图谱：

![](.\img\list-and-lambda-02.jpg)

当我们进行内部函数调用的时候，此时的情形是：

有一个包含10个函数的列表，变量 i=9

再加上**闭包的特性，i 会随着内部函数的一同返回**，所以调用依次内部函数的时候，情形就成了下面这样：以调用第1个函数为例：

![](.\img\list-and-lambda-03.jpg)

所以结果就成了 2 * 9 = 18

再调用别的内部函数都是如此逻辑。

知乎中此问题的回答者 @Op小剑 提到了延迟绑定

> 匿名函数的 i 并不是依次顺序指向0,1,2,...的，因为在迭代的过程中，创建匿名函数→创建完成后→才去找i 指向的值。

对上图做了很好的诠释。当我们将表达式修改为下面的样子，预期结果就出现了：

```python
li = []
 
def funcs():
    for i in range(10):
        def func(x, i=i):  # 给内部函数传参， i 是内部函数的局部变量，当所有内部函数定义完毕，i 的值也变成了一个列表（range(10)）
            return x*i  # 调用的时候其实是调用的内部函数的值
        li.append(func)
    return li
 
for func in funcs():
    func(2)
```

修改到列表推导式+lambda表达式就是下面的情况：

```python
funcs = [lambda x, i=i: x*i for i in range(10)]
for func in funcs:
    print(func(2))
```

此时调用的情形就像下面的图谱：

![](.\img\list-and-lambda-04.jpg)

### 2. 结尾

到此，此问题就分析结束了，涉及到的变量作用域，就不分析了，大家可以自行百度去理解。

最后分享一个[python的运行过程展示网站](http://www.pythontutor.com/visualize.html#mode=edit)，希望可以给大家带来帮助。:star:

