## 一、时间模块中的time模块

time模块提供的是跟时间相关的各种功能

**时间戳**：

​	表示从1970年1月1日凌晨开始按秒计算的一个偏移量

```python
#获取当地时间的时间元组
local_time = time.localtime()
#在时间元组中获得年
year = local_time[0]

#获取当前时间对应的时间戳
seconds = time.time()

#将时间戳转化为时间元组
local_time = time.localtime(seconds)

#返回一个可读的时间
asc_time = time.asctime()
print(asc_time)

#休眠
sleep = time.sleep(1)#休眠多少秒

'''
时间格式化
年 %y(减去1900年之后的年份) %Y(完成的年份) 一般使用Y
月 %m
日 %d
时 %H(24小时制)   %I(12小时制)
分 %M
秒 %S
标示上下午   %p
星期 %a
时区 %z
'''
	#设定一个格式化字符串
	format_str ="%Y-%m-%d %p %I:%M:%S"
		#将当前时间元组进行格式化
		format_time = time.strftime(format_str,local_time)
			print(format_time)

#逆向操作 由格式化时间 获得对应的时间元组
time_tuple = time.strptime(format_time,format_str)

#有一个字符串时间--->根据它 获取三天后的字符串时间
2018-02-11 06:12:28
--->"2018-02-14 06:12:28"
将指定的时间--->时间元祖--->获得秒数--->加三天时间--->时间元组--->格式化时间

#根据时间元组获得秒数
seconds = time.mktime(local_time)
time_tuple1 = time.strptime("2018-02-11 06:12:28","%Y -%m-%d %H:%M:%S)
time _seconds = time.mktime((time_tuple1))
res_seconds = time.localtime(res_time)
res_time = time.strftime("%Y -%m-%d %H:%M:%S,res_tuple)

```

## 二、时间模块 - datetime

```python
import datetime
#获取当前时间
current_time = datetime.datetime.now()

#只获取年月日
current_day = datetime.date.today()

#获取明天的日期
tomorrow = datetime.date.today() + datetime.timedelta(days = 1)

#获取昨天的几个小时后
yesterday= datetime.datetime.today() + datetime.timedelta(days = -1,hours = 2)

#自定义时间
cus_time = datetime.datetime(2018,5,20,12,24,30)
#与当前时间的时间差
time_diff = cus_time - current_time
#获取相差的天数
days = time_diff.days
print(days)

#获取差的总秒数
seconds = time_diff.total_seconds()

#格式化的操作 和time使用的方法差不多
cus_time2 = datetime.datetime(2018,5,20,12,24,30)
format_str = cus_time2.strftime("%Y-%m-%d %H:%M:%S")
#逆向
time_cus = datetime.datetime.strptime(format_str,"%Y-%m-%d %H:%M:%S")
print(time_cus)

#将datetime的时间转化为时间元组(可以通过脚标拿到想要的内容)
tuple_time = cus_time2.timetuple()
```

## 三、时间模块 -calendar

```python
import calendar
#获取指定年的日历
res = calendar.calendar(2018,w=5,c=5)#w是

#获取指定月的日历
month = calendar.month(2018,4,w=3)

#判定是否是闰年
is_leap = calendar.isleap(2020)
print(is_leap)

#获取两个年之间 闰年的个数
num = calendar.leapdays(1900,2018)

#将指定月的星期的嵌套列表
list_list=calendar.monthcalendar(2018,4)

#获取月的信息
info = calendar.monthrange(2018,4)

#设置指定日期，返回对应的星期数
week = calendar.weekday(2018,4,20)
```

## 四、os - 系统模块

包含了操作系统信息的功能 还可以处理一些文件

