---
title: Flask Web 框架学习笔记
date: 2021-08-18 19:37:48
categories: Python
tags: [Python, 程序]
description: Flask Web 框架全学习笔记(不断更新)
index_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210818152255.png
banner_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210818152255.png
sticky: 103
excerpt: Python 中三大名器的完全总结！个人总结，算是比较详细的了。
---

# Flask Web 框架学习笔记

![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210818152255.png)

## 1. Flask 初始化参数

> **Flask程序实例**在创建的时候，需要**默认传入当前Flask程序所指定的包**(模块).
- import name
  - Flask程序所在的包(模块),传 `__name__` 就可以
  - 其可以决定Flask在访问静态文件时查找的路径
- static_ _url_ _path
  - 静态文件访问路径，可以不传，默认为: /+静态文件目录名
- static_ folder
  -静态文件存储的文件夹，可以不传，默认为 static
- template_ folder
  - 模板文件存储的文件夹，可以不传，默认为 templates

### Flask 基本编写

```python
# 导入Flask类
from flask import Flask
#Flask类接收一个参数 __name__
app = Flask(__name__)

# 装饰器的作用是将路由映射到视图函数index
@app.route('/')
def index() :
	return 'Hello World'

# Flask应用程序实例的run方法启动WEB服务器
if __name__ == '__main__':
	app.run()
```

## 2. Flask 工程配置加载的方式

### 使用方式

Flask将配置信息保存到了`app.config`属性中，该属性可以按照**字典类型**进行操作。

```python3
app.config.get(name) # 读取
app.config[name] # 修改
```

### 设置

- **从配置对象中加载**

  ```python
  from flask import Flask
  class DefaultConfig(oject):
      # 密钥
      SECRET_KET = 'lovehyy2021209classpzezprem'
  app = Flask(__name__)
  app.config.from_object(DefaultConfig) # 从配置对象中加载
  
  @app.route('/') # 装饰器->路由
  def index(): # 视图函数
      print(app.config['SECRET_KEY'])
      return 'hello world'
  ```
  
- **从配置文件中加载**

  在项目目录下新建`setting.py`文件，存放大写常量

  ```python
  SECRET_KET = 'lovehyy2021209classpzezprem'
  ```

  在Flask程序文件中

  ```python
  app = Flask(__name__)
  app.config.from_pyfile('setting.py') # 从配置文件加载
  ```

- **从环境变量中加载**

  > **环境变量(environment variables)** 一般是指在操作系统中用来指定操作系统运行环境的一些参数，如:临时文件夹位置和系统文件夹位置等。环境变量是在操作系统中一个具有特定名字的对象，它包含了一个或者多个应用程序所将使用到的信息。

  **通俗的理解，环境变量就是我们设置在操作系统中，由操作系统代为保存的变量值**:

  在Linux系统中设置和读取环境变量的方式如下:

  ```shell
  export 变量名=变量值 # 设置
  echo $变量名 # 读取
  ```

  **Flask使用环境变量加载配置的本质是通过环境变量值找到配置文件**，再读取配置文件的信息，其使用方式为：
  `app.config.from_ envvar('环境变量名')`

  **环境变量的值为配置文件的绝对路径**先在终端中执行如下命令：

  `export PROJECT_ _SETTING= '~/ setting. py'`

  再运行如下代码

  ```python
  app = Flask(__ name__)
  app.config.from_envvar('PROJECT_ SETTING', silent=True)
  ```
  
- **各配置方式优缺点**
  
  - **app.config.from _object(配置对象)**
    - 继承- ->优点复用
    - 敏感数据暴露缺点
  - **app.config.from_pyfile(配置文件)**
    - 优点 --> 独立文件保护敏感数据
    - 缺点 --> 不能继承文件路径固定不灵活
  - **app.config.from_envvar("环境变量名")**
    - 优点 --> 独立文件保护敏感数据文件路径不固定灵活
    - 缺点 --> 不方便要记得设置环境量
  - **设置环境变量**
    - 终端export
    - pycharm设置

### 利用工厂函数设置

```python
from flask import Flask

class DefaultConfig(object):
    DEBUG = True
    SECRET_KEY = 'lovehyy2021pzez209class'

def create_flask_app(config):
    '''构建flask对象的工厂函数,传入配置对象,返回配置后的app对象'''
    app = Flask(__name__, static_url_path='/static', static_folder='static')
    app.config.from_object(config)
    return app

app = create_flask_app(DefaultConfig)

@app.route('/', methods=["GET"])
def index():
    return "Hello Flask"

if __name__ == '__mian__':
    app.run()
```

## 3. Flask 运行方式

### 直接运行py文件启动

可以指定运行的主机IP地址，端口，是否开启调试模式

```python
app.run(host="0.0.0.0", port=5000, debug=True)
```

关于**DEBUG调试模式**：

1. 程序代码修改后可以自动重启服务器
2. 在服务器出现相关错误的时候可以直接将错误信息返回到前端进行展示

### 新版启动方式

**开发服务器**启动方式：
在1.0版本之后，Flask调整了开发服务器的启动方式，由代码编写app. run()语句调整为**命令启动flask**

终端启动：

```shell
export FLASK_APP=main # 设置启动的python文件
flask run # 在项目文件下执行
```

### 说明

