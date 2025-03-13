## **Netfilter**

Netfilter 是 Linux 内核中的一个框架，用于实现数据包过滤、网络地址转换（NAT）和数据包修改等功能。它是防火墙（如 `iptables`）和 NAT 的基础。

### 主要功能：

- **数据包过滤**：根据规则允许或拒绝数据包。
    
- **网络地址转换（NAT）**：修改数据包的源或目标 IP 地址。
    
- **数据包修改**：修改数据包的内容（如 TTL、TOS 等）。
    

### Netfilter 的钩子点（Hook Points）：

Netfilter 在内核协议栈的关键位置设置了钩子点，允许用户注册回调函数来处理数据包：

1. [[NF_IP_PRE_ROUTING]]：数据包进入协议栈后，路由决策之前。
    
2. [[NF_IP_LOCAL_IN]]：数据包目标是本机。
    
3. [[NF_IP_FORWARD]]：数据包需要转发到其他主机。
    
4. [[NF_IP_LOCAL_OUT]]：数据包从本机发出。
    
5. [[NF_IP_POST_ROUTING]]：数据包发送到网卡之前。
    

### 示例：使用 `iptables` 配置防火墙规则

```
# 允许来自 192.168.1.0/24 的 SSH 连接
iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCEPT

# 拒绝所有其他流量
iptables -A INPUT -j DROP
```

