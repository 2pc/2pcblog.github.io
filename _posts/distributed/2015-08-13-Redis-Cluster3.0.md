---
layout: post
category : redis
tagline: "Redis-Cluster"
tags : [distributed, redis3.x]
---
{% include JB/setup %}
网上看着搭建一套redis集群好麻烦，搜了一圈blog,发现还是官网靠谱[Redis cluster tutorial](http://redis.io/topics/cluster-tutorial)
安装ruby（ruby安装我还是比较烦的，最开始ruby安装复杂，切jekyll需要本地编译而放弃了）

解压修改配置文件redis.conf,然后依次拷贝到其他节点，注意修改端口

```java
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

此时启动redis

```java
src/redis-server ./redis.conf
```

会出现如下日志

```java
18447:M 13 Aug 07:57:37.148 * Increased maximum number of open files to 10032 (it was originally set to 1024).
18447:M 13 Aug 07:57:37.148 * No cluster configuration found, I'm c8b35a4356445746a9855d384b1c2111eacded8c
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 3.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in cluster mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 10002
 |    `-._   `._    /     _.-'    |     PID: 18447
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

18447:M 13 Aug 07:57:37.158 # Server started, Redis version 3.0.3
18447:M 13 Aug 07:57:37.158 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
18447:M 13 Aug 07:57:37.158 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
18447:M 13 Aug 07:57:37.158 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
18447:M 13 Aug 07:57:37.158 * The server is now ready to accept connections on port 10002
```

建立集群

```java
# src/redis-trib.rb create --replicas 1 172.16.82.186:10001 172.16.82.186:10002 172.16.82.187:10005 172.16.82.187:10006 172.16.82.188:10003 172.16.82.188:10004
>>> Creating cluster
Connecting to node 172.16.82.186:10001: OK
Connecting to node 172.16.82.186:10002: OK
Connecting to node 172.16.82.187:10005: OK
Connecting to node 172.16.82.187:10006: OK
Connecting to node 172.16.82.188:10003: OK
Connecting to node 172.16.82.188:10004: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
172.16.82.186:10001
172.16.82.187:10005
172.16.82.188:10003
Adding replica 172.16.82.187:10006 to 172.16.82.186:10001
Adding replica 172.16.82.186:10002 to 172.16.82.187:10005
Adding replica 172.16.82.188:10004 to 172.16.82.188:10003
M: ea91dcad05e9d5b1ff46586a5cc7380133bdc72d 172.16.82.186:10001
   slots:0-5460 (5461 slots) master
S: c8b35a4356445746a9855d384b1c2111eacded8c 172.16.82.186:10002
   replicates 0fdc7426449ce43124b8da11f8d55e430dffaeeb
M: 0fdc7426449ce43124b8da11f8d55e430dffaeeb 172.16.82.187:10005
   slots:5461-10922 (5462 slots) master
S: 52018afc6822a0fe37b873d1ac1502c8c4eaf576 172.16.82.187:10006
   replicates ea91dcad05e9d5b1ff46586a5cc7380133bdc72d
M: 1fa25db9c20217a1adacdcd6705747ec58daac21 172.16.82.188:10003
   slots:10923-16383 (5461 slots) master
S: da09b9f765300ab64915685c2e5570da97d28813 172.16.82.188:10004
   replicates 1fa25db9c20217a1adacdcd6705747ec58daac21
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 172.16.82.186:10001)
M: ea91dcad05e9d5b1ff46586a5cc7380133bdc72d 172.16.82.186:10001
   slots:0-5460 (5461 slots) master
M: c8b35a4356445746a9855d384b1c2111eacded8c 172.16.82.186:10002
   slots: (0 slots) master
   replicates 0fdc7426449ce43124b8da11f8d55e430dffaeeb
M: 0fdc7426449ce43124b8da11f8d55e430dffaeeb 172.16.82.187:10005
   slots:5461-10922 (5462 slots) master
M: 52018afc6822a0fe37b873d1ac1502c8c4eaf576 172.16.82.187:10006
   slots: (0 slots) master
   replicates ea91dcad05e9d5b1ff46586a5cc7380133bdc72d
M: 1fa25db9c20217a1adacdcd6705747ec58daac21 172.16.82.188:10003
   slots:10923-16383 (5461 slots) master
M: da09b9f765300ab64915685c2e5570da97d28813 172.16.82.188:10004
   slots: (0 slots) master
   replicates 1fa25db9c20217a1adacdcd6705747ec58daac21
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

命令行下各种操作还不熟悉，试了下jedis3.x差不多

```java
Set<HostAndPort> jedisClusterNodes = new HashSet<HostAndPort>();  
jedisClusterNodes.add(new HostAndPort("172.16.82.186", 10001));  
jedisClusterNodes.add(new HostAndPort("172.16.82.186", 10002));  
jedisClusterNodes.add(new HostAndPort("172.16.82.187", 10005));  
jedisClusterNodes.add(new HostAndPort("172.16.82.187", 10006));  
jedisClusterNodes.add(new HostAndPort("172.16.82.188", 10003));  
jedisClusterNodes.add(new HostAndPort("172.16.82.188", 10004));  
JedisCluster jc = new JedisCluster(jedisClusterNodes);  
Random r = new Random(10000000);
System.out.println(new String(jc.get("k".getBytes())));
for (int i = 0; i < 1000000; i++) {
	String k = r.nextLong()+""+i;
	System.out.println(k);
	String v = r.nextLong()+""+i;
	jc.set(k, v);
	}
System.out.println(jc.del("*".getBytes()));
System.out.println("====");
```
