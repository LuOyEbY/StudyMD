# Docker运行常用中间件

### 1.docker 运行mysql

```java
docker run -v /Users/a58/Desktop/TestOwner/docker-mount/mysql/d_data/mysqldb/conf:/etc/mysql/conf.d -v /Users/a58/Desktop/TestOwner/docker-mount/mysql/d_data/mysqldb/logs:/logs -v /Users/a58/Desktop/TestOwner/docker-mount/mysql/d_data/mysqldb/data:/var/lib/mysql -d -p 3307:3306 -e MYSQL_ROOT_PASSWORD='root' --name mysql mysql
  
  
docker exec -it mysql bash
mysql -uroot -proot
```

#### 2.docker 运行redis

```java
docker run -p 6379:6379 --name redis -v /Users/a58/Desktop/TestOwner/docker-mount/redis/redis.conf:/etc/redis/redis.conf  -v /Users/a58/Desktop/TestOwner/docker-mount/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes 

##redis.config
#bind 0.0.0.0 #带bind的注释掉
protected-mode yes #开启密码 云服务上一定要开启密码，防火墙 不信你就试试！！！！！
daemonize no  #一定改为no 否则redis不能启动 而且没有日志！！！！！！！！！！
requirepass 123 #密码
  
docker exec -it redis bash
redis-cli  
```

#### 3.docker 运行ES

```java
docker run -p 9200:9200 --name elasticsearch \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms1g -Xmx2g" \
-v /Users/a58/Desktop/TestOwner/docker-mount/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /Users/a58/Desktop/TestOwner/docker-mount/elasticsearch/data:/usr/share/elasticsearch/data \
-v /Users/a58/Desktop/TestOwner/docker-mount/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.10.1

##elasticsearch.yml
network.host: 0.0.0.0   
network.bind_host: 0.0.0.0  #外网可访问

http.cors.enabled: true
http.cors.allow-origin: "*"
xpack.security.enabled: true # 这条配置表示开启xpack认证机制 spring boot连接使用
xpack.security.transport.ssl.enabled: true
  
#初始化
docker exec -it 容器id /bin/bash
bin/elasticsearch-setup-passwords interactive
  
0.0.0.0:9200 elastic/密码
```

#### 4.docker 运行kafka

```

```

#### 5.docker-compose.yaml（Es+Kibana,Redis,Mysql,minio）

```yaml
version: '2'

#networks:
#  app-tier:
#    driver: bridge

services:
  # kafka:
  #   image: 'bitnami/kafka:latest'
  #   container_name: kafka
  #   restart: always
  #   networks:
  #     - app-tier
  #   environment:
  #     - ALLOW_PLAINTEXT_LISTENER=yes

  elasticsearch:
    image: elasticsearch:8.8.1
    container_name: ly-elasticsearch
    restart: always
    environment:
      # 开启内存锁定
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      # 指定单节点启动
      - discovery.type=single-node
    ulimits:
      # 取消内存相关限制 用于开启内存锁定
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /Users/a58/Desktop/TestOwner/docker-mount/elasticsearch/data:/usr/share/elasticsearch/data
      - /Users/a58/Desktop/TestOwner/docker-mount/elasticsearch/logs:/usr/share/elasticsearch/logs
      - /Users/a58/Desktop/TestOwner/docker-mount/elasticsearch/plugins:/usr/share/elasticsearch/plugins
    ports:
      - 9200:9200

  kibana:   # 自定义服务名
    image: kibana:8.8.1  # 镜像名
    container_name: ly-kibana
    restart: always # 允许自动重启
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      # 自己本机电脑的ip
      - "ELASTICSEARCH_HOSTS=http://10.253.49.17:9200"
      - "ELASTICSEARCH_USERNAME=kibana"
      - "ELASTICSEARCH_PASSWORD=b3syQCoxRJp*4btAI=wf"
    volumes:
      - /etc/localtime:/etc/localtime
    ports:
      - 5601:5601/tcp

  redis:
    image: redis
    container_name: ly-redis
    environment:
      TZ: Asia/Shanghai
    ports: 
      - "6379:6379"
      # redis密码
    command: ["redis-server","--appendonly", "yes", "--requirepass","123456"]
    restart: always
    volumes:
      - /Users/a58/Desktop/TestOwner/docker-mount/redis/conf:/etc/redis/
      - /Users/a58/Desktop/TestOwner/docker-mount/redis/data:/data

  mysql:
    image: mysql
    container_name: ly-mysql  #容器名
    restart: always
    volumes:
      - /Users/a58/Desktop/TestOwner/docker-mount/mysql/data:/var/lib/mysql  #挂载目录，持久化存储
    ports:
      - '3306:3306'
    environment:
      TZ: Asia/Shanghai
      # root账户的密码
      MYSQL_ROOT_PASSWORD: "luoye@2023"   #设置root用户的密码

  minio:
    image: bitnami/minio:latest
    container_name: ly-minio
    restart: always  
    ports: 
      - "9001:9001"
      - "9000:9000"
    volumes:
      - "/Users/a58/Desktop/TestOwner/docker-mount/minio/data:/data"
    environment:
      - TZ=Asia/Shanghai
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=12345678
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
```

