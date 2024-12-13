# 单服务器下采用nginx作为反向代理转发到多应用(单部署文件版)

- 优势: 将项目的部署进行集中管理, 仅需单次对部署文件进行up操作, 无需单独进行创建network
- 劣势: 针对性较弱, 不能保证项目与部署文件一致

## 文件结构
```shell
single-deployment
|-- docker-compose.yml      # 全局部署文件
|-- README.md               # readme 文件
|-- nginx                   # nginx
|   |-- certs               # ssl 证书
|   |-- conf                # nginx 配置文件
```

## 快速开始
1. 更改 docker-compose.yml, 保留 nginx 和任意需要的 service
    ```yaml
    version: '3.8'

    services:
      nginx:
      image: nginx:stable
      restart: always
      ports:
        - "80:80"
      volumes:
        - ./conf:/etc/nginx/conf.d    # 配置文件挂载
        - ./certs:/etc/nginx/certs    # SSL 证书挂载, 如无, 请注释此行
      networks:
        - webnet

      backend-service:
        image: myregistry.com/namespace/image:0.0.1
        expose:
          - 80
        environment:
          TEST: 'fake'
        command: ["command", "args"]
        restart: always
        networks:
          - webnet

    networks:
      webnet:
        driver: bridge

    ```
2. 将更改的 `service name` 和暴露的端口改到对应的 `nginx/conf/config.conf` 文件, 若更改的是 api-project
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
3. 删除对应的 `signle-deployment/nginx/certs/dashboard.domain.com.pem`, `signle-deployment/nginx/certs/dashboard.domain.com.key`, `signle-deployment/nginx/conf/dashboard.conf`
