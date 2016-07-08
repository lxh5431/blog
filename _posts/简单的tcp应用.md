---
title: 简单的tcp应用
date: 2016-07-01 22:41:39
tags: 编程
categories: tcp
tag:
---
**tcp的定义**

TCP/IP协议族按照层次由上到下，层层包装。
##### 应用层
这里面有http，ftp,等等我们熟悉的协议。
##### 传输层
著名的TCP和UDP协议就在这个层次
##### 网络层
IP协议,它负责对数据加上IP地址和其他的数据（后面会讲到）以确定传输的目标。
##### 数据链路层
这个层次为待传送的数据加入一个以太网协议头，并进行CRC编码，为最后的数据传输做准备。
发送协议的主机从上自下将数据按照协议封装，而接收数据的主机则按照协议从得到的数据包解开，最后拿到需要的数据。这种结构非常有栈的味道，所以某些文章也把tcp/ip协议族称为tcp/ip协议栈。
而我现在要学习的就是tcp的简单实用
<!--more-->
在tcp的编程有两个关键字Socket和serverSocket,实现了基本的双向传输，下面是简单的例子让我们来简单的理解tcp的通信
![简单的协议通信](http://o94r16s1l.bkt.clouddn.com/%E5%A5%97%E6%8E%A5%E5%AD%97%E5%92%8C%E5%8D%8F%E8%AE%AE%E7%9A%84%E5%85%B3%E7%B3%BB.png)
首先创建tcp套接字,并制定服务器和端口号
获取套接字的输入输出流
发送字符串并回馈给服务器，从服务器接受回馈信息，
打印回馈的字符串
关闭套接字
服务器：
创建服务器套接字
永久迭代，迭代处理心连接请求
报告已连接的客户端
获取输入输出流
接收数据并复制数据，直到客户端关闭
如何让两个不同的终端通信，最简单的就是协议，按照什么规则去连接传输解码，这样才能够利于维护和扩展
java客户端
```java  




package tcpclient;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.net.SocketException;

public class TCPEchoClient {
	/**
	 *
	 * @author lxh
	 * @param args
	 * @throws IOException
	 * @version 1.0
	 * @throws
	 */
public static void main(String[] args)throws  IOException {
	//判断输入的参数
	if((args.length<2)||(args.length>3))
		throw new IllegalArgumentException("Parameter(s):<Server><word>[<Port>]");
	//转换为数组，
	String server=args[0];
	byte[] data=args[1].getBytes();
	int servPort=(args.length==3)?Integer.parseInt(args[2]):7;
	Socket socket=new Socket(server,servPort);
	System.out.println("Connected to server...sending echo string");
	//获取套接字的输入输出流
	InputStream in=socket.getInputStream();
	OutputStream out=socket.getOutputStream();
	out.write(data);//读写数据
	int totalBytesRcvd=0;
	int bytesRcvd;
	//接受服务器返回的数据
	while(totalBytesRcvd<data.length){
		if((bytesRcvd=in.read(data, totalBytesRcvd, data.length-totalBytesRcvd))==-1)
			throw new SocketException("Connection closed permaturely");
		totalBytesRcvd+=bytesRcvd;
	}
	System.out.println("Received:"+new String(data));
	socket.close();
}


}
```
java的服务器端
```java  

package tcpclient;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketAddress;


public class TCPEchoServer {

	private static final int BUFSIZE = 32;

	public static void main(String[] args)throws IOException {
		if(args.length!=1)
			throw new IllegalArgumentException("Parameter(s):<Port>");
		int servPort=Integer.parseInt(args[0]);
		ServerSocket servsock=new ServerSocket(servPort);
		int recvMsgSize;
		byte[] receiveBuf=new byte[BUFSIZE];
		while(true){
			Socket clntSock=servsock.accept();
			SocketAddress clientAddress=clntSock.getRemoteSocketAddress();
			System.out.println("Handing clinet at"+clientAddress);

		InputStream in=clntSock.getInputStream();
		OutputStream out=clntSock.getOutputStream();
		while((recvMsgSize=in.read(receiveBuf))!=-1){
			out.write(receiveBuf,0,recvMsgSize);
		}
		clntSock.close();
}
	}
}


```
