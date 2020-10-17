---
title: JavaNIO
date: 2020-10-10 16:31:32
tags: 
  - java
  - IO
categories: java
---

##  Java共支持3种网络编程模型/IO模式：BIO、NIO、AIO

1. ​	Java BIO ： 同步并阻塞(传统阻塞型)，服务器实现模式为一个连接一个线程，即客户端 有连接请求时服务器端就需要启动一个线程	进行处理，如果这个连接不做任何事情会造成 不必要的线程开销 。

2. ​	Java NIO ： 同步非阻塞，服务器实现模式为一个线程处理多个请求(连接)，即客户端发 送的连接请求都会注册到多路复用器上，多

    ​	路复用器轮询到连接有I/O请求就进行处理 。

3. ​	 Java AIO(NIO.2) ： 异步非阻塞，AIO 引入异步通道的概念，采用了 Proactor 模式，简 化了程序编写，有效的请求才启动线程，

    ​	它的特点是先由操作系统完成后才通知服务端程。

##  I/O模型 BIO、NIO、AIO适用场景分析

1. BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高， 并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解。
2. NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，弹幕 系统，服务器间通讯等。编程比较复杂，JDK1.4开始支持。
3. AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分 调用OS参与并发操作，编程比较复杂，JDK7开始支持。

## Java BIO 基本介绍

Java BIO 就是传统的java io 编程，其相关的类和接口在 java.io 2) BIO(blocking I/O) ： 同步阻塞，服务器实现模式为一个连接一个线程，即客户端有连 接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造 成不必要的线程开销，可以通过线程池机制改善(实现多个客户连接服务器)。 

 BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高， 并发局限于应用中，JDK1.4以前的唯一选择，程序简单易理解

### BIO编程简单流程

1. 服务器端启动一个ServerSocket
2. 客户端启动Socket对服务器进行通 信，默认情况下服务器端需要对每 个客户 建立一个线程与之通讯
3. 客户端发出请求后, 先咨询服务器 是否有线程响应，如果没有则会等 待，或者被拒绝
4. 如果有响应，客户端线程会等待请 求结束后，在继续执行

####  Java BIO 问题分析

1) 每个请求都需要创建独立的线程，与对应的客户端进行数据 Read，业务处理，数据 Write 。
2) 当并发数较大时，需要创建大量线程来处理连接，系统资源占 用较大。
3) 连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在 Read 操作上，造成线程资源浪费。

##  Java NIO 基本介绍

1. Java NIO 全称 java non-blocking IO，是指 JDK 提供的新 API。从 JDK1.4 开始，Java 提供了一系列改进的输入/输出 的新特性，被统称为 NIO(即 New IO)，是同步非阻塞的。
2. NIO 相关类都被放在 java.nio 包及子包下，并且对原 java.io 包中的很多类进行改写。
3. NIO 有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector(选择器) 4) NIO是 面向缓冲区 ，或者面向 块 编程的。数据读取到一个 它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就 增加了处理过程中的灵活性，使用它可以提供非阻塞式的高 伸缩性网络。
4. Java NIO的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得 到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线 程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞 写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这 个线程同时可以去做别的事情。
5. 通俗理解：NIO是可以做到用一个线程来处理多个操作的。假设有10000个请求过来, 根据实际情况，可以分配50或者100个线程来处理。不像之前的阻塞IO那样，非得分 配10000个。
6. HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求 的数量比HTTP1.1大了好几个数量级。

### NIO 和 BIO 的比较

1. BIO 以流的方式处理数据,而 NIO 以块的方式处理数据,块 I/O 的效率比流 I/O 高很多 。
2. BIO 是阻塞的，NIO 则是非阻塞的。
3. BIO基于字节流和字符流进行操作，而 NIO 基于 Channel(通道)和 Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。 Selector(选择器)用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道 

### NIO 三大核心关系

​	**Selector 、 Channel 和 Buffer**

1) 每个channel 都会对应一个Buffer 

2) Selector 对应一个线程， 一个线程对应多个channel(连接) 

3) 该图反应了有三个channel 注册到 该selector //程序 

4) 程序切换到哪个channel 是有事件决定的, Event 就是一个重要的概念 

5) Selector 会根据不同的事件，在各个通道上切换 

6) Buffer 就是一个内存块 ， 底层是有一个数组 

7) 数据的读取写入是通过Buffer, 这个和BIO , BIO 中要么是输入流，或者是 输出流, 不能双向，但是NIO的Buffer 是可以读也可以写, 需要 flip 方法切换 

8) channel 是双向的, 可以返回底层操作系统的情况, 比如Linux ， 底层的操作系统 通道就是双向的.

### 缓冲区(Buffer) 

#### 基本介绍

缓冲区（Buffer）：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个 容器对象(含数组)，该对象提供了一组方法，可以更轻松地使用内存块，，缓冲区对 象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、 网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer。

### 通道(Channel)

#### 基本介绍

