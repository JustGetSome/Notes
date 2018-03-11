## You don't call Twisted, Twisted calls you.

    from twisted.internet.protocol import Factory
    from twisted.internet.endpoints import TCP4ServerEndpoint
    from twisted.internet import reactor

    class QFactory(Factory):
        
        protocol = Q
        def __init__(self):
            do something...

    endpoint = TCP4ServerEndpoint(reactor, 8888)
    endpoint.listen(QFactory)
    reactor.run()


- Factory管理所有连接，创建protocol对象来处理各个成功的连接，然后由protocol内容接管剩余工作，Endpoint就是Server的抽象对象，可以为其添加监视者以及工厂, reactor即event loop。
