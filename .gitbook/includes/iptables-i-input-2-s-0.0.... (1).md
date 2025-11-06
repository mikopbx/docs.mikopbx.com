---
title: iptables -I INPUT 2 -s 0.0....
---

```
iptables -I INPUT 2 -s 0.0.0.0/0 -p udp -m multiport --dports 51820 -j ACCEPT
```