1. BIO 中的 stream 是单向的，例如 FileInputStream 对 象只能进行读取数据的操作，而 NIO 中的通道 (Channel)是双向的，可以读操作，也可以写操作。
2. Channel在NIO中是一个接口 public interface Channel extends Closeable{} 。
3. 常用的 Channel 类有：FileChannel、 DatagramChannel、ServerSocketChannel 和 SocketChannel。【ServerSocketChanne 类似 ServerSocket , SocketChannel 类似 Socket】
4. FileChannel 用于文件的数据读写， DatagramChannel 用于 UDP 的数据读写， ServerSocketChannel 和 SocketChannel 用于 TCP 的数据读写。
5. NIO的通道类似于流，但有些区别如下：

- 通道可以同时进行读写，而流只能读或者只能写。
- 通道可以实现异步读写数据。
- 通道可以从缓冲读数据，也可以写数据到缓冲。

### 关于Buffer 和 Channel的注意事项和细节

1. ByteBuffer 支持类型化的put 和 get, put 放入的是什么数据类型，get就应该使用 相应的数据类型来取出，否则可能有 BufferUnderflowException 异常。
2. 可以将一个普通Buffer 转成只读Buffer 。
3. NIO 还提供了 MappedByteBuffer， 可以让文件直接在内存（堆外的内存）中进 行修改， 而如何同步到文件由NIO 来完成。
4. 前面我们讲的读写操作，都是通过一个Buffer 完成的，NIO 还支持 通过多个 Buffer (即 Buffer 数组) 完成读写操作，即 Scattering 和 Gathering。

### Selector(选择器)

#### 基本介绍 

1. Java 的 NIO，用非阻塞的 IO 方式。可以用一个线程，处理多个的客户端连 接，就会使用到Selector(选择器)。
2. Selector 能够检测多个注册的通道上是否有事件发生(注意:多个Channel以 事件的方式可以注册到同一个Selector)，如果有事件发生，便获取事件然 后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个 通道，也就是管理多个连接和请求。
3. 只有在 连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少 了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程 4) 避免了多线程之间的上下文切换导致的开销。

#### 注意事项

1. NIO中的 ServerSocketChannel功能类似ServerSocket，SocketChannel功能类 似Socket。

2. 注册进Select的Channel必须是非阻塞的，所以FileChannel不适用Selector，因为FileChannel不能切换为非阻塞模式，更准确的来说是因为FileChannel没有继承SelectableChannel。Socketchannel可以正常使用。

3. selector 相关方法说明：

    ​	selector.select()//阻塞 

    ​	selector.select(1000);//阻塞1000毫秒，在1000毫秒后返回 

    ​	selector.wakeup();//唤醒selector 

    ​	selector.selectNow();//不阻塞，立马返还



### NIO写入文件代码示例

```java
import java.io.FileOutputStream;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class NIOFileChannel {
    public static void main(String[] args) throws Exception{

        String str = "hello,world";
        //创建一个输出流->channel
        FileOutputStream fileOutputStream = new FileOutputStream("d:\\file.txt");

        //通过 fileOutputStream 获取 对应的 FileChannel
        //这个 fileChannel 真实 类型是  FileChannelImpl
        FileChannel fileChannel = fileOutputStream.getChannel();

        //创建一个缓冲区 ByteBuffer
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        //将 str 放入 byteBuffer
        byteBuffer.put(str.getBytes());


        //对byteBuffer 进行flip
        byteBuffer.flip();

        //将byteBuffer 数据写入到 fileChannel
        fileChannel.write(byteBuffer);
        fileOutputStream.close();

    }
}

```



### NIO聊天室代码示例

Server端

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;

public class GroupChatServer {
    //定义属性
    private Selector selector;
    private ServerSocketChannel listenChannel;
    private static final int PORT = 6667;

    //构造器
    //初始化工作
    public GroupChatServer() {

        try {

            //得到选择器
            selector = Selector.open();
            //ServerSocketChannel
            listenChannel =  ServerSocketChannel.open();
            //绑定端口
            listenChannel.socket().bind(new InetSocketAddress(PORT));
            //设置非阻塞模式
            listenChannel.configureBlocking(false);
            //将该listenChannel 注册到selector
            listenChannel.register(selector, SelectionKey.OP_ACCEPT);

        }catch (IOException e) {
            e.printStackTrace();
        }

    }

    //监听
    public void listen() {

        System.out.println("监听线程: " + Thread.currentThread().getName());
        try {

            //循环处理
            while (true) {

                int count = selector.select();
                if(count > 0) {//有事件处理

                    //遍历得到selectionKey 集合
                    Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                    while (iterator.hasNext()) {
                        //取出selectionkey
                        SelectionKey key = iterator.next();

                        //监听到accept
                        if(key.isAcceptable()) {
                            SocketChannel sc = listenChannel.accept();
                            sc.configureBlocking(false);
                            //将该 sc 注册到seletor
                            sc.register(selector, SelectionKey.OP_READ);

                            //提示
                            System.out.println(sc.getRemoteAddress() + " 上线 ");

                        }
                        if(key.isReadable()) { //通道发送read事件，即通道是可读的状态
                            //处理读 (专门写方法..)

                            readData(key);

                        }
                        //当前的key 删除，防止重复处理
                        iterator.remove();
                    }

                } else {
                    System.out.println("等待....");
                }
            }

        }catch (Exception e) {
            e.printStackTrace();

        }finally {
            //发生异常处理....

        }
    }

