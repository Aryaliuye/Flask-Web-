下述命令从GitHub下载示例代码，并把程序文件夹切换到“1a”版本，即程序的初始版本：

$ git clone https://github.com/miguelgrinberg/flasky.git
$ cd flasky
$ git checkout 1a

查看git版本:
(flask_web_read) tarena@tarena:~/flask_web_read$ git --version
git version 2.17.1


检查虚拟环境是否安装:
输入以下命令可以检查系统是否安装了virtualenv
$ virtualenv --version

配置虚拟环境
tarena@tarena:~/flasky$ virtualenv venv
运行虚拟环境
tarena@tarena:~/flasky$ source venv/bin/activate
(venv) tarena@tarena:~/flasky$ 


接下来在虚拟环境下完成操作

执行下述命令可在虚拟环境中安装Flask：
(venv) $ pip install flask

要想验证Flask是否正确安装，你可以启动Python解释器，尝试导入Flask：

(venv) $ python
>>> import flask
>>>

程序和请求上下文
表2-1　Flask上下文全局变量

变量名                 上下文             说明

current_app         程序上下文       当前激活程序的程序实例

g                   程序上下文       处理请求时用作临时存储的对象。每次请求都会重设这个变量

request             请求上下文       请求对象，封装了客户端发出的HTTP请求中的内容

session             请求上下文        用户会话，用于存储请求之间需要“记住”的值的词典

Flask在分发请求之前激活（或推送）程序和请求上下文，请求处理完成后再将其删除。程序上下文被推送后，就可以在线程中使用current_app和g变量。类似地，请求上下文被推送后，就可以使用request和session变量。如果使用这些变量时我们没有激活程序上下文或请求上下文，就会导致错误。如果你不知道为什么这4个上下文变量如此有用，先别担心，后面的章节会详细说明。

下面这个Python shell会话演示了程序上下文的使用方法：

>>> from hello import app
>>> from flask import current_app
>>> current_app.name
>>> Traceback (most recent call last):
>>> ...
>>> RuntimeError: working outside of application context
>>> app_ctx = app.app_context()
>>> app_ctx.push()
>>> current_app.name
>>> 'hello'
>>> app_ctx.pop()
>>> 在这个例子中，没激活程序上下文之前就调用current_app.name会导致错误，但推送完上下文之后就可以调用了。注意，在程序实例上调用app.app_context()可获得一个程序上下文。

Flask使用app.route修饰器或者非修饰器形式的app.add_url_rule()生成映射。

2.6　Flask扩展

Flask被设计为可扩展形式，故而没有提供一些重要的功能，例如数据库和用户认证，所以开发者可以自由选择最适合程序的包，或者按需求自行开发。
社区成员开发了大量不同用途的扩展，如果这还不能满足需求，你还可使用所有Python标准包或代码库。

##### 2.6.1　使用Flask-Script支持命令行选项

Flask-Script扩展使用`pip`安装：

```shell
(venv) $ pip install flask-script
```

**示例2-3　hello.py：使用Flask-Script**

```python
from flask.ext.script import Manager
manager = Manager(app)

…

if __name__ == '__main__':
manager.run()
```

#### 3.1　Jinja2模板引擎

##### 3.1.1　渲染模板

Flask提供的`render_template`函数把Jinja2模板引擎集成到了程序中。`render_template`函数的第一个参数是模板的文件名。随后的参数都是键值对，表示模板中变量对应的真实值。

##### 3.1.2　变量

Jinja2能识别所有类型的变量，甚至是一些复杂的类型，例如列表、字典和对象。

<p>A value from a dictionary: {{ mydict['key'] }}.</p>
<p>A value from a list: {{ mylist[3] }}.</p>
<p>A value from a list, with a variable index: {{ mylist[myintvar] }}.</p>
<p>A value from an object's method: {{ myobj.somemethod() }}.</p>

可以使用**过滤器**修改变量，过滤器名添加在变量名之后，中间使用竖线分隔。例如，下述模板以首字母大写形式显示变量`name`的值：

```html
Hello, {{ name|capitalize }}
```

**表3-1　Jinja2变量过滤器**

| 过滤器名     |                    说明                    |
| ------------ | :----------------------------------------: |
| `safe`       |               渲染值时不转义               |
| `capitalize` | 把值的首字母转换成大写，其他字母转换成小写 |
| `lower`      |             把值转换成小写形式             |
| `upper`      |             把值转换成大写形式             |
| `title`      |     把值中每个单词的首字母都转换成大写     |
| `trim`       |             把值的首尾空格去掉             |
| `striptags`  |     渲染之前把值中所有的HTML标签都删掉     |

##### 3.1.3　控制结构

Jinja2提供了多种控制结构，可用来改变模板的渲染流程。

```html
{% if user %}
Hello, {{ user }}!
{% else %}
Hello, Stranger!
{% endif %}
```

```html
{% for comment in comments %}
<li>{{ comment }}</li>
{% endfor %}
```

Jinja2还支持**宏**。宏类似于Python代码中的函数。

```html
{% macro render_comment(comment) %}
<li>{{ comment }}</li>
{% endmacro %}
<ul>
    {% for comment in comments %}
        {{ render_comment(comment) }}
    {% endfor %}
</ul>
```

