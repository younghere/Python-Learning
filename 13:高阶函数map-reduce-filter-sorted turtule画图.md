## 高阶函数（python系统内置的）

### 1.map函数

```python
map(func, *iterables) 
#返回的是一个map对象
#参数1: 函数
#参数2: 序列
作用:将传入的函数依次作用于在序列中的每一个元素 并把结果返回作为一个新的可迭代对象(list)返回

>>> def f(x):
...     return x ** 2
...
>>> map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
[1, 4, 9, 16, 25, 36, 49, 64, 81]

#把列表中的所有数字转化为字符串
>>> map(str, [1, 2, 3, 4, 5, 6, 7, 8, 9])
['1', '2', '3', '4', '5', '6', '7', '8', '9']
```

### 2.reduce函数[不是内置的，这个需要导入functools模块]

```python
from functools import reduce
reduce(func, 序列)
参数1:函数(该函数需要两个形参)
参数2: 序列
作用: 将这个函数作用在序列上,将序列中的元素累计计算的过程(reduce每次需要向函数中传递两个值，第一次传入两个值，计算的结果作为其中一个值，再传入一个值，依次累计计算)
    
#求一个列表中的元素和
list0 = [23, 45, 67, 89]
def add(x, y):
    return x + y
res = reduce(add, list0)
print(res)

list1 = [1, 3, 5, 7]
def mult(x, y):
    return x * y

res = reduce(muilt, list1)
print(res)
```



### 3.filter函数

```python
筛选:
filter(func, 序列)
将序列中的元素依次传入到函数中 在函数中对每个元素进行判定
如果返回的是True ---> 表示保留该元素
如果返回为False ----> 表示去除该元素

#去除列表中的奇数
list0 = [15, 24, 32, 17, 55, 68, 91]
def even(item):
    if item % 2 == 0:
        return True
    else:
        return False

res = filter(even, list0)
print(res)
list1 = list(res)
print(list1)

去除的是字典中的键值对
    字典中的一个元素:
        key:value
dict0 = {"语文":68, "数学":77, "英语":56, "政治":48}
#去除掉字典中不及格的成绩

def del_item(item):#
    if item[1] < 60:
        return False
    else:
        return True

#dict.items()返回的数据格式：[(key1,value1),(key2,value2),...]
res = filter(del_item, dict0.items()) #这里要清楚filter函数有一个自动遍历的过程，将列表中每一个元组依次传入函数中，进行计算，而不是将整个items()列表传入函数中

print(dict(res))
```

### 4.sorted函数

排序

与列表中的sort的函数类型

```python
list0 = [87, 96, 42, 77, 59, 95]
#升序排序
list0.sort()#它对原对象起作用
print(list0)

降序
list0.sort(reverse=True)
print(list0)

list1 = ["abcd", "hello", "nice", "beauty"]
#按照字符串长度降序排序
list1.sort(key=len, reverse=True)
print(list1)

>>> sorted([36, 5, -12, 9, -21], key=abs)#根据绝对值排序
[5, 9, -12, -21, 36]

#内置函数sorted
list2 = [67, 89, 42, 11, 59, 76, 42]
#对它升序排序
n_list = sorted(list2)	#它对新生成的对象起作用，对原对象不起作用
print(list2)
print(n_list)

L = [("Bob",75),("Adam",92),("Bart",66),("Lisa",88)]
def by_name(t):
    return sorted(t[0])
L2 = sorted(L, key=by_name)     #sorted函数中的key也是针对于列表中的子元素，它自动将列表中的子元素与key中的函数对应
print(L2)

def by_score(t):
    return int(t[1])
L2 = sorted(L, key=by_score)
print(L2)

#使用sorted成绩升序
n_stus = sorted(stus, key=lambda s : s.score)
print(n_stus)

n_stus = sorted(stus, key=lambda s : s.age, reverse=True)
print(n_stus)
print("--------------------------------------")
#排序方式
for out in range(len(stus) - 1):
    for inner in range(out + 1, len(stus)):
        #成绩降序
        if stus[out].score < stus[inner].score:
            stus[out], stus[inner] = stus[inner], stus[out]
        elif stus[out].score == stus[inner].score:
            if stus[out].sid > stus[inner].sid:
                stus[out], stus[inner] = stus[inner], stus[out]
print(stus)

 #Python 定义了__str__()和__repr__()两种方法，__str__()用于显示给用户，而__repr__()用于显示给开发人员。
```

### 5.eval

```python
eval
    可以解析---字符串---代码
    把字符串中书写的代码 解析出来 映射到对应的类型
#"12 + 23"
res = eval("12 + 23")
print(res)

#"lambda x , y: x + y"
f = eval("lambda x , y: x + y")
print(f)
res = f(23, 56)
print(res)    
```

### 乌龟画图(Turtle)

在坐标系中(x,y)中进行绘制的，根据函数指令 在界面上旁 根据爬行路径绘制出对应的图像

1.画布(canves)

**import turtle**

设置画图区域的 大小

2.画笔

画笔的设置：

​	设置线宽，turtle.pensize(2)

​			turtle.pencolor("yellow")

​	turtle.speed(5)[在1—10范围内]

3.绘画的命令

画笔的运动命令（方法）

```python
import turtle

turtle.screensize(800,600)设定画布大小 #宽度可以设置小数 也可以设置整数
#小数的话表示是占屏幕的百分比
#整数表示的是像素

turtle.pensize(2)#设置笔迹的粗细

turtle.penup()起笔

turtle.goto(x,y) 笔的位置左右上下各移动多少距离【可以是负数】		

turtle.pendown()落笔

turtle.forward(位移)向画笔箭头的方向移动

turtle.backward()向画笔箭头背后移动多少距离

turtle.speed(10)笔移动的速度【1-10范围内】

turtle.hideturtle()隐藏画笔的形状

turtle.pencolor("red")笔迹的颜色
turtle.fillcolor("orange")填充色

turtle.color("red", "yellow")#前者表示字迹的颜色 后者表示填充的颜色

turtle.begin_fill()开始填充
turtle.end_fill()结束填充

turtle.done()绘画结束

#填充文本的
turtle.write("Done", font=("宋体",20, "normal"))

旋转：turtle.right(度数) 顺时针移动多少度 --操作画笔的，就是转动箭头方向

	 turtle.left(度数)逆时针转动多少度

      画圆:  turtle.circle(radius,entent,steps)

			radius --半径

			entent--弧度	

			steps--内切多边形的边数

	清空屏幕：turtle.clear() 清空后画笔还是在上一次结束的位置

	重置：trutle.reset()

	画点：turtle.dot(20,"red")(大小，颜色)

```

### 界面Tkinter

### pygame模块

