### Nginx负载均衡策略

| 策略               | 方式            |
| ------------------ | --------------- |
| 轮询               | 默认方式        |
| weight             | 权重方式        |
| ip_hash            | 依据ip分配方式  |
| least_conn         | 最少连接方式    |
| fair（第三方）     | 响应时间方式    |
| url_hash（第三方） | 依据URL分配方式 |

[Nginx服务器之负载均衡策略（6种）](https://www.cnblogs.com/1214804270hacker/p/9325150.html)

### nginx如何获取真实IP

在直接对外的Nginx反向代理服务器上配置：

`proxy_set_header X-Forwarded-For $remote_addr;`

如果有多层代理，那么在内层nginx还要配置

`proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`

$remote_addr是获取的是直接TCP连接的客户端IP（类似于Java中的request.getRemoteAddr()），这个是无法伪造的，即使客户端伪造也会被覆盖掉，而不是追加。


[http获取客户端真实ip的原理及利用X-Forwarded-For伪造客户端IP漏洞成因及防范](https://www.cnblogs.com/lovearpu/p/11187215.html)