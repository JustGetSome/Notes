## WSGI means Web Server Gateway Interface

### About Http：

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

