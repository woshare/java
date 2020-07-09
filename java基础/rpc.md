# RPC

## RPC:Remote Procedure Call ，远程过程调用。说白了就是从本地机器去执行服务器上的一个函数
>1,http,json-rpc,gRPC,其他，都可以看成是RPC的一种

## RPC & HTTP
>1，HTTP 请求往往围绕资源，而 RPC 的请求往往围绕一个动作


```
-> {"jsonrpc": "2.0", "method": "subtract", "params": [42, 23], "id": 1}
<-- {"jsonrpc": "2.0", "result": 19, "id": 1}
```

## gRPC vs. Restful API
>1，使用http作为底层的传输协议(严格地说, gRPC使用的http2.0，而restful api则不一定)
>2，gRPC基于http 2.0。有别于HTTP/1.1在连接中的明文请求，HTTP/2与SPDY一样，将一个TCP连接分为若干个流（Stream），每个流中可以传输若干消息（Message），每个消息由若干最小的二进制帧（Frame）组成。[12]这也是HTTP/1.1与HTTP/2最大的区别所在。 HTTP/2中，每个用户的操作行为被分配了一个流编号(stream ID)，这意味着用户与服务端之间创建了一个TCP通道；协议将每个请求分割为二进制的控制帧与数据帧部分。HTTP/2引入了服务器推送。
>3，单向流，双向流