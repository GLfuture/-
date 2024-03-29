### ARP协议

用于获取局域网内ip地址对应的mac地址的协议

原理：向局域网内所有主机发送一个包，对应ip地址的主机回复其mac给这个主机

ARP攻击：回应每一个请求mac地址的arp协议导致局域网内所有主机的arp表混乱

### Windows

查看arp列表

```
arp -a
```

查看网络连接状态

```
netsh i i show in
```

添加静态arp（需要管理员权限）

```
netsh -c i i add neighbors 20(网卡序号) $ip $mac 
//arp -c <interface> add <ip-address> <mac-address>
```

```
arp -s 192.168.0.120 00-0c-29-85-2e-88
arp -d 192.168.0.120
# The ARP entry addition failed: Access is denied
netsh i i show in
netsh -c i i add neighbors 23 192.168.0.120 00-0c-29-85-2e-88
netsh -c "i i" delete neighbors 23
```