为了重复使用宏，我们可以将其保存在单独的文件中，然后在需要使用的模板中**导入**：

```html
{% import 'macros.html' as macros %}

<ul>
{% for comment in comments %}
{{ macros.render_comment(comment) }}
{% endfor %}
</ul>
```

需要在多处重复使用的模板代码片段可以写入单独的文件，再**包含**在所有模板中，以避免重复：

{% include 'common.html' %}

另一种重复使用代码的强大方式是模板继承，它类似于Python代码中的类继承。首先，创建一个名为base.html的基模板：

```html
<html>

<head>
{% block head %}
<title>{% block title %}{% endblock %} - My Application</title>
{% endblock %}
</head>

<body>
{% block body %}
{% endblock %}
</body>
</html>
```

`block`标签定义的元素可在衍生模板中修改。在本例中，我们定义了名为`head`、`title`和`body`的块。注意，`title`包含在`head`中。下面这个示例是基模板的衍生模板：

```html
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
{{ super() }}

<style>
</style>

{% endblock %}
{% block body %}

<h1>Hello, World!</h1>

{% endblock %}
```

`extends`指令声明这个模板衍生自base.html。在`extends`指令之后，基模板中的3个块被重新定义，模板引擎会将其插入适当的位置。注意新定义的`head`块，在基模板中其内容不是空的，所以使用`super()`获取原来的内容。

#### 3.4　链接

在模板中直接编写简单路由的URL链接不难，但对于包含可变部分的动态路由，在模板中构建正确的URL就很困难。而且，直接编写URL会对代码中定义的路由产生不必要的依赖关系。如果重新定义路由，模板中的链接可能会失效。

为了避免这些问题，Flask提供了`url_for()`辅助函数，它可以使用程序URL映射中保存的信息生成URL。

使用`url_for()`生成动态地址时，将动态部分作为关键字参数传入。例如，`url_for('user', name='john', _external=True)`的返回结果是http://localhost:5000/user/john。

传入`url_for()`的关键字参数不仅限于动态路由中的参数。函数能将任何额外参数添加到查询字符串中。例如，`url_for('index', page=2)`的返回结果是/?page=2。

#### 3.5　静态文件

你可能还记得在第2章中检查hello.py程序的URL映射时，其中有一个`static`路由。这是因为对静态文件的引用被当成一个特殊的路由，即/*static*/<*filename*>。例如，调用`url_for('static', filename='css/styles.css', _external=True)`得到的结果是http://localhost:5000/static/css/styles.css。

#### 3.6　使用Flask-Moment本地化日期和时间

服务器需要统一时间单位，这和用户所在的地理位置无关，所以一般使用协调世界时（Coordinated Universal Time，UTC）。不过用户看到UTC格式的时间会感到困惑，他们更希望看到当地时间，而且采用当地惯用的格式。

要想在服务器上只使用UTC时间，一个优雅的解决方案是，把时间单位发送给Web浏览器，转换成当地时间，然后渲染。Web浏览器可以更好地完成这一任务，因为它能获取用户电脑中的时区和区域设置。

有一个使用JavaScript开发的优秀客户端开源代码库，名为moment.js（<http://momentjs.com/>），它可以在浏览器中渲染日期和时间。Flask-Moment是一个Flask程序扩展，能把moment.js集成到Jinja2模板中。Flask-Moment可以使用`pip`安装：

```bash
(venv) $ pip install flask-moment
```

这个扩展的初始化方法如示例3-11所示。

**示例3-11　hello.py：初始化Flask-Moment**

```python
from flask.ext.moment import Moment
moment = Moment(app)
```

除了moment.js，Flask-Moment还依赖jquery.js。要在HTML文档的某个地方引入这两个库，可以直接引入，这样可以选择使用哪个版本，也可使用扩展提供的辅助函数，从内容分发网络（Content Delivery Network，CDN）中引入通过测试的版本。Bootstrap已经引入了jquery.js，因此只需引入moment.js即可。示例3-12展示了如何在基模板的`scripts`块中引入这个库。

**示例3-12　templates/base.html：引入moment.js库**

```
{% block scripts %}
{{ super() }}
{{ moment.include_moment() }}
{% endblock %}
```

为了处理时间戳，Flask-Moment向模板开放了`moment`类。示例3-13中的代码把变量`current_time`传入模板进行渲染。

**示例3-13　hello.py：加入一个datetime变量**

```python
from datetime import datetime

@app.route('/')
def index():
    return render_template('index.html',
                           current_time=datetime.utcnow())
```

示例3-14展示了如何在模板中渲染`current_time`。

**代码3-14　templates/index.html：使用Flask-Moment渲染时间戳**

```html
<p>The local date and time is {{ moment(current_time).format('LLL') }}.</p>
<p>That was {{ moment(current_time).fromNow(refresh=True) }}</p>
```

Flask-Moment渲染的时间戳可实现多种语言的本地化。语言可在模板中选择，把语言代码传给`lang()`函数即可：

```html
{{ moment.lang('es') }}
```

### 第 4 章　Web表单

Flask-WTF（<http://pythonhosted.org/Flask-WTF/>）扩展可以把处理Web表单的过程变成一种愉悦的体验。这个扩展对独立的WTForms（<http://wtforms.simplecodes.com>）包进行了包装，方便集成到Flask程序中.

