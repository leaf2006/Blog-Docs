---
title: Python asyncio 异步协程百万并发
date: 2021-08-18 19:37:48
categories: Python
tags: [Python, 程序]
description: Python 异步协程百万并发
index_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630201524.png
banner_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210630201524.png
sticky: 103
excerpt: Python 异步百万并发全文最详细笔记！！
---
# Python asyncio 异步协程百万并发

## **协程（coroutine）**

> 本质就是一个**函数**

## **事件循环**——(event_loop)

**协程函数**，不是像普通函数那样直接调用运行的，必须**添加到事件循环**中，然后由**事件循环**去运行，单独运行协程函数是不会有结果的，看一个简单的例子：

```python
import time
import asyncio
async def say_after_time(delay,what):
        await asyncio.sleep(delay)
        print(what)
 
async def main():
        print(f"开始时间为： {time.time()}")
        await say_after_time(1,"hello")
        await say_after_time(2,"world")
        print(f"结束时间为： {time.time()}")
 
loop=asyncio.get_event_loop()    #创建事件循环对象
# loop=asyncio.new_event_loop()   #与上面等价，创建新的事件循环
loop.run_until_complete(main())  #通过事件循环对象运行协程函数
loop.close()
```

在python3.6版本中，如果我们单独像执行普通函数那样执行一个协程函数，只会返回一个coroutine对象（python3.7）如下所示：

```python
>>> main()
<coroutine object main at 0x1053bb7c8>
```

### 获取事件循环对象的几种方式

```
loop = asyncio.get_event_loop()
```
它是python3.7中新添加的，**获得一个事件循环**，如果当前线程**还没有事件循环**，则**创建一个新的事件循环loop；**

### 通过事件循环运行协程函数的两种方式

1. 创建事件循环对象loop，即`asyncio.get_event_loop()`，**通过事件循环运行协程函数**.

2. 直接通过`asyncio.run(function_name)`运行**协程函数。**

   > **但是需要注意的是，首先run函数是python3.7版本新添加的，前面的版本是没有的；**
   

	其次，这个run函数总是**会创建一个新的事件循环并在run结束之后关闭事件循环**，所以，如果在**同一个线程**中已经有了一个事件循环，则**不能再使用这个函数**了，因为**同一个线程不能有两个事件循环**，而且这个run函数**不能同时运行两次**，因为他已经创建一个了。即**同一个线程中是不允许有多个事件循环loop的**。

## Task任务

### 创建任务（两种方法）

1. `task = asyncio.create_task(coro())`

   > **这是3.7版本新添加的**,**可以传协程函数**

2. `task = asyncio.ensure_future(coro())`

3. 也可以：

   ```python
   loop.create_future()
   loop.create_task(coro)
   ```

### 获取某一个任务的方法

1. `task=asyncio.current_task(loop=None)`

   > 返回在某一个指定的loop中，**当前正在运行的任务**，**如果没有任务正在运行，则返回None**；
   > 如果loop为None，**则默认为在当前的事件循环中获取**.

2. `asyncio.all_tasks(loop=None)`

   > 返回某一个**loop中还没有结束的任务**

## 异步函数的结果获取

对于异步编程、异步函数而言，最重要的就是**异步函数调用结束之后，获取异步函数的返回值**，我们可以有以下几种方式**来获取函数的返回值**，第一是直接通过`Task.result()`来获取；第二种是**绑定一个回调函数**来获取，**即函数执行完毕后调用一个函数来获取异步函数的返回值。**

1. 直接通过`result`来获取.

   ```python
   import asyncio
   import time
    
    
   async def hello1(a,b):
       print("Hello world 01 begin")
       await asyncio.sleep(3)  #模拟耗时任务3秒
       print("Hello again 01 end")
       return a+b
    
   coroutine=hello1(10,5)
   loop = asyncio.get_event_loop()                #第一步：创建事件循环
   task = asyncio.ensure_future(coroutine)         #第二步:将多个协程函数包装成任务列表
   loop.run_until_complete(task)                  #第三步：通过事件循环运行
   print('-------------------------------------')
   print(task.result())
   loop.close() 
    
   '''运行结果为
   Hello world 01 begin
   Hello again 01 end
   -------------------------------------
   15
   '''
   ```
   
