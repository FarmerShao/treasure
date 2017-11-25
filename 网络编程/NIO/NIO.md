# NIO 
---

## **Selector(选择器)**
    Selector 允许单个线程处理多个Channel。这样就可以防止线程的无限制增长，以及减少线程的上下文切换。
    
+ Selector的创建： Selector selector = Selector.open();
+ 向Selector 注册通道：
    1. 将通道设置为非阻塞的（因为FileChannel不能置为非阻塞模式，所以FileChannel不能注册到Selector中） channel.configureBlocking(flase)
    2. 将通道注册到 Selector: SelectionKey sk = channel.register(selector, SelectionKey.OP_ACCEPT | SelectionKey.OP_READ); 第二个参数 SelectionKey可以通过“位或”运算符添加多个,意思是 selector对监听的 channel 的什么事件感兴趣。具体使用请看下一节。
+ 通过 selector 选择通道：
|Method|Target| 
|:---|:---|
|int select()|阻塞到至少有一个通道在你注册的事件上就绪了|
|int select(long timeout)|和select()一样，除了最长会阻塞timeout毫秒(参数)|
|int selectNow()| 此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。|
|close()| 用完Selector后调用其close()方法会关闭该Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。|

### **SelectionKey**
+ "OP_READ" : 读
+ "OP_WRITE"： 写
+ "OP_ACCEPT" ： 接收连接
+ "OP_CONNECT"：连接
|Method|Target| 
|:---|:---|
|SelectableChannel channel()|获取当前 key 关联的 channel |
|Selector selector()|获取当前 key 关联的 selector |
|int interestOps()|获取当前 Key 对应的操作集 |
|int readyOps()|通道已经准备就绪的操作的集合 |
|boolean isValid()|当前 key 是否有效 |
|boolean isReadable()|当前 key 是否读就绪 |
|boolean isWritable()|当前 key 是否写就绪 |
|boolean isConnectable()|当前 key 是否连接就绪 |
|boolean isAcceptable()|当前 key 是否接收就绪 |
|Object attach(Object ob)|为Key 添加附加对象，这样可以方便识别某个指定的channel|
          
## **Channel(通道)** 
    Channel 类似于 IO 中的流，但也有不同之处：
    1. Channel 是双向的，流是单向的只能输入/输出
    2. Channel 的数据必须写入 Buffer 或者从 Buffer 中读取
    3. Channel 可以异步的读写

    
### **FileChannel**： 
    从文件中读取数据。FileChannel 无法设置为非阻塞模式，它总是运行在阻塞模式下。FileChannel 无法直接获取，它总是要通过流（InputStream、OutputStream或RandomAccessFile）来获取一个FileChannel。
|Method|Target| 
|:---|:---|
|int read(ByteBuffer bf)|从Buffer 中读取数据，返回值为读取数据的大小，若为-1则表示文件结束|
|int write(ByteBuffer src)|将Buffer中的数据写到文件中。|
|long position()|获取当前文件的指针位置|
|FileChannel position(long newPosition)|将指针指向 newPosition 位置，返回当前FileChannel。|
|long size()|返回文件的大小|
```
try {
    RandomAccessFile raf = new RandomAccessFile("e:\\test.txt","rw");
    RandomAccessFile raf1 = new RandomAccessFile("e:\\test1.txt","rw");
    FileChannel fromChannel = raf.getChannel();
    FileChannel toChannel = raf1.getChannel();
    //初始化一个 1024byte大小的 ByteBuffer
    ByteBuffer bf = ByteBuffer.allocate(1024);
    bf.clear();
    //读取数据，若 read()返回值等于 -1 ，则表示文件结束
    while ((toChannel.read(bf)) != -1){
        //将Buffer 从"读"模式切换到"写"模式
        bf.flip();
        while(bf.hasRemaining()) {
            fromChannel.write(bf);
        }
        //清空Buffer
        bf.clear();
    }
    // 关闭FileChannel
    fromChannel.close();
    toChannel.close();
    
    // 当2个通道传输时，其中一个是FileChannle 就可以使用下面2种方式来做通道间得读写
    // 1. transferTo(long position, long count, WritableByteChannel target) 
    // 2 .long transferFrom(ReadableByteChannel src, long position, long count)  
    // 此外要注意，transferFrom 在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据
    //（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中
    long count = fromChannel.size();
    fromChannel.transferTo(0L, count, toChannel);
    //上面的方法等同于下面使用 transferFrom()
    toChannel.transferFrom(fromChannel, 0L, count);
} catch (Exception e) {
    e.printStackTrace();
}
```

### **DatagramChannel**：能通过UDP读取网络中数据
+ DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。
+ 打开 DatagramChannel
```
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(8080));
```