Flask-WTF及其依赖可使用`pip`安装：

```bash
(venv) $ pip install flask-wtf
```

#### 4.1　跨站请求伪造保护

默认情况下，Flask-WTF能保护所有表单免受跨站请求伪造（Cross-Site Request Forgery，CSRF）的攻击。恶意网站把请求发送到被攻击者已登录的其他网站时就会引发CSRF攻击。

为了实现CSRF保护，Flask-WTF需要程序设置一个密钥。Flask-WTF使用这个密钥生成加密令牌，再用令牌验证请求中表单数据的真伪。设置密钥的方法如示例4-1所示。

**示例4-1　hello.py：设置Flask-WTF**

```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
```

`app.config`字典可用来存储框架、扩展和程序本身的配置变量。使用标准的字典句法就能把配置值添加到`app.config`对象中。这个对象还提供了一些方法，可以从文件或环境中导入配置值。

`SECRET_KEY`配置变量是通用密钥，可在Flask和多个第三方扩展中使用。如其名所示，加密的强度取决于变量值的机密程度。不同的程序要使用不同的密钥，而且要保证其他人不知道你所用的字符串。

> 

#### 4.2　表单类

使用Flask-WTF时，每个Web表单都由一个继承自`Form`的类表示。这个类定义表单中的一组字段，每个字段都用对象表示。字段对象可附属一个或多个**验证函数**。验证函数用来验证用户提交的输入值是否符合要求。

示例4-2是一个简单的Web表单，包含一个文本字段和一个提交按钮。

**示例4-2　hello.py：定义表单类**

```python
from flask.ext.wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required

class NameForm(Form):
    name = StringField('What is your name?', validators=[Required()])
    submit = SubmitField('Submit')
```

这个表单中的字段都定义为类变量，类变量的值是相应字段类型的对象。在这个示例中，`NameForm`表单中有一个名为`name`的文本字段和一个名为`submit`的提交按钮。`StringField`类表示属性为`type="text"`的`<input>`元素。`SubmitField`类表示属性为`type="submit"`的`<input>`元素。字段构造函数的第一个参数是把表单渲染成HTML时使用的标号

`StringField`构造函数中的可选参数`validators`指定一个由验证函数组成的列表，在接受用户提交的数据之前验证数据。验证函数`Required()`确保提交的字段不为空。

WTForms支持的HTML标准字段如表4-1所示。

**表4-1　WTForms支持的HTML标准字段**

| 字段类型              | 说明                                  |
| --------------------- | ------------------------------------- |
| `StringField`         | 文本字段                              |
| `TextAreaField`       | 多行文本字段                          |
| `PasswordField`       | 密码文本字段                          |
| `HiddenField`         | 隐藏文本字段                          |
| `DateField`           | 文本字段，值为`datetime.date`格式     |
| `DateTimeField`       | 文本字段，值为`datetime.datetime`格式 |
| `IntegerField`        | 文本字段，值为整数                    |
| `DecimalField`        | 文本字段，值为`decimal.Decimal`       |
| `FloatField`          | 文本字段，值为浮点数                  |
| `BooleanField`        | 复选框，值为`True`和`False`           |
| `RadioField`          | 一组单选框                            |
| `SelectField`         | 下拉列表                              |
| `SelectMultipleField` | 下拉列表，可选择多个值                |
| `FileField`           | 文件上传字段                          |
| `SubmitField`         | 表单提交按钮                          |
| `FormField`           | 把表单作为字段嵌入另一个表单          |
| `FieldList`           | 一组指定类型的字段                    |

WTForms内建的验证函数如表4-2所示。

**表4-2　WTForms验证函数**

| 验证函数      | 说明                                                   |
| ------------- | ------------------------------------------------------ |
| `Email`       | 验证电子邮件地址                                       |
| `EqualTo`     | 比较两个字段的值；常用于要求输入两次密码进行确认的情况 |
| `IPAddress`   | 验证IPv4网络地址                                       |
| `Length`      | 验证输入字符串的长度                                   |
| `NumberRange` | 验证输入的值在数字范围内                               |
| `Optional`    | 无输入值时跳过其他验证函数                             |
| `Required`    | 确保字段中有数据                                       |
| `Regexp`      | 使用正则表达式验证输入值                               |
| `URL`         | 验证URL                                                |
| `AnyOf`       | 确保输入值在可选值列表中                               |
| `NoneOf`      | 确保输入值不在可选值列表中                             |

#### 4.3　把表单渲染成HTML

表单字段是可调用的，在模板中调用后会渲染成HTML。假设视图函数把一个`NameForm`实例通过参数`form`传入模板，在模板中可以生成一个简单的表单，如下所示：

```html
<form method="POST">
    {{form.hidden_tag()}}
    {{ form.name.label }} {{ form.name() }}
    {{ form.submit() }}
</form>
```

#### 5.6　定义模型

**模型**这个术语表示程序使用的持久化实体。在ORM中，模型一般是一个Python类，类中的属性对应数据库表中的列。

Flask-SQLAlchemy创建的数据库实例为模型提供了一个基类以及一系列辅助类和辅助函数，可用于定义模型的结构。图5-1中的`roles`表和`users`表可定义为模型`Role`和`User`，如示例5-2所示。

**示例5-2　hello.py：定义Role和User模型**