2. 通过定义**回调函数**来获取
	```python
	import asyncio
	import time
    
   async def hello1(a,b):
       print("Hello world 01 begin")
       await asyncio.sleep(3)  #模拟耗时任务3秒
       print("Hello again 01 end")
	    return a+b
    
	def callback(future):   #定义的回调函数,需要传future参数
	    print(future.result())
	 
	loop = asyncio.get_event_loop()                #第一步：创建事件循环
	task=asyncio.ensure_future(hello1(10,5))       #第二步:将多个协程函数包装成任务
	task.add_done_callback(callback)                      #并被任务绑定一个回调函数，默认传入结果参数
	 
	loop.run_until_complete(task)                  #第三步：通过事件循环运行
	loop.close()                                   #第四步：关闭事件循环
	 
	 
	'''运行结果为：
	Hello world 01 begin
	Hello again 01 end
	15
	'''
	```
	
	> **注意：**所谓的**回调函数**，就是指协程函数coroutine**执行结束时候会调用回调函数**。并通过**参数future获取协程执行的结果**。我们创建的**task和回调里的future对象**，实际上是**同一个对象**，因为task是future的子类。

## asyncio异步编程的基本模板

### **第一步：构造事件循环**

1. `loop = asyncio.get_running_loop()`

   > 返回（获取）在当前线程中**正在运行的事件循环**，如果没有正在运行的事件循环，则会显示错误；它是**python3.7中新添加的**

2. `loop = asyncio.get_event_loop() `

   > **获得一个事件循环**，如果当前线程还没有事件循环，则**创建一个新的事件循环loop**；

3. `loop=asyncio.set_event_loop(thread) `

   > 设置一个事件循环**为当前线程的事件循环**；

4. `loop=asyncio.new_event_loop()`

   > **创建一个新的事件循环**

### **第二步：将一个或者是多个协程函数包装成任务Task**

1. `task = asyncio.create_task(coro(参数列表))` 

   > **这是3.7版本新添加的**

2. `task = asyncio.ensure_future(coro(参数列表))`

> 需要注意的是，在使用`Task.result()`获取**协程函数结果**的时候，使用`asyncio.create_task()`却会显示错，但是使用`asyncio.ensure_future`却正确

### **第三步：通过事件循环运行**

1. `loop.run_until_complete(asyncio.wait(tasks))`

   > 通过`asyncio.wait()`**整合多个task**

2. `loop.run_until_complete(asyncio.gather(tasks))`

   > 通过`asyncio.gather()`**整合多个task**

3. `loop.run_until_complete(task_1)  `

   > **单个任务则不需要整合**

4. ~~loop.run_forever()~~

   > ~~但是这个方法在新版本已经取消，不再推荐使用，因为使用起来不简洁~~

#### 使用`gather`和`wait`整合task注册多个服务

1. #### **参数形式不一样**
   
    **gather**的参数为 coroutines_or_futures,即如这种形式：
    
    ```python
    tasks = asyncio.gather(*[task1,task2,task3])
    tasks = asyncio.gather(task1,task2,task3)
    loop.run_until_complete(tasks)
   ```
   **wait**的参数为**列表或者集合**的形式，如下:
   
    ```python
    tasks = asyncio.wait([task1,task2,task3])
    loop.run_until_complete(tasks)
    ```
   
2. #### **返回的值不一样**
   
   **gather返回的是每一个任务运行的结果**：
   
   > ###### 要以传入一个列表可变参数
   
   **可变参数允许在调用参数的时候传入多个参数,这些参数在调用时被自动组装为一个tuple**
   
   `results = await asyncio.gather(*[tasks])` 
   
   `results = await asyncio.gather(task1,task2,task3)` 
   
   **wait返回dones是已经完成的任务，pending是未完成的任务，都是集合类型**：
   `done, pending = yield from asyncio.wait(fs)`
> 简单来说：**async.wait会返回两个值:done和pending**，done为已完成的协程Task，pending为超时未完成的协程Task，**需通过future.result调用Task的result。**
   >
   > 而`async.gather`返回的是**已完成Task的result**。

###    **第四步：关闭事件循环**

