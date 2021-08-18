---
title: Python 并发编程笔记
date: 2021-06-18 19:37:48
categories: Python
tags: [Python, 程序]
description: Python并发编程(多线程,多进程,多协程)
index_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630201524.png
banner_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630201524.png
sticky: 97
---

# Python 并发编程

## 各并发技术 

![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630200927.png)

## 1. 多线程 Thread（threading）

> - 优点：**相比进程，更轻量级、占用资源少.**
>
> - 缺点：
>   - 相比进程：多线程只能**并发执行**，**不能利用多CPU（GIL）**
>   - 相比协程：启动数目**有限制**，占用内存资源，**有线程切换开销**
>
> - 适用于：**IO密集型计算**、同时运行的任务数目要求不多

### 多进程普通写法

```python
import threading
threads = [] # 线程列表

# 为每个url创建一个线程对象并加入线程列表里
# target: 函数名 args: 函数参数(元组格式)
for url in urls:
    threads.append(
        threading.Thread(target=craw, args=(url,))
    )

# 遍历线程列表启动线程
for thread in threads:
    thread.start()

# 等待线程结束再执行主线程
for thread in threads:
    thread.join()
```

### 利用线程池技术

> 好处:新建线程系统需要**分配资源**、终止线程系统需要回收资源,线程池可以**重用线程**，则可以**减去新建/终止的开销**

- 原理

  ![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630204323.png)

  ![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630204353.png)

