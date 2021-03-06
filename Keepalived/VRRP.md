# VRRP

__VRRP协议与工作原理__
* 在现实的网络环境中，主机之间的通信都是通过配置静态路由（默认网关）完成的，而主机之间的路由器一旦出现故障，通信就会失败，因此，在这种通信模式中，路由器就成了一个单点瓶颈，为了解决这个问题，就引入了VRRP协议。

* VRRP是一种主备模式的协议，通过VRRP可以在网络发生故障时透明地进行设备切换而不影响主机间的数据通信，这其中涉及两个概念：物理路由器和虚拟路由器。

* VRRP可以将两台或多台物理路由器设备虚拟成一个虚拟路由器，这个虚拟路由器通过虚拟IP（一个或多个）对外提供服务，而在虚拟路由器内部，是多个物理路由器协同工作，同一时间只有一台物理路由器对外提供服务，这台物理路由器被称为主路由器（处于MASTER角色）。一般情况下MASTER由选举算法产生，它拥有对外服务的虚拟IP，提供各种网络功能，如ARP请求、ICMP、数据转发等。而其他物理路由器不拥有对外的虚拟IP，也不提供对外网络功能，仅仅接收MASTER的VRRP状态通告信息，这些路由器被统称为备份路由器（处于 BACKUP 角色）。当主路由器失效时，处于BACKUP角色的备份路由器将重新进行选举，产生一个新的主路由器进入MASTER角色继续提供对外服务，整个切换过程对用户来说完全透明。

* 在一个虚拟路由器中，只有处于MASTER角色的路由器会一直发送VRRP数据包，处于BACKUP角色的路由器只接收MASTER发过来的报文信息，用来监控MASTER运行状态，因此，不会发生MASTER抢占的现象，除非它的优先级更高。而当MASTER不可用时， BACKUP也就无法收到MASTER发过来的报文信息，于是就认定MASTER出现故障，接着多台BACKUP就会进行选举，优先级最高的BACKUP将成为新的MASTER，这种选举并进行角色切换的过程非常快，因而也就保证了服务的持续可用性。