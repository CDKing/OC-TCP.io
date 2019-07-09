## iOS TCP简易交互

懒人理解：socket通信，双方必须知道对方是谁，数据的发送接收也要管理

**1）服务端：开启服务，打开端口，TCP客户端也要直接服务端的ip和端口才行**

```
- (void)tcpServer {
   // 第一步：创建socket
   int error = -1;
   
   // 创建socket套接字
   int serverSocketId = socket(AF_INET, SOCK_STREAM, 0);
   // 判断创建socket是否成功
   BOOL success = (serverSocketId != -1);
   
   // 第二步：绑定端口号
   if (success) {
      // Socket 地址
      struct sockaddr_in addr;
      
      // 初始化全置为0
      memset(&addr, 0, sizeof(addr));
      // 指定socket地址长度
      addr.sin_len = sizeof(addr);
      // 指定网络协议，比如这里使用的是TCP/UDP则指定为AF_INET
      addr.sin_family = AF_INET;
      // 指定端口号
      addr.sin_port = htons(150431);
      // 指定监听的ip，指定为INADDR_ANY时，表示监听所有的ip
      addr.sin_addr.s_addr = INADDR_ANY;
      // 绑定套接字
      error = bind(serverSocketId, (const struct sockaddr *)&addr, sizeof(addr));
      success = (error == 0);
   }
   // 第三步：监听
   if (success) {
      NSLog(@"bind server socket success");
      error = listen(serverSocketId, 5);
      success = (error == 0);
   }
   if (success) {
      // 监听成功
      while (true) {
          struct sockaddr_in peerAddr;
          int peerSocketId;
          socklen_t addrLen = sizeof(peerAddr);
          
          // 第四步：等待客户端连接
          // 服务器端等待从编号为serverSocketId的Socket上接收客户连接请求
          peerSocketId = accept(serverSocketId, (struct sockaddr *)&peerAddr, &addrLen);
          success = (peerSocketId != -1);
          if (success) {
              NSLog(@"连接成功,获取地址:%s,端口:%d",inet_ntoa(peerAddr.sin_addr),ntohs(peerAddr.sin_port));
              char buf[1024];
              size_t len = sizeof(buf);
              
              // 第五步：接收来自客户端的信息
              // 当客户端输入exit时才退出
              do {
                  // 接收来自客户端的信息
                  recv(peerSocketId, buf, len, 0);
                  if (strlen(buf) != 0) {
                      NSString *str = [NSString stringWithCString:buf encoding:NSUTF8StringEncoding];
                      if (str.length >= 1) {
                          NSLog(@"received message from client：%@",str);
                      }
                  }
              } while (strcmp(buf, "exit") != 0);
              NSLog(@"收到exit信号，本次socket通信完毕");
              // 第六步：关闭socket
              close(peerSocketId);
          }
      }
   }
}
```

**2）客户端：eg.手机客户端，模拟服务端，电脑开热点手机连接即可进行socket通信test，哈哈**

```
- (void)tcpClient {
    // 第一步：创建soket
    // TCP是基于数据流的，因此参数二使用SOCK_STREAM
    int error = -1;
    int clientSocketId = socket(AF_INET, SOCK_STREAM, 0);
    BOOL success = (clientSocketId != -1);
    struct sockaddr_in addr;
    
    // 第二步：绑定端口号
    if (success) {
        NSLog(@"绑定成功");
        // 初始化
        memset(&addr, 0, sizeof(addr));
        addr.sin_len = sizeof(addr);
        // 指定协议簇为AF_INET，比如TCP/UDP等
        addr.sin_family = AF_INET;
        // 监听任何ip地址
        addr.sin_addr.s_addr = INADDR_ANY;
        error = bind(clientSocketId, (const struct sockaddr *)&addr, sizeof(addr));
        success = (error == 0);
    }
    if (success) {
        struct sockaddr_in peerAddr;
        memset(&peerAddr, 0, sizeof(peerAddr));
        peerAddr.sin_len = sizeof(peerAddr);
        peerAddr.sin_family = AF_INET;
        peerAddr.sin_port = htons(150431);
        
        // 指定服务端的ip地址，测试时，修改成对应自己服务器的ip
        peerAddr.sin_addr.s_addr = inet_addr("10.0.143.179");
        
        socklen_t addrLen;
        addrLen = sizeof(peerAddr);
        
        // 第三步：连接服务器
        error = connect(clientSocketId, (struct sockaddr *)&peerAddr, addrLen);
        success = (error == 0);
        NSLog(@"开始连接！");
        if (success) {
            // 第四步：获取套接字信息
            error = getsockname(clientSocketId, (struct sockaddr *)&addr, &addrLen);
            success = (error == 0);
            if (success) {
                NSLog(@"连接成功, 本地地址:%s,端口:%d", inet_ntoa(addr.sin_addr), ntohs(addr.sin_port));
                // 这里只发送10次
                int count = 10;
                do {
                    // 第五步：发送消息到服务端
                    send(clientSocketId, "哈哈，server您好！", 1024, 0);
                    count--;
                    
                    // 告诉server，客户端退出了
                    if (count == 0) {
                        send(clientSocketId, "exit", 1024, 0);
                    }
                } while (count >= 1);
                
                // 第六步：关闭套接字
                close(clientSocketId);
            }
        } else {
           NSLog(@"连接失败了 ：%d",error);
           // 失败后关闭套接字
           close(clientSocketId);
        }
    }
}
```