```python
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)

    def __repr__(self):
        return '<Role %r>' % self.name

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, index=True)

    def __repr__(self):
        return '<User %r>' % self.username
```

类变量`__tablename__`定义在数据库中使用的表名。如果没有定义`__tablename__`，Flask-SQLAlchemy会使用一个默认名字，但默认的表名没有遵守使用复数形式进行命名的约定，所以最好由我们自己来指定表名。其余的类变量都是该模型的属性，被定义为`db.Column`类的实例。

`db.Column`类构造函数的第一个参数是数据库列和模型属性的类型。表5-2列出了一些可用的列类型以及在模型中使用的Python类型。

**表5-2　最常用的SQLAlchemy列类型**

| 类型名       | Python类型           | 说明                                                |
| ------------ | -------------------- | --------------------------------------------------- |
| Integer      | `int`                | 普通整数，一般是32位                                |
| SmallInteger | `int`                | 取值范围小的整数，一般是16位                        |
| BigInteger   | `int`或`long`        | 不限制精度的整数                                    |
| Float        | `float`              | 浮点数                                              |
| Numeric      | `decimal.Decimal`    | 定点数                                              |
| String       | `str`                | 变长字符串                                          |
| Text         | `str`                | 变长字符串，对较长或不限长度的字符串做了优化        |
| Unicode      | `unicode`            | 变长Unicode字符串                                   |
| UnicodeText  | `unicode`            | 变长Unicode字符串，对较长或不限长度的字符串做了优化 |
| Boolean      | `bool`               | 布尔值                                              |
| Date         | `datetime.date`      | 日期                                                |
| Time         | `datetime.time`      | 时间                                                |
| DateTime     | `datetime.datetime`  | 日期和时间                                          |
| Interval     | `datetime.timedelta` | 时间间隔                                            |
| Enum         | `str`                | 一组字符串                                          |
| PickleType   | 任何Python对象       | 自动使用Pickle序列化                                |
| LargeBinary  | `str`                | 二进制文件                                          |

`db.Column`中其余的参数指定属性的配置选项。表5-3列出了一些可用选项。

**表5-3　最常使用的SQLAlchemy列选项**

| 选项名        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| `primary_key` | 如果设为`True`，这列就是表的主键                             |
| `unique`      | 如果设为`True`，这列不允许出现重复的值                       |
| `index`       | 如果设为`True`，为这列创建索引，提升查询效率                 |
| `nullable`    | 如果设为`True`，这列允许使用空值；如果设为`False`，这列不允许使用空值 |
| `default`     | 为这列定义默认值                                             |

#### 5.7　关系

关系型数据库使用关系把不同表中的行联系起来。图5-1所示的关系图表示用户和角色之间的一种简单关系。这是角色到用户的**一对多**关系，因为一个角色可属于多个用户，而每个用户都只能有一个角色。

图5-1中的一对多关系在模型类中的表示方法如示例5-3所示。

**示例5-3　hello.py：关系**

```python
class Role(db.Model):
    # ...
    users = db.relationship('User', backref='role')

class User(db.Model):
    # ...
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
```

如图5-1所示，关系使用`users`表中的外键连接了两行。添加到`User`模型中的`role_id`列被定义为外键，就是这个外键建立起了关系。传给`db.ForeignKey()`的参数`'roles.id'`表明，这列的值是`roles`表中行的`id`值。

添加到`Role`模型中的`users`属性代表这个关系的面向对象视角。对于一个`Role`类的实例，其`users`属性将返回与角色相关联的用户组成的列表。`db.relationship()`的第一个参数表明这个关系的另一端是哪个模型。如果模型类尚未定义，可使用字符串形式指定。

`db.relationship()`中的`backref`参数向`User`模型中添加一个`role`属性，从而定义反向关系。这一属性可替代`role_id`访问`Role`模型，此时获取的是模型对象，而不是外键的值。

大多数情况下，`db.relationship()`都能自行找到关系中的外键，但有时却无法决定把哪一列作为外键。例如，如果`User`模型中有两个或以上的列定义为`Role`模型的外键，SQLAlchemy就不知道该使用哪列。如果无法决定外键，你就要为`db.relationship()`提供额外参数，从而确定所用外键。表5-4列出了定义关系时常用的配置选项。

**表5-4　常用的SQLAlchemy关系选项**

| 选项名          | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| `backref`       | 在关系的另一个模型中添加反向引用                             |
| `primaryjoin`   | 明确指定两个模型之间使用的联结条件。只在模棱两可的关系中需要指定 |
| `lazy`          | 指定如何加载相关记录。可选值有`select`（首次访问时按需加载）、`immediate`（源对象加载后就加载）、`joined`（加载记录，但使用联结）、`subquery`（立即加载，但使用子查询），`noload`（永不加载）和`dynamic`（不加载记录，但提供加载记录的查询） |
| `uselist`       | 如果设为`Fales`，不使用列表，而使用标量值                    |
| `order_by`      | 指定关系中记录的排序方式                                     |
| `secondary`     | 指定**多对多**关系中关系表的名字                             |
| `secondaryjoin` | SQLAlchemy无法自行决定时，指定**多对多**关系中的二级联结条件 |

> 

#### 5.8　数据库操作

##### 5.8.1　创建表