- 代码实现

  1. **map方式提交**
  
  > `map` 的结果和入参是顺序对应的,且map传入函数参数时要传入**参数列表**

  > [Python Zip()函数](https://www.runoob.com/python3/python3-func-zip.html)
  >
  > > **zip()** 函数用于将**可迭代的对象作为参数**，将对象中**对应的元素打包成一个个元组**，然后返回由这些**元组组成的对象**，这样做的好处是**节约了不少的内存**。
  > >
  > > 我们可以使用 `list()` 转换来输出列表。
  > >
  > > **元素个数与最短的列表一致**.
  > >
  > > 如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同，利用 ***** 号操作符，可以将元组解压为列表。
  >
  > !["形象"](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630211154.png)
  
  ```python
  from concurrent.futures import ThreadPoolExecutor
  # 可以设置线程数
  with ThreadPoolExecutor(max_workers=10) as pool:
      # map 向线程池提交任务,传入方法名,以及参数列表
      htmls = pool.map(craw, urls)
      # htmls urls(列表) 都是可迭代对象,可以用 zip 函数将他们打包成一个元组
      htmls = list(zip(urls, htmls))
      # 结果和入参是顺序对应的
      for url, html in htmls:
          print(url, len(html))
  ```
  
  2. **submit 方式提交**
  
  > **future模式，更强大**,注意如果用`as_completed`顺序是不定的
  
  > [Python3 字典 items方法](https://www.runoob.com/python3/python3-att-dictionary-items.html)
  >
  > > Python 字典 items() 方法以列表**返回视图对象**，是一个**可遍历的 key/value 对**。
  >
  > 将: `{'Name': 'Runoob', 'Age': 7}` 变为: `[('Age', 7), ('Name', 'Runoob')]`
  
  - **方法一**
  
    ```python
    from concurrent.futures import ThreadPoolExecutor,as_completed
    with ThreadPoolExecutor() as pool:
        # 利用线程列表启动
        futures=[
            pool.submit(craw,url)
            for url in urls
        ]
    
        # 通过遍历线程列表取出结果(要等所有结果运行完才有)
        for future in futures:
            print(future.result())
            
        # as_completed 不需要等待所有结果运行完才输出结果
        # 一旦有结果运行完就会输出
        for future in as_completed(futures):
            print(future.result())
    ```
  
  - **方法二**
  
    > **适合需要与某一个量一一对应建立联系**
  
    ```python
    from concurrent.futures import ThreadPoolExecutor
    with ThreadPoolExecutor() as pool:
        futures = {}  # 字典,便于一一对应
        # htmls格式: ([],[],[],..)
        # 遍历元组的每个列表,把列表中的两个参数赋值给 url,html
        for url, html in htmls:
            # submwit 提交,传入方法名和单个参数
            future = pool.submit(parse, html)
            # 使用字典 将 future 对象与链接一一对应
            futures[future] = url
        
        # 将转化为元组的字典遍历出来
        # future->线程对象 url->线程对应的url
        for future,url in futures.items():
            # 线程对象通过 result 取出运行结果
            print(url,future.result())
    ```

### 生产者消费者模型  ( Producer, Consumer)

1. 多组件的`Pipeline`技术架构

   > 复杂的事情一般都不会一下子做完，而是会分**很多中间步骤一步步完成**

   ![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210703065209.png)

   ![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210703065255.png)

2. **多线程数据通信的`queue.Queue`**

   > `queue.Queue`可以用于**多线程之间**的、**线程安全**的数据通信
   >
   > **多个线程** 可以 **同时** 读取 **同一个队列** 

   ```python
   # 1、导入类库
   import queue
   # 2、创建Queue
   q = queue.Queue()
   # 3、添加元素
   q.put(item)
   # 4、获取元素
   item = q.get()
   # 5、查询状态
   # 查看元素的多少
   q.qsize()
   # 判断是否为空
   q.empty()
   # 判断是否已满
   q.full()
   ```

3. 实例:

   ```python
   import threading
   import queue
   import random,time
   
   # 传入 url 队列 和 Html 队列
   def do_craw(url_queue: queue.Queue, html_queue: queue.Queue):
       while True:
           # 从 Url 队列里获取一个链接
           url = url_queue.get()
           html = blog_spider.craw(url)
           # 将获取到的 html 内容放入 html 队列
           html_queue.put(html)
           # current_thead: 当前线程 .name: 获取线程的名字
           print(threading.current_thread().name, f"craw {url}",
                 "url_queue.size=", url_queue.qsize())
           time.sleep(random.randint(1, 2))
   
   
   def do_parse(html_queue: queue.Queue, fout):
       while True:
           html = html_queue.get()
           results = blog_spider.parse(html)
           for result in results:
               fout.write(str(result) + "\n")
           print(threading.current_thread().name, f"results.size", len(results),
                 "html_queue.size=", html_queue.qsize())
           time.sleep(random.randint(1, 2))
   
   
   if __name__ == "__main__":
       url_queue = queue.Queue() # 链接队列
       html_queue = queue.Queue() # Html 文件队列
       # 将所有链接添加到链接队列里面
       for url in urls:
           url_queue.put(url)
   
       # 创建链接访问线程
       for idx in range(3):
           t = threading.Thread(target=do_craw, args=(url_queue, html_queue),
                                name=f"craw{idx}")
           t.start()
   
       # 创建 HTML 解析线程
       fout = open("02.data.txt", "w")
       for idx in range(2):
           # name: 设置线程的名字.
           t = threading.Thread(target=do_parse, args=(html_queue, fout),
                                name=f"parse{idx}")
           t.start()
   
   ```

   

## 2. 多进程 (MultiProcess)

### 全局解释器锁 (Python的大缺点)

1. **任何时刻仅有一个线程**在执行。
2. 在**多核心处理器上**，使用 GIL 的解释器也只允许同一时间执行一个线程
3. GIL目的：为了解决多线程之间**数据完整性**和状态同步问题
4. GIL带来的**问题**
- 即使使用了多线程，同一时刻也只有**单个线程使用CPU**，导致多核CPU的浪费
- GIL只会对**CPU密集型**的程序产生影响
- 如果程序主要是在做**I/O操作**，比如**处理网络连接**，那么**多线程技术**常常是一个明智的选择
5. **规避GIL**的方法
- 规避方法2： 使用`multiprocessing`多进程，对CPU密集型计算，单独启动子进程解释器去执行
- 规避方法2： ﻿将计算密集型的任务转移到**C语言**中，因为C语言比Python快得多，注意要在C语言中自己释放GIL

![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210701000441.png)

> ### 多进程适用于 CPU密集型计算
>
> 多进程的API与 **多线程的实现十分类似**

| 语法条目             |                            多线程                            |                            多进程                            |
| :------------------- | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 引入模块             |                 from threading import Thread                 |             from multiprocessing import Process              |
| 新建  启动  等待结束 |   t=Thread(target=func, args=(100, ))  t.start()  t.join()   |  p = Process(target=f, args=('bob',))  p.start()  p.join()   |
| 数据通信             | import queue  q = queue.Queue()  q.put(item)  item = q.get() | from multiprocessing import Queue  q = Queue()  q.put([42, None, 'hello'])  item = q.get() |
| 线程安全加锁         | from threading import Lock  lock = Lock()  with lock:      # do something | from multiprocessing import Lock  lock = Lock()  with lock:      # do something |
| 池化技术             | from concurrent.futures import  ThreadPoolExecutor    with ThreadPoolExecutor() as executor:      # 方法1      results = executor.map(func, [1,2,3])        # 方法2      future = executor.submit(func, 1)      result = future.result() | from concurrent.futures import ProcessPoolExecutor    with ProcessPoolExecutor() as executor:      # 方法1      results = executor.map(func, [1,2,3])        # 方法2      future = executor.submit(func, 1)      result = future.result() |

### **利用进程池技术实现多进程**

```python
from concurrent.futures import ProcessPoolExecutor
import time

def test(lover):
    print("我喜欢:", lover)
    time.sleep(1)

# 任何池化技术都需要写程序入口
if __name__ == "__main__":
    # 喜欢的人列表
    lovers = ["颖怡", "菲菲", "詹天佑"]
    start_time = time.time()
    # 1. 使用 map 方式提交
    with ProcessPoolExecutor(max_workers=10) as pool:
        # map 方式提交进程池需要传入函数名和多参数元组
        pool.map(test, lovers)

    # 2. 使用 submit 方式提交
    with ProcessPoolExecutor() as pool:
        # 利用进程列表启动
        futures=[
            pool.submit(test,lover)
            for lover in lovers
        ]

    end_time = time.time()
    print("cost Time:", end_time-start_time)
```

## 3. 并发锁 (Concurrent Lock)

> **线程安全**指某个函数、函数库在**多线程环境**中被调用时，能够正确地处理多个线程之间的**共享变量**，使程序功能正确完成。
>
> 由于**线程的执行随时会发生切换**，就造成了不可预料的结果，出现**线程不安全**(简单来说就是程序串行)

![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210702130844.png)

```python
import threading
# 实例化锁对象
lock = threading.Lock()
# 上锁
lock.locked()
# 释放锁
lock.release()
```

### 用法1:  `try-finally` 模式

```python
import threading
lock = threading.Lock()
lock.acquire()
try:
    # do something
finally:
    lock.release()
```

### 用法2: `with` 模式  **(推荐这种方式)**

```python
import threading
lock = threading.Lock()
with lock:
    # do something
```

### 什么时候需要上锁

>  **多人读,不需要|一读一写要加|多人写要加**

只是操作**共享变量**部分的代码要**上锁**,而不是多线程中所有代码都要上锁,并**不影响多线程**的执行效果.

![image-20210702143100587](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210702143102.png)

## 4. 异步协程

### 原理

> 同步：执行 IO 操作时，**必须等待执行完成才得到返回结果**。
> 异步：执行 IO 操作时，**不必等待执行就能得到返回结果**。

![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210702150837.jpeg)

![image-20210702143832079](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210702143833.png)

> 注意：要用在**异步IO编程中**,依赖的**库必须支持异步IO特性.**
>
> 爬虫引用中：`requests` 不支持异步|需要用 `aiohttp`

### 补充

#### 1. 什么是协程、异步

举个例子：假设有1个洗衣房，里面有10台洗衣机，有一个洗衣工在负责这10台洗衣机。那么**洗衣房就相当于1个进程**，洗衣工就相当**1个线程**。如果有10个洗衣工，就相当于**10个线程**，**1个进程是可以开多线程的**。这就是多线程！

那么协程呢？先不急。大家都知道，洗衣机洗衣服是**需要等待时间**的，如果10个洗衣工，1人负责1台洗衣机，这样效率肯定会提高，但是不觉得**浪费资源**吗？明明1 个人能做的事，却要10个人来做。只是把衣服放进去，打开开关，**就没事做**了，等衣服洗好再拿出来就可以了。就算很多人来洗衣服，**1个人也足以应付了**，开好第一台洗衣机，在等待的时候去**开第二台洗衣机，再开第三台**，……直到有衣服洗好了，就回来把衣服取出来，接着**再取另一台**的（哪台洗好先就取哪台，所以**协程是无序的**）。这就是计算机的协程！洗衣机就是**执行的方法**。

当你程序中方法需要等待时间的话，就可以用**协程**，**效率高，消耗资源少。**

**洗衣房 ==> 进程 | 洗衣工 ==> 线程 | 洗衣机 ==> 方法(函数)**

#### 2. `async` \ `await`的使用

正常的函数在执行时是**不会中断**的，所以你要写一个**能够中断**的函数，就需要添加`async`关键字。

`async` 用来声明一个函数为异步函数，异步函数的特点是**能在函数执行过程中挂起，去执行其他异步函数**，等到**挂起条件**（假设挂起条件是sleep(5)）消失后，也就是5秒到了**再回来执行**。

`await` 用来用来声明**程序挂起**，比如异步程序执行到**某一步时需要等待的时间很长，就将此挂起**，去**执行其他的异步程序**。`await` 后面只能跟 **异步函数** 或有`__await__`属性的对象，因为**异步程序与一般程序不同**。

假设有两个异步函数`async a，async b`，a中的某一步有`await`，当程序碰到关键字`await b()`后，异步程序挂起后去执行另一个异步b程序，就是从**函数内部跳出去执行其他函数**，**当挂起条件消失后，不管b是否执行完，要马上从b程序中跳出来**，**回到原程序执行原来**的操作。如果await后面跟的**b函数不是异步函数**，那么操作就**只能等b执行完再返回**，无法在**b执行的过程中返回**。如果要在**b执行完**才返回，也就**不需要用await关键字了**，直接**调用b函数**就行。所以这就需要`await`后面跟的是**异步函数**了。在一个异步函数中，可以**不止一次挂起**，也就是可以用多个`await`。



### **基本实现:**

> - 1. 在普通的函数前面加 **async** 关键字；
> - 2. `await` 表示在这个地方**等待子函数执行完成**，再往下执行。（在并发操作中，把程序控制权交给主程序，让他分配其他协程执行。） `await` 只能在带有 `async` 关键字的函数中运行。

```python
# 异步io简单实现
import asyncio
import time
# 获取超级事件循环
# event:事件 loop:环
loop = asyncio.get_event_loop()
urls = [
    f"https://skyxinye.xyz/#{page}"
    for page in range(1, 99)
]

# 定义协程函数
async def myfunc(url):
    await time.sleep(2) # 这样写不行,因为 time.sleep 不是异步函数,不能中途跳出执行另一个协程.
    await asyncio.sleep(2) # 用协程函数.
# 创建 task 列表
# create:创造
tasks = [
    # 在超级循环中创造多个任务,并设置等待任务完成.(如果协程函数中没有同步操作的话就不用设置)
    # 任务没有运行
    loop.create_task(asyncio.wait(myfunc(url)))
    for url in urls
]
# 执行异步事件列表,运行直到完成,如果是协程列表就要设置等待事件完成.
loop.run_until_complete(asyncio.wait(tasks))
```

#### 利用 `aiohttp` 模块发送网络请求

```python
# 异步模块 aiohttp 使用
import asyncio
import aiohttp


# Http请求 协程函数
# GET
async def async_get(url):
    """
    aiohttp:发送http请求
    1.创建一个ClientSession对象
    2.通过ClientSession对象去发送请求（get, post, delete等）
    3.await 异步等待返回结果
    """
    print("Get Url:", url)
    async with aiohttp.ClientSession() as se:
        async with se.get(url) as resp:
            result = await resp.text()
            print(len(result))

urls = [
    f"https://skyxinye.xyz/#{page}"
    for page in range(1, 99999)
]

# 超级循环
loop = asyncio.get_event_loop()
# 事件列表
tasks=[
    loop.create_task(async_get(url))
    for url in urls 
]
# 用 wait 方法遍历事件列表.协程列表需要设置等待运行完成
loop.run_until_complete(asyncio.wait(tasks))

"""
    异步协程不用事件列表的另一种实现方法
    aiohttp:发送POST请求
"""
# POST
async def main():
    data = {'key1': 'value1', 'key2': 'value2'}
    url = 'http://httpbin.org/post'
    async with aiohttp.ClientSession() as session:
        async with session.post(url, data=data) as res:
            print(res.status)
            print(await res.text())
loop = asyncio.get_event_loop()
# 如果是单个协程就不需要设置 asyncio.wait()
task = loop.create_task(main())
loop.run_until_complete(task)
```

#### 利用 `aiofile` 异步操作文件

```python
import aiofiles
import asyncio

async def write():
    async with aiofiles.open("test.txt","w",encoding="utf8") as fp:
        await fp.write("异步写入文件")
        print("文件写入成功")

async def read():
    async with aiofiles.open("test.txt","r",encoding="utf8") as fp:
        content = await fp.read()
        print(content)

async def read2_demo():
    async with aiofiles.open("text.txt","r",encoding="utf-8") as fp:
        # 读取每行
        async for line in fp:
            print(line)

if __name__=="__main__":
    asyncio.run(write())
```