    //读取客户端消息
    private void readData(SelectionKey key) {

        //取到关联的channle
        SocketChannel channel = null;

        try {
           //得到channel
            channel = (SocketChannel) key.channel();
            //创建buffer
            ByteBuffer buffer = ByteBuffer.allocate(1024);

            int count = channel.read(buffer);
            //根据count的值做处理
            if(count > 0) {
                //把缓存区的数据转成字符串
                String msg = new String(buffer.array());
                //输出该消息
                System.out.println("form 客户端: " + msg);

                //向其它的客户端转发消息(去掉自己), 专门写一个方法来处理
                sendInfoToOtherClients(msg, channel);
            }

        }catch (IOException e) {
            try {
                System.out.println(channel.getRemoteAddress() + " 离线了..");
                //取消注册
                key.cancel();
                //关闭通道
                channel.close();
            }catch (IOException e2) {
                e2.printStackTrace();;
            }
        }
    }

    //转发消息给其它客户(通道)
    private void sendInfoToOtherClients(String msg, SocketChannel self ) throws  IOException{

        System.out.println("服务器转发消息中...");
        System.out.println("服务器转发数据给客户端线程: " + Thread.currentThread().getName());
        //遍历 所有注册到selector 上的 SocketChannel,并排除 self
        for(SelectionKey key: selector.keys()) {

            //通过 key  取出对应的 SocketChannel
            Channel targetChannel = key.channel();

            //排除自己
            if(targetChannel instanceof  SocketChannel && targetChannel != self) {

                //转型
                SocketChannel dest = (SocketChannel)targetChannel;
                //将msg 存储到buffer
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                //将buffer 的数据写入 通道
                dest.write(buffer);
            }
        }

    }

    public static void main(String[] args) {

        //创建服务器对象
        GroupChatServer groupChatServer = new GroupChatServer();
        groupChatServer.listen();
    }
}

```



Client端

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Scanner;

public class GroupChatClient {

    //定义相关的属性
    private final String HOST = "127.0.0.1"; // 服务器的ip
    private final int PORT = 6667; //服务器端口
    private Selector selector;
    private SocketChannel socketChannel;
    private String username;

    //构造器, 完成初始化工作
    public GroupChatClient() throws IOException {

        selector = Selector.open();
        //连接服务器
        socketChannel = socketChannel.open(new InetSocketAddress("127.0.0.1", PORT));
        //设置非阻塞
        socketChannel.configureBlocking(false);
        //将channel 注册到selector
        socketChannel.register(selector, SelectionKey.OP_READ);
        //得到username
        username = socketChannel.getLocalAddress().toString().substring(1);
        System.out.println(username + " is ok...");

    }

    //向服务器发送消息
    public void sendInfo(String info) {

        info = username + " 说：" + info;

        try {
            socketChannel.write(ByteBuffer.wrap(info.getBytes()));
        }catch (IOException e) {
            e.printStackTrace();
        }
    }

    //读取从服务器端回复的消息
    public void readInfo() {

        try {

            int readChannels = selector.select();
            if(readChannels > 0) {//有可以用的通道

                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()) {

                    SelectionKey key = iterator.next();
                    if(key.isReadable()) {
                        //得到相关的通道
                       SocketChannel sc = (SocketChannel) key.channel();
                       //得到一个Buffer
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        //读取
                        sc.read(buffer);
                        //把读到的缓冲区的数据转成字符串
                        String msg = new String(buffer.array());
                        System.out.println(msg.trim());
                    }
                    iterator.remove(); //删除当前的selectionKey, 防止重复操作
                }
            } else {
                System.out.println("没有可以用的通道...");

            }

        }catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws Exception {

        //启动我们客户端
        GroupChatClient chatClient = new GroupChatClient();

        //启动一个线程, 读取从服务器发送数据
        new Thread() {
            public void run() {

                while (true) {
                    chatClient.readInfo();
//                    try {
//                        Thread.currentThread().sleep(3000);
//                    }catch (InterruptedException e) {
//                        e.printStackTrace();
//                    }
                    System.out.println("11111111");
                }


            }
        }.start();

        //发送数据给服务器端
        Scanner scanner = new Scanner(System.in);

        while (scanner.hasNextLine()) {
            String s = scanner.nextLine();
            chatClient.sendInfo(s);
        }
    }

}

```

## Java AIO 基本介绍

1. JDK 7 引入了 Asynchronous I/O，即 AIO。在进行 I/O 编程中，常用到两种模式： Reactor和 Proactor。Java 的 NIO 就是 Reactor，当有事件触发时，服务器端得 到通知，进行相应的处理 。
2. AIO 即 NIO2.0，叫做异步不阻塞的 IO。AIO 引入异步通道的概念，采用了 Proactor 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作 系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时 间较长的应用。

##  END

注：笔记参考自尚硅谷-韩顺平的Netty相关教程