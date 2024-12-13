# 单服务器下采用nginx作为反向代理转发到多应用(多部署文件版)

- 优势: 将项目的部署文件与项目进行关联, 针对性强
- 劣势: 部署文件相对分散, 需要提前创建 network, 需要多次对部署文件进行up操作

## 文件结构
```shell
multiple-deployment
|-- network.yml                 # 统一network
|-- api-project                 # 接口项目
|       |-- docker-compose.yml  # 接口部署文件
|       |-- ...
|
|-- dashboard-project           # web 项目
|       |-- docker-compose.yml  # web 部署文件
|       |-- ...
|
|-- nginx                       # nginx
|       |-- docker-compose.yml  # nginx 部署文件
|       |-- certs               # ssl 证书
|       |-- conf                # nginx 配置文件
```

## 快速开始
1. 创建 network
    ```shell
    # 需保证当前路径处于 multiple-deployment 中
    docker network create webnet
    # 或者
    docker-compose -f network.yml up -d
    ```
2. 更改任意 project 内的 `docker-compose.yml` 文件, 假设更改的是 api-project
    ```yaml
    version: '3.8'
    
    services:
    backend-service:
      image: myregistry.com/namespace/image:0.0.1
      expose:
        - 80
      environment:
        TEST: 'fake'
      command: ["command", "args"]
      networks:
        - webnet
    
    networks:
      webnet:
        external: true
    ```
3. 将更改的 `service name` 和暴露的端口改到对应的 `nginx/conf/project.conf` 文件, 若更改的是 api-project
    ```nginx
    server {
        listen                  80;
        server_name             api.mydomain.com;

        location / {
            # {api-service} 换成 backend-service, {expose-port} 为 80
            proxy_pass          http://backend-service:80;
            proxy_set_header    Host $host;
            proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    X-Forwarded-Proto $scheme;
        }
    }

    # 如无ssl证书, 将此块进行注释
    server {
        listen                  443;
        server_name             api.mydomain.com;

        # SSL 证书路径
        ssl_certificate         /etc/nginx/certs/api.mydomain.com.pem;  # 与 ../certs/api.mydomain.com.pem 文件名一致
        ssl_certificate_key     /etc/nginx/certs/api.mydomain.com.key;  # 与 ../certs/api.mydomain.com.key 文件名一致


        location / {
            # {api-service} 换成 backend-service, {expose-port} 为 80
            proxy_pass          http://backend-service:80;
            proxy_set_header    Host $host;
            proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    X-Forwarded-Proto $scheme;
        }
    }
    ```
