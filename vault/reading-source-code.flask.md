---
id: 7cm5owhntfy7sanywya6zhf
title: Flask
desc: ''
updated: 1678618062198
created: 1678586510048
---

## 信息

- repos: https://github.com/pallets/flask

## tag 0.1

### 环境准备

Flask tag 0.1 的代码需要在 Python 2.7 环境运行,
我用的是 `2.7.18`.

- 虚拟环境创建:
1. 使用 `pyenv local 2.7.18` 指定目录的 Python 版本.
2. 用 `virtualenv -p <Python 解释器路径> venv` 创建虚拟环境
3. `source venv/bin/activate` 激活虚拟环境

- 按照代码库 readme 提示安装 2 个依赖:
1. Jinja 2.4
2. Werkzeug 0.6.1

*TIP*: Python 2 安装依赖可以使用 `easy_install`

### 概况

Flask tag 0.1 代码量只有 600 行, 1 个单文件 flask.py

核心类: Flask

### 核心代码

- Flask.wsgi_app 方法, 在一个请求上下文中, 依次做如下操作:
  1. preprocess_request 处理请求前钩子
  2. dispath_request 委派请求, 理由 werkzeug 的 routing.Map 对象,
  匹配请求对应的视图函数
  3. process_response 处理请求后钩子
  4. 返回 Response 对象

```python
class Flask:
    ...
    def run(self, host='localhost', port=5000, **options):
        """Runs the application on a local development server.  If the
        :attr:`debug` flag is set the server will automatically reload
        for code changes and show a debugger in case an exception happened.

        :param host: the hostname to listen on.  set this to ``'0.0.0.0'``
                     to have the server available externally as well.
        :param port: the port of the webserver
        :param options: the options to be forwarded to the underlying
                        Werkzeug server.  See :func:`werkzeug.run_simple`
                        for more information.
        """
        from werkzeug import run_simple
        if 'debug' in options:
            self.debug = options.pop('debug')
        options.setdefault('use_reloader', self.debug)
        options.setdefault('use_debugger', self.debug)
        return run_simple(host, port, self, **options)


    def wsgi_app(self, environ, start_response):
        """The actual WSGI application.  This is not implemented in
        `__call__` so that middlewares can be applied:

            app.wsgi_app = MyMiddleware(app.wsgi_app)

        :param environ: a WSGI environment
        :param start_response: a callable accepting a status code,
                               a list of headers and an optional
                               exception context to start the response
        """
        with self.request_context(environ):
            rv = self.preprocess_request()
            if rv is None:
                rv = self.dispatch_request()
            response = self.make_response(rv)
            response = self.process_response(response)
            return response(environ, start_response)
    ...
```

### 关键点

1. Werkzeug Local
1. Flask.route 装饰器
1. Flask.before_request 和 Flask.after_request 装饰器
1. werkzeug Map 匹配路由的时候引入中间对象 MapAdapter 增加代码可读性.

### 疑问

1. 为什么 Flask 路由映射需要搞成 2 级的？