首先，我们要让Flask-SQLAlchemy根据模型类创建数据库。方法是使用`db.create_all()`函数：

```bash
(venv) $ python hello.py shell
>>> from hello import db
>>> db.create_all()
```

##### 5.8.2　插入行

下面这段代码创建了一些角色和用户：

```bash
>>> from hello import Role, User
>>> admin_role = Role(name='Admin')
>>> mod_role = Role(name='Moderator')
>>> user_role = Role(name='User')
>>> user_john = User(username='john', role=admin_role)
>>> user_susan = User(username='susan', role=user_role)
>>> user_david = User(username='david', role=user_role)
```

现在这些对象只存在于Python中，还未写入数据库。因此`id`尚未赋值：

```bash
>>> print(admin_role.id)
None
>>> print(mod_role.id)
None
>>> print(user_role.id)
None
```

通过数据库**会话**管理对数据库所做的改动，在Flask-SQLAlchemy中，会话由`db.session`表示。准备把对象写入数据库之前，先要将其添加到会话中：

```bash
>>> db.session.add(admin_role)
>>> db.session.add(mod_role)
>>> db.session.add(user_role)
>>> db.session.add(user_john)
>>> db.session.add(user_susan)
>>> db.session.add(user_david)
```

为了把对象写入数据库，我们要调用`commit()`方法**提交**会话：

```bash
>>> db.session.commit()
```

再次查看`id`属性，现在它们已经赋值了：

```bash
>>> print(admin_role.id)
1
>>> print(mod_role.id)
2
>>> print(user_role.id)
3
```

> 据库会话`db.session`和第4章介绍的Flask`session`对象没有关系。数据库会话也称为事务。
>
> 数据库会话也可**回滚**。调用`db.session.rollback()`后，添加到数据库会话中的所有对象都会还原到它们在数据库时的状态。

##### 5.8.3　修改行

在数据库会话上调用`add()`方法也能更新模型。我们继续在之前的shell会话中进行操作，下面这个例子把`"Admin"`角色重命名为`"Administrator"`：

```bash
>>> admin_role.name = 'Administrator'
>>> db.session.add(admin_role)
>>> db.session.commit()
```

##### 5.8.4　删除行

数据库会话还有个`delete()`方法。下面这个例子把`"Moderator"`角色从数据库中删除：

```bash
>>> db.session.delete(mod_role)
>>> db.session.commit()
```

注意，删除与插入和更新一样，提交数据库会话后才会执行

##### 5.8.5　查询行

Flask-SQLAlchemy为每个模型类都提供了`query`对象。最基本的模型查询是取回对应表中的所有记录：

```bash
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>]
>>> User.query.all()
[<User u'john'>, <User u'susan'>, <User u'david'>]
```

使用**过滤器**可以配置`query`对象进行更精确的数据库查询。下面这个例子查找角色为`"User"`的所有用户：

```bash
>>> User.query.filter_by(role=user_role).all()
[<User u'susan'>, <User u'david'>]
```

若要查看SQLAlchemy为查询生成的原生SQL查询语句，只需把`query`对象转换成字符串：

```bash
>>> str(User.query.filter_by(role=user_role))
'SELECT users.id AS users_id, users.username AS users_username,
users.role_id AS users_role_id FROM users WHERE :param_1 = users.role_id'
```

如果你退出了shell会话，前面这些例子中创建的对象就不会以Python对象的形式存在，而是作为各自数据库表中的行。如果你打开了一个新的shell会话，就要从数据库中读取行，再重新创建Python对象。下面这个例子发起了一个查询，加载名为`"User"`的用户角色：

```bash
>>> user_role = Role.query.filter_by(name='User').first()
```

`filter_by()`等过滤器在`query`对象上调用，返回一个更精确的`query`对象。多个过滤器可以一起调用，直到获得所需结果。

表5-5列出了可在`query`对象上调用的常用过滤器。完整的列表参见SQLAlchemy文档（<http://docs.sqlalchemy.org>）。

**表5-5　常用的SQLAlchemy查询过滤器**

| 过滤器        | 说明                                                 |
| ------------- | ---------------------------------------------------- |
| `filter()`    | 把过滤器添加到原查询上，返回一个新查询               |
| `filter_by()` | 把等值过滤器添加到原查询上，返回一个新查询           |
| `limit()`     | 使用指定的值限制原查询返回的结果数量，返回一个新查询 |
| `offset()`    | 偏移原查询返回的结果，返回一个新查询                 |
| `order_by()`  | 根据指定条件对原查询结果进行排序，返回一个新查询     |
| `group_by()`  | 根据指定条件对原查询结果进行分组，返回一个新查询     |

在查询上应用指定的过滤器后，通过调用`all()`执行查询，以列表的形式返回结果。除了`all()`之外，还有其他方法能触发查询执行。表5-6列出了执行查询的其他方法。

**表5-6　最常使用的SQLAlchemy查询执行函数**

| 方法             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| `all()`          | 以列表形式返回查询的所有结果                                 |
| `first()`        | 返回查询的第一个结果，如果没有结果，则返回`None`             |
| `first_or_404()` | 返回查询的第一个结果，如果没有结果，则终止请求，返回404错误响应 |
| `get()`          | 返回指定主键对应的行，如果没有对应的行，则返回`None`         |
| `get_or_404()`   | 返回指定主键对应的行，如果没找到指定的主键，则终止请求，返回404错误响应 |
| `count()`        | 返回查询结果的数量                                           |
| `paginate()`     | 返回一个`Paginate`对象，它包含指定范围内的结果               |

