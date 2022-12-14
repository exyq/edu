# 华为1+x证书考试中拓展指令

### 链路聚合

- 将多条线路变为一条逻辑线路
- 最大线路数为8, 且需为同速率接口
```
in et1 进入Eth-Trunk1链路聚合端口中 //interface Eth-Trunk 1
mo la 设定链路聚合模式为Lacp
trun g 0/0/3 to 0/0/4 0/0/5 将g速率的3, 4, 5接口添加为成员接口
p l t 将链路聚合端口工作模式设为trunk
p t a v 10 20 将vlan10 vlan20放行
max act <接口数> 设置最大活动接口数 //max active-linknumber <接口数>
la p <优先级> 设置当前接口优先级(从少到多,从高到低) //lacp priority <优先级>
dis et 1 显示eth1链路聚合端口的状态
```

### 单臂路由(路由器子接口)

- loopback0 为测试端口又名环回端口
```
in g0/0/0.10 进入g0/0/0.10的vlan10的子接口
dot t v 10 设定dot1q为vlan10
ip add 10.1.1.254 24 设定vlan10设备的网关
arp b e 开启arp广播
```

### 生成树协议

- 生成树协议是为了断开回路, 保证网络线路正常运转
```
stp e 开启生成树协议(需在每台设备会产生回路的设备中设置)
stp mo rs 生成树模式设为快速生成树模式(须在所有开启stp e的设备中启用)
```
##### 设定优先级与设定根桥
```
stp pri 4096 设定此台设备的优先级数目为4096(优先级数目越少, 等级越高)
stp ro pri 设定此台设备为根桥
stp ro se 设定此台设备为备根
```
###### 边缘端口
- 边缘端口为无需参与生成树计算的端口, 即为链接客户机的端口
```
in g0/0/1 
stp e e 开启边缘端口模式
```
##### 查看指令
- root 根端口
- alte 阻塞端口
- desi 指定端口
```
dis stp 查看stp设定状态
```
##### 多生成树协议
- mstp(多生成树协议)用来解决stp与rstp同一缺点(浪费带宽)
- mstp兼容了stp与rstp
- mstp域内科生成多棵生成树, 每棵生成树被称为Msti, Msti彼此独立, 且每个Msti皆与rstp计算过程相同
- 每个Vlan只能对应一个Msti, 每个Msti可以对应多个Vlan
- 多生成树体现在实例(Instance)(Msti)
```
stp mo ms 设定生成树模式为多生成树 //stp mode mstp
stp reg 配置mstp域 //stp region-configuration
reg <域名> 配置mstp域名 //region-name
in <ID> v <Vid> 将Vlan引入实例中 //Instance <InstanceID> vlan <vid>
act reg 保存mstp域配置 //active region-configuration
stp in <ID> pri <优先级> 为指定的实例设定优先级 //stp instance <InstanceID> priority <优先级>
stp in <ID> root <pri/sec> 设定实例为根桥或备份根桥 //stp instance <InstanceID> roo <primary/secondary>
dis stp in <ID> b 查看特定实例的运行状态 //display stp instance <InstanceID> brief
```

### 路由设置
- 路由表 可为数据通过查看转发至目的地
- 查看命令 dis ip ro
- 直连(Direct) 目的地址为路由器连接, 无需人工干预
- 静态(Static) 路由表中没有的条目通过手动添加
	- 静态路由添加命令 ip rout <目的网段> <子网掩码> <下一跳的ip地址或接口>
	- 如需ping通需目的路由器配置本路由器
- 缺省路由添加
	- ip route 0.0.0.0 0 <下一跳>

##### ospf动态路由协议
- 动态路由自动添加只目标网段及中途的路由表信息
- ospf以设备接口为单位
- ospf区域(area) 
	- 单区域 仅有area0(骨干区域)
	- 多区域 都通过area0来中转数据包
- RouterID 在ospf网络中的每台路由器都有一个单独且不重复的routerId来区分(通常设置为最大的ip地址)
```
os 1 ro 10.1.1.1 设定ospf进程1的routerId为10.1.1.1 //ospf 1 router-id 10.1.1.1
ar 0 进入骨干区域 //area 0
ne <网段> <反掩码> 进行宣告配置 //network 3.0.0.0 0.0.0.255
re os <进程号> pro 重启ospf进程(需在用户模式下) // reset ospf 1 process
```

##### 路由引入
- 衔接不同路由协议
``` 
im <protocol> 引入<protocol(协议)> //import-route <protocol>
im static 引入静态路由
```
- 在ospf引入其他协议
```
ospf <进程号> 进入ospf进程
im static 引入静态路由 //import-route static
default-r 发布默认路由到ospf中 //default-route-advertise
```

##### OSPF认证
- 区域认证, 接口认证
- 认证方式
	- 空 不认证
	- md5 加密认证
	- simple 明文方式
	- keychain 多个密钥滚动加密
