# 基本环境安装说明

## 一、基础配置

### 1、以root用户安装应用

### 2、目录规范
 ```
/usr/local/
3、软件配置文件目录
/etc/
4、数据存储目录    
/data
5、docker镜像存储目录
/data/docker

# JENKINS项目备份
cd ~./jenkins/
tar -zcvf ../jenkins.tar.gz --exclude=workspace --exclude=logs *
```

### 3、下载相关软件包文件
```
# 更新镜像源
cp /etc/apt/sources.list /etc/apt/sources.list.bak
vi /etc/apt/sources.list

deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

apt update

# 安装系统基础软件包
apt install -y net-tools lrzsz zip

cd 
# 下载安装包
wget https://miyun.archermind.com/file/zjt/01_install.zip
unzip 01_install.zip
cd 01_install/zjt/01_base

# 下载软件包
wget https://miyun.archermind.com/file/zjt/software/openjdk-17.0.0.1+2_linux-x64_bin.tar.gz
wget https://miyun.archermind.com/file/zjt/software/apache-tomcat-9.0.82.tar.gz
wget https://miyun.archermind.com/file/zjt/software/docker-23.0.6.tgz
wget https://miyun.archermind.com/file/zjt/software/docker-compose-linux-x86_64
wget https://miyun.archermind.com/file/zjt/software/jenkins.tar.gz
wget https://miyun.archermind.com/file/zjt/software/database.zip
```

### 3、创建目录
```
# 创建数据存储目录
mkdir /data
# 创建docker容器目录
mkdir /data/docker
# 创建langflow的数据存储目录
mkdir -p /data/flow/data/
```

##  二、JDK安装 
### 1、解压
```
tar -zxvf openjdk-17.0.0.1+2_linux-x64_bin.tar.gz
mv jdk-17.0.0.1 /usr/local/
```
### 2、配置环境变量
```
vi ~/.bashrc

# JDK环境配置
export JAVA_HOME=/usr/local/jdk-17.0.0.1
export PATH=$PATH:$JAVA_HOME/bin

# 应用JDK配置
source ~/.bashrc

java -version
```

## 三、Docker安装 
### 1、解压Docker安装包
 ```
tar -zxvf docker-23.0.6.tgz
mv docker/* /usr/local/bin/
rm -rf docker
```

### 2、Docker自启动
```
# vi /usr/lib/systemd/system/docker.service

# 将下面的内容复制到刚创建的docker.service文件中
cat > /usr/lib/systemd/system/docker.service  <<EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/local/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
EOF


# 执行权限 
chmod +x /usr/lib/systemd/system/docker.service
```
### 3、配置Docker
```
# 创建目录
mkdir /etc/docker/

# 写入配置文件

cat > /etc/docker/daemon.json <<EOF
{
"insecure-registries": ["https://llmhub.archermind.com"],
"data-root":"/data/docker/",
"registry-mirrors": [
"https://llmhub.archermind.com",
"http://hub-mirror.c.163.com",
"https://docker.mirrors.ustc.edu.cn",
"https://dockerhub.azk8s.cn",
"https://mirror.ccs.tencentyun.com",
"https://registry.cn-hangzhou.aliyuncs.com",
"https://docker.mirrors.ustc.edu.cn",
"https://docker.1panel.live",
"https://atomhub.openatom.cn/",
"https://hub.uuuadc.top",
"https://docker.anyhub.us.kg",
"https://dockerhub.jobcher.com",
"https://dockerhub.icu",
"https://docker.ckyl.me",
"https://docker.awsl9527.cn",
"https://docker.m.daocloud.io",
"https://dockerhub.timeweb.cloud",
"https://noohub.ru",
"https://huecker.io"
]
}
EOF

```

### 4、启动Docker
```
systemctl daemon-reload
systemctl start docker
systemctl enable docker
docker -v
```

### 5、创建服务通讯网络
```
docker network create kg-network
```

## 四、DockerCompose的安装
```
cp docker-compose-linux-x86_64 /usr/bin/docker-compose && chmod +x /usr/bin/docker-compose
docker-compose version
```

## 五、Nginx安装
### 1、安装
```
apt install -y nginx
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```
### 2、配置
```
# /etc/nginx/nginx.conf
cp ./nginx/* /etc/nginx/conf.d/
rm -f /etc/nginx/sites-enabled/default
nginx -t
nginx -s reload
```


