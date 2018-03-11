## WSGI (Web Server Gateway Interface)

### About Http

* Request

    * 请求方法 Method

    * 请求地址 URL

    * 请求内容

    * 请求头部 Header

    * 请求环境信息 Environ

* Response

    * 状态码 Status Code

    * 响应数据

    * 响应头部

### About Application

WSGI规定Python Web App必须是一个Callable对象，并接受WSGI传递的`environ`和`start_response`参数，并返回以一个Iterable对象。P.S. :

1. `environ` 是一个包含Http信息的字典
2. `start_response`是一个可调用的函数，接受来自`app`的`status`和`reqponse_headers`参数
3. `environ`和`start_response`由HttpServer生成

Sample:

    def app(env, start_resp):
        resp_body = 'Request source is {}.'.format(env['PATH_INFO'])
        status = '200 OK'
        resp_header = [
                        ('Content-Type', 'text/plain'),
                        ('Content-Length', str(len(resp_body)))
                        ]
        start_resp(status, resp_header)
        return [resp_body]


### Middleware

在一个完整的Web Application流程中，有些程序的作用看起来介于服务器和Web应用之间，我们称之为中间件，它被一方调用，同时又调用另一方，类似代理或管道一般，连接了服务器和应用，可对中间数据进行一些处理，So it looks like:

![middleware](http://ww4.sinaimg.cn/large/005yyi5Jjw1em2p14p5z9j30rs0cqwg1.jpg)

一些可能的应用场景：
* url routing
* 把一个请求传递给多个应用
* 均衡负载
* 应答过滤

Sample:

    class MW_router(object):

        def __init__(self):
            self.path = dict()

        def __call__(self, path):
            def wrapper(app):
                self.path[path] = app
            return wrapper

        def route(self, env, start_resp):
            req_path = env['PATH_INFO']
            app = self.path[req_path]
            return app(env, start_resp)

    router = MW_router()

    @router('/hello')
    def hello(env, start_resp):
        resp_body = 'Request source is {}.'.format(env['PATH_INFO'])
        status = '200 OK'
        resp_header = [
                        ('Content-Type', 'text/plain'),
                        ('Content-Length', str(len(resp_body)))
                        ]
        start_resp(status, resp_header)
        return [resp_body]

    @router('/hi')
    def hi(env, start_resp):
        resp_body = 'Request source is {}.'.format(env['PATH_INFO'])
        status = '200 OK'
        resp_header = [
                        ('Content-Type', 'text/plain'),
                        ('Content-Length', str(len(resp_body)))
                        ]
        start_resp(status, resp_header)
        return [resp_body]

    res = router.route(env, start_resp)


>以上代码看似实现了多个应用处理一个请求，其实，不存在的 ; )