```python
loop.close()
# 以上示例都没有调用 loop.close，好像也没有什么问题。所以到底要不要调 loop.close 呢？
```

## 注意

### 协程阻塞问题

**异步方式依然会有阻塞的**，当我们定义的很多个异步方法**彼此之间有一来**的时候，比如，我必须要等到函数1执行完毕，**函数2需要用到函数1的返回值**，就会造成**阻塞**，这也是异步编程的难点之一，如何合理配置这些资源，尽量**减少函数之间的明确依赖**，这是很重要的。

**结论**：在有**很多个异步方式**的时候，一定要尽量避免这种**异步函数的直接调用**，这和同步是没什么区别的，一定要**通过事件循环loop**，**“让事件循环在各个异步函数之间不停游走”**，这样才不会造成阻塞。

## 代码片段

### 使用gather同时注册多个任务，实现并发

```python
import asyncio
import time
async def hello1(a,b):
    print("Hello world 01 begin")
    await asyncio.sleep(3)  #模拟耗时任务3秒
    print("Hello again 01 end")
    return a+b
 
async def hello2(a,b):
    print("Hello world 02 begin")
    await asyncio.sleep(2)   #模拟耗时任务2秒
    print("Hello again 02 end")
    return a-b
 
async def hello3(a,b):
    print("Hello world 03 begin")
    await asyncio.sleep(4)   #模拟耗时任务4秒
    print("Hello again 03 end")
    return a*b
 
async def main():  #封装多任务的入口函数
    # 用列表表达式创建任务
    tasks = [
        asyncio.ensure_future(hello1(10,5))
        for i in range(10)
    ]
    results = await asyncio.gather(tasks)   
    for result in results:    #通过迭代获取函数的结果，每一个元素就是相对应的任务的返回值，顺序都没变
        print(result)
 
 
loop = asyncio.get_event_loop()               
loop.run_until_complete(main())
loop.close()
```

### 异步+多线程

```python
import asyncio 
import asyncio,time,threading
 
#需要执行的耗时异步任务
async def func(num):
    print(f'准备调用func,大约耗时{num}')
    await asyncio.sleep(num)
    print(f'耗时{num}之后,func函数运行结束')
 
#定义一个专门创建事件循环loop的函数，在另一个线程中启动它
def start_loop(loop):
    asyncio.set_event_loop(loop)
    loop.run_forever()
 
#定义一个main函数
def main():
    coroutine1 = func(3)
    coroutine2 = func(2)
    coroutine3 = func(1)
 
    new_loop = asyncio.new_event_loop()                        #在当前线程下创建时间循环，（未启用），在start_loop里面启动它
    t = threading.Thread(target=start_loop,args=(new_loop,))   #通过当前线程开启新的线程去启动事件循环
    t.start()
 
    asyncio.run_coroutine_threadsafe(coroutine1,new_loop)  #这几个是关键，代表在新线程中事件循环不断“游走”执行
    asyncio.run_coroutine_threadsafe(coroutine2,new_loop)
    asyncio.run_coroutine_threadsafe(coroutine3,new_loop)
 
    for i in "iloveu":
        print(str(i)+"    ")
 
if __name__ == "__main__":
    main()
 
'''运行结果为：
i    准备调用func,大约耗时3
l    准备调用func,大约耗时2
o    准备调用func,大约耗时1
v
e
u
耗时1之后,func函数运行结束
耗时2之后,func函数运行结束
耗时3之后,func函数运行结束
'''
```

### httpx aiohttp 之异步请求

- **aiohttp实现**

```python
import aiohttp
import asyncio
 
async def main():
    async with aiohttp.ClientSession() as client:
         async with client.get('http://httpbin.org/get') as resp:
              assert resp.status == 200
              html= await resp.text()
              print(html)
```

- **httpx实现**

```python
async with httpx.AsyncClient() as client:
    resp = await client.get('http://httpbin.org/get')
    assert resp.status_code == 200
    html = resp.text
```

感觉总体上比较`aiohttp`写起来舒服多了**，少写很多异步代码。**

> 之前使用 aiohttp 中的 resp.status 来获取状态码的时候写了status_code，应该是使用 requests 习惯了吧，这下好了使用 httpx 不用担心这个写错的问题了。
