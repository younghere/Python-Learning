## 1.枚举

**该类型的对象的值是固定的，且对象个数是有限的**

一系列常量的**集合**

设置一个枚举类 需要继承自Enum/IntEnum

这些内容是放置**在enum模块中**的--->python3.4的时候出现

python3.4之前需要使用 pip install enum 【不是系统提供的 】

在这个模块下使用的有Enum/IntEnum/ 装饰器：@unique

IntEnum:限定枚举成员都必须是 整型或者可以转化为整型

unique：限定枚举成员的值不可以重复

```python
1.定义枚举时，成员名称不允许重复　　　
2.默认情况下，不同的成员值允许相同。但是两个相同值的成员，第二个成员的名称被视作第一个成员的别名　　
3.如果枚举中存在相同值的成员，在通过值获取枚举成员时，只能获取到第一个成员
4.如果要限制定义枚举时，不能定义相同值的成员。可以使用装饰器@unique【要导入unique模块】

from enum import Enum
class Season(Enum):
    #常量 声明名字的话使用是大写字母
    SPRING = "春"
    WINTER = "冬"
    WINTER = '仲夏' 	#会报错，因为定义枚举时，成员名称不允许重复
#获取枚举类型中的对象
print(Season.SPRING)
#获得枚举类型对象对应的值
print(Season.SPRING.value)

from enum import IntEnum
#这种枚举类型要求 枚举对象的值必须是整型或者是可以转化为整型的数据

class Week(IntEnum):
    MON = 1
    TUE = "2"
    WED = "wed" 	#会报错，因为枚举对象的值不能转化为整型
print(Week.TUE)
print(Week.TUE.value)


from enum import Enum,unique
@unique #作用是枚举类型的值是不能重复的
class Month(Enum):
    JAN = 1
    FEB = 1  #会报错，值不能相同
print(Month.JAN)

#枚举类型对象本身还是枚举类型
#print(Month.JAN.MAR.MAY)
```

**枚举类型对象本身还是枚举类型**



## 2.异常机制

**异常出现的时机**：

​	当程序可能会发生异常的位置 如果检测到有异常 创建一个对应的一场信息

**程序对异常机制的处理**：

​	默认采用的是系统抛出，系统捕获

如果让系统进行捕获 -->程序就直接中断 -->会影响用户的使用---->程序员会采用手动捕获

### 2.1 手动捕获异常的方式

**Python中常见的异常类型**:

```python
AttributeError 试图访问一个对象没有的树形，比如foo.x，但是foo没有属性x
IOError 输入/输出异常；基本上是无法打开文件
ImportError 无法引入模块或包；基本上是路径问题或名称错误
IndentationError 语法错误（的子类） ；代码没有正确对齐
IndexError 下标索引超出序列边界，比如当x只有三个元素，却试图访问x[5]
KeyError 试图访问字典里不存在的键
KeyboardInterrupt Ctrl+C被按下
NameError 使用一个还未被赋予对象的变量
SyntaxError Python代码非法，代码不能编译(个人认为这是语法错误，写错了）
TypeError 传入对象类型与要求的不符合
UnboundLocalError 试图访问一个还未被设置的局部变量，基本上是由于另有一个同名的全局变量，
导致你以为正在访问它
ValueError 传入一个调用者不期望的值，即使值的类型是正确的
```



```python
try-except 
格式：
	try:
		存放的是可能会出现异常的代码
	except 异常类型 as 变量名：
		捕获带了异常
        
#try-except-else

while True:
	try:
		num = int(input("请输入一个数字："))
	except ValueError as e:  #可以用Exception替代ValueError
		print(e)
		print("输入的不是数字，请重新输入")
		continue
			#要想继续运行就用else：
    esle:     #try中没有异常才会执行
		break
print(int(num))

#只会执行第一个except
except IndexError as e:
	print(e,"索引异常")
except NameError as e:
	print(e,")
	

```

如果抛出的不知道是什么异常 使用异常的祖宗来接受**Exception**

如果try中有多个异常 -- 遇到第一个异常 try的代码就会中断 不会向下运行

**try-except-finally**

​	格式：

​		try:

​			可能出现异常的代码

​		except 异常类型 as 变量：

​			捕获到了异常

​		finally:

​			不管有没有异常都会执行此处

这种格式使用的位置：

​	输入/输出 网络请求 一般涉及到**资源关闭**的问题 

```python
#读取文件 -->对于程序而言是外界资源
#1.打开文件 获得操作文件的手柄
fp = open("test.py","rb") -->以二进制读取数据
#读取数据
data = fp.read()
del data
print(data)      #会出现NameError
#关闭资源
fp.close()#此时关闭资源就不会执行到，就会产生垃圾

fp = None
try:
	fp = open("test.py","rb")     #赋值的优先级是最低的
	data = fp.read()
	del data
	print(data)
except Exception as e:
	print(e)
finally:     #finally是无论有没有出现异常都会运行这里
	if fp != None:#如果不做判断会出现None.close()  会出现异常
		fp.close()
```

手动抛出异常 **raise**语句

​	捕获到的是异常对象	

```python
def div(a,b):
	if b == 0:
		raise Exception("value error:%d" % b)
		#手动抛出的语句后面不能直接跟随其他语句 因为该语句无法执行 执行完raise就不会向下执行了
	return a/b
```

**手动抛出异常 无论该条语句是否出现异常 都会中断代码  不会向下继续执行**

自定义异常：

所谓异常就是不正常的状态 只要不满足生活实际需求的话 就是一种不正常的状态

去自定义异常类型：

​	需要继承自**Exception** 这个类才是异常类

谁调用谁处理异常-->如果调用者不处理 ，继续向上抛出异常-->依然没有地方处理，系统会做处理

```python 
#自定义异常的用法
from cus_exception.age_negative import AgeNegative

class Person(object):
    def __init__(self, name, age, sex):
        self.set_name(name)
        self.set_age(age)
        self.set_sex(sex)

    def get_name(self):
        return self.__name
    def set_name(self, name):
        self.__name = name

    def get_age(self):
        return self.__age

    def set_age(self, age):
        if age <= 0:
            # print("年龄不合法")
            # self.__age = 1
               #自定义抛出异常
            raise AgeNegative("年龄为负异常age=%d" % (age))
        else:
            self.__age = age

    def get_sex(self):
        return self.__sex
    def set_sex(self, sex):
        if sex == "男" or sex == "女":
            self.__sex = sex
        else:
            print("性别不合法")
            self.__sex = "男"
    def __str__(self):
        return "[Person name=%s, age=%d, sex=%s]" % (self.__name, self.__age, self.__sex)
    __repr__ = __str__
 
#自定义异常类型：
class AgeNegative(Exception):
    def __init__(self, msg):
        super().__init__()
        self.__msg = msg
    def __str__(self):
        return self.__msg
    __repr__ = __str__
    
    
def main():   #test模块
    try:
        p = Person("小沐", -20, "女")
        print(p)
    except AgeNegative as e:
        print(e)
if __name__ == "__main__":
    main()
```

