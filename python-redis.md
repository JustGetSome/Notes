### Basic operation

Connection:

    import redis
    conn = redis.Redis(host='localhost', port=6379)

Basic set and get:

    conn.set('A', 'apple')
    conn.get('A')

设置过期时间：

    setex(name, value, time)

Redis的连接实例据说是线程安全的，很6，另外可以用维持一个连接池的方式来减少每次短连接带来的开销：

    pool = redis.ConnectionPool(host='localhost', port=6379)
    conn = redis.Redis(connection_pool=pool)

Redis提供了`pipeline`对象来支持批量操作，据说有原子性：

    pipe = conn.piprline()  # transaction 参数默认为 True
    conn.set(bla bla)
    conn.setex(bla bla)
    conn.mget(bla bla)
    pipe.execute()


### 一些应用场景

* 访问统计


    page1 = select count from page1
    conn.set("page:1", page1)
    for every visit:
        conn.incr("page:1")
    conn.get("page:1") if page1 has been loaded and show()



* 社交圈应用

根据社交用户的个人信息设置社交圈，建立一些集合网络(感觉MongoDB更适合 XD )，通过集合可以向一些新加入的用户推荐好友以及圈子等：

    conn.sadd('game:WOW', 'user:a')
    conn.sadd('game:WOW', 'user:b')
    conn.sadd('game:WOW', 'user:c')
    conn.sadd('sport:basketball', 'user:c')
    conn.sadd('sport:basketball', 'user:d')
    conn.sadd('sport:basketball', 'user:e')

获取某圈子成员：

    conn.smembers('game:WOW')

通过集合运算找到共同成员：

    conn.sinter('game:WOW', 'sport:basketball')



* 消息队列

`Master`负责作业的生产，分发以及获取结果。`Slaver`负责消费作业并返回结果。

Sample:

    import time, threading
    import random, redis

    HOST = 'localhost'
    PORT = 6379
    chd = 'CHANNEL_DISPATCH'
    chr = 'CHANNEL_RESULT'


    def Master():
        def start():
            ServerResultHandleThread().start()
            ServerDispatchThread().start()
        return start


    class ServerDispatchThread(threading.Thread):
        def __init__(self):
            super(ServerDispatchThread, self).__init__()

        def run(self):
            conn = redis.Redis(HOST, PORT)
            for i in range(100):
                channel = chd + '_' + str(random.randint(1, 5))
                print 'Dispatch job {} to {}'.format(str(i), channel)
                ret = conn.publish(channel, str(i))
                if not ret:
                    print 'Dispatch failed'
                    time.sleep(3)


    class ServerResultHandleThread(threading.Thread):
        def __init__(self):
            super(ServerResultHandleThread, self).__init__()

        def run(self):
            conn = redis.Redis(HOST, PORT)
            p = conn.pubsub()
            p.subcribe(chr)
            for res in p.listen():
                if res['type'] != 'message': continue
                print 'Received job {}'.format(res['data'])


    def Slaver():
        def start():
            for i in range(1, 6):
                WorkerThread(chd+'_'+str(i)).start()
        return start


    class WorkerThread(threading.Thread):
        def __init__(self, channel):
            super(WorkerThread, self).__init__()
            self.channel = channel

        def run(self):
            conn = redis.Redis(HOST, PORT)
            p = conn.pubsub()
            p.subcribe(self.channel)
            for job in p.listen():
                if job['type'] != 'message': continue
                print 'Worker{} run job {}'.format(self.channel, job['data'])
                time.sleep(2)
                print 'Job finished'
                ret = r.publish(chr, job['data'])
                if not ret:
                    print 'Worker{} return job {} failed'.format(self.channel, job['data'])


    if __name__ == '__main__':
        Slaver = Slaver()
        Master = Master()
        Slaver()
        Master()        
