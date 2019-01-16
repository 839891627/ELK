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
1. 创建并启动容器 
    ```bash
    docker run -d -v  /data/wwwroot/filebeat/filebeat.yml:/filebeat.yml -v /path/需要搜集的日志目录/logs:/home/logs --restart=always --name filebeat prima/filebeat
    ```
    
## 通过nginx认证登录kibana
1. `yum install httpd-tools -y`
1. `htpasswd -c -b /usr/local/nginx/conf/passwd/kibana.passwd username passwd`
1. nginx配置：
    ```
    server {
          listen       80;
          server_name kibana.jt.com;
          #proxy_connect_timeout 6500s;
          location / {
             #proxy_connect_timeout 6500s;
             #proxy_read_timeout 6500s;
             auth_basic "secret";
             auth_basic_user_file /usr/local/nginx/conf/kibana.passwd;
             proxy_pass http://127.0.0.1:5601;
             proxy_set_header Host $host:5601;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header Via "nginx";
          }
          access_log off
    }
    ```
1. 同时需要配置 `/opt/kibana/config/kibana.yml`;
    ```
    # 只允许本地访问
    server.host: "localhost"
    ```
    
### 本地命令备份    
docker run -d -v /root/ELK/logstash/conf.d:/etc/logstash/conf.d -p 5601:5601 -p 9200:9200 -p 5044:5044 --restart=always -it --name elk sebp/elk
docker run -d -v  /data/wwwroot/filebeat/filebeat.yml:/filebeat.yml -v /data/wwwroot/lyq.naqujia/storage/logs:/home/logs --restart=always --name filebeat prima/filebeat