- 区域认证
``` 
au <加密方式> <密钥ID(可不填)> <cipher/simple> <Passwd> 指定加密方式进行区域认证 //authentication-mod <认证方式> <认证密钥ID(可不填)> <不显示/显示输入> <Passwd>
```
- 区域认证密钥需相同, 且需要在区域视图下配置
- 接口认证
```
ospf au <认证方式> <密钥ID(可不填)> <cipher/simple> <passwd> 基本与区域认证相同 //ospf authentication-mode <认证方式> <密钥ID(可不填)> <不显示/显示输入> <passwd>
```
- 接口认证与区域认证指令类似, 区别在于指令前方有ospf指令段且需要在接口下进行配置

### ACL访问控制列表
- ACL(访问控制列表)是数据包过滤访问控制的技术涉及网络安全部分
- ACL是一条或多条判断语句的集合
- 需要在网路通常后才可配置
- permit 允许通过
- deny 拒绝通过

| ACL类别 | 编号范围 |
| :-- | :--: |
| 基本ACL | 2000~2999 |
| 高级ACL | 3000~3999 |
| 二层ACL | 4000~4999 |
| 用户自定义ACL | 5000~5999 |
| 用户ACL | 6000~9999 |

- rule ACL规则
- rule的默认编号为5, 默认步长为5
- ACL语序
	- 紧密要求指令优先, 松散要求指令靠后
	- 在dney指令后需一条permit指令
- ACL指令编写
``` 
acl <ACLID> 设定ACL编号并进入ACL
ru <ruleID> <per/den> sou <ip> <通配符> 基本ACL语句格式 //rule <RuleID(可不填)> <permit/deny> source <核对ip> <通配符>
``` 
- 通配符与反掩码类似, 用途为核对ip, 控制ip是否通过, 转换为二进制后1的位置忽略, 0的位置检查, 若允许所有IP则为any
- 启用ACL
``` 
traffic-f <out/in> acl <ACL编号> 在接口模式下启用指定编号的ACL并设定方向 //traffic-filter <outbound/inbound> acl <ACL编号>
```
##### Easy IP
- EasyIP需使用ACL语句
- Nat 将公网与私网地址进行转换的一种技术
- EasyIP属于Nat中的一种技术
- EasyIP命令
``` 
nat <out/in> <ACL编号> 应用已经编写好的ACL并设定出入方向 //nat <outbound/inbound> <ACL编号>
```

### VRRP
- VRRP(虚拟网关技术)
- 网关(关口): 在一个网络下,需去往其他网段的一个关口
- 在两个网段之间有两个网关, 两个网段也分别有自己的网关
- 网关配置位置
	- VlanIF
	- 路由器接口与子接口下
- VRRP是为了使网络增加可靠性
	- 使用VRRP结合多个网关, 即可在其中一个网关故障后使用另外一个网关
```
v v <VRID> v <IP> 在配置网关的接口下进行配置vrid及虚拟网关 //vrrp vrid <VRID> vritual-ip <IP>
vrrp v <VRID> p <优先级> 设定指定vrrp的优先级(默认优先级为100, 数值越大,优先级越高) //vrrp vrid <VRID> priority <优先级>
```
- VRRP通过以下指令才能在特定接口出现故障后来切换另一台路由器
```
vrrp v <VRID> t in <接口(不是网关配置接口)> re <优先级> 在需检测接口出现故障后, 立即削减指定优先级, 使本路由器的VRRP优先级降低而使用备用路由器的VRRP //vrrp vrid <VRID> track in <需检测接口> reduced <优先级>
vrrp v <VRID> pre ti de <Seconds> 设定VRRP出故障后的抢占延时 //vrrp vrid <VRID> preemtp-mode timer delay <抢占秒数>
```
- 查看VRRP状态
```
dis vrrp 查看VRRP详细配置情况 //display vrrp
dis vrrp b 查看VRRP配置 //display vrrp brief
```


### DHCP
- 动态分配IP地址协议
- 交换机配置命令
```
dhcp e 开启dhcp服务 //dhcp enable
ip po <IP地址池名称> 设定IP池 //ip pool <地址池名称>
gat <网关> 设定该dhcp网关 //gatway <网关>
ne <网段> ma <子网掩码> 设定子网掩码及网段 //network <网段> mask <子网掩码>
dns <DNS> 设定DNS服务器地址 //dns-list <DNS>
exc <IP> 将特定ip排除IP池 //exculded-ip-address <IP>
le d <NUM> h <NUM> m <NUM> 设定租约时间 //lease day <天数> hour <小时数> minute <分数>
in v<VID> 进入VlanIF //interface vlanif <VID>
ip a <IP> <子网掩码> 设定网关 //ip address <IP> <子网掩码>
dhcp sel glo 设定dhcp范围为全局 //dhcp select global
```
