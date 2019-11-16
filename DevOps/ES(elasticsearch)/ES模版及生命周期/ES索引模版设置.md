# ES索引模版设置：
---
### 1.创建模版
    PUT _template/template_1

    {
      "index_patterns": [
        "te*"
      ],
      "order": 1,
      "settings": {
        //"index.lifecycle.name": "my_policy",
        "number_of_shards": 6,
        "number_of_replicas": 1
      },
      "aliases": {
        "alias1": {},
        "alias2": {
          "filter": {
            "term": {
              "user": "kimchy"
            }
          },
          "routing": "kimchy"
        },
        "{index}-alias": {}
      },
      "mappings": {}
    }

### 2.查看模版：
  	GET /_template/template_1
  	GET /_template/template_1,template_2
  	GET /_template/temp*
  	GET /_template	获取所有

### 3.删除模版：
    DELETE /_template/template_1

### 4.模版是否存在：
    HEAD _template/template_1
