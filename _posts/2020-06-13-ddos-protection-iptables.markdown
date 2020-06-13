---
layout: post
title:  "DDoS Protection With IPtables"
subtitle: "The Ultimate IPtables Guide"
date:   2020-06-13 13:33:37
categories: [tweaks]
---

There are different ways of building your own Anti-DDoS rules for IPTables. We will be discussing the most effective protection methods in this comprehensive tutorial.

While one can do a lot with IPTables, there isn’t a way around actual hardware firewalls - sometimes the right hardware is a must. However, it is not impossible to filter most of the bad traffic at line rate using IPTables!

**NOTE:** We’ll only cover protection from TCP-based attacks. Most UDP-based attacks are amplified reflection attacks that will exhaust the network interface card of any common server. The only mitigation approach that makes sense against these types of attacks is to block them at the edge,core network or even the carrier. If such attacks are able to reach your server, there isn’t much you can do against those multi-Gbit/s attacks except to move to a DDoS protected network. 


**The Best Linux Kernel Settings to Mitigate DDoS**

A common mistake is that people do not use optimized kernel settings to better mitigate the effects of such attacks.
Note that this guide focuses on CentOS 7 & Debian 9/10 as the operating systems of choice. CentOS 7 & Debian 9/10 include a recent version of IPTables and support of the new SYNPROXY target.

We won’t cover every single kernel setting that you need to adjust in order to better mitigate DDoS with IPTables.

Instead, we provide a set of CentOS 7 (they work for Debian 9 and 10 too) kernel settings that we would use. Just put the below in your ``/etc/sysctl.conf`` file and apply the settings with ``sysctl -p``.

```conf
kernel.printk = 4 4 1 7 
kernel.panic = 10 
kernel.sysrq = 0 
kernel.shmmax = 4294967296 
kernel.shmall = 4194304 
kernel.core_uses_pid = 1 
kernel.msgmnb = 65536 
kernel.msgmax = 65536 
vm.swappiness = 20 
vm.dirty_ratio = 80 
vm.dirty_background_ratio = 5 
fs.file-max = 2097152 
net.core.netdev_max_backlog = 262144 
net.core.rmem_default = 31457280 
net.core.rmem_max = 67108864 
net.core.wmem_default = 31457280 
net.core.wmem_max = 67108864 
net.core.somaxconn = 65535 
net.core.optmem_max = 25165824 
net.ipv4.neigh.default.gc_thresh1 = 4096 
net.ipv4.neigh.default.gc_thresh2 = 8192 
net.ipv4.neigh.default.gc_thresh3 = 16384 
net.ipv4.neigh.default.gc_interval = 5 
net.ipv4.neigh.default.gc_stale_time = 120 
net.netfilter.nf_conntrack_max = 10000000 
net.netfilter.nf_conntrack_tcp_loose = 0 
net.netfilter.nf_conntrack_tcp_timeout_established = 1800 
net.netfilter.nf_conntrack_tcp_timeout_close = 10 
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 10 
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 20 
net.netfilter.nf_conntrack_tcp_timeout_last_ack = 20 
net.netfilter.nf_conntrack_tcp_timeout_syn_recv = 20 
net.netfilter.nf_conntrack_tcp_timeout_syn_sent = 20 
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 10 
net.ipv4.tcp_slow_start_after_idle = 0 
net.ipv4.ip_local_port_range = 1024 65000 
net.ipv4.ip_no_pmtu_disc = 1 
net.ipv4.route.flush = 1 
net.ipv4.route.max_size = 8048576 
net.ipv4.icmp_echo_ignore_broadcasts = 1 
net.ipv4.icmp_ignore_bogus_error_responses = 1 
net.ipv4.tcp_congestion_control = htcp 
net.ipv4.tcp_mem = 65536 131072 262144 
net.ipv4.udp_mem = 65536 131072 262144 
net.ipv4.tcp_rmem = 4096 87380 33554432 
net.ipv4.udp_rmem_min = 16384 
net.ipv4.tcp_wmem = 4096 87380 33554432 
net.ipv4.udp_wmem_min = 16384 
net.ipv4.tcp_max_tw_buckets = 1440000 
net.ipv4.tcp_tw_recycle = 0 
net.ipv4.tcp_tw_reuse = 1 
net.ipv4.tcp_max_orphans = 400000 
net.ipv4.tcp_window_scaling = 1 
net.ipv4.tcp_rfc1337 = 1 
net.ipv4.tcp_syncookies = 1 
net.ipv4.tcp_synack_retries = 1 
net.ipv4.tcp_syn_retries = 2 
net.ipv4.tcp_max_syn_backlog = 16384 
net.ipv4.tcp_timestamps = 1 
net.ipv4.tcp_sack = 1 
net.ipv4.tcp_fack = 1 
net.ipv4.tcp_ecn = 2 
net.ipv4.tcp_fin_timeout = 10 
net.ipv4.tcp_keepalive_time = 600 
net.ipv4.tcp_keepalive_intvl = 60 
net.ipv4.tcp_keepalive_probes = 10 
net.ipv4.tcp_no_metrics_save = 1 
net.ipv4.ip_forward = 0 
net.ipv4.conf.all.accept_redirects = 0 
net.ipv4.conf.all.send_redirects = 0 
net.ipv4.conf.all.accept_source_route = 0 
net.ipv4.conf.all.rp_filter = 1
```
These kernel tweaks help to maximize the performance of your server under DDoS and should somewhat increase the effectiveness of the IPTables rules that we’re going to provide in this guide.


