###### 系统版本
	CentOS release 6.6 (Final)
###### Elasticsearch版本
	Elasticsearch-7.2.1

----------

# 环境要求
	1.Java 运行环境。
		①查看 Java 版本(java -version)，保证 Java 版本在 8u40 +。
		②Elasticsearch-7.2.1 版本已自带 OpenJDK 捆绑版本(安装包 jdk 目录中)。注：当环境变量中获取不到 JAVA_HOME 变量值时，会使用自带的 JDK 。

	2.查看硬盘大小(df -h)。ES 数据需要较大硬盘空间，尽量部署在空间较大的目录中。

	3.ES 需要非 root 用户启动，具体操作见 [安装问题1]。


# 部署
	1.解压：
		tar -xvf elasticsearch-7.2.1-linux-x86_64.tar.gz
	2.切换为非 root 用户：
		su es
	3.启动：
		./Elasticsearch-7.2.1/bin/elasticsearch
	  
	  守护进程方式启动：
			nohup ./Elasticsearch-7.2.1/bin/elasticsearch >/dev/null 2>&1 &
		或：
			./Elasticsearch-7.2.1/bin/elasticsearch -d


# 确认部署成功
	1.查看该节点信息：
		http://127.0.0.1:9200/
	2.健康状态查询：
		http://127.0.0.1:9200/_cat/health?v
	3.节点查询：
		http://127.0.0.1:9200/_cat/nodes?v


# 启动日志
	tail -100f logs/集群名.log


# 集群配置
	
	# 集群名称 同一个集群此参数要一致
	cluster.name: elasticsearch
	
	# 节点名称 同一集群节点之间需要不一样
	node.name: node-1
	
	# 绑定本机ip，不是本机ip启动会失败
	network.host: 192.168.3.91

	# 是否为主节点(默认为 true)
	node.master: true

	# 是否为数据节点(默认为 true)
	node.data: true

	# 详看[安装问题2]
	bootstrap.system_call_filter: false
	
	# elasticsearch-head 等插件 for Elasticsearch 5.x+ 跨域问题解决
	http.cors.enabled: true
	# 可设置单个地址
	http.cors.allow-origin: "*"
	
	# 单播发现集群其他节点
	discovery.seed_hosts: ["192.168.3.91", "192.168.3.92", "192.168.3.93"]

	# 生产模式启动一个新集群时，必须显式地列出在第一次选举中应该计算其选票的符合主节点资格的节点
	cluster.initial_master_nodes: ["192.168.3.91", "192.168.3.92", "192.168.3.93"]
	
	# 单个查询最大的桶数，默认10000，特殊需求下才会用到。
	# search.max_buckets: 50000
	
	# ping 超时时间，默认30s 
	discovery.zen.fd.ping_timeout: 60s

	# ping 间隔，默认1s 
	discovery.zen.fd.ping_interval: 10s

# 安装问题

### 问题1：
	问题：
		java.lang.RuntimeException: can not run elasticsearch as root
	
	原因：
		不能使用 root 用户运行 elasticsearch
	
	解决：
		创建其他用户并赋予操作目录权限
	
	步骤：
		root 用户(或使用root权限)操作：
			①创建 es 用户：useradd es
			②创建 es 用户密码：passwd 123456
			③创建 /opt/elasticsearch 目录：mkdir elsticsearch
			④赋予 es 用户操作 /opt/elasticsearch 目录权限：chmod 777 -R /opt/elasticsearch
	
### 问题2：
	问题：
		java.lang.UnsupportedOperationException: seccomp unavailable: requires kernel 3.5+ with CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER compiled
	原因：
		centos6的内核不支持SecComp
	解决：
		config/elasticsearch.yml 中加入 bootstrap.system_call_filter: false


