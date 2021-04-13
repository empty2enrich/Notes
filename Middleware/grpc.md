# grpc

## channle

### options 参数（https://github.com/grpc/grpc/blob/v1.37.x/include/grpc/impl/codegen/grpc_types.h）

* grpc.keepalive_timeout_ms： 设置 keepalive ping 的超时时间
* grpc.server_handshake_timeout_ms： 设置连接握手时间（网络连接三次握手）
* grpc.grpclb_call_timeout_ms：调用 grpclb load balancer 的超时时间（感觉像是集群环境试验的）
* grpc.grpclb_fallback_timeout_ms： 等待从 grpclb load balancer 获取服务器列表的超时时间
* grpc.dns_ares_query_timeout：