#### 5.9　在视图函数中操作数据库

前一节介绍的数据库操作可以直接在视图函数中进行。示例5-5展示了首页路由的新版本，已经把用户输入的名字写入了数据库。

**示例5-5　hello.py：在视图函数中操作数据库**

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username = form.name.data)
            db.session.add(user)
            session['known'] = False
        else:
            session['known'] = True
        session['name'] = form.name.data
        form.name.data = ''
        return redirect(url_for('index'))
    return render_template('index.html',
        form = form, name = session.get('name'),
        known = session.get('known', False))
```

在这个修改后的版本中，提交表单后，程序会使用`filter_by()`查询过滤器在数据库中查找提交的名字。变量`known`被写入用户会话中，因此重定向之后，可以把数据传给模板，用来显示自定义的欢迎消息。

对应的模板新版本如示例5-6所示。这个模板使用`known`参数在欢迎消息中加入了第二行，从而对已知用户和新用户显示不同的内容。

**示例5-6　templates/index.html**

```html
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Flasky{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Hello, {% if name %}{{ name }}{% else %}Stranger{% endif %}!</h1>
    {% if not known %}
    <p>Pleased to meet you!</p>
    {% else %}
    <p>Happy to see you again!</p>
    {% endif %}
</div>
{{ wtf.quick_form(form) }}
{% endblock %}
```

5.10　集成Python shell

每次启动shell会话都要导入数据库实例和模型，这真是份枯燥的工作。为了避免一直重复导入，我们可以做些配置，让Flask-Script的`shell`命令自动导入特定的对象。

若想把对象添加到导入列表中，我们要为`shell`命令注册一个`make_context`回调函数，如示例5-7所示。

**示例5-7　hello.py：为shell命令添加一个上下文**

```python
from flask.ext.script import Shell

def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)
manager.add_command("shell", Shell(make_context=make_shell_context))
```

`make_shell_context()`函数注册了程序、数据库实例以及模型，因此这些对象能直接导入shell：

```bash
$ python hello.py shell
>>> app
<Flask 'app'>
>>> db
<SQLAlchemy engine='sqlite:////home/flask/flasky/data.sqlite'>
>>> User
<class 'app.User'>
```



# 第 7 章　大型程序的结构

尽管在单一脚本中编写小型Web程序很方便，但这种方法并不能广泛使用。程序变复杂后，使用单个大型源码文件会导致很多问题。

不同于大多数其他的Web框架，Flask并不强制要求大型项目使用特定的组织方式，程序结构的组织方式完全由开发者决定。在本章，我们将介绍一种使用包和模块组织大型程序的方式。本书后续示例都将采用这种结构。

## 7.1　项目结构

Flask程序的基本结构如示例7-1所示。

**示例7-1　多文件Flask程序的基本结构**

```
|-flasky
  |-app/
    |-templates/
    |-static/
    |-main/
      |-__init__.py
      |-errors.py
      |-forms.py
      |-views.py
    |-__init__.py
    |-email.py
    |-models.py
  |-migrations/
  |-tests/
    |-__init__.py
    |-test*.py
  |-venv/
  |-requirements.txt
  |-config.py
  |-manage.py
```

这种结构有4个顶级文件夹：

- Flask程序一般都保存在名为app的包中；
- 和之前一样，migrations文件夹包含数据库迁移脚本；
- 单元测试编写在tests包中；
- 和之前一样，venv文件夹包含Python虚拟环境。

同时还创建了一些新文件：

- requirements.txt列出了所有依赖包，便于在其他电脑中重新生成相同的虚拟环境；
- config.py存储配置；
- manage.py用于启动程序以及其他的程序任务。

## 7.2　配置选项

程序经常需要设定多个配置。这方面最好的例子就是开发、测试和生产环境要使用不同的数据库，这样才不会彼此影响。

我们不再使用hello.py中简单的字典状结构配置，而使用层次结构的配置类。config.py文件的内容如示例7-2所示。

**示例7-2　config.py：程序的配置**

```python
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard to guess string'
    SQLALCHEMY_COMMIT_ON_TEARDOWN = True
    FLASKY_MAIL_SUBJECT_PREFIX = '[Flasky]'
    FLASKY_MAIL_SENDER = 'Flasky Admin <flasky@example.com>'
    FLASKY_ADMIN = os.environ.get('FLASKY_ADMIN')

    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config):
    DEBUG = True
    MAIL_SERVER = 'smtp.googlemail.com'
    MAIL_PORT = 587
    MAIL_USE_TLS = True
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-dev.sqlite')

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-test.sqlite')

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data.sqlite')