## 六、Jenkins安装
### 1、解压文件
```
tar -zxvf apache-tomcat-9.0.82.tar.gz -C /usr/local/
mv /usr/local/apache-tomcat-9.0.82 /usr/local/jenkins
chown root.root -R  /usr/local/jenkins
```
### 2、自启动
```
cat > /usr/lib/systemd/system/jenkins.service  <<EOF
[Unit]
Description=Jenkins Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/local/jdk-17.0.0.1
Environment=CATALINA_PID=/usr/local/jenkins/tomcat.pid
Environment=CATALINA_HOME=/usr/local/jenkins
Environment=CATALINA_BASE=/usr/local/jenkins
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseG1GC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/usr/local/jenkins/bin/startup.sh
ExecStop=/usr/local/jenkins/bin/shutdown.sh

User=root
Group=root
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
EOF

```
### 3、启动服务
```
chmod +x /usr/lib/systemd/system/jenkins.service
systemctl daemon-reload
systemctl start jenkins
systemctl enable jenkins

netstat -an|grep 8080|grep LISTEN
```

### 配置jenkins
```
# 解压配置文件
tar -zxvf jenkins.tar.gz -C /root/.jenkins/ 

systemctl restart jenkins
```

### 访问地址
http://xxxx.xxx.xxx.xxxx:8080/jenkins
用户名：llm
密码：llm123

# cat /root/.jenkins/secrets/initialAdminPassword


## 七、postgreSQL安装

### 创建数据目录
```
mkdir -p /data/postgresql/data
```

### 拉取镜像
```
docker pull llmhub.archermind.com/external/postgres:16
```
### 启动
```
docker run --name postgresql -d \
-p 5432:5432 \
-e POSTGRES_USER=kguser \
-e POSTGRES_PASSWORD=kgpassword \
-e POSTGRES_DB=test \
-e TZ=Asia/Shanghai \
-v /data/postgresql/data:/var/lib/postgresql/data \
--network kg-network \
--restart=unless-stopped \
--health-cmd="pg_isready -U kguser" \
--health-interval=30s \
--health-timeout=5s \
--health-retries=3 \
llmhub.archermind.com/external/postgres:16 \
postgres -c max_connections=1000 -c shared_buffers=4GB

# shared_buffers 通常不大于物理内存的25% 

```

###  验证是否生效
```
# docker exec -it postgresql psql -U test -c "SHOW max_connections; SHOW shared_buffers;"

# docker exec -it docker-db-1 psql -U kguser -c "SHOW max_connections; SHOW shared_buffers;"
```
### 数据导入导出
```
mkdir database
cd database
unzip ../database.zip 

# langflow
# docker run --rm -ti -v $(pwd):/backup   llmhub.archermind.com/external/postgres:16   pg_dump -h 192.168.102.19 -p 15432 -U langflow -d langflow -Fc -Z 9 -f /backup/flow_backup.dump

docker run --rm -ti -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16   psql -h 192.168.102.18 -p 5432 -U kguser -d postgres -c "CREATE DATABASE flow OWNER kguser;"
docker run --rm -ti -v $(pwd):/backup -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16  pg_restore -h 192.168.102.18 -p 5432 -U kguser -d flow -j 8 --clean /backup/flow_backup.dump

# kg-master
# docker run --rm -ti -v $(pwd):/backup  llmhub.archermind.com/external/postgres:16   pg_dump -h 192.168.102.21 -p 5432 -U kguser  -d kg-master-dev -Fc -Z 9 -f /backup/kg-master_backup.dump

docker run --rm -ti -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16    psql -h 192.168.102.18 -p 5432 -U kguser -d postgres -c "CREATE DATABASE \"kg-master-prod\" OWNER kguser;"
docker run --rm -ti -v $(pwd):/backup -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16  pg_restore -h 192.168.102.18 -p 5432 -U kguser -d kg-master-prod -j 8 --clean /backup/kg-master_backup.dump

# kg
# docker run --rm -ti -v $(pwd):/backup   llmhub.archermind.com/external/postgres:16   pg_dump -h 192.168.102.21 -p 5432 -U kguser  -d kg-dev -Fc -Z 9 -f /backup/kg_backup.dump

docker run --rm -ti -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16   psql -h 192.168.102.18 -p 5432 -U kguser -d postgres -c "CREATE DATABASE \"kg-prod\" OWNER kguser;"
docker run --rm -ti -v $(pwd):/backup -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16  pg_restore -h 192.168.102.18 -p 5432 -U kguser -d kg-prod -j 8 --clean /backup/kg_backup.dump

# kg-partner
# docker run --rm -ti -v $(pwd):/backup   llmhub.archermind.com/external/postgres:16   pg_dump -h 192.168.102.21 -p 5432 -U kguser  -d kg-partner -Fc -Z 9 -f /backup/kg-partner_backup.dump

docker run --rm -ti -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16   psql -h 192.168.102.18 -p 5432 -U kguser -d postgres -c "CREATE DATABASE \"kg-partner-prod\" OWNER kguser;"
docker run --rm -ti -v $(pwd):/backup -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16  pg_restore -h 192.168.102.18 -p 5432 -U kguser -d kg-partner-prod -j 8 --clean /backup/kg-partner_backup.dump

# kg-data
# docker run --rm -ti -v $(pwd):/backup   llmhub.archermind.com/external/postgres:16   pg_dump -h 192.168.102.21 -p 5432 -U kguser  -d kg-data-dev -Fc -Z 9 -f /backup/kg-data_backup.dump

docker run --rm -ti -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16   psql -h 192.168.102.18 -p 5432 -U kguser -d postgres -c "CREATE DATABASE \"kg-data-prod\" OWNER kguser;"
docker run --rm -ti -v $(pwd):/backup -e PGPASSWORD='kgpassword' llmhub.archermind.com/external/postgres:16  pg_restore -h 192.168.102.18 -p 5432 -U kguser -d kg-data-prod -j 8 --clean /backup/kg-data_backup.dump


cd ../
```



