# 生产环境的部署结构
![Model](https://github.com/DDD-MicroService-Kata/petstore-materials/raw/master/image/pet-shop.jpg)

## 网络环境
在 docker中，搭建了一个名为`petstoreinfrastructure_petshop-network`的网路，所有的服务和基础设置都部署在这个网络中

## 基础设施：

- 数据库：mysql server
- 服务注册和发现：consul

### 启动说明

1. `git clone https://github.com/DDD-MicroService-Kata/petstore-infrastructure.git`
1. 执行`docker-compose up -d`
1. 完成后，执行`docker-compose ps`，可以看到启动了两个 container(`petstoreinfrastructure_petshop-consul_1`和`petstoreinfrastructure_petshop-mysql_1`)
1. 执行`docker network ls | grep petstoreinfrastructure`。可以看到一个名为`petstoreinfrastructure_petshop-network`的网络被创建好
1. 连接入 mysql（端口和密码见下节）, 可以看到3个数据库已经建立完毕

### 基础设施参数说明

| 基础设施 |  映射本机地址   | docker 网络内地址 |
|--------|-----------|--------------|
| consul |      8505 |         8500 |
| mysql  |      3306 |             3306 |

mysql 用户名`root`；密码`dev`

## 服务

- account [source code](https://github.com/DDD-MicroService-Kata/petstore-account)
- inventory [source code](https://github.com/DDD-MicroService-Kata/petstore-inventory)
- order [source code](https://github.com/DDD-MicroService-Kata/petstore-order)

所有服务都会配置一个 swagger，作为 api 调用的界面, 路径为`htttp://localhost:${servic-port}/swagger-ui.html`

### 服务的部署
#### 数据库 migration
每个服务的数据库 migration 会在服务启动的过程中，自动由 flyway 完成。  
flyway 的 migration 脚本位置 resouces/db/migration/ 文件夹下

#### 服务的构造和启动
在每个服务的代码库下，如下执行：

1. 构造 jar 包 `gradle build`
1. 构造 image 镜像 `docker-compose build --force-rm`
1. 更新 container `docker-compose up -d`
1. 执行`docker-compose ps`可以看到一个`petstore*****_1`实例已经启动
1. 稍后1分钟，等待 spring 启动完毕
1. 打开[consul ui](http://localhost:8505/ui/#/dc1/services)，可以看到对应的服务已经完成注册。

#### 服务的端口
| 服务 |  服务端口   | 本地映射端口 |
|--------|-----------|--------------|
| account |      9092 |         11092 |
| inventory  |      9093 |             11093 |
| order  |      9094 |             11094 |

# 开发环境
## 技术栈

- spring boot
- spring boot jpa
- spring cloud consul
- spring cloud feign

## 开发环境和部署环境的差异
### 数据库

- 开发环境的运行时将和产品环境共用一个数据库
- 测试环境使用 H2数据库，通过 JPA 来直接生成 schema.不使用 flyway 进行migration

### 服务注册和发现
开发环境中不进行服务的注册和发现。需要自行配置服务间调用的地址：  
配置地点在bootstrap-local.yml和bootstrap-test.yml，模板如下：

``` yml
服务名称:
  ribbon:
    listOfServers: 服务地址
```

具体案例可见： https://github.com/DDD-MicroService-Kata/petstore-account/blob/master/src/main/resources/bootstrap-local.yml