环境变量 **FLASK_APP** 指明flask的启动实例：

- `flask run -h 0.0.0.0 -p 8000`绑定地址端口
- `flask run --help` 获取帮助
- 生产模式与开发模式的控制
  通过`FLASK_ENV`环境变量指明
  - `export FLASK_ ENV=production` 运行在生产模式，未指明则默认为此方式
  - `export FLASK_ ENV=development` 运行在开发模式

## 4. Flask 路由

### 查询路由信息

- 命令行方式

  ```shell
  flask routes
  ```

- 在应用中的`url_ map`属性中保存着整个Flask应用的**路由映射信息**，可以通过读取这个属性获取**路由信息**。

  `print(app.url_map)`
  如果想在程序中遍历路由信息，可以采用如下方式：
  ```python
  for rule in app.url_map.iter_rules():
      # endpoint 视图函数的名字
      # rule 路由的路径
  	print( 'name={} path={}'. format(rule.endpoint, rule.rule))

- 搭建一个**返回所有路由信息**的json接口

  ```python
  @app.route('/', methods=["GET"])
  def index():
      # for rule in app.url_map.iter_rules():
      #     print(f"name={rule.endpoint};path={rule.rule}")
      rules_iterator = app.url_map.iter_rules()
      # 字典推导式
      dic = {
          rule.endpoint: rule.rule
          for rule in rules_iterator
      }
      # 利用jsonify返回json数据
      return jsonify(dic)
  ```


## 5. 路由 options 限定请求方式 methods

### 请求方式

- **GET**(自带)

- **OPTIONS**(自带)  -> 简化版的GET请求用于询问服务器接口信息的
  比如接口允许的请求方式允许的

  > CORS跨域:
  > www.meiduo.site -> api.meiduo.site/users/1
  > options api.meiduo.site/uses/1

- **HEAD**(自带) -> 简化版的GET请求

  > 只返回GET请求处理时的响应头，不返回响应体。

### 指定接口请求方式

```python
# 利用修饰器中的methods返回设定请求接口方式
# 定义视图
@app.route('/', methods=["POST"])
def index():
    return 'POST API'
```

## 6. 蓝图

### 引入

在一个**Flask应用项目**中，如果**业务视图**过多，可以用某种方式划分出的**业务单元单独维护**，将每个单元用到的**视图、静态文件、模板文件**等独立分开。

例如从**业务角度**上，可将整个应用划分为**用户模块单元、商品模块单元、订单模块单元**，分别开发这些不同单元，并最终整合到一个项目应用中。

### 蓝图

在Flask中，使用**蓝图Blueprint**来分模块组织管理。

蓝图实际可以理解为是**一个存储一组视图方法的容器对象**，其具有如下特点:

- 一个应用可以具有**多个Blueprint**
- 可以将一个**Blueprint注册到**任何一个未使用的**URL**下，比如“/user”、 “/godds"。
- **Blueprint**可以单独具有**自己的模板、静态文件**或者其它的通用操作方法，它并不是必须要实现应用的视图和函数的
- 在一个**应用初始化**时，就应该要**注册需要使用的Blueprint**。★
- 但是一个**Blueprint**并不是一个完整的应用，它**不能独立于应用运行**，而必须要**注册到某一个应用中**。★

### 定义蓝图

1. 定义

   ```python
   user_bp = Blueprint('user', __name__)
   ```

2. 在这个**蓝图对象上进行操作**,**注册路由**,**指定静态文件夹**,**注册模版过滤器**

   ```python
   @user_bp.route('/')
   def user_profile():
       return 'user_profile'
   ```

3. 在应用对象中注册这个蓝图对象

   ```python
   app.register_blueprint(user_bp)
   ```

### 单文件蓝图

可以将创建蓝图对象与定义视图函数放在一个文件中。

###  目录（包）蓝图

对于一个打算包含**多个文件的蓝图**，通常将**创建蓝图对象**放到Python包的`__init__.py` 文件中

```
--------- project # 工程目录
	|---- main.py # 启动文件
	|---- user # 用户蓝图
	|	|---- __init__.py # 此处创建蓝图对象
	|	|---- views.py
```

![蓝图包目录实例](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210725224412.png)

#### 循环引用问题

![循环引用问题](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210725224744.png)

> 所以导入视图函数要放在 `__init__.py` 文件的最后

### 蓝图内部静态文件

和**应用对象**不同，蓝图对象创建时**不会默认注册静态目录的路由**。需要我们在创建时指定`static_folder`参数。

下面的示例将蓝图所在目录下的`static_admin`目录设置为静态目录
```python
admin = Blueprint ("admin",__ name__ , static_folder='static_admin')
app.register_blueprint (admin, url_prefix='/admin' )
```

现在就可以使用`/admin/static_admin/<filename>`访问`static_ admin`目录下的静态文件了。
也可通过`static_url_path`改变访问路径

```python
admin = Blueprint("admin",__name__, static_folder='static_admin' ,static_url_path='/lib')
app.register_blueprint(admin, url_prefix='/admin')
```

## 7. 处理请求

> 在视图编写中需要读取客户端请求携带的数据时，如何才能正确的**取出数据**呢?
> **请求携带的数据**可能出现在HTTP报文中的不同位置，需要**使用不同的方法来获取参数**。