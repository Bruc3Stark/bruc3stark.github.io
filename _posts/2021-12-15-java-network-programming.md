---
layout: mypost
title: Java网络通信
categories: [Java]
---

### 协议和Socket

TCP/IP协议是一个协议簇，里面包括很多协议的。TCP/UDP只是其中的一个。之所以命名为TCP/IP协议，因为TCP,IP协议是两个很重要的协议，就用他两命名了

网络由下往上分为物理层、数据链路层、网络层、传输层、会话层、表示层和应用层。

IP 协议对应于网络层，TCP协议对应于传输层，HTTP协议对应于应用层，三者从本质上来说没有可比性

可以说，TPC/IP协议是传输层协议，主要解决数据如何在网络中传输，而HTTP是应用层协议，主要解决如何包装数据

Socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Socket，我们才能使用TCP/IP协议

实际上，Socket跟TCP/IP协议没有必然的联系。Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所以说，Socket的出现只是使得程序员更方便地使用TCP/IP协议栈而已，是对TCP/IP协议的抽象。

### TCP协议

TCP（Transmission Control Protocol，传输控制协议）是面向连接的协议，也就是说，在收发数据前，必须和对方建立可靠的连接。一个TCP连接必须要经过三次“对话”才能建立起来，其中的过程非常复杂，只简单的描述下这三次对话的简单过程：主机A向主机B发出连接请求数据包：“我想给你发数据，可以吗？”，这是第一次对话；主机B向主机A发送同意连接和要求同步（同步就是两台主机一个在发送，一个在接收，协调工作）的数据包：“可以，你什么时候发？”，这是第二次对话；主机A再发出一个数据包确认主机B的要求同步：“我现在就发，你接着吧！”，这是第三次对话。三次“对话”的目的是使数据包的发送和接收同步，经过三次“对话”之后，主机A才向主机B正式发送数据。

### UDP协议

UDP（User Data Protocol，用户数据报协议）

1. UDP是一个非连接的协议，传输数据之前源端和终端不建立连接，当它想传送时就简单地去抓取来自应用程序的数据，并尽可能快地把它扔到网络上。在发送端，UDP传送数据的速度仅仅是受应用程序生成数据的速度、计算机的能力和传输带宽的限制；在接收端，UDP把每个消息段放在队列中，应用程序每次从队列中读一个消息段。

2. 由于传输数据不建立连接，因此也就不需要维护连接状态，包括收发状态等，因此一台服务机可同时向多个客户机传输相同的消息。

3. UDP信息包的标题很短，只有8个字节，相对于TCP的20个字节信息包的额外开销很小。

4. 吞吐量不受拥挤控制算法的调节，只受应用软件生成数据的速率、传输带宽、源端和终端主机性能的限制。

5. UDP使用尽最大努力交付，即不保证可靠交付，因此主机不需要维持复杂的链接状态表（这里面有许多参数）。

6. UDP是面向报文的。发送方的UDP对应用程序交下来的报文，在添加首部后就向下交付给IP层。既不拆分，也不合并，而是保留这些报文的边界，因此，应用程序需要选择合适的报文大小。

我们经常使用ping命令来测试两台主机之间TCP/IP通信是否正常，其实ping命令的原理就是向对方主机发送UDP数据包，然后对方主机确认收到数据包，如果数据包是否到达的消息及时反馈回来，那么网络就是通的。

### HTTP协议

HTTP协议即超文本传送协议(Hypertext Transfer Protocol )，是Web联网的基础，也是手机联网常用的协议之一，HTTP协议是建立在TCP协议之上的一种应用。

HTTP连接最显著的特点是客户端发送的每次请求都需要服务器回送响应，在请求结束后，会主动释放连接。从建立连接到关闭连接的过程称为“一次连接”。

1. 在HTTP 1.0中，客户端的每次请求都要求建立一次单独的连接，在处理完本次请求后，就自动释放连接。

2. 在HTTP 1.1中则可以在一次连接中处理多个请求，并且多个请求可以重叠进行，不需要等待一个请求结束后再发送下一个请求。

由于HTTP在每次请求结束后都会主动释放连接，因此HTTP连接是一种“短连接”，要保持客户端程序的在线状态，需要不断地向服务器发起连接请求。通常的 做法是即时不需要获得任何数据，客户端也保持每隔一段固定的时间向服务器发送一次“保持连接”的请求，服务器在收到该请求后对客户端进行回复，表明知道客 户端“在线”。若服务器长时间无法收到客户端的请求，则认为客户端“下线”，若客户端长时间无法收到服务器的回复，则认为网络已经断开。

