---
title: lvs
tags:
author:
---



## ipvsadm

查看vip的监听地址

```
$ ipvsadm -ln | grep 10.139.180.179
TCP  10.139.180.179:80                               0          wrr      N/A    
```

查看某个监听的详细信息，其中Weight列为0，说明健康检查失败。

```
$ ipvsadm -lnt 10.139.180.179:80
Prot LocalAddress:Port                               VID        Scheduler Established(Sec.) bandwidth(Bps) flags
  -> RemoteAddress:Port                              VID        Forward Weight ActiveConn InActConn Vxlan-Addr Status
TCP  10.139.180.179:80                               0          wrr      N/A         
  -> 10.139.48.5:80                                  0          Masq    100    0          0          N/A                                     U  
  -> 10.139.50.246:80                                0          Masq    100    0          0          N/A                                     U  
  -> 10.139.50.248:80                                0          Masq    100    0          0          N/A                                     U  
```

