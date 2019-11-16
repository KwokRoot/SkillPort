# ES索引生命周期设置：
---
### 1.创建索引生命周期计划：

  	PUT _ilm/policy/delete_policy


  	{
  	  "policy": {
  	    "phases": {
  	      "delete": {
  	          "min_age": "7d",
  	        "actions": {
  	          "delete" : { }
  	        }
  	      }
  	    }
  	  }
  	}


### 2.索引使用生命周期：

    PUT index/_settings

    {
      "index.lifecycle.name": "delete_policy"
    }

### 3.索引模版使用生命周期：

    PUT _template/template_1
    {
        "index_patterns" : ["te*"],
        "order" : 1,
        "settings" : {
            "number_of_shards" : 6,
            "number_of_replicas": 1,
            "index.lifecycle.name": "my_policy"
        },
        "aliases" : {
            "alias1" : {},
            "alias2" : {
                "filter" : {
                    "term" : {"user" : "kimchy" }
                },
                "routing" : "kimchy"
            },
            "{index}-alias" : {} 
        },
        "mappings": {}
    }

### 4.查看索引生命周期：
    GET _ilm/policy     获取所有
    GET _ilm/policy/<policy_id>

### 5.查看单个索引的生命周期计划：
    GET test/_ilm/explain

### 6.索引生命周期计划执行轮询时间：
    PUT _cluster/settings

    By default, ILM only checks rollover conditions every 10 minutes

    {
      "transient": {
        "indices.lifecycle.poll_interval": "1m" 
      }
    }


### 7.删除生命周期：
    DELETE _ilm/policy/<policy_id>

### 8.生命周期(Managing the index lifecycle)其他配置：

    ①索引生命周期状态：
      GET _ilm/status

    ②索引生命周期开启(关闭)：
      POST _ilm/start(stop)

    ③生命周期运行失败重试：
      POST /index/_ilm/retry
