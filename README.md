# 单服务器下采用nginx作为反向代理转发到多应用
基于docker下采用nginx容器作为流量转发入口, 将多应用部署在单服务器上
## 项目结构
```shell
root
|-- multiple-deployment                 # 单服务器下采用nginx作为反向代理转发到多应用(多部署文件版)
|       |-- README.md                   # readme 文件
|       |-- network.yml                 # 统一network
|       |-- api-project                 # 接口项目
|       |    |-- docker-compose.yml     # 接口部署文件
|       |    |-- ...
|       |
|       |-- dashboard-project           # web 项目
|       |    |-- docker-compose.yml     # web 部署文件
|       |    |-- ...
|       |
|       |-- nginx                       # nginx
|            |-- docker-compose.yml     # nginx 部署文件
|            |-- certs                  # ssl 证书
|            |-- conf                   # nginx 配置文件
|
|-- single-deployment                   # 单服务器下采用nginx作为反向代理转发到多应用(单部署文件版)
|       |-- docker-compose.yml          # 全局部署文件
|       |-- README.md                   # readme 文件
|       |-- nginx                       # nginx
|            |-- certs                  # ssl 证书
|            |-- conf                   # nginx 配置文件
|       
```
`multiple-deployment` 和 `single-deployment` 完全一致, 只是部署文件是否位于一处进行统一管理的区别  
详情见: [多部署文件版](./multiple-deployment/README.md)  [单部署文件版](./single-deployment/README.md)