### 区别

1. 基于连接与无连接

2. 对系统资源的要求（TCP较多，UDP少）

3. UDP程序结构较简单

4. 流模式与数据报模式

5. TCP保证数据正确性，UDP可能丢包，TCP保证数据顺序，UDP不保证

### 使用

想用着省事就用TCP协议

UDP协议有可能无法保障文件的一致性，因为无法保证接收顺序。

同时DatagramSocket对数据的结构也比Socket要稍微的严格一点

可以说UDP唯一的优势就是速度，免去了各种繁琐的稳定传输的步骤，性能比TCP要好得

### 代码实现

### InetAddress

在java中用InetAddress类来表示一个Internet地址

```java
InetAddress inetAddress = InetAddress.getLocalHost();
System.out.println(inetAddress);

inetAddress = InetAddress.getLoopbackAddress();
System.out.println(inetAddress);

inetAddress = InetAddress.getByAddress("baidu.com", new byte[]{(byte) 192,0,0,1});
System.out.println(inetAddress);

//自动进行DNS解析
inetAddress = InetAddress.getByName("baidu.com");
System.out.println(inetAddress);
```

输出

```
TMaize/192.168.56.1
localhost/127.0.0.1
baidu.com/192.0.0.1
baidu.com/111.13.101.208
```

### UDP通信

接收端代码

```java
//打开一个UDP服务并设置端口
DatagramSocket datagramSocket = new DatagramSocket(7777);

while(true){

    //用于接收数据
    byte[] buf = new byte[1024];
    DatagramPacket datagramPacket = new DatagramPacket(buf, buf.length);
    
    //等待数据
    datagramSocket.receive(datagramPacket);
    
    //处理数据
    String ip = datagramPacket.getAddress().getHostAddress();
    System.out.println("来自"+ip+":"+datagramPacket.getPort());
    String rec = new String(datagramPacket.getData(),0,datagramPacket.getLength());
    System.out.println("内容:"+rec);
    System.out.println("------------------");
}
```

发送端代码

```java
//打开一个UDP服务并设置端口
DatagramSocket datagramSocket = new DatagramSocket(6666);

//封装数据包，这里是发送给端口为7777的自己
byte[] data = "Hello".getBytes();
InetAddress inetAddress = InetAddress.getLocalHost();
DatagramPacket datagramPacket = new DatagramPacket(data, data.length,inetAddress, 7777);

//发送数据包
datagramSocket.send(datagramPacket);

//关闭
datagramSocket.close();
datagramSocket.close();
```

结果

```
来自192.168.56.1:6666
内容:Hello
------------------
```

### TCP通信

接收端代码

```java
//在端口6666上建立服务
ServerSocket serverSocket = new ServerSocket(6666);

while(true){
	//获得发送的数据
	Socket socket = serverSocket.accept();
	
	//获得流，同时转换一下便于读取
	BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
	String line;
	while((line = br.readLine())!=null){
		System.out.println(line);
	}

	br.close();
	socket.close();
}
```

发送端代码

```java
//连接到本机端口6666的TCP服务
Socket socket = new Socket(InetAddress.getLocalHost(), 6666);
OutputStream os = socket.getOutputStream();
//发送内容
os.write("Hello\nWorld".getBytes());
//关闭
os.close();
socket.close();
```

### Demo实例

http是要基于TCP连接基础上的，通过对报头的解析处理可以简单地实现一个http服务器

```
ServerSocket serverSocket = new ServerSocket(8080);

while(true){
	Socket socket =serverSocket.accept();
	
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
    String line;
    while(true){
    	line=bufferedReader.readLine();
    	if (line==null||line.equals("")) {
			break;
		}
    	System.out.println(line);
    	
    }
    System.out.println("-------------");
    
    OutputStream outputStream = socket.getOutputStream();  
    PrintWriter printWriter = new PrintWriter(outputStream);  
    printWriter.println("<b>successful!</b>");
    printWriter.flush();//刷新缓冲区
    
    socket.close();
} 
```

浏览器访问`http://127.0.0.1:8080/?name=123`,控制台输出

```
GET /?name=123 HTTP/1.1
Host: 127.0.0.1:8080
Connection: keep-alive
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
-------------
GET /favicon.ico HTTP/1.1
Host: 127.0.0.1:8080
Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36
Accept: */*
Referer: http://127.0.0.1:8080/?name=1
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
-------------
```

### 参考

> [TCP协议与UDP协议的区别](http://www.cnblogs.com/bizhu/archive/2012/05/12/2497493.html)
