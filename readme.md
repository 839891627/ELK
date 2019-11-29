# ELK搭建--Docker方式
## 环境要求
1. Docker
1. A minimum of 4GB RAM assigned to Docker
1. A limit on mmap counts equal to 262,144 or more
    > Linux 需要修改系统 *sysctl vm.max_map_count* 参数
1. Access to TCP port 5044 from log-emitting clients    

## 相关文件
 主要是 logstash 的配置以及 filebeat 配置
 > [https://github.com/839891627/ELK](https://github.com/839891627/ELK)


## 目录结构
```bash
├── client  客户端
│   ├── docker-compose.yml
│   ├── filebeat.yml
│   ├── filebeat�\205\215置详解.yml
│   ├── logs  # 日志目录
│   │   ├── 3.log
│   │   ├── laravel
│   │   ├── readme.md
│   └── logstash
│       └── conf.d
├── readme.md
└── server  # 服务器端
    ├── docker-compose.yml
    └── logstash
        └── conf.d

```

## 服务端
### 安装 ELK
在  `server` 目录下，`composer-compose up -d`

> - 5601 (Kibana web interface). 这里做了 5601端口本地的映射限制，是为了搭配后面nginx的认证（如果不需要，则去掉 127.0.0.1）
> - 9200 (Elasticsearch JSON interface).
> - 5044 (Logstash Beats interface, receives logs from Beats such as Filebeat).
  
1. 现在即可通过 `ip:port` 方式访问 Kibana web 后台了    
    > 后面会通过 nginx 做 web 认证登录


## 客户端
### filebeat 安装    
  > 客户端使用 `filebeat` 来搜集日志
在  `client` 目录下， `docker-compose up -d ` 启动
    
## 通过nginx认证登录kibana
1. 安装 apache 工具： `yum install httpd-tools -y`
1. 生成密码配置文件： `htpasswd -c -b /usr/local/nginx/conf/passwd/kibana.passwd username passwd`  
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
             auth_basic_user_file /usr/local/nginx/conf/passwd/kibana.passwd;
             proxy_pass http://127.0.0.1:5601;
             proxy_set_header Host $host:5601;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header Via "nginx";
          }
          access_log off
    }
    ```
1. ~~同时需要配置 `/opt/kibana/config/kibana.yml`;~~
    > 不再需要。因为在创建 elk 容器的时候，指定了 `127.0.0.1:5601` 端口的映射
    
    ```
    # 只允许本地访问
    server.host: "localhost"
    ```

### 其他命令
#### 安装 elasticsearch 插件
1. `docker exec -it elk bash`
1. `cd /opt/elasticsearch/bin`
1. `elasticsearch-plugin install  xxx`