config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,

    'default': DevelopmentConfig
}
```

基类`Config`中包含通用配置，子类分别定义专用的配置。如果需要，你还可添加其他配置类。

为了让配置方式更灵活且更安全，某些配置可以从环境变量中导入。例如，`SECRET_KEY`的值，这是个敏感信息，可以在环境中设定，但系统也提供了一个默认值，以防环境中没有定义。

在3个子类中，`SQLALCHEMY_DATABASE_URI`变量都被指定了不同的值。这样程序就可在不同的配置环境中运行，每个环境都使用不同的数据库。

配置类可以定义`init_app()`类方法，其参数是程序实例。在这个方法中，可以执行对当前环境的配置初始化。现在，基类`Config`中的`init_app()`方法为空。

在这个配置脚本末尾，`config`字典中注册了不同的配置环境，而且还注册了一个默认配置（本例的开发环境）。

## 7.3　程序包

程序包用来保存程序的所有代码、模板和静态文件。我们可以把这个包直接称为app（应用），如果有需求，也可使用一个程序专用名字。templates和static文件夹是程序包的一部分，因此这两个文件夹被移到了app中。数据库模型和电子邮件支持函数也被移到了这个包中，分别保存为app/models.py和app/email.py。

### 7.3.1　使用程序工厂函数

在单个文件中开发程序很方便，但却有个很大的缺点，因为程序在全局作用域中创建，所以无法动态修改配置。运行脚本时，程序实例已经创建，再修改配置为时已晚。这一点对单元测试尤其重要，因为有时为了提高测试覆盖度，必须在不同的配置环境中运行程序。

这个问题的解决方法是延迟创建程序实例，把创建过程移到可显式调用的**工厂函数**中。这种方法不仅可以给脚本留出配置程序的时间，还能够创建多个程序实例，这些实例有时在测试中非常有用。程序的工厂函数在app包的构造文件中定义，如示例7-3所示。

构造文件导入了大多数正在使用的Flask扩展。由于尚未初始化所需的程序实例，所以没有初始化扩展，创建扩展类时没有向构造函数传入参数。`create_app()`函数就是程序的工厂函数，接受一个参数，是程序使用的配置名。配置类在config.py文件中定义，其中保存的配置可以使用Flask`app.config`配置对象提供的`from_object()`方法直接导入程序。至于配置对象，则可以通过名字从`config`字典中选择。程序创建并配置好后，就能初始化扩展了。在之前创建的扩展对象上调用`init_app()`可以完成初始化过程。

**示例7-3　app/__init__.py：程序包的构造文件**

```python
from flask import Flask, render_template
from flask.ext.bootstrap import Bootstrap
from flask.ext.mail import Mail
from flask.ext.moment import Moment
from flask.ext.sqlalchemy import SQLAlchemy
from config import config

bootstrap = Bootstrap()
mail = Mail()
moment = Moment()
db = SQLAlchemy()

def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)

    bootstrap.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    db.init_app(app)

    # 附加路由和自定义的错误页面

    return app
```

工厂函数返回创建的程序示例，不过要注意，现在工厂函数创建的程序还不完整，因为没有路由和自定义的错误页面处理程序。这是下一节要讲的话题。

### 7.3.2　在蓝本中实现程序功能

转换成程序工厂函数的操作让定义路由变复杂了。在单脚本程序中，程序实例存在于全局作用域中，路由可以直接使用`app.route`修饰器定义。但现在程序在运行时创建，只有调用`create_app()`之后才能使用`app.route`修饰器，这时定义路由就太晚了。和路由一样，自定义的错误页面处理程序也面临相同的困难，因为错误页面处理程序使用`app.errorhandler`修饰器定义。

幸好Flask使用**蓝本**提供了更好的解决方法。蓝本和程序类似，也可以定义路由。不同的是，在蓝本中定义的路由处于休眠状态，直到蓝本注册到程序上后，路由才真正成为程序的一部分。使用位于全局作用域中的蓝本时，定义路由的方法几乎和单脚本程序一样。

和程序一样，蓝本可以在单个文件中定义，也可使用更结构化的方式在包中的多个模块中创建。为了获得最大的灵活性，程序包中创建了一个子包，用于保存蓝本。示例7-4是这个子包的构造文件，蓝本就创建于此。

**示例7-4　app/main/__init__.py：创建蓝本**

```python
from flask import Blueprint

main = Blueprint('main', __name__)

from . import views, errors
```

通过实例化一个`Blueprint`类对象可以创建蓝本。这个构造函数有两个必须指定的参数：蓝本的名字和蓝本所在的包或模块。和程序一样，大多数情况下第二个参数使用Python的`__name__`变量即可。

程序的路由保存在包里的app/main/views.py模块中，而错误处理程序保存在app/main/errors.py模块中。导入这两个模块就能把路由和错误处理程序与蓝本关联起来。注意，这些模块在app/main/__init__.py脚本的末尾导入，这是为了避免循环导入依赖，因为在views.py和errors.py中还要导入蓝本`main`。

蓝本在工厂函数`create_app()`中注册到程序上，如示例7-5所示。

**示例7-5　app/_init_.py：注册蓝本**

```python
def create_app(config_name):
    # ...
    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)

    return app
```

示例7-6显示了错误处理程序。

**示例7-6　app/main/errors.py：蓝本中的错误处理程序**

```python
from flask import render_template
from . import main

@main.app_errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@main.app_errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500
```

在蓝本中编写错误处理程序稍有不同，如果使用`errorhandler`修饰器，那么只有蓝本中的错误才能触发处理程序。要想注册程序全局的错误处理程序，必须使用`app_errorhandler`。

在蓝本中定义的程序路由如示例7-7所示。

**示例7-7　app/main/views.py：蓝本中定义的程序路由**

```python
from datetime import datetime
from flask import render_template, session, redirect, url_for