+ 接收数据： SocketAddress receive(ByteBuffer dst)
```
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
// 若 Buffer 不能完全接收数据包中的数据，多余的数据将会被丢弃
channel.receive(buf);
```

+ 发送数据： int send(ByteBuffer src, SocketAddress target)
```
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();
// 注意UDP在数据传送方面没有任何保证
int bytesSent = channel.send(buf, new InetSocketAddress(80));
```

+ 连接到特定的地址
```
// 可以将DatagramChannel“连接”到网络中的特定地址的。
//由于UDP是无连接的，连接到特定地址并不会像TCP通道那样创建一个真正的连接。
//而是锁住DatagramChannel ，让其只能从特定地址收发数据。
channel.connect(new InetSocketAddress(80));
//当连接后，也可以使用read()和write()方法，就像在用传统的通道一样。只是在数据传送方面没有任何保证。
int bytesRead = channel.read(buf);
int bytesWritten = channel.write(but);
```


### **SocketChannel**：能通过TCP获取网络中的数据
+ 打开SocketChannel： 
```
SocketChannel socketChannel = SocketChannel.open(); 
socketChannel.connect(new InetSocketAddress(80));
```

+  关闭SocketChannel： 

```
socketChannel.close();
```

+  从SocketChannel中读数据： 

```
ByteBuffer bf = ByteBuffer.allocate(1024);
socketChannel.read(bf);
```

+  从SocketChannel中写数据： 

```
ByteBuffer bf = ByteBuffer.allocate(1024);
bf.clear();
bf.put("Hello Java".getBytes());
bf.flip();
//注意SocketChannel.write()方法的调用是在一个while循环中的。
//Write()方法无法保证能写多少字节到SocketChannel。所以，我们重复调用write()直到Buffer没有要写的字节为止。
while(bf.hasRemaining()){
    socketChannel.writer(bf);
}
```

+ 非阻塞模式下的connect()、read()、write():
    + connect(): 如果SocketChannel在非阻塞模式下，此时调用connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用finishConnect()的方法
    
    ```
    socketChannel.configureBlocking(false);
    socketChannel.connect(new InetSocketAddress(80));
    while(! socketChannel.finishConnect() ){
        //wait, or do something else...
    }
    ```
    
    + write() : 非阻塞模式下，write()方法在尚未写出任何内容时可能就返回了。所以需要在循环中调用write()。
    + read(): 非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值，它会告诉你读取了多少字节。
    
### **ServerSocketChannel**：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。
+ 打开 ServerSocketChannel:
```
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(81));
```    

+ 关闭 ServerSocketChannel:
```
serverSocketChannel.close();
``` 

+ 监听新进来的连接
```
while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    //do something with socketChannel...
}

```

+ 非阻塞模式： ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null

    
## **Buffer(缓冲区)**
### **BUffer 基本用法**
1. 分配Buffer， ByteBuffer bf = ByteBuffer.allocate(1024);不同类型的Buffer都是调用对应类型的allocate(int) 方法获得。
2. 写入数据到 Buffer， 使用 put()方法，Buffer中有很多put()方法，自己查找所需用的。
3. 调用 filp() 切换模式，将Buffer 从"写"模式切换到"读"模式
4. 从 Buffer 中读取数据， 使用get()方法，Buffer中有很多get()方法，自己查找所需用的。
5. 调用 clear() 清空或者compact()方法；2者的区别： clear() 会清空所有的数据，将postion 置为 0，limit置为 capacity; compact() 只会清空已读的数据，并将未读数据移动到 Buffer 的前端，并将 position 的位置移动到未读数据的尾部，limit置为 capacity。
6. rewind(): 可以利用rewind()重复读取Buffer中的数据，因为rewind()只是将postion 置为0，并未重置limit。
7. mark()与reset()： mark()标记当前position 位置，reset()将position重置为上次mark()的位置。
    

### **Buffer 的 position、limit 和 capacity**
+ capacity: 指 Buffer 的大小，"读"模式与"写"模式相同，不会随着模式的切换而改变。
+ position: 当前游标所处位置，初始值为0，在"写"模式下，指数据写入的位置；"读"模式下指当前读取的位置；position <= limit
+ limit: 在"写"模式下，等于 capacity；在"读"模式，等于在模式切换前 position 的位置。
+ 调用filp(): 将 limit 置为"写"模式下最终读取数据的位置（即原position）；postion = 0；capacity 不变


### **Buffer的类型 (包含了所有的基本类型)**
+ ByteBuffer
+ CharBuffer
+ DoubleBuffer
+ FloatBuffer
+ IntBuffer
+ LongBuffer
+ ShortBuffer






