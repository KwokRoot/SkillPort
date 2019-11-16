## ES插件安装(elasticsearch-head、Cerebro、Kibana)
##### **注：elasticsearch-head 插件需要安装 nodejs 环境，Cerebro 插件需要 Java 运行环境。**

---

### 插件一：elasticsearch-head

      ① git clone git://github.com/mobz/elasticsearch-head.git
      ② cd elasticsearch-head
      ③ npm install
      ④ npm run start
      ⑤ open http://localhost:9100/


### 插件二：Cerebro

      ① Download from https://github.com/lmenezes/cerebro/releases
      ② Run bin/cerebro(or bin/cerebro.bat if on Windows)
      ③ Access on http://localhost:9000
    
    安装详情：
      
      1.解压:
        tar -zxvf cerebro-0.8.3.tgz
      
      2.修改配置：
        cd cerebro-0.8.3
        vim conf/application.conf

        修改(样例)：

        hosts = [
          {
           host = "http://192.168.3.91:9200"
           name = "elasticsearch"
          }
        ]

        权限验证：

        替换：

        auth = {
          type: basic
          settings {
            username = es
            password = 123456
          }
        }

      3.系统防火墙设置：
        /sbin/iptables -I INPUT -p tcp --dport 9000 -j ACCEPT
        /etc/rc.d/init.d/iptables save
        /etc/rc.d/init.d/iptables restart

      4.启动：
        nohup ./bin/cerebro &
      
      5.访问路径：
        http://ip:9000


### 插件三：Kibana插件安装：

      1.官方下载 
          kibana-7.2.1-linux-x86_64.tar.gz

      2.解压:
          tar -zxvf kibana-7.2.1-linux-x86_64.tar.gz

      3.修改配置：
          cd kibana-7.2.1-linux-x86_64
          vim config/kibana.yml

        追加配置项(样例)：
          # Kibana 服务端口 默认：5601
          server.port: 5601
          # 本机ip
          server.host: "192.168.3.91"
          # ES 集群
          elasticsearch.hosts: ["http://集群节点IP:9200","http://集群节点IP:9200","http://集群节点IP:9200"]

      4.系统防火墙设置：
        /sbin/iptables -I INPUT -p tcp --dport 5601 -j ACCEPT
        /etc/rc.d/init.d/iptables save
        /etc/rc.d/init.d/iptables restart

      5.启动：
        nohup ./bin/kibana &

      6.访问路径：
        http://ip:5601