## 八、minIO安装
```
# 创建数据目录
mkdir -p /data/minio/{data,config}

# 拉取镜像
docker pull llmhub.archermind.com/external/minio:latest

# 安装镜像
docker run --name minio -d \
-p 9001:9001  \
-p 9090:9090  \
--restart=always \
--network kg-network \
-e "MINIO_ROOT_USER=minioadmin" \
-e "MINIO_ROOT_PASSWORD=minioadmin" \
-e "MINIO_ACCESS_KEY=minioadmin" \
-e "MINIO_SECRET_KEY=minioadmin" \
-v /data/minio/data:/data \
-v /data/minio/config:/root/.minio \
llmhub.archermind.com/external/minio:latest server  /data --console-address ":9090" --address ":9001"
    
# 访问测试    
http://xxx.xxx.xxx.xxx:9090/
用户名：minioadmin
密码：minioadmin

# 创建默认存储桶

kg-light
设置该桶为public

    
```


## 十、MYSQL 安装
```
mkdir -p /data/mysql8/{data,conf,logs,mysql-files}

cat > /data/mysql8/conf/custom.cnf <<'EOF'
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
EOF

 
docker pull llmhub.archermind.com/external/mysql:8.0.27
 
docker run \
-p 3306:3306 \
--name mysql \
--privileged=true \
--restart unless-stopped \
--network kg-network \
-v /data/mysql8/conf:/etc/mysql \
-v /data/mysql8/logs:/logs \
-v /data/mysql8/data:/var/lib/mysql \
-v /data/mysql8/mysql-files:/var/lib/mysql-files \
-v /etc/localtime:/etc/localtime \
-e MYSQL_ROOT_PASSWORD=123456 \
-d llmhub.archermind.com/external/mysql:8.0.27
```
### 确认字符集：

SHOW VARIABLES LIKE 'character_set%';

## 业务库
CREATE DATABASE `yudao-vue-pro-sec` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE `kg-business-hubei` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

## 十一、redis安装
```

mkdir -p /data/redis/{conf,data}
 
redis.conf:
bind 0.0.0.0
port 6379
dir /data
# 如果启用了 AOF
appendonly yes


cp ./redis/redis.conf /data/redis/conf/

docker pull llmhub.archermind.com/external/redis:7.4

docker run -d \
--name redis \
--memory=1g \
--cpus=1 \
-p 6379:6379 \
--network kg-network \
-v /data/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf \
-v /data/redis/data:/data \
--restart unless-stopped \
--log-driver json-file \
--log-opt max-size=100m \
llmhub.archermind.com/external/redis:7.4 \
redis-server /usr/local/etc/redis/redis.conf
  
```

