打印日历：

```python
def year_leap(year):
    if year % 4 == 0 and year % 100 != 0 or year % 400 == 0:
        return True
    else:
        return  False
def month_leap(year,month):
    if month in (1,3,5,7,8,10,12):
        return 31
    elif month in (4,6,9,11):
        return 30
    else:
        return 29 if year_leap(year) else 28
year = int(input("请输入年份："))
month = int(input("请输入月份："))
def tian_shu(year):
    day = 0
    for y in range(1900,year):
        day += 366 if year_leap(year) else 365
        return day
def tian_shu2(year,month):
    c = 0
    for m in range(1,month):
        c += month_leap(year,m)
        return c
total_day = tian_shu(year) + tian_shu2(year,month)
week = (total_day + 1) % 7
print("星期日\t星期一\t星期二\t星期三\t星期四\t星期五\t星期六")
for j in range(week):
    print(" ",end="")
days = month_leap(year,month)
for b in (1,days + 1):
    print(b,end="\t\t")
    if (b + week) % 7 == 0:
        print()

```

**return 后面是确定的值**

知识拓展：

1. 三元运算表达式：

​	条件表达式 and 表达式1 or 表达式2

2. 三元运算符的另外一种格式：

​	表达式1    if    条件表达式     else     表达式2

num1 = int(input("请输入一个值："))

num2 = int(input("请输入第二个个值："))

**max_value = num1 if num1 > num2 else num2**

**print(max_ value)**

## 一、global关键字

变量作用域引起的一个修饰词

在函数内声明的变量 只对当前函数有效

在函数内部 想修改函数外部声明的变量的值的话，如果不加修饰的话 在函数体内默认又声明一个变量

```python
 age = 18
 def update_age(): 
   #想要操作函数外部的变量 需要在函数体内添加修饰词 标注一下这个变量函数外部的    
   global age    
   #这种操作是在函数内部又声明了一个与函数外部重名的变量而已    
   age = 20    
   print("函数内部修改的age = ", age)
    #调用方法
    update_age()
    #打印函数外部的age变量
    print(age)
```



## 二、可变参数(Variable Argument)

**调用参数的时候可以传任意个实参**

```python
#对多个数据相加求和
*只是一个标识  numbers 才是一个变量名
def add(*numbers)：  #*args表示任何多个无名参数,它是一个tuple
	print (type(numbers))
	#采用元组存放的形式将数据存放在元组中
	total = 0 
    for n in numbers:
      total += n
      return total
t = (12,23,534,4656,767,34)   #传某个元组 也可以是某个列表
res = add(*t)            #一定要写*
print(res)	
```

格式：

​	def   函数名(*变量名)：

​		函数体

​		return

## 三、命名关键字参数

传值的时候 x = 2, y= 3  防止传错

```python
#自定义默认参数
def test_defargs(one, two = 2):
   print ('Required argument: ', one)
   print ('Optional argument: ', two)

test_defargs(1)
# result:
# Required argument: 1
# Optional argument: 2

test_defargs(1, 3)
# result:
# Required argument: 1
# Optional argument: 3
```

**函数可以接受任意个包含参数名的数据，这些数据在函数体内封装成dict**

```python
#用户的信息
def user(name, age):
    print("姓名:", name, "年龄:", age)

user("小沐", 18)

'''
注册网站的时候:
    必填项 ----> 字段必须清晰
    选填项 ----> 用户随意性
                传了多少个不清楚 -- 就没办法定义个数 模糊接收法
'''
def users(name, age, sex, **kw):
    print("姓名:", name, "年龄:", age, "性别:", sex, "other:", kw)
    if 'city' in kw:
        print("城市是", kw["city"])

users("李家欣", 25, "男", city="北京", birth="1994-01-01")
```

也可以传入字典

```python
#字典
dict0 = {"city":"深圳", "job":"developer"}
users("小沐", 18, "女", **dict0)
```



## 四、闭包

