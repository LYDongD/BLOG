## pb 安装流程

### mac环境快速安装最新版本pb

```
brew search protobuf
brew install protobuf

```

### 源码包安装编译

> 1 找到指定版本的源码包

[github地址](https://github.com/protocolbuffers/protobuf/releases)

> 2 配置构建

```
cd protobuf-xxxx
./configure
make
make check
sudo make install

```

> 3 验证安装成功

```
which protoc
protoc --version

```

> 4 生成java协议

```
protoc -I . --java_out=. edms-sync.proto

```

> 5 可能出现的问题

#### 执行./configure时，找不到动态链接库

解决方案：

```
ln -s /usr/local/opt/readline/lib/libreadline.8.dylib /usr/local/opt/readline/lib/libreadline.7.dylib

```

#### 多版本问题

使用不同pb版本生成的java序列化协议文件很可能不一致，例如用v3.6和v2.5生成的协议中，使用了两个不同的类：

GeneratedMessage 和 GeneratedMessageV3

在项目中使用时需要考虑兼容问题，需要与项目中的pb版本保持一致

```
<dependency>
     <groupId>com.google.protobuf</groupId>
     <artifactId>protobuf-java</artifactId>
     <version>${version.protobuf}</version>
 </dependency>
 <dependency>
     <groupId>com.googlecode.protobuf-java-format</groupId>
     <artifactId>protobuf-java-format</artifactId>
     <version>${version.protobuf.format}</version>
 </dependency>

```
