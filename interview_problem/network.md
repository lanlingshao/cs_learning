#### 查看远程服务器端口是否打开
1、telnet [IP] [端口号]
如果返回Connected to [IP]，则证明端口是打开的
2、使用python的套接字模块
```import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
result = sock.connect_ex(('127.0.0.1',80))
if result == 0:
   print "Port is open"
else:
   print "Port is not open"
sock.close()```
