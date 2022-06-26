---
id: m6julyz338jprui7eohujt7
title: How to Integrate Django and Vue CLI by Webpack Loader
desc: ''
updated: 1656231113447
created: 1656231095508
---

# 如何集成 Django 和 Vue CLI -- Extension: django-webpack-loader

搭建 Django 和 Vue 的集成开发环境有多种方式，
本篇总结如何利用 Django 扩展包 `django-webpack-loader` 
配合 node 包 `webpack-bundle-tracker` 实现 Django 和 Vue 的集成.

这种方式的优点是配置简单易于理解， 缺点是需要安装额外的第三方包。

## 工具

### [webpack-bundle-tracker](https://github.com/django-webpack/webpack-bundle-tracker)

`webpack-bundle-tracker` 是一个 webpack 的 pulgin,
作用是把 webpack 打包的过程信息存储在一个 json 文件中。

通过 npm 安装, 本文档中使用当前最新版本 `1.6.0`。

### [django-webpack-loader](https://github.com/django-webpack/django-webpack-loader)

`django-webpack-loader` 是一个 django 的第三方扩展。
通过读取 `webpack-bundle-tracker` 生成的打包过程信息, 获取静态资源路径。
在 django 模版文件使用自定义标签 `render_bundle` 把打包生成的静态资源文件引入模版文件。

通过 python pip 安装, 本文档中使用当前最新版本 `1.6.0`。

### Django 版本 `4.0.5`

### Vue CLI 版本 `5.0.6`

## 配置

如果使用的 Django, Vue CLI, webpack-bundle-tracker 和 django-webpack-loader 版本不同,
配置可能略有不同。

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
│       ├── base.html
│       └── index.html
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
    ├── vue.config.js         # Vue 配置文件
    └── webpack-stats.json    # webpack-bundle-tracker 生成的 webpack 打包过程信息
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

创建模版文件 `backend/templates/index.html`

```
{% load render_bundle from webpack_loader %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue Django Example</title>
</head>
<body>
    <h1>Hello From Django</h1>
    <div id="app"></div>
    {% render_bundle 'app' %}
</body>
</html>
```

3. 配置静态文件目录

创建静态文件目录 `backend/static`, 修改 Django `settings.py` 配置文件。

添加 `STATICFILES_DIRS` 配置

```
STATICFILES_DIRS = [
    BASE_DIR.joinpath('static'),
]
```

4. 安装配置 django-webpack-loader

安装 `pip install django-webpack-loader`

修改 Django `settings.py` 配置文件。

在 `INSTALLED_APPS` 添加 `webpack_loader`

```
INSTALLED_APPS = [
    ...
    'webpack_loader',
    ...
]
```

在 Django `settings.py` 中添加 `django-webpack-loader` 配置:

```
WEBPACK_LOADER = {
    'DEFAULT': {
        'STATS_FILE': BASE_DIR.parent.joinpath('frontend', 'webpack-stats.json'),
    },
}
```

5. 配置 Django 路由

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

2. 安装配置 `webpack-bundle-tracker`

```
npm install --save-dev webpack-bundle-tracker
```

修改 `frontend/vue.config.js` 配置文件:

```
const BundleTracker = require('webpack-bundle-tracker')

// Change this to match the path to your files in production (could be S3, CloudFront, etc.)
const DEPLOYMENT_PATH = '/static/dist/'

module.exports = {
    publicPath: process.env.NODE_ENV === 'production' ? DEPLOYMENT_PATH : 'http://localhost:8080/',
    outputDir: '../backend/static/dist',

    devServer: {
        host: 'localhost',
        port: 8080,
        headers: {
            'Access-Control-Allow-Origin': '*',
        },
    },

    configureWebpack: {
        plugins: [
            new BundleTracker({ path: __dirname, filename: 'webpack-stats.json' }),
        ],
    },
}
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

[Vue.js and Django: Modern Multi-Page Website](https://shumingxu.com/posts/vue-django-multi-page/#vue-js-webpack-setup)
[Vue CLI Guide](https://cli.vuejs.org/guide/)