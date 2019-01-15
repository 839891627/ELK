# ELK搭建--Docker方式
## 环境要求
1. Docker
1. A minimum of 4GB RAM assigned to Docker
1. A limit on mmap counts equal to 262,144 or more
    > Linux 需要修改系统 *sysctl vm.max_map_count* 参数
1. Access to TCP port 5044 from log-emitting clients    
## 服务端
### 安装 ELK
  > 这里使用 `sebp/elk` 镜像来安装
1. `docker pull sebp/elk`
1. `docker run -d -v /path/ElK/logstash/conf.d:/etc/logstash/conf.d -p 5601:5601 -p 9200:9200 -p 5044:5044 --restart=always -it --name elk sebp/elk`    
    > - 5601 (Kibana web interface).
    > - 9200 (Elasticsearch JSON interface).
    > - 5044 (Logstash Beats interface, receives logs from Beats such as Filebeat).
1. 现在即可通过 `ip:host` 方式访问 Kibana web 后台了    

### 安装插件
#### 安装 elasticsearch 插件
1. `docker exec -it elk bash`
1. `cd /opt/elasticsearch/bin`
1. `elasticsearch-plugin install  xxx`

## 客户端
### filebeat 安装    
  > 客户端使用 `filebeat` 来搜集日志
1. `docker pull prima/filebeat`
1. 配置文件 `filebeat.yml`
    ```yml
        #filebeat.modules:
        #  - module: system
        filebeat.prospectors:
          - type: log
            paths:
              - /home/logs/*.log
        output.logstash:
            hosts: ["ip:5044"]
        #filebeat.config.modules:
        #   path: ${path.config}/modules.d/*.yml
    ```
1. 创建并启动容器 
    ```bash
    docker run -d -v  /path/ELK/filebeat.yml:/filebeat.yml --name filebeat prima/filebeat
    ```

### todo
- [] 各种日志的搜集配置
- [] docker file    
