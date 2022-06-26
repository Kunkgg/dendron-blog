---
id: m1lynul9moi6nnbdr50ay81
title: How to Integrate Django and Vue CLI
desc: ''
updated: 1656231171713
created: 1656231044515
---

# 如何集成 Django 和 Vue CLI -- 不依赖额外第三方包

搭建 Django 和 Vue 的集成开发环境有多种方式，
这里总结一种不依赖额外第三方包的方式。

## [html-webpack-plugin](https://webpack.js.org/plugins/html-webpack-plugin/)

html-webpack-plugin, 是 webpack 最常使用的 plugin 之一。 
在每次 webpack 打包过程中, 通常静态资源文件名都会包含一段 hash, 而且每次打包都不一样,
通过手动的方式把这些名称不断变化的静态资源引入 html 是不可能的。
html-webpack-plugin 可以自动生成包含静态资源的 html 文件。

Vue CLI 已经默认安装了 html-wepack-plugin。 可以通过 `vue inspect` 命令查看 vue 的 `webpack` 配置。
其中 plugins 部分一定包含了 html-wepack-plugin。

这个方案就是利用 html-webpack-plugin 的能力, 
生成包含打包的静态资源标签的模版文件到 Django 模版路径下,
再通过 Django 模版继承在 Django 中作为前端页面使用。

## 工具版本

### Django 版本 `4.0.5`

### Vue CLI 版本 `5.0.6`

如果使用的 Django, Vue CLI 版本不同, 配置可能略有不同。

## 配置

### 目录结构

```
.
├── backend                   # Django 后端目录
│   ├── db.sqlite3
│   ├── example               # django startproject 生成的目录
│   │   ├── asgi.py
│   │   ├── __init__.py
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── manage.py
│   ├── static                # 静态文件目录
│   │   └── dist              # Vue build 打包输出目录
│   └── templates             # 模版目录
│       ├── base-vue.html     # html-webpack-plugin 注入静态资源的 base 模版
│       ├── base.html         # 未注入静态资源的 base 模版
│       └── index.html        # 继承 base-vue.html, 最终在页面渲染显示
└── frontend                  # Vue 前端目录
    ├── babel.config.js
    ├── jsconfig.json
    ├── package.json
    ├── package-lock.json
    ├── public
    │   ├── favicon.ico
    │   └── index.html
    ├── README.md
    ├── src
    │   ├── App.vue
    │   ├── assets
    │   ├── components
    │   └── main.js
    └── vue.config.js         # Vue 配置文件
```

### Django 方面配置

1. 创建 Django 项目

```
pip install django

django-admin startproject example

mv example backend
```

建议先创建 python 虚拟环境, 再在虚拟环境中创建 Django 项目。

`example` 为举例的 project name, 根据实际情况替换。

为了便于区分, 这里把 django 的 BASE_DIR 重命名为了 `backend`

2. 配置模版

创建 `backend/templates` 目录, 修改 Django `settings.py` 配置文件。

在默认的 `TEMPLATES` 中的 `DIRS` 加入刚创建的模板路径。

```
TEMPLATES = [
    {
        ...
        'DIRS': [
            BASE_DIR.joinpath('templates'),
        ],
        ...
    },
]

```

创建模版文件 `backend/templates/base.html`

```
{% block html %}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        {% block head %}
            <meta charset="UTF-8">
            <meta http-equiv="X-UA-Compatible" content="IE=edge">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Document</title>
        {% endblock %}
    </head>
    <body>
        {% block body %}
            {% block content %}
        
            {% endblock %}
        {% endblock %}
    </body>
    </html>
{% endblock %}

```

创建模版文件 `backend/templates/index.html`

```
{% extends 'base-vue.html' %}
{% block content %}
    <h1>From Vue Django index.html</h1>
    <div id="app"></div>
{% endblock %}
```

3. 配置静态文件目录

创建静态文件目录 `backend/static`, 修改 Django `settings.py` 配置文件。

添加 `STATICFILES_DIRS` 配置

```
STATICFILES_DIRS = [
    BASE_DIR.joinpath('static'),
]
```

4. 配置 Django 路由

修改 `backend/example/urls.py`, 添加路由关联刚创建的模版文件

```
from django.views.generic import TemplateView

urlpatterns = [
    path('', TemplateView.as_view(template_name='index.html')),
    path('admin/', admin.site.urls),
]
```

### Vue CLI 方面配置

1. 创建 Vue frontend 项目

```
npm install -g @vue/cli

vue create frontend --no-git
```

本篇以 Vue 3 为例

2. 修改 `frontend/vue.config.js` 配置文件:

```
module.exports = {
  publicPath: process.env.NODE_ENV === 'production' ? '/static/dist/' : 'http://localhost:8080',
  outputDir: '../backend/static/dist',
  indexPath: '../../templates/base-vue.html',
  devServer: {
    host: 'localhost',
    port: 8080,
    devMiddleware: {
          writeToDisk: true,
        },
    headers: {
        'Access-Control-Allow-Origin': '*',
    },
  },
}

```

3. 修改 `frontend/public/index.html`, 注入 Django 标签, 用于生成 `base-vue.html`

```
{% extends 'base.html' %}

{% block html %}
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    {% block body %}
      {{ block.super }}
    {% endblock %}
    <!-- built files will be auto injected -->
  </body>
</html>

{% endblock %}
```

## 使用

1. 启动 Django

```
python manage.py runserver
```

2. 启动 Vue DevServer

```
npm run serve
```

3. 打包为用于部署的静态文件, 输出至 `backend/static/dist` 目录

```
npm run build
```

## 参考

- [django-vue-cli-webpack-demo](https://github.com/EugeneDae/django-vue-cli-webpack-demo)
- [How to use HtmlWebpackPlugin to load Webpack bundle in Django](https://www.accordbox.com/blog/how-use-htmlwebpackplugin-load-webpack-bundle-django/)
- [Why would one prefer 'django-webpack-loader' over 'html-webpack-plugin' #209](https://github.com/django-webpack/django-webpack-loader/issues/209)
[Vue CLI Guide](https://cli.vuejs.org/guide/)