## 高压缩文件系统squshfs

#### 安装压缩工具

```
yum install -y squashfs-tools

```

#### 构建suashfs

以 /etc为例，构建环文件系统(在文件中而非物理设备中的文件系统)

```
mksquashfs  /etc test.squashfs

```

#### 挂载

**什么是挂载**


挂载即将硬盘(存储资源)加载到linux root 下的某个目录下，访问目录就相当于访问该存储资源


将高压缩文件系统挂载到/mnt/squash

```
mount -o loop test.squashfs /mnt/squash

```

#### 查看

```
//查看文件大小
du -h --max-depth=1 /mnt

//查看挂载
mount -l | grep squash

```


