# Socket和ServerSocket的简单介绍及例子

### Socket 类

　　socket可以使一个应用从网络中读取和写入数据，不同计算机上的两个应用可以通过连接发送和接受字节流，当发送消息时，你需要知道对方的ip和端口，在java中，socket指的是java.net.Socket类。 
　　在java.net.Socket中，可以看到socket有多种构造函数 
　　 
　　![这里写图片描述](https://img-blog.csdn.net/20151217223244553)

　　以public Socket(String host, int port)为例，host为远程机器名称或ip地址，port为端口号。若连接本地的Server，其端口号为8080，可以写成如下格式 
　　new Socket(“localhost”, 8080)； 
　　一旦成功创建一个Socket类的实例，可以用它来发送和接收字节流，发送时调用getOutputStream方法获取一个java.io.OutputStream对象，接收远程对象发送来的信息可以调用getInputStream方法来返回一个java.io.InputStream对象。 

　　

### **ServerSocket类**

**ServerSocket详解**：https://www.jianshu.com/p/665994c2e784

　　Socket类代表一个客户端套接字，即任何时候连接到一个远程服务器应用时构建所需的socket。现在，要实现一个服务器应用，需要不同的做法。服务器需随时待命，因为不知道客户端什么时候会发来请求，此时，我们需要使用ServerSocket，对应的是java.net.ServerSocket类。 
　　ServerSocket与Socket不同，ServerSocket是等待客户端的请求，一旦获得一个连接请求，就创建一个Socket示例来与客户端进行通信。 
　　ServerSocket的构造函数也有多种重载形式： 
　　![这里写图片描述](https://img-blog.csdn.net/20151217224727410)

　　ServerSocket 有一个不带参数的默认构造方法。通过该方法创建的 ServerSocket 不与任何端口绑定，接下来还需要通过 bind()方法与特定端口绑定。这个默认构造方法的用途是，允许服务器在绑定到特定端口之前，先设置ServerSocket 的一些选项。因为一旦服务器与特定端口绑定，有些选项就不能再改变了。 
　　例如

```java
ServerSocket serverSocket=new ServerSocket();serverSocket.setReuseAddress(true); //设置 ServerSocket 的选项serverSocket.bind(new InetSocketAddress(8080)); //与 8080 端口绑定
```

把以上程序改成

```java
ServerSocket serverSocket=new ServerSocket(8080);serverSocket.setReuseAddress(true); //设置 ServerSocket 的选项
```

-  

那 么 serverSocket.setReuseAddress(true) 方 法 就 不 起 任何作用了 
　　我们也可以使用如下构造函数创建一个ServerSocket实例

```java
ServerSocket serverSocket = new ServerSocket(port,3); 
```

　　把连接请求队列的长度设为 3。这意味着当队列中有了 3 个连接请求时，如果 Client 再请求连接，就会被 Server拒绝，因为服务器队列已经满了。我们使用的 serverSocket.accept()方法就是从队列中取出连接请求。 
　　总之， 
　　客户端向服务器发送请求可分为以下步骤： 
　　1.创建一个Socket实例 
　　2.利用I/O流与服务器进行通信 
　　3.关闭socket 
　　 
　　服务器接收客户端请求步骤： 
　　１.创建一个ServerSocket实例，监听客户端发来的请求。 
　　2.与客户端获取连接后，创建一个Socket实例，利用I/O流与客户端进行通信，完毕后关闭Socket。

　　当然，服务器可以接收多个客户端的请求，所以如果服务器是一个一个顺序相应肯定会带来不好的体验，因此使用多线程来为多个客户端提供服务 
Client代码：

```java
package com.zhoufenqin.socket.client; import java.io.BufferedReader;import java.io.IOException;import java.io.InputStreamReader;import java.io.PrintStream;import java.net.Socket; public class Client {    public static final int port = 8080;       public static final String host = "localhost";    public static void main(String[] args) {            System.out.println("Client Start...");            while (true) {                Socket socket = null;              try {                  //创建一个流套接字并将其连接到指定主机上的指定端口号                  socket = new Socket(host,port);                     //读取服务器端数据                    BufferedReader input = new BufferedReader(new InputStreamReader(socket.getInputStream()));                    //向服务器端发送数据                    PrintStream out = new PrintStream(socket.getOutputStream());                    System.out.print("请输入: \t");                    String str = new BufferedReader(new InputStreamReader(System.in)).readLine();                    out.println(str);                     String ret = input.readLine();                     System.out.println("服务器端返回过来的是: " + ret);                    // 如接收到 "OK" 则断开连接                    if ("OK".equals(ret)) {                        System.out.println("客户端将关闭连接");                        Thread.sleep(500);                        break;                    }                     out.close();                  input.close();              } catch (Exception e) {                  System.out.println("客户端异常:" + e.getMessage());               } finally {                  if (socket != null) {                      try {                          socket.close();                      } catch (IOException e) {                          socket = null;                           System.out.println("客户端 finally 异常:" + e.getMessage());                       }                  }              }          }        }    }
```

-  

Server代码:

```java
package com.zhoufenqin.socket.server; import java.io.BufferedReader;import java.io.InputStreamReader;import java.io.PrintStream;import java.net.ServerSocket;import java.net.Socket; public class Server {    public static final int port = 8080;//监听的端口号          public static void main(String[] args) {            System.out.println("Server...\n");            Server server = new Server();            server.init();        }         public void init() {            try {                //创建一个ServerSocket，这里可以指定连接请求的队列长度              //new ServerSocket(port,3);意味着当队列中有3个连接请求是，如果Client再请求连接，就会被Server拒绝             ServerSocket serverSocket = new ServerSocket(port);                while (true) {                    //从请求队列中取出一个连接                Socket client = serverSocket.accept();                    // 处理这次连接                    new HandlerThread(client);                }            } catch (Exception e) {                System.out.println("服务器异常: " + e.getMessage());            }        }         private class HandlerThread implements Runnable {            private Socket socket;            public HandlerThread(Socket client) {                socket = client;                new Thread(this).start();            }             public void run() {                try {                    // 读取客户端数据                    BufferedReader input = new BufferedReader(new InputStreamReader(socket.getInputStream()));                    String clientInputStr = input.readLine();//这里要注意和客户端输出流的写方法对应,否则会抛 EOFException                  // 处理客户端数据                    System.out.println("客户端发过来的内容:" + clientInputStr);                     // 向客户端回复信息                    PrintStream out = new PrintStream(socket.getOutputStream());                    System.out.print("请输入:\t");                    // 发送键盘输入的一行                    String s = new BufferedReader(new InputStreamReader(System.in)).readLine();                    out.println(s);                     out.close();                    input.close();                } catch (Exception e) {                    System.out.println("服务器 run 异常: " + e.getMessage());                } finally {                    if (socket != null) {                        try {                            socket.close();                        } catch (Exception e) {                            socket = null;                            System.out.println("服务端 finally 异常:" + e.getMessage());                        }                    }                }           }        }    }
```

-  

结果如下所示： 
![这里写图片描述](https://img-blog.csdn.net/20151217222304502) 
![这里写图片描述](https://img-blog.csdn.net/20151217222315564)