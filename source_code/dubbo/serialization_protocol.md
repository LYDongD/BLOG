## dubbo序列化协议

### 模块： dubbo-serialization:

* api 
* api扩展


### 核心对象

ChannelBuffer -> ChannelBufferOutputStream -> ObjectOutput

serialization = ObjectOutput + request/response

ObjectOutput.flushBuffer() -> ChannelBufferOutputStream


### 编码/序列化入口

NettyClient

```
@Override
    protected void doOpen() throws Throwable {
        NettyHelper.setNettyLoggerFactory();
        bootstrap = new ClientBootstrap(channelFactory);
        // config
        // @see org.jboss.netty.channel.socket.SocketChannelConfig
        bootstrap.setOption("keepAlive", true);
        bootstrap.setOption("tcpNoDelay", true);
        bootstrap.setOption("connectTimeoutMillis", getConnectTimeout());
        final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
        bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
            @Override
            public ChannelPipeline getPipeline() {
                NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyClient.this);
                ChannelPipeline pipeline = Channels.pipeline();
                pipeline.addLast("decoder", adapter.getDecoder());
                pipeline.addLast("encoder", adapter.getEncoder());
                pipeline.addLast("handler", nettyHandler);
                return pipeline;
            }
        });
    }

```

netty信道上添加了encoder和decoder，来自NettyCodecAdapter

NettyCodecAdaptor适配了dubbo的Codec2接口，该接口扩展了多个不同的编码器

**编码的核心方法：**

通过ChannelBuffers.wrappedBuffer(buffer.toByteBuffer())，将dubbo的buffer桥接到netty的buffer

```
 @Sharable
    private class InternalEncoder extends OneToOneEncoder {

        @Override
        protected Object encode(ChannelHandlerContext ctx, Channel ch, Object msg) throws Exception {
            org.apache.dubbo.remoting.buffer.ChannelBuffer buffer =
                    org.apache.dubbo.remoting.buffer.ChannelBuffers.dynamicBuffer(1024);
            NettyChannel channel = NettyChannel.getOrAddChannel(ch, url, handler);
            try {
                codec.encode(channel, buffer, msg);
            } finally {
                NettyChannel.removeChannelIfDisconnected(ch);
            }
            return ChannelBuffers.wrappedBuffer(buffer.toByteBuffer());
        }
    }

```

注意，dubbo2.7版本之后，默认使用netty4, 除非显示指定netty3，否则:

netty或netty4都指向netty4

---

#### 如何避免在重试机制下，打印过多日志导致jvm崩溃?

1. reconnect_count 记录重连次数，按固定频率打印warn
2. reconnect_error_log_flag 记录错误打印标志，仅记录一次

```
private synchronized void initConnectStatusCheckCommand() {
        //reconnect=false to close reconnect， 默认开启
        int reconnect = getReconnectParam(getUrl());
        if (reconnect > 0 && (reconnectExecutorFuture == null || reconnectExecutorFuture.isCancelled())) {
            Runnable connectStatusCheckCommand = new Runnable() {
                @Override
                public void run() {
                    try {
                        if (!isConnected()) {
                            connect();
                        } else {
                            lastConnectedTime = System.currentTimeMillis();
                        }
                    } catch (Throwable t) { //日志优化，不重复打印重连失败日志
                        String errorMsg = "client reconnect to " + getUrl().getAddress() + " find error . url: " + getUrl();
                        // wait registry sync provider list， 超过一定时间才打印错误日志，通过flag标记，该日志仅打印一次
                        if (System.currentTimeMillis() - lastConnectedTime > shutdown_timeout) {
                            if (!reconnect_error_log_flag.get()) {
                                reconnect_error_log_flag.set(true);
                                logger.error(errorMsg, t);
                                return;
                            }
                        }
                        //周期性打印warn日志，
                        if (reconnect_count.getAndIncrement() % reconnect_warning_period == 0) {
                            logger.warn(errorMsg, t);
                        }
                    }
                }
            };
            //发起定时任务
            reconnectExecutorFuture = reconnectExecutorService.scheduleWithFixedDelay(connectStatusCheckCommand, reconnect, reconnect, TimeUnit.MILLISECONDS);
        }
    }

```
----

#### transporter层核心接口

* ChannelHandler/Channe/Endpoint

定义了基于信道的各种事件处理能力：

```

1. 处理连接
2. 处理断开
3. 发送消息
4. 接收消息
5. 处理异常
```

* Dispatcher

采用装饰器模式，装饰ChannelHandler，根据不同配置实现不同的线程调度模型

* Codec2

编解码器，包含不同的序列化协议，对消息进行编解码