## 十二、nacos安装
https://blog.csdn.net/boywcx/article/details/125758198
### docker拉取nacos镜像
```
docker pull llmhub.archermind.com/external/nacos-server:v2.3.2
```
### 创建映射容器的文件目录
```
# 创建logs目录
mkdir -p /data/nacos/{logs,conf}
# 创建配置文件目录
#授予权限
chmod 777 /data/nacos/{logs,conf}

CREATE DATABASE nacos DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
EXIT;

```
### 创建application.properties文件并放入/data/nacos/conf/中
### 如果当前没有 nacos.sql，从镜像里拷贝：
docker create --name nacos-tmp llmhub.archermind.com/external/nacos-server:v2.3.2
docker cp nacos-tmp:/home/nacos/conf/mysql-schema.sql /data/nacos.sql
docker rm nacos-tmp

### 导入 Nacos 表结构：
docker exec -i mysql mysql -uroot -p123456 --default-character-set=utf8mb4 --database=nacos < /data/nacos.sql


### 创建 /data/nacos/conf/application.properties：
cat > /data/nacos/conf/application.properties <<'EOF'
server.servlet.contextPath=/nacos
server.port=8848
server.error.include-message=ALWAYS

spring.sql.init.platform=mysql
db.num=1
db.url.0=jdbc:mysql://mysql:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
db.user.0=root
db.password.0=123456

nacos.core.auth.enabled=false
nacos.core.auth.system.type=nacos

management.endpoints.web.exposure.include=*
EOF


### 创建容器
```
docker run --name nacos -d \
-p 8848:8848 \
-p 7848:7848 \
-p 9848:9848 \
-p 9849:9849 \
--network kg-network \
--restart unless-stopped \
-m 2g \
-e JVM_XMS=1024m \
-e JVM_XMX=1024m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
-v /data/nacos/logs:/home/nacos/logs \
-v /data/nacos/conf/application.properties:/home/nacos/conf/application.properties \
llmhub.archermind.com/external/nacos-server:v2.3.2
  
  

```
### 测试访问
http://192.168.110.128:8848/nacos  
http://ip:8848/nacos

### 创建命名空间
```
curl -X POST 'http://127.0.0.1:8848/nacos/v1/console/namespaces' \
-H 'Content-Type: application/x-www-form-urlencoded' \
-d 'customNamespaceId=kg-prod&namespaceName=kg-prod&namespaceDesc=kg_prod'
```

## 十三、toolbox安装

### 创建配置目录
```
mkdir -p /data/toolbox/conf/
cp ./toolbox/tools-tianyancha.yaml /data/toolbox/conf/
```


### 启动容器
```
docker pull llmhub.archermind.com/kg/toolbox:latest

docker run -d --name toolbox \
-p 5000:5000 \
--restart always \
--network kg-network \
-v /data/toolbox/conf:/app/yaml \
llmhub.archermind.com/kg/toolbox:latest
```

http://xx.xx.xx.xx:5000/ui

## 十四、ES安装
```
# 创建本地目录（可选但推荐）
mkdir -p /data/elasticsearch/{data,logs,plugins}

# Elasticsearch 容器内以用户 elasticsearch（UID 1000）运行，必须确保挂载目录可写：
chown -R 1000:1000 /data/elasticsearch/{data,logs,plugins}

# 将 IK 插件复制到 plugins 目录（确保权限正确）
cp -r ./elasticsearch/es_plugins/ik /data/elasticsearch/plugins/


# 启动 Elasticsearch 容器
# 低配
docker run -d \
--name elasticsearch \
--restart=always \
-p 9200:9200 \
--network kg-network \
-e "discovery.type=single-node" \
-e "cluster.max_shards_per_node=10000" \
-e "ES_JAVA_OPTS=-Xms1g -Xmx1g" \
-e "xpack.security.enabled=false" \
-e "TZ=Asia/Shanghai" \
-e "http.max_content_length=500mb" \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/elasticsearch/logs:/usr/share/elasticsearch/logs \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
llmhub.archermind.com/external/elasticsearch:8.14.2

# 高配
docker run -d \
--name elasticsearch \
--restart=always \
-p 9200:9200 \
--network kg-network \
--memory=60g \
-e "discovery.type=single-node" \
-e "cluster.max_shards_per_node=10000" \
-e "ES_JAVA_OPTS=-Xms31g -Xmx31g" \
-e "xpack.security.enabled=false" \
-e "TZ=Asia/Shanghai" \
-e "http.max_content_length=500mb" \
-v /data/elasticsearch/data:/usr/share/elasticsearch/data \
-v /data/elasticsearch/logs:/usr/share/elasticsearch/logs \
-v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
llmhub.archermind.com/external/elasticsearch:8.14.2


# 优化性能版
docker run -d \
  --name elasticsearch \
  --restart=always \
  --network kg-network \
  --memory=6g \
  --memory-swap=6g \
  --ulimit memlock=-1:-1 \
  --ulimit nofile=65536:65536 \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e "cluster.max_shards_per_node=10000" \
  -e "ES_JAVA_OPTS=-Xms4g -Xmx4g" \
  -e "xpack.security.enabled=false" \
  -e "TZ=Asia/Shanghai" \
  -e "http.max_content_length=500mb" \
  -v /data/elasticsearch/data:/usr/share/elasticsearch/data \
  -v /data/elasticsearch/logs:/usr/share/elasticsearch/logs \
  -v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
  -v /data/elasticsearch/config:/usr/share/elasticsearch/config \
  llmhub.archermind.com/external/elasticsearch:8.14.2
  
  
# 验证
curl http://localhost:9200
  
```

