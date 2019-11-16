# ES分词器
---
### 1.ES自带标准分词器(standard)加入按“.”分词：

    创建 standard_plus 分词器(在默认 standard 分词器功能基础上加入按“.”分词),并在 text 字段上使用  standard_plus 分词器。
    {
      "settings": {
        "analysis": {
          "analyzer": {
            "standard_plus": {
              "tokenizer": "standard",
              "char_filter": [
                "my_char_filter"
              ]
            }
          },
          "char_filter": {
            "my_char_filter": {
              "type": "pattern_replace",
              "pattern": "\\.",
              "replacement": " "
            }
          }
        }
      },
      "mappings": {
        "properties": {
          "text": {
            "type": "text",
            "analyzer": "standard_plus"
          }
        }
      }
    }


# 拓展：
---
### 1.IK 分词器 热更新 词库配置：

    ①插件地址：https://github.com/medcl/elasticsearch-analysis-ik
        Analyzer: ik_smart , ik_max_word , Tokenizer: ik_smart , ik_max_word

    ②远程词库(可 Nginx 部署)：
      http://192.168.3.91:60/ik_dic.txt

    ③修改配置：
      cd /opt/elasticsearch/elasticsearch-7.2.1/config/analysis-ik/IKAnalyzer.cfg.xml

      <?xml version="1.0" encoding="UTF-8"?>
      <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
      <properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict">custom_ik.txt</entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <entry key="remote_ext_dict">http://192.168.3.91:60/ik_dic.txt</entry>
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
      </properties>