> 1. 定义：如果在一个内部函数里，对在外部作用域（但不是在全局作用域）的变量进行引用，那么内部函数就被认为是闭包(closure).
>
> 闭包=函数+引用环境)
>
> 闭包的书写格式:
>
> ​		def  A(参数):
>
> ​			def B(参数):
>
> ​				B的函数体
>
> ​			return B
>
> ​	这个时候A的形参就变成了自由变量, 因为在A中声明的变量可以在B中使用  当执行A结束之后 A中的参数依然被占用着, 只要被占用着就不会释放
>
> ​	如果在内部函数要修改外部函数的变量的值  需要使用nonlocal来修饰
>
> 2. 闭包的用途：(注意闭包的传值)
>
> - 当闭包执行完后，仍然能够保持住当前的运行环境。
> - 用途2，闭包可以根据外部作用域的局部变量来得到不同的结果，这有点像一种类似配置功能的作用，我们可以修改外部的变量，闭包根据这个变量展现出不同的功能。比如有时我们需要对某些文件的特殊行进行分析，先要提取出这些特殊行。
>
> ```python
> def make_filter(keep):
>     def the_filter(file_name):
>         file = open(file_name)
>         lines = file.readlines()
>         file.close()
>         filter_doc = [i for i in lines if keep in i]
>         return filter_doc
>     return the_filter
> ```
>
> 如果我们需要取得文件"result.txt"中含有"pass"关键字的行，则可以这样使用例子程序
>
> ```python
> filter = make_filter("pass")
> filter_result = filter("result.txt")
> ```

## 五、nonlocal关键字

```python
def outer():
    num = 10
    def inner():
        #在内部使用的num这个变量 被标注为外部函数的变量
        nonlocal num
        #如果直接这样操作 意味着在内部函数中声明了一个与外部函数同名变量 但是两者没有关联
        num = 20
        print(num, "内部函数被调用了")
    inner()
    print(num) #20
    return inner

#闭包的调用
res = outer()
print(res)
'''
<function outer.<locals>.inner at 0x00000188223AA8C8>

res是一个函数
'''
#调用内部函数
res()
```

## 六、装饰器

装饰器的函数声明就是一个闭包

装饰器的作用为一个函数在不修改原函数代码的前提下增加额外的功能

分为：带有给定参数和不给定参数的，不给定参数，就是使用(*args,**kawrgs)

案例: 计算一段代码的执行时间

​	模块: time

```python
#装饰器: 还是计算时间
def get_time_deco(func):#外部函数的形参就是需要装饰器修饰的函数
    print("装饰器的外部函数")
    def inner(*args, **kw):   #适合任何不定参数的函数
        start = time.time()
        res = func(*args, **kw)  #这里的参数要和内部函数的形参相同
        end = time.time()
        print(end - start)
        return res

    return inner

@get_time_deco  #get_time_deco(func5)
def func5():
    print("func5")

'''
为func5这个功能添加额外的计算时间的功能
'''
@get_time_deco
def func6(num):
    pass

#执行函数的功能
func6()

'''
添加上装饰器之后  就已经将函数本身传递给外部函数 外部已执行  返回了内部函数
    函数本身就和内部函数绑定在一起
        如果函数本身有参数  --- 内部函数有参数
            函数本身有返回值 ---- 内部函数有返回值
    想执行内部函数 -- 执行函数本身就可以
'''
```

装饰器本质上就是一个函数

```python
def get_time(func):
	 	start = time.time()
        func()
        end = time.time()
        print(end - start)
def code():
	pass
	
get_time(code)
```

本质上的执行

```python 
code = get_time_deco(code)
code()
```

@ --- 称之为糖语法

```python
#多个装饰器
import time

def deco01(func):
    def wrapper(*args, **kwargs):
        print("this is deco01")
        startTime = time.time()
        func(*args, **kwargs)
        endTime = time.time()
        msecs = (endTime - startTime)*1000
        print("time is %d ms" %msecs)
        print("deco01 end here")
    return wrapper

def deco02(func):
    def wrapper(*args, **kwargs):
        print("this is deco02")
        func(*args, **kwargs)

        print("deco02 end here")
    return wrapper

@deco01
@deco02
def func(a,b):
    print("hello，here is a func for add :")
    time.sleep(1)
    print("result is %d" %(a+b))

if __name__ == '__main__':
    f = func
    f(3,4)
    #func()

'''
this is deco01
this is deco02
hello，here is a func for add :
result is 7
deco02 end here
time is 1003 ms
deco01 end here
'''
```

**多个装饰器执行的顺序就是从最后一个装饰器开始，执行到第一个装饰器，再执行函数本身。**

```python
ef dec1(func):  
    print("1111")  
    def one():  
        print("2222")  
        func()  
        print("3333")  
    return one  

def dec2(func):  
    print("aaaa")  
    def two():  
        print("bbbb")  
        func()  
        print("cccc")  
    return two  

@dec1  
@dec2  
def test():  
    print("test test")  

test()  

#输出：

aaaa  
1111  
2222  
bbbb  
test test  
cccc  
3333
```



## 七、常用类型中方法的讲解

### 1.Number

