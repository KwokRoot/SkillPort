# ES集群性能优化及维护
###### **注：集群 elasticsearch 版本为 v7.2.1。**
---
### 1.ES索引刷新间隔设置：
		index.refresh_interval 刷新时间，默认1
		
		PUT index(_all)/_settings?preserve_existing=false

		{
		  "index.refresh_interval": "15s"
		}


### 2.ES索引备份数设置：
		index.number_of_replicas 备份数，默认1

		PUT index/_settings?preserve_existing=false

		{
		  "index.number_of_replicas": "1"
		}

### 3.ES索引分片数设置：
		index.number_of_shards：分片数，默认1，最大：1024
		注：在创建索引时设置！

		例：
			{
			  "settings": {
			    "index": {
			      "number_of_shards": 5,
			      "number_of_replicas": 1,
			      "refresh_interval": "5s",
			      "max_result_window": "50000"
			    }
			  }"aliases": {
			    "aliase": {}
			  },
			  "mappings": {}
			}

### 4.ES搜索返回结果数最大10000条数据设置：
		index.max_result_window: 10000 (默认)

		PUT _all/_settings?preserve_existing=false

		{
		  "index.max_result_window": "50000"
		}

### 5.ES搜索聚合最大桶数10000设置：
		search.max_buckets：10000(默认)

		方式①：配置文件 elasticsearch.yml
			增加配置项：

			# 单个查询最大的桶数，默认10000
			search.max_buckets: 50000


		方式②：动态修改该配置项：
			PUT _cluster/settings

			{
			  "transient": {
			    "search.max_buckets": "50000"
			  }
			}

### 6.ES集群设置删除性操作不允许使用_all及通配符设置：
	
		PUT /_cluster/settings

		{
		  "persistent": {
		    "action.destructive_requires_name": true
		  }
		}

### 7.ES集群总分片数不足，每台机器最大分片数设置：
		ES7.0.0版本以上，默认只允许1000个分片。
		查询ES集群节点分片数：  
			 _cat/allocation?v

		方式①：配置文件 elasticsearch.yml
			增加配置项：

			cluster.max_shards_per_node: 2000


		方式②：动态修改该配置项：
			_cluster/settings GET、PUT请求

			{
			  "transient": {
			    "cluster": {
			      "max_shards_per_node":2000
			    }
			  }
			}

		恢复之前默认：

			_cluster/settings GET、PUT请求

			{
			  "transient": {
			    "cluster": {
			      "max_shards_per_node":null
			    }
			  }
			}

### 8.ES集群API请求设置与配置文件设置优先级说明：
		集群属性设置方式：
			(1)API方式设置：
				a.临时设置(集群节点全部重启失效)：

					{
					    "transient" : {
					        "xxx" : "yyy"
					    }
					}

				b.永久设置(集群节点全部重启依然生效)：

					PUT /_cluster/settings
					{
					    "persistent" : {
					        "xxx" : "yyy"
					    }
					}

			(2)配置文件设置(集群节点全部重启依然生效)：elasticsearch.yml

		优先级：

			transient cluster settings > persistent cluster settings >	settings in the elasticsearch.yml configuration file.

### 9.ES集群状态异常，一直出现UnassignedShards解决方案：
		
		①查询索引分片信息：
			GET _cat/shards?v
		结果中查找：state 为 UNASSIGNED 的索引。

		②当索引无数据写入时可执行关闭、开启索引操作，重新触发索引分片：
			POST index/_close
			POST index/_open

### 10.ES节点延迟分片分配-临时重启操作：
		index.unassigned.node_left.delayed_timeout 默认 1m

		PUT _all/_settings

		{
		  "settings": {
		    "index.unassigned.node_left.delayed_timeout": "5m"
		  }
		}

### 11.ES节点禁用分片转移-临时重启操作：
	
		PUT _cluster/settings

		{
		  "transient": {
		    "cluster.routing.allocation.enable": "none"
		  }
		}

		恢复分片转移：

		PUT _cluster/settings

		{
		  "transient": {
		    "cluster.routing.allocation.enable": "all"
		  }
		}

### 12.ES节点手动分片转移-永久下线操作：

		PUT _cluster/settings

		{
		  "transient": {
		    "cluster.routing.allocation.exclude._name": "node-1,node-2"
		  }
		}

		重新加入集群：

		PUT _cluster/settings

		{
		  "transient": {
		    "cluster.routing.allocation.exclude._name": ""
		  }
		}
