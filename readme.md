## MongoDB集群部署 ##

## 1.启动复制集中mongod实例，运行: ##

```
	start_rs0_0.bat		-> 4000 (PRIMARY)
	start_rs0_1.bat		-> 4001 (SECONDARY)
	start_rs0_2.bat		-> 4002 (ARBITER)
	
	start_rs1_0.bat		-> 4003 (PRIMARY)
	start_rs1_1.bat		-> 4004 (SECONDARY)
	start_rs1_2.bat		-> 4005 (ARBITER)
```

## 2.初始化配置复制集rs0 ##

```
	//初始化，配置
	> mongo --port 4000
	> rs.initiate()
	rs0:OTHER> rs.conf()
	
	//添加SECONDARY和ARBITER节点
	rs0:PRIMARY> rs.add("jock-PC:4001")
	rs0:PRIMARY> rs.addArb("jock-PC:4002")
	rs0:PRIMARY> rs.status()
```

## 3.初始化配置复制集rs1（同上） ##

```
	//初始化，配置
	> mongo --port 4003
	> rs.initiate()
	rs0:OTHER> rs.conf()
	
	//添加SECONDARY和ARBITER节点
	rs0:PRIMARY> rs.add("jock-PC:4004")
	rs0:PRIMARY> rs.addArb("jock-PC:4005")
	rs0:PRIMARY> rs.status()
```

## 4.启动3个配置服务器，运行： ##

```
	start_cfg_0.bat		-> 4006
	start_cfg_1.bat		-> 4007
	start_cfg_2.bat		-> 4008
```

## 5.启动mongos路由服务器，运行 ##

```
	start_mongos.bat	-> 4009
```

## 6.添加分片到集群，mongos分片操作会保存到配置服务器中 ##

```
	> mongo --port 4009
	mongos> sh.addShard("rs0/jock-PC:4000,jock-PC:4001")
	mongos> sh.addShard("rs1/jock-PC:4003,jock-PC:4004")
	mongos> sh.status()
```

## 部署原则： ##

部署一个MongoDB集群逻辑上需要10台机器（9个mongod进程，一个mongos进程）。为了节省机器，一些进程可以部署在同一台机器上，部署的总原则是：保证一个复制集中的所有节点(PRIMARY, SECONDARY, ARBITER)和三台配置服务器分开在不同的物理机上。
	
## 最简部署案例： ##

> A,B: 表示mongodb复制集实例  
> C: 表示配置实例  
> M: 表示物理主机  

```
	A1 -> shard1 rs0 PRIMARY   4000
	A2 -> shard1 rs0 SECONDARY 4001
	A3 -> shard1 rs0 ARBITER   4002
	
	B1 -> shard2 rs1 PRIMARY   4003
	B2 -> shard2 rs1 SECONDARY 4004
	B3 -> shard2 rs1 ARBITER   4005
	
	C1 -> cfg1 				   4006
	C2 -> cfg2 				   4007
	C3 -> cfg3 				   4008
	
	
	M1: (A1, B3, C1)
	M2: (A3, B1, C2)
	M3: (A2, B2, C3)
```
	
应用服务器和mongos部署在一起，一个应用服务器只能连接集群中的一个mongos

把A,B,C中的节点分开到不同物理主机上，避免一台主机宕机后，整个复制集失效，无法自动故障转移

> PRIMARY,SECONDARY可互相转换，要求磁盘和CPU高  
> ARBITER节点只做仲裁，不保存数据，要求磁盘和CPU低  
> cfg节点只做简单数据保存，要求磁盘低  

如果机器配置相差不多，交差部署搭配原则:

> PRIMARY优先搭配ARBITER和cfg节点  
> SECONDARY优先搭配cfg节点  
	
	
如果有4个相同配置的机器，以下布署方案能让资源得到最高利用

```
	M1: (A1, B3, C1)
	M2: (A3, B1)
	M3: (A2, C3)
	M4: (B2, C2)
```


	
	