**The Actual IPTables Anti-DDoS Rules**

DDoS attacks are complex. There are many different types of them and it’s close to impossible to maintain signature-based rules against all. But luckily there is something called connection tracking (nf_conntrack kernel module), which can help us to mitigate almost any TCP-based DDoS attack that doesn’t use SYN packets that seem legitimate - this includes all types of ACK, SYN-ACK and bogus TCP flag DDoS attacks. Anyways, here are the rules.

**1) Drop invalid packets**

This rule blocks all packets that are not a SYN packet and don’t belong to an established TCP connection.
```bash
iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP
```

**2) Drop TCP packets that are new and are not SYN**

This blocks all packets that are new (don’t belong to an established connection) and don’t use the SYN flag. This rule is similar to the "Block Invalid Packets" one, but we found that it catches some packets that the other one doesn’t.
```bash
iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP 
```

**3) Drop SYN packets with suspicious MSS value**

The followings rule blocks new packets (only SYN packets can be new packets as per the two previous rules) that use a TCP MSS value that is not common. This helps to block dumb SYN floods.
```bash
iptables -t mangle -A PREROUTING -p tcp -m conntrack --ctstate NEW -m tcpmss ! --mss 536:65535 -j DROP  
```

**4) Block packets with bogus TCP flags**

The ruleset below blocks packets that use bogus TCP flags, ie. TCP flags that legitimate packets wouldn’t use.
```bash
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN FIN,SYN -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags SYN,RST SYN,RST -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,RST FIN,RST -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,ACK FIN -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,URG URG -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,FIN FIN -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,PSH PSH -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL ALL -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL NONE -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL FIN,PSH,URG -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,FIN,PSH,URG -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -j DROP   
```

**5) Drop packets from private subnets (Spoofing)**

These rules block spoofed packets originating from private (local) subnets. On your public network interface you usually don’t want to receive packets from private source IPs.These rules assume that your loopback interface uses the 127.0.0.0/8 IP space.
These five sets of rules alone already block many TCP-based DDoS attacks at very high packet rates. With the kernel settings and rules mentioned above, you’ll be able to filter ACK and SYN-ACK attacks at line rate.
```bash
iptables -t mangle -A PREROUTING -s 224.0.0.0/3 -j DROP 
iptables -t mangle -A PREROUTING -s 169.254.0.0/16 -j DROP 
iptables -t mangle -A PREROUTING -s 172.16.0.0/12 -j DROP 
iptables -t mangle -A PREROUTING -s 192.0.2.0/24 -j DROP 
iptables -t mangle -A PREROUTING -s 192.168.0.0/16 -j DROP 
iptables -t mangle -A PREROUTING -s 10.0.0.0/8 -j DROP 
iptables -t mangle -A PREROUTING -s 0.0.0.0/8 -j DROP 
iptables -t mangle -A PREROUTING -s 240.0.0.0/5 -j DROP 
iptables -t mangle -A PREROUTING -s 127.0.0.0/8 ! -i lo -j DROP  
```

**6)  Drop ICMP (you usually don't need this protocol)**

This rule drops all ICMP packets. ICMP is only used to ping a host to find out if it’s still alive. Because it’s usually not needed and only represents another vulnerability that attackers can exploit, we block all ICMP packets to mitigate Ping of Death (ping flood), ICMP flood and ICMP fragmentation flood.
```bash
iptables -t mangle -A PREROUTING -p icmp -j DROP  
```

**7) Drop fragments in all chains**

