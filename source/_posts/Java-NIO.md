---
title: Java NIO
date: 2019-09-15 16:46:17
tags:
- java
- 面试
- nio
---
主要分析java nio的基础知识和关键组件
<!--more-->

# 三个关键组件
## Channel
基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。

| Channel                                     |
| ------------------------------------------- |
| FileChannel : 从文件中读写数据              |
| DatagramChannel : 能通过UDP读写网络中的数据 |
| SocketChannel : 能通过TCP读写网络中的数据   |
| ServerSocketChannel : 监听新进来的TCP连接   |

## Buffer

### Buffer的实现包含 : 
ByteBuffer / CharBuffer / DoubleBuffer / FloatBuffer / IntBuffer / LongBuffer / ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。Java NIO 还有个 MappedByteBuffer，用于表示内存映射文件。

### Buffer使用步骤：
1. 写入数据到Buffer
2. 调用`flip()`方法
3. 从Buffer中读取数据
4. 调用`clear()`方法或者`compact()`方法

### 使用方法：
```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer from channel.
//也可以直接使用buf.put()
while (bytesRead != -1) {
	System.out.println("Read " + bytesRead);
	buf.flip(); //flip()方法将Buffer从写模式切换到读模式

	while(buf.hasRemaining()){
		System.out.print((char) buf.get()); // read 1 byte at a time
    //也可以使用inChannel.write将buffer中的数据读入channel
	}

	buf.clear(); //clear()方法会清空整个缓冲区，compact()方法只会清除已经读过的数据。
	bytesRead = inChannel.read(buf);
}
aFile.close();
```

### Buffer的三个重要属性：
- capacity - 作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

- position - 当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

  当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。

- limit - 在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。

  当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

![img](http://ifeve.com/wp-content/uploads/2013/06/buffers-modes.png)

### Buffer重要API : 
| 功能 | buf.get()          | 说明                           |
| ---- | ------------------ | ------------------------------ |
| 读   | buf.get()          | 读取一个字节                   |
|      | channel.write(buf) | 读取buf中的数据，并写入channel |
| 写   | buf.put()          | 向buf写入数据，有多个重载方法  |
|      | channel.read(buf)  | 读取channel中的数据，并写入buf |
| 清除 | buf.clear()        | 清空整个缓存区                 |
|      | buf.compact()      | 清除已经读过的数据             |

## Selector
Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

![img](http://ifeve.com/wp-content/uploads/2013/06/overview-selectors.png)

要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。