from . import main
from .forms import NameForm
from .. import db
from ..models import User

@main.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        # ...
        return redirect(url_for('.index'))
    return render_template('index.html',
                           form=form, name=session.get('name'),
                           known=session.get('known', False),
                           current_time=datetime.utcnow())
```

在蓝本中编写视图函数主要有两点不同：第一，和前面的错误处理程序一样，路由修饰器由蓝本提供；第二，`url_for()`函数的用法不同。你可能还记得，`url_for()`函数的第一个参数是路由的端点名，在程序的路由中，默认为视图函数的名字。例如，在单脚本程序中，`index()`视图函数的URL可使用`url_for('index')`获取。

在蓝本中就不一样了，Flask会为蓝本中的全部端点加上一个命名空间，这样就可以在不同的蓝本中使用相同的端点名定义视图函数，而不会产生冲突。命名空间就是蓝本的名字（`Blueprint`构造函数的第一个参数），所以视图函数`index()`注册的端点名是`main.index`，其URL使用`url_for('main.index')`获取。

`url_for()`函数还支持一种简写的端点形式，在蓝本中可以省略蓝本名，例如`url_for('.index')`。在这种写法中，命名空间是当前请求所在的蓝本。这意味着同一蓝本中的重定向可以使用简写形式，但跨蓝本的重定向必须使用带有命名空间的端点名。

为了完全修改程序的页面，表单对象也要移到蓝本中，保存于app/main/forms.py模块。

## 7.4　启动脚本

顶级文件夹中的manage.py文件用于启动程序。脚本内容如示例7-8所示。

**示例7-8　manage.py：启动脚本**

```python
#!/usr/bin/env python
import os
from app import create_app, db
from app.models import User, Role
from flask.ext.script import Manager, Shell
from flask.ext.migrate import Migrate, MigrateCommand

app = create_app(os.getenv('FLASK_CONFIG') or 'default')
manager = Manager(app)
migrate = Migrate(app, db)

def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)
manager.add_command("shell", Shell(make_context=make_shell_context))
manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()
```

这个脚本先创建程序。如果已经定义了环境变量`FLASK_CONFIG`，则从中读取配置名；否则使用默认配置。然后初始化Flask-Script、Flask-Migrate和为Python shell定义的上下文。

出于便利，脚本中加入了shebang声明，所以在基于Unix的操作系统中可以通过`./manage.py`执行脚本，而不用使用复杂的`python manage.py`。

## 7.5　需求文件

程序中必须包含一个requirements.txt文件，用于记录所有依赖包及其精确的版本号。如果要在另一台电脑上重新生成虚拟环境，这个文件的重要性就体现出来了，例如部署程序时使用的电脑。`pip`可以使用如下命令自动生成这个文件：

```bash
(venv) $ pip freeze >requirements.txt
```

安装或升级包后，最好更新这个文件。需求文件的内容示例如下：

```text
Flask==0.10.1
Flask-Bootstrap==3.0.3.1
Flask-Mail==0.9.0
Flask-Migrate==1.1.0
Flask-Moment==0.2.0
Flask-SQLAlchemy==1.0
Flask-Script==0.6.6
Flask-WTF==0.9.4
Jinja2==2.7.1
Mako==0.9.1
MarkupSafe==0.18
SQLAlchemy==0.8.4
WTForms==1.0.5
Werkzeug==0.9.4
alembic==0.6.2
blinker==1.3
itsdangerous==0.23
```

如果你要创建这个虚拟环境的完全副本，可以创建一个新的虚拟环境，并在其上运行以下命令：

```bash
(venv) $ pip install -r requirements.txt
```

当你阅读本书时，该示例requirements.txt文件中的版本号可能已经过期了。如果愿意，你可以试着使用这些包的最新版。如果遇到问题，你可以随时换回这个需求文件中的版本，因为这些版本和程序兼容。

## 7.6　单元测试

这个程序很小，所以没什么可测试的。不过为了演示，我们可以编写两个简单的测试，如示例7-9所示。

**示例7-9　tests/test_basics.py：单元测试**

```python
import unittest
from flask import current_app
from app import create_app, db

class BasicsTestCase(unittest.TestCase):
    def setUp(self):
        self.app = create_app('testing')
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()

    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()

    def test_app_exists(self):
        self.assertFalse(current_app is None)

    def test_app_is_testing(self):
        self.assertTrue(current_app.config['TESTING'])
```

这个测试使用Python标准库中的`unittest`包编写。`setUp()`和`tearDown()`方法分别在各测试前后运行，并且名字以`test_`开头的函数都作为测试执行。

`setUp()`方法尝试创建一个测试环境，类似于运行中的程序。首先，使用测试配置创建程序，然后激活上下文。这一步的作用是确保能在测试中使用`current_app`，像普通请求一样。然后创建一个全新的数据库，以备不时之需。数据库和程序上下文在`tearDown()`方法中删除。

第一个测试确保程序实例存在。第二个测试确保程序在测试配置中运行。若想把tests文件夹作为包使用，需要添加tests/__init__.py文件，不过这个文件可以为空，因为`unittest`包会扫描所有模块并查找测试