This rule blocks fragmented packets. Normally you don’t need those and blocking fragments will mitigate UDP fragmentation flood. But most of the time UDP fragmentation floods use a high amount of bandwidth that is likely to exhaust the capacity of your network card, which makes this rule optional and probably not the most useful one.
```bash
iptables -t mangle -A PREROUTING -f -j DROP
```

**8) Limit connections per source IP**

This rule blocks simply limits the number of connections a single IP can have with your server.
```bash
iptables -A INPUT -p tcp -m connlimit --connlimit-above 111 -j REJECT --reject-with tcp-reset  
```

**9) Limit connections per source IP**

This limits incoming TCP RST packets to mitigate TCP RST floods. Effectiveness of this rule is questionable.
```bash
iptables -A INPUT -p tcp --tcp-flags RST RST -m limit --limit 2/s --limit-burst 2 -j ACCEPT 
iptables -A INPUT -p tcp --tcp-flags RST RST -j DROP
```

**10) Limit new TCP connections per second per source IP**

Limits the new TCP connections that a client can establish per second. This can be useful against connection attacks, but not so much against SYN floods because the usually use an endless amount of different spoofed source IPs.
```bash
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m limit --limit 60/s --limit-burst 20 -j ACCEPT 
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -j DROP  
```

**11) Protection against port scanning**

Stops snoops from checking which ports are running and what services are running on your server.
```bash
iptables -N port-scanning 
iptables -A port-scanning -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s --limit-burst 2 -j RETURN 
iptables -A port-scanning -j DROP 
```

```bash
### 1: Drop invalid packets ### 
iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP  

### 2: Drop TCP packets that are new and are not SYN ### 
iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP 
 
### 3: Drop SYN packets with suspicious MSS value ### 
iptables -t mangle -A PREROUTING -p tcp -m conntrack --ctstate NEW -m tcpmss ! --mss 536:65535 -j DROP  

### 4: Block packets with bogus TCP flags ### 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,SYN FIN,SYN -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags SYN,RST SYN,RST -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,RST FIN,RST -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags FIN,ACK FIN -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,URG URG -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,FIN FIN -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ACK,PSH PSH -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL ALL -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL NONE -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL FIN,PSH,URG -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,FIN,PSH,URG -j DROP 
iptables -t mangle -A PREROUTING -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -j DROP  

### 5: Block spoofed packets ### 
iptables -t mangle -A PREROUTING -s 224.0.0.0/3 -j DROP 
iptables -t mangle -A PREROUTING -s 169.254.0.0/16 -j DROP 
iptables -t mangle -A PREROUTING -s 172.16.0.0/12 -j DROP 
iptables -t mangle -A PREROUTING -s 192.0.2.0/24 -j DROP 
iptables -t mangle -A PREROUTING -s 192.168.0.0/16 -j DROP 
iptables -t mangle -A PREROUTING -s 10.0.0.0/8 -j DROP 
iptables -t mangle -A PREROUTING -s 0.0.0.0/8 -j DROP 
iptables -t mangle -A PREROUTING -s 240.0.0.0/5 -j DROP 
iptables -t mangle -A PREROUTING -s 127.0.0.0/8 ! -i lo -j DROP  

### 6: Drop ICMP (you usually don't need this protocol) ### 
iptables -t mangle -A PREROUTING -p icmp -j DROP  

### 7: Drop fragments in all chains ### 
iptables -t mangle -A PREROUTING -f -j DROP  

### 8: Limit connections per source IP ### 
iptables -A INPUT -p tcp -m connlimit --connlimit-above 111 -j REJECT --reject-with tcp-reset  

### 9: Limit RST packets ### 
iptables -A INPUT -p tcp --tcp-flags RST RST -m limit --limit 2/s --limit-burst 2 -j ACCEPT 
iptables -A INPUT -p tcp --tcp-flags RST RST -j DROP  

### 10: Limit new TCP connections per second per source IP ### 
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m limit --limit 60/s --limit-burst 20 -j ACCEPT 
iptables -A INPUT -p tcp -m conntrack --ctstate NEW -j DROP  

### 11: Protection against port scanning ### 
iptables -N port-scanning 
iptables -A port-scanning -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s --limit-burst 2 -j RETURN 
iptables -A port-scanning -j DROP
```

[CalifaxYT Discord](https://discord.gg/WufbYbv)

---

CalifaxYT - <https://twitter.com/ytcalifax>
