# [Flask知识总结](https://juejin.im/entry/5af569076fb9a07aa34a5077)

## 

```bash
pip install flask-wtf
```

## 流程

视图函数坐镇八方：关联URL、静态模板、以及逻辑类的关系。

```python
# app/route.py
from flask import render_template, flash, redirect

@app.route('/login', method=['GET', 'POST'])
def login():
    from = LoginFrom()
    if form.validate_onsubmit():
        flash('Login requested for user {}, remember_me={}'.format(form.username.data, form.remeber_me.data))
        return redirect('/index')
    return render_template('login.html', title='Sign In', form=form)
```



静态模板之间可以继承来保证页面的风格一致而无需太多劳动。

```php+HTML
<!--base.html-->
<html>
    <head>
        {% if title %}
        <title>{{title}} - microblog</title>
        {% else %}
        <title>microblog</title>
        {% endif %}
    </head>
    <body>
        <div>
            Microblog:
            <a href='/index'>Home</a>
            <a href='/login'>Login</a>
        </div>
        <hr>
        {% with messages = get_flashed_messaged() %}
        {% if messages %}
        <ul>
            {% for message in messages %}
            <li>{{ messge }}</li>
        	{% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </body>
</html>
```



```html
<!--login.html-->
{% extends "base.html" %}
{% block content%}
	<form action="" method="post" novalidate>
		{{ form.hidden_tag()}}
		<p>
			{{ form.username.label }}<br>
			{{ form.username(size=32) }}
		</p>
	</form>
{% endblock %}
```

### 配置文件

利用类来存储配置变量，并将其导入。

首先从环境变量中读取配置，为获取到就使用默认值，这样做是个好习惯。

```python
# config.py
import os 
class Config(object):
    SECTET_KEY = os.environ.get('SECRET+KEY') or 'you-will-never-geuss'
```

```python
# app/__init__.py
from flask import Flask
from config import Config

app = Flask(__name__)
app.config.forom_object(Config)

from app import routes
```

### 字段验证

已经有了错误检查机制和错误描述，需要做的就是将错误信息渲染出来。

```html
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}<br>
            {% for error in form.username.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
```

### 生成链接

为了减少工作量和日后维护方便，应将硬编码的URL替换为url_for()的视图函数名称。

## 数据库

### Flask-SQLAlchemy：操作数据库

操作数据库的包统称Object Relational Mapper（ORM）。其允许应用程序使用高级实体，如类、对象和方法，而不是表和SQL来管理数据库。

SQLAlchemy即是典型的ORM，支持MySQL、PostgreSQL和SQLite在内的很多数据库软件。可以在开发过程中灵活选择数据库而无需大费周章。如，在开发初期使用简单的SQLite，部署到服务器上时，则选用更健壮的MySQL或PostgreSQL服务，此过程中不需要修改应用代码，仅修改应用配置即可。

Flask-SQLAIchemy是对SQLAIchemy的针对Flask的封装。

### Flask-Migrate：数据库迁移

Flask-Migrate是对插件Alembic的一个Flask封装，可以使将来的数据库结构在更新数据库或添加表结构时变得更稳健。

### Flask-SQLAlchemy配置

SQLite是开发小型乃至中型应用最方便的选择，因为每个数据库都存储在磁盘上的单个文件中，并且不需要像MySQL和PostgreSQL那样 运行数据库服务。

```python
# config.py
import os 
basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    # ...
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
    	'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

注册Flask插件的模式大抵如此。

```python
# app/__init__.py
from flask import Flask 
from config import Config
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

from app import routes, models
```

### 数据库模型

```python
# app/models.py
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_has = db.Column(db.String(128))
    
    def __repr__(self):
        return '<User {}>'.fromat(self.username)
```

### 创建数据库迁移存储库并迁移

```bash
flask db init
# creating upgrade autoly or manmade
flask db migrate -m "users table"
flask db upgrade
```

有了数据库迁移技术，就可以将比照目标数据库与现有数据库结构的任务自动化，从而将数据库层的任务封装隔离。无论是在开发机还是服务器上都会大大节省人力。

### 加入发表动态后的model.py

```python
from datetime import datetime
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Columm(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))
    posts = db.relationship('Post', backref='author', lazy='dynamic')
    
    def __repr__(self):
        return '<User {}'.format(self.username)
class Post(db.Model):
    id = db.column(db.Integer, primary_key=True)
    body = db.Column(db.String(140))
    # utc is the local time according to the user self
    # datetime.utcnow没有（），所以传递的是函数本身。注意！！！
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    
    def __repr__(self):
        return '<Post {}>'.format(self.body)
```



## Flask子命令

`flask`依赖于`FLASK_APP`环境变量来知道`Flask`应用入口在哪里，本项目中设置为`FLASK_APP = microblog.py`。

```bash
flask run
flask db init #创建迁移数据库
```



## 环境安装

```bash
pip install flask-sqlalchemy
pip install flask-migrate
```



