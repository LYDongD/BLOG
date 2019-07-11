## java io 总结

> IO 

面向流，字节流和字符流

#### 字节流

[参考](https://www.jianshu.com/p/cbd3795fed84)

* InputStream
    * FileInputStream
    * BufferedInputSteam
    * ObjectInputStream
    * ByteArrayInputStream
    * ...
* OutputStream
    * ...


#### 字符流

* Reader
    * FileReader
    * BufferReader
    * ...
* Writer
    * ...

> NIO

面向管道和缓冲区，网络IO通过selector可实现非阻塞


* buffer
    * ByteBuffer, CharBuffer…
    * MappedByteBuffer 内存文件映射

* channel
    * FileChannel
    * SocketChannel & ServerSocketChannel

* selector
    * SocketChannel通过selector实现非阻塞IO


> 总结

* 普通IO， 面向流，无法使用PageCached(预读机制)
* FileChannel + buffer机制，可以使用PageCached预读，对顺序度友好
* FileChannel + mmap 机制，文件内存映射，省略一次数据拷贝(内核buffer -> 用户空间，黑盒)
* direct io， 省略PageCached，对随机读来说比较友好

> 代码示例

#### NIO

```
public void write() throws Exception {

        FileChannel fileChannel = new RandomAccessFile("/Users/lee/Desktop/test.txt", "rw").getChannel();

        //构造4kb的数据
        byte[] datas = new byte[4 * 1024];
        byte[] tmps = new byte[]{'a', 'b', 'c', 'd', 'e', 'f', 'g', '1', '2', '3', '4'};
        for (int i = 0; i < datas.length; i++) {
            datas[i] = tmps[i % tmps.length];
        }

        //从指定位置写入
        long position = 1024L;
        fileChannel.write(ByteBuffer.wrap(datas), position);

        //从当前文件指针写入, 会覆盖前面写入的3k数据，最终文件大小应该是5k
        fileChannel.write(ByteBuffer.wrap(datas));

        //关闭通道
        fileChannel.close();
    }

    public void read() throws Exception {
        FileChannel fileChannel = new RandomAccessFile("/Users/lee/Desktop/test.txt", "rw").getChannel();

        //使用对内内存缓冲数据, buffer 大小为1k
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        //从当前文件指针开始读取，一次读取1k, 需要读取5次
        int bytes = fileChannel.read(byteBuffer);
        //读取buffer数据并打印
        while (bytes != -1) {
            //每次读取让buffer position回归0
            byteBuffer.flip();
            //逐个字符读取
            while (byteBuffer.hasRemaining()) {
                System.out.println((char) byteBuffer.get());
            }
            //清空buffer
            byteBuffer.clear();
            bytes = fileChannel.read(byteBuffer);
        }

        //关闭
        fileChannel.close();
    }

    /**
     *  通过内存映射方式写
     */
    public void mmapWrite() throws Exception{

        FileChannel fileChannel = new RandomAccessFile("/Users/lee/Desktop/test.txt", "rw").getChannel();


        //构造4kb数据
        byte[] datas = new byte[4 * 1024];
        byte[] tmps = new byte[]{'a', 'b', 'c', 'd', 'e', 'f', 'g', '1', '2', '3', '4'};
        for (int i = 0; i < datas.length; i++) {
            datas[i] = tmps[i % tmps.length];
        }

        //通过mmap写入
        MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, datas.length);

        //从当前文件指针写入
        mappedByteBuffer.put(datas);

    }

    /**
     * 通过文件映射方式读
     */
    public void mmapRead() throws Exception {

        FileChannel fileChannel = new RandomAccessFile("/Users/lee/Desktop/test.txt", "rw").getChannel();

        MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, fileChannel.size());
        while (mappedByteBuffer.hasRemaining()) {
            System.out.println((char) mappedByteBuffer.get());
        }
    }


```

#### IO

```
   /**
     * 采用IO面向流的方式读取文件数据，这里是有BufferReader
     * @throws IOException
     */
    public void readDataIO() throws IOException {
        InputStreamReader inputStreamReader = new InputStreamReader(new FileInputStream(FILE_NAME), "UTF-8");
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

        String data;
        while ((data = bufferedReader.readLine()) != null) {
            System.out.println(data);
        }

        inputStreamReader.close();
        bufferedReader.close();
    }

```
