
redis集群搭建

1.启动示例 配置使用绝对路径

    ./redis-server /path/to/redis.conf

2.集群启动命令(redis客户端先各自启动，后以下命令创建集群,有配置密钥的时候使用 "-a 123456" )

    redis-cli --cluster create 172.16.163.2:6379 172.16.163.3:6379 172.16.163.4:6379 -a 123456 --cluster-replicas 0

3.redis.conf文件配置

    集群设置
    cluster-announce-ip 172.16.163.2
    cluster-announce-tls-port 6379
    cluster-announce-port 0
    cluster-announce-bus-port 16379

    密码设置
    requirepass 123456

    开启集群模式
    cluster-enabled yes

    需要监听所有请求时，注释"bind 127.0.0.1 -::1"
    #bind 127.0.0.1 -::1
    
    protected-mode yes
    
    定义暴露的端口
    port 6379
    
    是否后台启动
    daemonize no 
    
    定义日志路径
    logfile /var/log/redis/redis.log
    dir ./

4.相关错误

    错误1:
        [ERR] Node 172.16.163.3:6379 is not empty. Either the node
        already knows other nodes (check with CLUSTER NODES)
        or contains some key in database 0.
    
    错误2:
        Waiting for the cluster to join
    解决方式:
        查看redis.conf文件里的"
                cluster-announce-ip 172.16.163.2
                cluster-announce-tls-port 6379
                cluster-announce-port 0
                cluster-announce-bus-port 16379
                "是否有配置

5.use java to operation redis cluster

    <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <versiong>3.3.0</versiong>
    </dependency>

    public class Cluster {
        public static void main(String[] args) {
            JedisPoolConfig poolConfig = new JedisPoolConfig();
            // 最大连接数
            poolConfig.setMaxTotal(1);
            // 最大空闲数
            poolConfig.setMaxIdle(1);
            // 最大允许等待时间，如果超过这个时间还未获取到连接，则会报JedisException异常：
            // Could not get a resource from the pool
            poolConfig.setMaxWaitMillis(1000);
            Set<HostAndPort> nodes = new LinkedHashSet<HostAndPort>();
            nodes.add(new HostAndPort("172.16.163.2", 6379));
            nodes.add(new HostAndPort("172.16.163.3", 6379));
            nodes.add(new HostAndPort("172.16.163.4", 6379));
    
            JedisCluster cluster = new JedisCluster(nodes,10,10,100,"123456", poolConfig);
    
            cluster.set("a", "1");
            cluster.set("b", "2");
            cluster.set("c", "3");
            cluster.set("d", "4");
    
            SetParams setParams = new SetParams();
            setParams.nx();
            setParams.ex(1000);
            cluster.set("lock","lock",setParams);
    
            //cluster.zadd("1",1,"1111");
            //cluster.zadd("1",3,"3333");
            //cluster.zadd("1",0,"0000");
    
            //Set<String> zRange = cluster.zrange("1", -10, 100);
            //System.out.println(zRange);
            System.out.println(cluster.get("age"));
            cluster.close();
        }
    }