### 问题3：
	问题：
		ERROR: [1] bootstrap checks failed
		[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
	原因：
		操作系统vm.max_map_count参数设置太小
	解决：
		root 用户(或使用root权限)操作：
			临时设置：
				① 设置：sysctl -w vm.max_map_count=262144
				② 查看：sysctl -a | grep "vm.max_map_count"
			永久性设置:
				① cp /etc/sysctl.conf /etc/sysctl.conf.bak
				② vim /etc/sysctl.conf
					追加：
					# elasticsearch config start
					vm.max_map_count=262144
					# elasticsearch config end

### 问题4：
	问题：
		java.lang.OutOfMemoryError: Java heap space
		Dumping heap to data/java_pidxxxx.hprof ...
		Heap dump file created [1079957084 bytes in 2.366 secs]
	
	原因：
		Elasticsearch JVM 默认使用堆大小过小
	解决：
		① 修改 config/jvm.options
		② vim config/jvm.options
			修改：
				-Xms1g
				-Xmx1g
			为：
				-Xms2g
				-Xmx2g
			
	建议（需验证）：
    	① 将最小堆大小（Xms）和最大堆大小（Xmx）设置为彼此相等。
    	② Elasticsearch可用的堆越多，它可用于缓存的内存就越多。但请注意，过多的堆可能会陷入长时间的垃圾收集暂停。所以设置的堆不能太大， 尽量设置到内存的50%。
    	③ 将Xmx设置为不超过物理RAM的50％，以确保有足够的物理内存给内核文件系统缓存。
    	④ 内存heap size配置不要超过32G, 基本上大多数系统最多只配置到30G.

### 问题5：
	问题：
		集群不进行分片分配操作，磁盘剩余空间较小。
	原因：
		ES在磁盘空间占用超过 85% 时不在给该主机分配分片。
	解决：
		排除由于数据原因造成的磁盘占用过大，很可能是日志原因。
		①不产生 nohup.out 日志：nohup ./Elasticsearch-7.2.1/bin/elasticsearch >/dev/null 2>&1 &
		②定期清理(！不是直接删除) nohup.out 日志：cp /dev/null nohup.out
		③配置 elasticsearch/config 目录下日志级别。

### 问题6：
	问题：
		ERROR: [1] bootstrap checks failed
		[1]: max number of threads [1024] for user [es] is too low, increase to at least [4096]
	原因：
		操作系统安全配置影响
	解决：
		① cd /etc/security/limits.d/
		② vim 90-nproc.conf
			修改：
				*          soft    nproc     1024
				root       soft    nproc     unlimited

			为：
				*          soft    nproc     4096
				root       soft    nproc     unlimited

		③ 重新登录 es 用户

### 问题7：
	问题：
		failed to join
		org.elasticsearch.transport.RemoteTransportException: [...][*.*.*.*:9300][internal:cluster/coordination/join]
		Caused by: org.elasticsearch.transport.ConnectTransportException: [...][*.*.*.*:9300] connect_exception
		...		
		Caused by: java.io.IOException: No route to host...
	原因：
		连接失败
	解决：
		查看防火墙设置等
	系统防火墙设置打开 9200、9300 输入端口：
		/sbin/iptables -I INPUT -p tcp --dport 9200 -j ACCEPT
		/sbin/iptables -I INPUT -p tcp --dport 9300 -j ACCEPT
		/etc/rc.d/init.d/iptables save
		/etc/rc.d/init.d/iptables restart

### 问题8：
	问题：
		ES假死：集群列表中丢失该节点，该节点的机器中es进程在，本地访问http端口访问异常，没有错误日志。
	原因：
		暂定为垃圾回收出现的问题，或 短暂网络连接异常造成节点拒绝服务的问题。
	解决：
		①修改jvm.options文件，将下面几行:

			-XX:+UseConcMarkSweepGC
			-XX:CMSInitiatingOccupancyFraction=75
			-XX:+UseCMSInitiatingOccupancyOnly
		更改为：
			-XX:+UseG1GC
			-XX:MaxGCPauseMillis=50

		②配置文件 config/elasticsearch.yml 中添加：
			# ping 超时时间，默认30s 
			discovery.zen.fd.ping_timeout: 60s

### 问题9：
	问题：
		ES插件 Cerebro 错误信息： error[_nodes/stats/jvm,fs,os,process]
	插件错误信息：
		{
		  "error": {
		    "root_cause": [
		      {
		        "type": "illegal_argument_exception",
		        "reason": "Values less than -1 bytes are not supported: -1325121536b"
		      }
		    ],
		    "type": "illegal_argument_exception",
		    "reason": "Values less than -1 bytes are not supported: -1325121536b",
		    "suppressed": [
		      {
		        "type": "illegal_state_exception",
		        "reason": "Failed to close the XContentBuilder",
		        "caused_by": {
		          "type": "i_o_exception",
		          "reason": "Unclosed object or array found"
		        }
		      }
		    ]
		  },
		  "status": 400
		}

	ES错误信息：
		[o.e.x.m.MonitoringService] [es_real_m02] monitoring execution failed
		org.elasticsearch.xpack.monitoring.exporter.ExportException: failed to add documents to export bulks
		......

	原因：
		获取节点信息失败，可能节点系统信息有问题。
		如 Swap 异常，被用 18446744073707805560 used 显然是机器信息错误，统计的结果会超出插件的允许值范围。
	解决：
		重启节点服务器。

