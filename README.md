# 传送门

[Condition](https://github.com/seaswalker/JDK/blob/master/note/Condition/Condition.md)

[CountDownLatch](https://github.com/seaswalker/JDK/blob/master/note/CountDownLatch/CountDownLatch.md)

[CyclicBarrier](https://github.com/seaswalker/JDK/blob/master/note/CyclicBarrier/CyclicBarrier.md)

[ReadWriteLock](https://github.com/seaswalker/JDK/blob/master/note/ReadWriteLock/ReadWriteLock.md)

[ReentrantLock](https://github.com/seaswalker/JDK/blob/master/note/ReentrantLock/ReentrantLock.md)

[Socket](https://github.com/seaswalker/JDK/blob/master/note/Socket/socket.md)

[UDP](https://github.com/seaswalker/JDK/blob/master/note/UDP/udp.md)

[IO](https://github.com/seaswalker/JDK/blob/master/note/IO/io.md)

[FileChannel](https://github.com/seaswalker/JDK/blob/master/note/FileChannel/filechannel.md)

[Buffer](https://github.com/seaswalker/JDK/blob/master/note/Buffer/buffer.md)

[URLConnection](https://github.com/seaswalker/JDK/blob/master/note/URLConnection/urlconnection.md)

[NIO](https://github.com/seaswalker/JDK/blob/master/note/NIO/nio.md)

[Process](https://github.com/seaswalker/JDK/blob/master/note/Process/process.md)

[HashMap](https://github.com/seaswalker/JDK/blob/master/note/HashMap/hashmap.md)

[LinkedHashMap](https://github.com/seaswalker/JDK/blob/master/note/LinkedHashMap/linkedhashmap.md)

[TreeMap](https://github.com/seaswalker/JDK/blob/master/note/TreeMap/treemap.md)

[ConcurrentHashMap](https://github.com/seaswalker/JDK/blob/master/note/ConcurrentHashMap/concurrenthashmap.md)

[ConcurrentLinkedQueue](https://github.com/seaswalker/JDK/blob/master/note/ConcurrentLinkedQueue/concurrentlinkedqueue.md)

[ThreadPool](https://github.com/seaswalker/JDK/blob/master/note/ThreadPool/threadpool.md)

[ThreadLocal](https://github.com/seaswalker/JDK/blob/master/note/ThreadLocal/threadlocal.md)

[Reflection](https://github.com/seaswalker/JDK/blob/master/note/Reflection/reflection.md)

[ScheduledThreadPool](https://github.com/seaswalker/JDK/blob/master/note/ScheduledThreadPool/scheduledthreadpool.md)

[AsynchronousFileChannel](https://github.com/seaswalker/JDK/blob/master/note/AsynchronousFileChannel/asynchronousfilechannel.md)

[BufferedInputStream](https://github.com/seaswalker/JDK/blob/master/note/BufferedInputStream/bufferedinputstream.md)

[Enum](https://github.com/seaswalker/JDK/blob/master/note/Enum/enum.md)
### 1.ES集群搭建(三个节点)

拷贝三份es(版本为6.1.2)，修改elasticsearch.yml配置文件

```
cluster.name: elastic-test #集群名称，要保持一致。
node.name: node-1 #节点名称，每个节点不一样，系统会自动生成，建议自己命名，利于后期维护。
#node.master: true #是否作为主节点，默认为true
#node.data: true #是否作为数据节点，默认为true，一个节点可以同时作为主节点和数据节点
path.data: D:\elasticsearch1\data #es数据路径
path.logs: D:\elasticsearch1\logs #es日志路径
bootstrap.memory_lock: true #锁定内存，不做交换，建议打开，特别是机器内存小的时候
network.host: localhost #es对外访问ip,当为localhost则为开发环境，替换成ip后变为生产环境
http.port: 9200 #es对外端口
transport.tcp.port: 9300 #节点间的通信端口
discovery.zen.ping.unicast.hosts: ["localhost:9300", "localhost:9301", "localhost:9302"] #节点间的发现，用于节点加入集群
discovery.zen.minimum_master_nodes: 2 #集群所需最少主节点，值设定为（总的主节点数/2 + 1）
action.destructive_requires_name: true #删除索引时，是否再说一次指定名称，该值为false时，可以通过正则或_all进行删除。
http.cors.enabled: true #设置是否激活插件访问
http.cors.allow-origin: "*" #设置允许访问的源，*表示所有源都可以访问
http.cors.allow-credentials: true #设置是否允许证书访问，本次未设置此项
# 线程池的配置（es内部维护了多个线程池，如index、search、get、bulk）
#thread_pool.index.type: fixed，经测试所有的type设置已经失效，网上的还有此配置，可能版本不一致。
thread_pool.index.size: 3 # 经测试，size最大值为cpu核数+1，数值过大es启动报错。
thread_pool.index.queue_size: 1000 # 队列数量
thread_pool.bulk.size: 3
thread_pool.bulk.queue_size: 1000
```

#### 1.1：格式规范

```
elasticsearch.yml配置文件中，如
	cluster.name: elastic-test
：后面必须有一个空格，否则报错。
```

#### 1.2：主节点和数据节点的配置

##### 1.2.1.默认配置

```
node.master: true
node.data: true
节点既可以竞争主节点也可以存储数据，对于主节点压力会增大
```

##### 1.2.2.数据存储

```
node.master: false
node.data: true
节点只用于数据存储，不竞争主节点，可作为负载器
```

##### 1.2.3.节点协调

```
node.master: true
node.data: false
节点只用于竞争主节点，不存储数据，保有空闲资源，可作为协调器
```

##### 1.2.4.数据搜索

```
node.master: false
node.data: false
节点不存储数据，也不竞争主节点，可作为一个搜索器，从节点中获取数据，生成搜索结果
```

### 2.JVM配置

```
-Xms2g #一般分配机器内存的一半，且与-Xmx保持一致，此处分配2g，因底层Lucene处理需要使用内存，实际该节点占用内存为2-4g
-Xmx2g
-Dfile.encoding=UTF-8 #如果es或者logstash控制台输出中文乱码，可将该值修改为GBK
```

### 3.Logstash配置

修改logstash.conf配置文件

3.1 filter插件中的Ruby代码，超过15天的日志不读取。

```
ruby {
	code => "
		current = Time.now
		today = Time.local(current.year,current.month,current.day)
		timespot = (today - 15 * 24 * 60 * 60).to_i
		logtime = Time.parse(event.get('logdate')).to_i
		if (timespot >= logtime)
			event.set('hisdata','Y')
		else
			event.set('hisdata','N')
		end
	"
}
```

3.2 模板配置

es默认索引有5个分片，项目中日志量不大，不需要这么多分片，可以使用模板来指定分片数量，要指定索引其他配置也是一样可以的。

模板内容：由于logstash.conf中的变量无法传递给模板，所以需要以固定自符串开头

```
{
	"template": "logs*",
	"settings": {
		"index.number_of_shards": "1"
	}
}
```

使用模板logstash.conf中output应为

```
output {
	if [hisdata] == "N" {
		elasticsearch {
			index => "logs-%{type}-%{+yyyy.MM.dd}" #需要以固定自符串开头
			hosts => ["localhost:9200"]
			codec => plain {
				charset => ["UTF-8"]
			}
			template => {
                "template.json"
			}
			template_name => "123456" #随意
			template_overwrite => true
		}
	}
}
```

3.3 output插件中的代码，历史数据直接过滤

```
output {
	if [hisdata] == "N" {
		elasticsearch {
			index => "logs-%{type}-%{+yyyy.MM.dd}"
			hosts => ["localhost:9200"]
			codec => plain {
				charset => ["UTF-8"]
			}
		}
	}
}
```

### 4.定时删除es索引

以下为python脚本，需要搭建python环境，设置环境变量，版本为python-3.6.2。

脚本写完后要添加到windows的任务计划里。

```
import urllib
import urllib.request
import re
import datetime
import time

def getIndices(es_url):
    es_request = urllib.request.Request(es_url)
    es_response_indices = urllib.request.urlopen(es_request).read()
    return es_response_indices

def match(indices):
    pattern = re.compile(r'\d{4}\.\d{2}\.\d{2}') #正则匹配yyyy.MM.dd的索引
    indices = indices.decode('utf-8')
    date_match = pattern.findall(indices)
    date_now = datetime.date.today()
    days_before_7 = date_now - datetime.timedelta(days=15) #删除15天以前的日志
    date_format = days_before_7.__format__('%Y.%m.%d')
    types = {'type1','type2','type3'} #索引的前缀，比如索引为logs1-2018.11.22，logs1就是type
    for index_format in date_match:
        for type in types:
            if date_format > index_format:
                deleteIndicex(index_format,type)
            else:
                pass

def deleteIndicex(date_wanna_delete,type):
    url_delete = 'http://localhost:9200/%s-%s?pretty' % (type,date_wanna_delete)
    print(url_delete)
    es_request_delete = urllib.request.Request(url_delete)
    es_request_delete.get_method = lambda:'DELETE'
    try:
        response_delete = urllib.request.urlopen(es_request_delete).read()
    except:
        pass
    else:
        return response_delete
    
if __name__ == '__main__':
    es_url = 'http://localhost:9200/_cat/indices?v'
    match(getIndices(es_url))
    print('delete success!')
```

### 5.问题及处理

#### 5.1 error1 映射出错

```
failed to put mappings on indices [[[logs4-2018.10.12/d4QBGUZ-RIGTkEt1jaFsKw]]], type [doc]
org.elasticsearch.cluster.metadata.ProcessClusterEventTimeoutException: failed to process cluster event (put-mapping) within 30s
```

该错误可以考虑自定义mapping，由于日志系统是通过logstash直接生成的索引，无法自定义，output中需要使用到template。

#### 5.2 error2 频繁垃圾回收

```
[WARN ][o.e.m.j.JvmGcMonitorService] [node-1] [gc][3856] overhead, spent [2.3s] collecting in the last [3s]
```

过于频繁的垃圾回收，一般xms和xmx过低造成的

#### 5.3 error3 Ruby编解码错误

```
Grok regexp threw exception {:exception=>"incompatible encoding regexp match (UTF-8 regexp with ASCII-8BIT string)
```

项目中同一个filebeat传过来的数据编码有UTF-8和GBK两种，而input中只配了一种，考虑可能是这个原因。建议不同的编码使用不同的filebeat传输(未验证。)。input中加入

```
input {
    beats {
		host => "127.0.0.1"
		port => 5044
		codec => plain {
			charset => ["UTF-8"] #如GBK则写GBK
		}
	}
}
```

#### 5.4 error4 读取大量历史日志报错

Logstash日志出现大量429错误码，提示es拒绝了请求，导致logstash队列中数据积累越来越多，这是由于ES的写入瓶颈，但是官方不建议修改es的线程数。

解决方案一：

	增大logstash.yml文件配置，提升logstash的吞吐量

```
pipeline.workers: 5 #官方建议等于CPU数，可以设置稍大一点
pipeline.output.workers: 5 #实际output时的线程数
pipeline.batch.size: 1000 #每次发送的事件数，一条日志被称作一个事件，主要是修改这个值，需要反复测试来获取最优值。
pipeline.batch.delay: 10 #发送延时
```

解决方案二：

	修改elasticsearch.yml配置文件

```
thread_pool.index.size: 3 
thread_pool.index.queue_size: 1000
thread_pool.bulk.size: 3
thread_pool.bulk.queue_size: 1000 # 队列数量,主要是修改这个值，需要反复测试来获取最优值。
```

解决方案三：

	修改es的垃圾回收机制为G1，未使用过。

#### 5.5 es无法新增document

当es所在磁盘超过90%以上，该节点上的索引会变为只读索引，只允许查询和删除操作。

解决办法：

```
put http://localhost:9200/*/_settings?pretty -H "" -d
'{
    "index":{
        "blocks":{
            "read_only_allow_delete":"false"
        }
    }
}'
```