## 十五、mcp-server-chart安装
 ```
 # 这里要注意，docker-compose文件的配置了服务器的IP，要注意修改与当前服务器的IP一致
 # 以docker-compose进行安装
 
 mkdir -p /data/mcp-server-chart/{sse,streamable,images}
 cd ./mcp-server-chart/
 docker-compose up -d
 cd ../
 ```

## 十六、OpenAI路由服务升级
```
mkdir -p /data/openai-router/data/

cp ./openai-router/routes.db /data/openai-router/data/

docker run -d \
--name openai-router \
--restart unless-stopped \
-v /data/openai-router/data/:/app/.venv/lib/python3.12/data/ \
-p 8000:8000 \
llmhub.archermind.com/external/openai-router:latest \
openai-router --host 0.0.0.0

```  

## 十七、升级页面安装
### 1、创建数据目录
```
mkdir -p /data/upgrade/{www,bin}
```
### 2、安装 本地版本获取程序
```
cp ./upgrade/generate.sh /data/upgrade/bin/

# chmod +x /data/upgrade/bin/generate.sh

(crontab -l 2>/dev/null; echo "* * * * * /bin/bash /data/upgrade/bin/generate.sh >> /var/log/generate.sh.log 2>&1") | crontab -

crontab -l
```

### 部署jenkins升级pipeline
```

```

### 安装升级后台服务
```
docker run -d --name operation-system-prod \
-p 41086:41086 \
-v /docker/operation-system-prod/logs:/root/logs/ \
-e TZ=Asia/Shanghai \
-e SPRING_PROFILES_ACTIVE=prod \
-e SPRING_CLOUD_NACOS_CONFIG_SERVER_ADDR=nacos:8848 \
-e SPRING_CLOUD_NACOS_CONFIG_NAMESPACE=kg-prod \
-e SPRING_CLOUD_NACOS_SERVER_ADDR=nacos:8848 \
-e SPRING_CLOUD_NACOS_DISCOVERY_NAMESPACE=kg-prod \
-e SPRING_CLOUD_NACOS_DISCOVERY_FAIL-FAST=false \
-e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_URL=jdbc:postgresql://postgresql:5432/kg-master-prod \
-e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_USERNAME=kguser \
-e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_PASSWORD=kgpassword \
-e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_URL=jdbc:postgresql://postgresql:5432/kg-master-prod \
-e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_USERNAME=kguser \
-e SPRING_DATASOURCE_DYNAMIC_DATASOURCE_MASTER_PASSWORD=kgpassword \
-e SPRING_REDIS_HOST=redis -e SPRING_REDIS_PORT=6379 -e SPRING_REDIS_DATABASE=9 \
--network kg-network \
--restart=always \
llmhub.archermind.com/zjt/operation-system-prod:v1.0.0.1
```

### 升级页面安装
```
# 解压升级管理页面
unzip ./upgrade/upgrade.zip -d /data/upgrade/www/

# 修改配置文件
vi /data/upgrade/www/config/index.js

const config = {
  jenkinsUrl: '/jenkins/job/', // 通过nginx代理的jenkins地址
  jenkinsUserName: 'llm',
  jenkinsToken: '11415571e6b62fd2bc5a75398f9e4c68a7',
  backendUrl: 'http://192.168.102.22:41089/admin-api/', // 后端接口地址
}

```

## 十七、系统配置
# minio的地址
打开文件配置功能页
配置
![img.png](img.png)
配置名称：minIO
节点地址：http://192.168.102.19:9001
存储 bucket：kg-light
自定义域名：http://192.168.102.22:41089/upload-proxy/kg-light

