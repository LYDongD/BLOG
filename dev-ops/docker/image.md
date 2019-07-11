## 高效构建docker镜像

### 目录

- 高效构建docker镜像
    - [镜像操作](#镜像操作)
    - [镜像构建优化建议](#镜像构建优化建议)

### <a name="镜像操作"> 镜像操作</a>

* docker image 查看命令列表
* docker image save 以tar包的形式存储镜像


> 查看一个镜像的结构

```

//将mysql镜像以tar包的形式保存
docker image save -o /root/tmp/mysql_image.tar docker.io/mysql

//解压到指定目录
tar -C mysql_image/ -xf mysql_image.tar

//查看目录树
tree mysql_image/


输入如下：

├── bb85cfe43af5173f7cddd98b7d6668f1ac3205c765466c113e995696862b3695
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── e30e1a28b79cf97d5c6569ec962128046a291d0f4a7dcf85304866cccdba235e
│   ├── json
│   ├── layer.tar
│   └── VERSION
├── manifest.json
└── repositories

//安装jq，一个json格式化工具
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum repolist
yum install jq

//查看manifest.json, 顶层配置
cat manifest.json | jq

输出如下：

[
  {
    "Config": "7bb2586065cd50457e315a5dab0732a87c45c5fad619c017732f5a13e58b51dd.json",
    "RepoTags": [
      "docker.io/mysql:latest"
    ],
    "Layers": [
      "55c7861c336131f1b3921e8dc215ef274d8c0c10da13873b1ae63e1cfcd384de/layer.tar",
      "065d66aa7bdce863e2d259ee753cc89c9819e40d39139ac4696b61bfca53da13/layer.tar",
      "93db928c56f51cf9a8183d50dbd1f7ec09f07f4f2d8bcee9dc71ed6a3a4538dd/layer.tar",
      "69296249841433bb268584c09aa6bdcbd4e8700827f7fb583a4496167ce82d76/layer.tar",
      "32a48283de1266d2fef73fc15a6df86cc0fb6b7502227e28bfa120f97ea5b440/layer.tar",
      "bb85cfe43af5173f7cddd98b7d6668f1ac3205c765466c113e995696862b3695/layer.tar",
      "179bd73fc7524feca3e08aea400d07d6246bc9490319d52de73a1571620618b5/layer.tar",
      "83073238cf107f4436ac5f3342cf0e8bfb28fd1c0ae9b796589d24f424ee41b5/layer.tar",
      "e30e1a28b79cf97d5c6569ec962128046a291d0f4a7dcf85304866cccdba235e/layer.tar",
      "0579984b6ddc507146841b57e8b9dc677d224922631109393c626871b9ec1fbf/layer.tar",
      "5de6be230bf556e9a000d65911e0b5c29d4d80f7afc4f1ee9b7b035ac7bd3169/layer.tar",
      "279e4c5c1afdbc6488bc0778c56f03258fa1d9eeee306881f05acba4ed428d7e/layer.tar"
    ]
  }
]

//可进一步查看配置
cat 7bb2586065cd50457e315a5dab0732a87c45c5fad619c017732f5a13e58b51dd.json | jq

``` 

> 镜像的构建

* 从容器创建： docker container commit
* 从dockerfile构建： docker image build -t imageName:tag .

建议从dockerfile进行构建

### <a name="镜像构建优化建议"> 镜像构建优化建议 </a>

> 利用缓存， 为了更有效的利用构建缓存，将更新最频繁的步骤放在最后面

```

//由于copy每次都会发生变化，导致接下来的两个操作无法被缓存
FROM debian

COPY . /app

RUN apt update
RUN apt install -y openjdk-8-jdk

CMD [ "java", "-jar", "/app/target/gs-spring-boot-0.1.0.jar" ]


//优化后，将变更操作放在后面
FROM debian

RUN apt update
RUN apt install -y openjdk-8-jdk

COPY . /app

CMD [ "java", "-jar", "/app/target/gs-spring-boot-0.1.0.jar" ]

```

> 部分拷贝, 避免将全部内容拷贝至镜像中, 至保留需要的内容即可

```

FROM debian

RUN apt update
RUN apt install -y openjdk-8-jdk

//仅copy必要的内容
COPY target/gs-spring-boot-0.1.0.jar /app/

CMD [ "java", "-jar", "/app/gs-spring-boot-0.1.0.jar" ]

```

> 防止包缓存过期

将包管理器的缓存生成与安装包的命令写到一起可防止包缓存过期

```
FROM debian

//防止包缓存过期
RUN apt update && apt install -y openjdk-8-jdk

COPY target/gs-spring-boot-0.1.0.jar /app/

CMD [ "java", "-jar", "/app/gs-spring-boot-0.1.0.jar" ]

```

> 保持构建环境一致

在创建镜像时进行应用构建，替代copy构建好的包，保证构建环境一致

```
FROM maven:3.6.1-jdk-8-alpine

WORKDIR /app

//构建依赖，一般依赖不会发生变化，可有效利用缓存
COPY pom.xml /app/
RUN mvn dependency:go-offline

//构建项目
COPY src /app/src
    
RUN mvn -e -B package

CMD [ "java", "-jar", "/app/target/gs-spring-boot-0.1.0.jar" ]

```

> 多阶段构建

```
FROM maven:3.6.1-jdk-8-alpine AS builder

WORKDIR /app

COPY pom.xml /app/
RUN mvn dependency:go-offline
COPY src /app/src
RUN mvn -e -B package

FROM openjdk:8-jre-alpine

COPY --from=builder /app/target/gs-spring-boot-0.1.0.jar /

CMD [ "java", "-jar", "/gs-spring-boot-0.1.0.jar" ]

```