```python
import os
#获得操作系统的类型
type = os.name #nt表示的是Windows  posix-->Linux 和 Mac
#获取操作系统的详细信息
info = os.uname()
	windows不支持
#获得操作系统下的所有的环境变量
print(os.environ)

#获取指定变量下的内容
print(os.environ.get("path"))

#相对路径和绝对路径
平级直接拿文件
下一级直接\文件名
上一级文件..\文件名
		上上一级..\..\文件名
#获取当前目录
print(os.curdir) 采用的相对路径

#获取当前程序所在的目录
print(os.getcwd()) #绝对路径

#操作目录
1.获取指定路径下的直接子目录和子文件
all_list = os.listdir(".")<----路径(可以是相对的也可以是绝对的)
#避免路径解析出错 前面加r

#2.创建目录
os.mkdir("test")#是普通文件夹的形式
os.mkdir(绝对路径)#只能创建最后一级目录

#创建多级目录
os.mkdirs("目录")

#删除目录
os.rmdir("目录")#只能删除空目录

os.chdir(os.chdir("/home/newdir"))#将当前目录更改为

#重命名
os.rename("原目录名或者文件名","新目录名或者目录名")
      
#获得文件或者目录的信息
  print(os.stat("目录或者文件名"))

#针对文件的操作
#移除文件
os.remove("文件名")#只能删除文件，不能删除目录
          
#运行dos命令
os.system("echo hello>text.text")
          
#通过dos命令打开其他程序
os.system("notepad")#记事本
          
#通过dos将程序毙掉
os.system("taskkill /f /im notepad.exe")  

#查看目录的绝对路径
print(os.path.abspath("."))    
#路径拼接          
p1 =路径
p2 =目录
res = os.path.join(p1,p2)          
          
#拆分路径
>>> os.path.split('/Users/michael/testdir/file.txt')#会有两项，后一项为 最后一级目录或者文件名
('/Users/michael/testdir', 'file.txt')     
          
os.path.splitext()#拆分路径，获得文件扩展名
>>> os.path.splitext('/path/to/file.txt')
('/path/to/file', '.txt')          
          
#检测指定路径是否是目录          
res = os.path.isdir(p2) 
          
#检测指定路径是否是文件
res = os.path.isfile(p2)
          
#判定指定路径是否存在        
res = os.path.exists("路径") 
          
#获得文件的字节大小          
size = os.path.getsize(p2)          
          
#获得文件的目录  (在哪个目录下)
res = os.path.dirname(p1)   

#返回的是路径最后一个反斜线后的部分  （最后一级路径）       
res = os.path.basename(p1)          
          
#获取文件名
filename = os.path.basename(p3)          
          
递归实现文件遍历（找到指定路径下的所有目录和文件）
def get_files(path):
    #判定一下路径是否存在
    res = os.path.exists(path)
    if not(res): #not(res) == True ---> res === False
        print("指定目录不存在")
        return
    #路径存在的情况下
    #判断传进来的是不是文件
    is_file = os.path.isfile(path)
    if is_file:
        print("该方法的功能是遍历目录 获得目录下的文件")
        return
    #就是一个目录
    #获得该目录下的所有子文件和子目录
    all_files = os.listdir(path)
    #遍历
    for p in all_files:
        #拼接
        join_path = os.path.join(path, p)
        #判断是否是文件
        if os.path.isfile(join_path):
            print(p)
        elif os.path.isdir(join_path):
            get_files(join_path)
          '''
path  = D:\ClassContent\BJ_Python1805\day01

获得直接子路径
    listdir(path)
        子目录: code  
                    获得直接子路径
                    遍历
                notes  
                video
        子文件: test.txt
'''          
               
```

## 五、模拟栈(stack)和队列(queue)

**使用队列来模拟栈的思路**：

​	栈是后进先出，队列是先进先出，队列在python中要使用collections.dequeue()  要用队列表示栈，则在元素入栈时对栈元素的顺序进行调整，将最后元素之前的元素按顺序移动到最后一个元素的后面，使最后一个进入的元素位于队列的前面。

```python
#1.栈(stacks)是一种只能通过访问其一端来实现数据存储与检索的线性数据结构，具有后进先出(last in first out，LIFO)的特征

#2.队列(queue)是一种具有先进先出特征的线性数据结构，元素的增加只能在一端进行，元素的删除只能在另一端进行。能够增加元素的队列一端称为队尾，可以删除元素的队列一端则称为队首。

class MyStack(object):

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.stack = collections.deque([])

    def push(self, x):
        """
        self.stack = []
        Push element x onto stack.
        :type x: int
        :rtype: void
        """
        self.stack.append(x)
        q = self.stack
        for i in range(len(q) - 1):
            q.append(q.popleft()) #一边删除之前的元素，一边添加元素，达到调整队列中元素顺序的目的

    def pop(self):
        """
        Removes the element on top of the stack and returns that element.
        :rtype: int
        """
        return self.stack.popleft()

    def top(self):
        """
        Get the top element.
        :rtype: int
        """
        return self.stack[0]

    def empty(self):
        """
        Returns whether the stack is empty.
        :rtype: bool
        """
        if len(self.stack) == 0:
            return True
        else:
            return False
```

## 六、python内置四种队列

1.LILO队列（Last In Last Out)

```python
from queue import Queue
q=Queue() #创建队列对象
q.put(0)#在队列尾部插入元素
print(q.queue)#查看队列中的所有元素
print(q.get()) #返回并删除队列头部元素
```

2.LIFO队列(Last In First Out)

```python
from queue import LifoQueue 
lifoQueue=LifoQueue()
lifoQueue.put(1)
lifoQueue.get()#返回并删除队列头部元素
```

3.PriorityQueue#优先队列

```python
from queue import PriorityQueue
priorityQueue=PriorityQueue()
priorityQueue.put(3)
print(priorityQueue.queue) #查看优先级队列中的所有元素
priorityQueue.get()#返回并删除优先级最低的元素
```

4.双端队列

```python
from collections import deque
dequeQueue=deque(['Eric','John','Smith'])
print(dequeQueue)
dequeQueue.append('Tom') #在右侧插入新元素
dequeQueue.appendleft('Terry') #在左侧插入新元素
dequeQueue.rotate(2) #循环右移两次
dequeQueue.popleft()#返回并删除队列最左端元素
dequeQueue.pop()#返回并删除队列最右端元素
```

**关于栈的额外补充**：https://blog.csdn.net/haiyu94/article/details/79413992

Input 可以接受一个python表达式作为输入，并将元素结果返回

```python
请输入：[x*5 for x in range(2,10,2)]
你输入的内容是:  [10, 20, 30, 40]
```