> 在程序中使用的内容都是在内存中存在的. 声明一个变量 也是存放在内存中
>
> 这个变量什么被释放
>
> **内存机制**:
>
> ​	垃圾自动回收机制.
>
> ​	python 中所有的内容都是对象
>
> ​	当创建出来一个对象之后, 与这个对象绑定的有一个引用计数器[指向这个对象的引用个数]. 有一个引用变量指向,计数器+1. 当引用计数器为0的时候,这个对象会被标记上垃圾的标识, 等待着垃圾回收机制回收
>
> 如何将内存中的对象引用计数设置为0:
>
> ​	----> 1. 将引用指向其他的地方
>
> ​			a = 10
>
> ​			a  = 11 
>
> ​			10的位置就没用引用指向 
>
> ​	---->2. del 删除对应的引用变量
>
> ​		a = 10
>
> ​		del a ----> 删除之后 相当于就没有声明变量a这个内容了

#### 关于number的操做

> ##### 支持三种类型: 整型 浮点型  复数型
>
> 整型:
>
> ​	int
>
> ​	在python3中整型可以当做长整型来使用
>
> ​	所谓长整型也就意味着声明变量开辟内存时 内存大小为8B (64位)
>
> 浮点型
>
> ​	float
>
> ​	可以使用科学计数法来表示
>
> ​		2.5e2 ----> 2.5 * 10 ^ 2
>
> ​		2.5e-2 ----> 2. 5*10 ^ -2
>
> ​	在程序中尽量避免使用浮点型进行比较 ----> 因为精确的问题可能会造成判断失误
>
> 复数类型
>
> ​	complex
>
> ​	由实部和虚部组成
>
> ​	3 + 4j

#### 与number类型相关的一些内置方法

> 1. 求绝对值
>
>    ​	abs(num)
>
> 2. 比较两个数值的大小
>
>    ​	python2.x中方式是 cmp(x, y)
>
>    ​				如果x > y ----> 返回1
>
>    ​					x < y ----> -1
>
>    ​					x == y ----> 返回0
>
>    ​	python3中的比较方式:
>
>    ​		(x>y) - (x<y)
>
>    ​		通过比较运算符与布尔类型的结合产生的结果
>
>    ​				如果成立  视作1
>
>    ​					不成立视作0 
>
> 3. 求幂数
>
>    ​	pow(x, y)  求x的y次方
>
> 4. 四舍五入
>
>    ​	round(x, [n])
>
>    ​		如果只传x 返回来的是一个四舍五入的整数
>
>    ​		n表示的是保留小数点的位数
>
> 一下这两个方法针对于序列也可以使用
>
> 1. max ----> 取序列中的最大值
>
>    ​	max(x,x1,...)
>
>    1. min ----> 取序列中的最小值
>
>       min(....)

对于数学中的一些方法还单独的封装在一个math的模块中, 有所有的关于数学的方法:三角函数

> math中的
>
> ```python
> ceil:取大于等于x的最小的整数值，如果x是一个整数，则返回x
> 
> copysign:把y的正负号加到x前面，可以使用0
> 
> cos:求x的余弦，x必须是弧度
> 
> degrees:把x从弧度转换成角度
> 
> e:表示一个常量
> 
> exp:返回math.e,也就是2.71828的x次方
> 
> expm1:返回math.e的x(其值为2.71828)次方的值减１
> 
> fabs:返回x的绝对值
> 
> factorial:取x的阶乘的值
> 
> floor:取小于等于x的最大的整数值，如果x是一个整数，则返回自身
> 
> fmod:得到x/y的余数，其值是一个浮点数
> 
> frexp:返回一个元组(m,e),其计算方式为：x分别除0.5和1,得到一个值的范围
> 
> fsum:对迭代器里的每个元素进行求和操作
> 
> gcd:返回x和y的最大公约数
> 
> hypot:如果x是不是无穷大的数字,则返回True,否则返回False
> 
> isfinite:如果x是正无穷大或负无穷大，则返回True,否则返回False
> 
> isinf:如果x是正无穷大或负无穷大，则返回True,否则返回False
> 
> isnan:如果x不是数字True,否则返回False
> 
> ldexp:返回x*(2**i)的值
> 
> log:返回x的自然对数，默认以e为基数，base参数给定时，将x的对数返回给定的base,计算式为：log(x)/log(base)
> 
> log10:返回x的以10为底的对数
> 
> log1p:返回x+1的自然对数(基数为e)的值
> 
> log2:返回x的基2对数
> 
> modf:返回由x的小数部分和整数部分组成的元组
> 
> pi:数字常量，圆周率
> 
> pow:返回x的y次方，即x**y
> 
> radians:把角度x转换成弧度
> 
> sin:求x(x为弧度)的正弦值
> 
> sqrt:求x的平方根
> 
> tan:返回x(x为弧度)的正切值
> 
> trunc:返回x的整数部分
> ```



