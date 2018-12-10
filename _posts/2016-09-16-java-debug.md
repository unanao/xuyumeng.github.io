# 1.1 docker中的参数设置
> docker让部署更方便，但是调试起来比本地开发稍微有些麻烦。 

## 1 docker-compose 文件修改

1. 指定启动参数：在 Dockerfile 中有一个指令叫做 ENTRYPOINT 指令，用于指定接入点，在docker-compose.yml 中可以定义接入点，覆盖 Dockerfile 中的定义就可以指定启动的调试参数了。
2. 暴露调试端口

### 1.1.1 配置举例
```
  export-distro:
    image: edgexfoundry/docker-export-distro
    ports:
      - "48070:48070"
      - "5566"
      - "58070:58070"                           # 暴露调试端口
      - "38070:38070"                           # 暴露jmx端口
    container_name: edgex-export-distro
    hostname: edgex-export-distro
    networks:
      - edgex-network
    volumes:
      - db-data:/data/db
      - log-data:/edgex/logs
      - consul-config:/consul/config
      - consul-data:/consul/data
    depends_on:
      - volume
      - config-seed
      - mongo
      - logging
      - notifications
      - metadata
      - data
      - export-client
                                               # 覆盖docker entrypoint
    entrypoint: java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=58070 -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=38070 -Dcom.sun.management.jmxremote.rmi.port=38070 -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=192.168.88.235 -Djava.security.egd=file:/dev/urandom export.jar

```

### 1.1.2 开启远程调试
```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=58070
```
将58080换成实际的端口， 只要端口不冲突就可以使用。

### 1.1.3 开启jmx支持visualVM远程监控

```
-Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=38071 -Dcom.sun.management.jmxremote.rmi.port=38071 -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=192.168.88.235
```
- 将38071换成jmx的实际端口号， 只要端口不冲突就可以使用。
- 192.168.88.235, 换成docker所在的服务器的地址。

## 1.2 让配置生效

启动镜像
```
docker-compose up -d image-name
```
也可以不加-d，查看启动的详细信息。

# 2. idea远程调试
1. 进入运行选项配置的
![edit config](image/edit-config.png) 

2. 添加remote运行的选项
![remote add](image/config-ip-port.png)

# 3. Java VisualVM远程监控
## 3.1 方法一
“文件” -> "添加JMX连接"

## 3.2 方法二
1. “文件” -> "添加远程主机"
2. 在远程主机上， 右键。“添加JMX连接”

添加完成后，“远程“的下面就会出现添加的“远程主机”， 然后点开“+”，就可以看到想要监视的进程啦。

# 4 问题定位
## 4.1 登录到docker里面查看启动的参数

```
E:\project\edgex\images>docker exec -it edgex-export-client sh
/edgex/edgex-export-client # ps aux
PID   USER     TIME   COMMAND
    1 root       0:00 /bin/sh -c java -jar -Djava.security.egd=file:/dev/urando
    5 root       1:43 java -jar -Djava.security.egd=file:/dev/urandom -Xmx100M
  108 root       0:00 sh
  114 root       0:00 ps aux

```
查看ps到的参数是否与docker-compoer.yml中配置的一致。

## 4.2 调试端口是否已经被占用
关闭镜像，查看配置的调试端口是否还在
```
netstat -n
```