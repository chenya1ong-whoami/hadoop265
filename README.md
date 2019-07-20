


windows
  软件  必须从G盘加载一个文件 conf  xml
linux
	/a  /b  /c /d /e /f /g
	mount  /g  ->  disk:G分区   /b  -> disk:B分区
	软件  必须从/g 文件

好处：软件具备了 移动性

hdfs  目录树结构

角色 即  JVM进程

-----------------------------------------------
数据持久化：
	日志文件： 记录实时发生的增删改的操作  mkdir /abc   append  文本文件
		完整性比较好
		加载恢复数据：慢/占空间   <  比如说，我NN  内存4G  运行了10年  日志 是多大
						5年恢复，内存会不会溢出

	镜像、快照、dump、db、序列化
			间隔（小时，天，10分钟，1分钟，5秒钟），内存全量数据基于某一个时间点做的向磁盘的溢写
							I/O ：慢
		恢复速度快过  日志文件
		因为是间隔的，容易丢失一部分数据

HDFS：
	EditsLog：	日志
		体积小，记录少：必然有优势
	FsImage：	镜像、快照
		如果能更快的滚动更新时点

	最近时点的FsImage + 增量的EditsLog
	现在10点
	FI:9点+ 9点到10点的增量的EL
	1，加载FI
	2,加载EL
	3，内存就得到了关机前的全量数据！！！！
	问题：
		那么：  FI  时点是怎么滚动更新的！！！！！？
			由 NN  8点溢写，9点溢写。。。
			NN ：第一次开机的时候，只写一次FI ，假设8点，到9点的时候，EL  记录的是8~9的日志
				只需要将8~9的日志的记录，更新到8点的FI中，FI的数据时点就变成了9点~！
				脱了裤子放屁
				寻求另外一台机器来做



知识点：
	NN 存元数据：  文件属性 / 每个块存在哪个DN上
				属性：path  /a/b/c.txt  32G  root:root  rwxrwxrwx
				                 blk01
						 blk02
		在持久化的时候：文件属性会持久化，但是文件的每一个块不会持久化
			恢复的时候，NN会丢失块的位置信息

			分布式时代，数据一致性~！！！
			等。DN 会和 NN 建立心跳，汇报块信息~！！！！








请问：999999999  请问占用多大的磁盘空间
	文件编码：txt  byte  9个字节
	int a=9999999        4个字节  <  二进制文件














hadoop  安装：

centos 6.5
jdk 1.8
hadoop 2.6.5

1基础设施：
	设置网络：
	设置IP
		*  大家自己看看自己的vm的编辑->虚拟网络编辑器->观察 NAT模式的地址
	vi /etc/sysconfig/network-scripts/ifcfg-eth0
		DEVICE=eth0
		#HWADDR=00:0C:29:42:15:C2
		TYPE=Ethernet
		ONBOOT=yes
		NM_CONTROLLED=yes
		BOOTPROTO=static
		IPADDR=192.168.150.11
		NETMASK=255.255.255.0
		GATEWAY=192.168.150.2
		DNS1=223.5.5.5
		DNS2=114.114.114.114
	设置主机名
	vi /etc/sysconfig/network
		NETWORKING=yes
		HOSTNAME=node01
	设置本机的ip到主机名的映射关系
	vi /etc/hosts
		192.168.150.11 node01
		192.168.150.12 node02

	关闭防火墙
	service iptables stop
	chkconfig iptables off
	关闭 selinux
	vi /etc/selinux/config
		SELINUX=disabled

	做时间同步
	yum install ntp  -y
	vi /etc/ntp.conf
		server ntp1.aliyun.com
	service ntpd start
	chkconfig ntpd on

	安装JDK：
	rpm -i   jdk-8u181-linux-x64.rpm
		*有一些软件只认：/usr/java/default
	vi /etc/profile
		export  JAVA_HOME=/usr/java/default
		export PATH=$PATH:$JAVA_HOME/bin
	source /etc/profile   |  .    /etc/profile

	ssh免密：  ssh  localhost  1,验证自己还没免密  2,被动生成了  /root/.ssh
		ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
		cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
		如果A 想  免密的登陆到B：
			A：
				ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
			B：
				cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
		结论：B包含了A的公钥，A就可以免密的登陆
			你去陌生人家里得撬锁
			去女朋友家里：拿钥匙开门
2，Hadoop的配置（应用的搭建过程）
	规划路径：
	mkdir /opt/bigdata
	tar xf hadoop-2.6.5.tar.gz
	mv hadoop-2.6.5  /opt/bigdata/
	pwd
		/opt/bigdata/hadoop-2.6.5

	vi /etc/profile
		export  JAVA_HOME=/usr/java/default
		export HADOOP_HOME=/opt/bigdata/hadoop-2.6.5
		export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	source /etc/profile

	配置hadoop的角色：
	cd   $HADOOP_HOME/etc/hadoop
		必须给hadoop配置javahome要不ssh过去找不到
	vi hadoop-env.sh
		export JAVA_HOME=/usr/java/default
		给出NN角色在哪里启动
	vi core-site.xml
		    <property>
				<name>fs.defaultFS</name>
				<value>hdfs://node01:9000</value>
			</property>
		配置hdfs  副本数为1.。。。
	vi hdfs-site.xml
		    <property>
				<name>dfs.replication</name>
				<value>1</value>
			</property>
			<property>
				<name>dfs.namenode.name.dir</name>
				<value>/var/bigdata/hadoop/local/dfs/name</value>
			</property>
			<property>
				<name>dfs.datanode.data.dir</name>
				<value>/var/bigdata/hadoop/local/dfs/data</value>
			</property>
			<property>
				<name>dfs.namenode.secondary.http-address</name>
				<value>node01:50090</value>
			</property>
			<property>
				<name>dfs.namenode.checkpoint.dir</name>
				<value>/var/bigdata/hadoop/local/dfs/secondary</value>
			</property>


		配置DN这个角色再那里启动
	vi slaves
		node01

3,初始化&启动：
	hdfs namenode -format
		创建目录
		并初始化一个空的fsimage
		VERSION
			CID

	start-dfs.sh
		第一次：datanode和secondary角色会初始化创建自己的数据目录

	http://node01:50070
		修改windows： C:\Windows\System32\drivers\etc\hosts
			192.168.150.11 node01
			192.168.150.12 node02
			192.168.150.13 node03
			192.168.150.14 node04

4，简单使用：
	hdfs dfs -mkdir /bigdata
	hdfs dfs -mkdir  -p  /user/root



5,验证知识点：
	cd   /var/bigdata/hadoop/local/dfs/name/current
		观察 editlog的id是不是再fsimage的后边
	cd /var/bigdata/hadoop/local/dfs/secondary/current
		SNN 只需要从NN拷贝最后时点的FSimage和增量的Editlog


	hdfs dfs -put hadoop*.tar.gz  /user/root
	cd  /var/bigdata/hadoop/local/dfs/data/current/BP-281147636-192.168.150.11-1560691854170/current/finalized/subdir0/subdir0


	for i in `seq 100000`;do  echo "hello hadoop $i"  >>  data.txt  ;done
	hdfs dfs -D dfs.blocksize=1048576  -put  data.txt
	cd  /var/bigdata/hadoop/local/dfs/data/current/BP-281147636-192.168.150.11-1560691854170/current/finalized/subdir0/subdir0
	检查data.txt被切割的块，他们数据什么样子

-----------------------------------------------------------------------------------------

伪分布式：  在一个节点启动所有的角色： NN,DN,SNN
完全分布式：
	基础环境
	部署配置
		1）角色在哪里启动
			NN： core-site.xml:  fs.defaultFS  hdfs://node01:9000
			DN:  slaves:  node01
			SNN: hdfs-siet.xml:  dfs.namenode.secondary.http.address node01:50090
		2) 角色启动时的细节配置：
			dfs.namenode.name.dir
			dfs.datanode.data.dir
	初始化&启动
		格式化
			Fsimage
			VERSION
		start-dfs.sh
			加载我们的配置文件
			通过ssh 免密的方式去启动相应的角色

伪分布式到完全分布式：角色重新规划

	node01:
		stop-dfs.sh

	ssh 免密是为了什么 ：  启动start-dfs.sh：  在哪里启动，那台就要对别人公开自己的公钥
		这一台有什么特殊要求吗： 没有
	node02~node04:
		rpm -i jdk....
	node01:
		scp /root/.ssh/id_dsa.pub  node02:/root/.ssh/node01.pub
		scp /root/.ssh/id_dsa.pub  node03:/root/.ssh/node01.pub
		scp /root/.ssh/id_dsa.pub  node04:/root/.ssh/node01.pub
	node02:
		cd ~/.ssh
		cat node01.pub >> authorized_keys
	node03:
		cd ~/.ssh
		cat node01.pub >> authorized_keys
	node04:
		cd ~/.ssh
		cat node01.pub >> authorized_keys

配置部署：
	node01:
		cd $HADOOP/etc/hadoop
		vi core-site.xml    不需要改
		vi hdfs-site.xml
			    <property>
				<name>dfs.replication</name>
				<value>2</value>
			    </property>
			    <property>
				<name>dfs.namenode.name.dir</name>
				<value>/var/bigdata/hadoop/full/dfs/name</value>
			    </property>
			    <property>
				<name>dfs.datanode.data.dir</name>
				<value>/var/bigdata/hadoop/full/dfs/data</value>
			    </property>
			    <property>
				<name>dfs.namenode.secondary.http-address</name>
				<value>node02:50090</value>
			    </property>
			    <property>
				<name>dfs.namenode.checkpoint.dir</name>
				<value>/var/bigdata/hadoop/full/dfs/secondary</value>
			    </property>
		vi slaves
			node02
			node03
			node04

		分发：
			cd /opt
			scp -r ./bigdata/  node02:`pwd`
			scp -r ./bigdata/  node03:`pwd`
			scp -r ./bigdata/  node04:`pwd`

		格式化启动
			hdfs namenode -format
			start-dfs.sh

-----------------------------------------------------

做减法：
作业：笔记~！
发到群里
人名作为笔记的文件名
周六晚上之前


-----------------------------------------------------

FULL  ->  HA:
HA模式下：有一个问题，你的NN是2台？在某一时刻，谁是Active呢？client是只能连接Active

core-site.xml
fs.defaultFs -> hdfs://node01:9000

配置：
	core-site.xml
		<property>
		  <name>fs.defaultFS</name>
		  <value>hdfs://mycluster</value>
		</property>

		 <property>
		   <name>ha.zookeeper.quorum</name>
		   <value>node02:2181,node03:2181,node04:2181</value>
		 </property>

	hdfs-site.xml
		#以下是  一对多，逻辑到物理节点的映射
		<property>
		  <name>dfs.nameservices</name>
		  <value>mycluster</value>
		</property>
		<property>
		  <name>dfs.ha.namenodes.mycluster</name>
		  <value>nn1,nn2</value>
		</property>
		<property>
		  <name>dfs.namenode.rpc-address.mycluster.nn1</name>
		  <value>node01:8020</value>
		</property>
		<property>
		  <name>dfs.namenode.rpc-address.mycluster.nn2</name>
		  <value>node02:8020</value>
		</property>
		<property>
		  <name>dfs.namenode.http-address.mycluster.nn1</name>
		  <value>node01:50070</value>
		</property>
		<property>
		  <name>dfs.namenode.http-address.mycluster.nn2</name>
		  <value>node02:50070</value>
		</property>

		#以下是JN在哪里启动，数据存那个磁盘
		<property>
		  <name>dfs.namenode.shared.edits.dir</name>
		  <value>qjournal://node01:8485;node02:8485;node03:8485/mycluster</value>
		</property>
		<property>
		  <name>dfs.journalnode.edits.dir</name>
		  <value>/var/bigdata/hadoop/ha/dfs/jn</value>
		</property>

		#HA角色切换的代理类和实现方法，我们用的ssh免密
		<property>
		  <name>dfs.client.failover.proxy.provider.mycluster</name>
		  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
		</property>
		<property>
		  <name>dfs.ha.fencing.methods</name>
		  <value>sshfence</value>
		</property>
		<property>
		  <name>dfs.ha.fencing.ssh.private-key-files</name>
		  <value>/root/.ssh/id_dsa</value>
		</property>

		#开启自动化： 启动zkfc
		 <property>
		   <name>dfs.ha.automatic-failover.enabled</name>
		   <value>true</value>
		 </property>



流程：
	基础设施
		ssh免密：
			1）启动start-dfs.sh脚本的机器需要将公钥分发给别的节点
			2）在HA模式下，每一个NN身边会启动ZKFC，
				ZKFC会用免密的方式控制自己和其他NN节点的NN状态
	应用搭建
		HA 依赖 ZK  搭建ZK集群
		修改hadoop的配置文件，并集群同步
	初始化启动
		1）先启动JN   hadoop-daemon.sh start journalnode
		2）选择一个NN 做格式化：hdfs namenode -format   <只有第一次搭建做，以后不用做>
		3)启动这个格式化的NN ，以备另外一台同步  hadoop-daemon.sh start namenode
		4)在另外一台机器中： hdfs namenode -bootstrapStandby
		5)格式化zk：   hdfs zkfc  -formatZK     <只有第一次搭建做，以后不用做>
		6) start-dfs.sh
	使用

------实操：
	1）停止之前的集群
	2）免密：node01,node02
		node02:
			cd ~/.ssh
			ssh-keygen -t dsa -P '' -f ./id_dsa
			cat id_dsa.pub >> authorized_keys
			scp ./id_dsa.pub  node01:`pwd`/node02.pub
		node01:
			cd ~/.ssh
			cat node02.pub >> authorized_keys
	3)zookeeper 集群搭建  java语言开发  需要jdk  部署在2,3,4
		node02:
			tar xf zook....tar.gz
			mv zoo...    /opt/bigdata
			cd /opt/bigdata/zoo....
			cd conf
			cp zoo_sample.cfg  zoo.cfg
			vi zoo.cfg
				datadir=/var/bigdata/hadoop/zk
				server.1=node02:2888:3888
				server.2=node03:2888:3888
				server.3=node04:2888:3888
			mkdir /var/bigdata/hadoop/zk
			echo 1 >  /var/bigdata/hadoop/zk/myid
			vi /etc/profile
				export ZOOKEEPER_HOME=/opt/bigdata/zookeeper-3.4.6
				export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin
			. /etc/profile
			cd /opt/bigdata
			scp -r ./zookeeper-3.4.6  node03:`pwd`
			scp -r ./zookeeper-3.4.6  node04:`pwd`
		node03:
			mkdir /var/bigdata/hadoop/zk
			echo 2 >  /var/bigdata/hadoop/zk/myid
			*环境变量
			. /etc/profile
		node04:
			mkdir /var/bigdata/hadoop/zk
			echo 3 >  /var/bigdata/hadoop/zk/myid
			*环境变量
			. /etc/profile

		node02~node04:
			zkServer.sh start

	4）配置hadoop的core和hdfs
	5）分发配置
		给每一台都分发
	6）初始化：
		1）先启动JN   hadoop-daemon.sh start journalnode
		2）选择一个NN 做格式化：hdfs namenode -format   <只有第一次搭建做，以后不用做>
		3)启动这个格式化的NN ，以备另外一台同步  hadoop-daemon.sh start namenode
		4)在另外一台机器中： hdfs namenode -bootstrapStandby
		5)格式化zk：   hdfs zkfc  -formatZK     <只有第一次搭建做，以后不用做>
		6) start-dfs.sh
使用验证：
	1）去看jn的日志和目录变化：
	2）node04
		zkCli.sh
			ls /
			启动之后可以看到锁：
			get  /hadoop-ha/mycluster/ActiveStandbyElectorLock
	3）杀死namenode 杀死zkfc
		kill -9  xxx
		a)杀死active NN
		b)杀死active NN身边的zkfc
		c)shutdown activeNN 主机的网卡 ： ifconfig eth0 down
			2节点一直阻塞降级
			如果恢复1上的网卡   ifconfig eth0 up
			最终 2编程active


==================================================================================

1，hdfs 的权限
2，hdfs java api idea
3，mapreduce 启蒙

Permission	Owner	Group		Size	Replication	Block Size	Name
drwxr-xr-x	root	supergroup	0 B	0		0 B		user
-rw-r--r--	root	supergroup	8.61 KB	2		128 MB		install.log

hdfs是一个文件系统
	类unix、linux
	有用户概念

	hdfs没有相关命令和接口去创建用户
		信任客户端 <- 默认情况使用的 操作系统提供的用户
				扩展 kerberos LDAP  继承第三方用户认证系统
	有超级用户的概念
		linux系统中超级用户：root
		hdfs系统中超级用户： 是namenode进程的启动用户

	有权限概念
		hdfs的权限是自己控制的 来自于hdfs的超级用户

-----------实操：（一般在企业中不会用root做什么事情）
	面向操作系统		root是管理员  其他用户都叫【普通用户】
	面向操作系统的软件	谁启动，管理这个进程，那么这个用户叫做这个软件的管理员

	实操：
	切换我们用root搭建的HDFS  用god这个用户来启动

	node01~node04:
		*)stop-dfs.sh
		1)添加用户：root
			useradd god
			passwd god
		2）讲资源与用户绑定（a,安装部署程序；b,数据存放的目录）
			chown -R god  src
			chown -R god /opt/bigdata/hadoop-2.6.5
			chown -R god /var/bigdata/hadoop
		3）切换到god去启动  start-dfs.sh  < 需要免密
			给god做免密
			*我们是HA模式：免密的2中场景都要做的
			ssh localhost   >> 为了拿到.ssh
			node01~node02:
				cd /home/god/.ssh
				ssh-keygen -t dsa -P '' -f  ./id_dsa
			node01:
				ssh-copy-id -i id_dsa node01
				ssh-copy-id -i id_dsa node02
				ssh-copy-id -i id_dsa node03
				ssh-copy-id -i id_dsa node04
			node02
				cd /home/god/.ssh
				ssh-copy-id -i id_dsa node01
				ssh-copy-id -i id_dsa node02
		4)hdfs-site.xml
			<property>
			  <name>dfs.ha.fencing.ssh.private-key-files</name>
			  <value>/home/god/.ssh/id_dsa</value>
			</property>

			分发给node02~04
		5)god  :  start-dfs.sh

----------用户权限验证实操：
	node01:
		su god
		hdfs dfs -mkdir   /temp
		hdfs dfs -chown god:ooxx  /temp
		hdfs dfs -chmod 770 /temp
	node04:
		root:
			useradd good
			groupadd ooxx
			usermod -a -G ooxx good
			id good
		su good
			hdfs dfs -mkdir /temp/abc  <失败
			hdfs groups
				good:        <因为hdfs已经启动了，不知道你操作系统又偷偷摸摸创建了用户和组
	*node01:
		root:
			useradd good
			groupadd ooxx
			usermod -a -G ooxx good
		su god
			hdfs dfsadmin -refreshUserToGroupsMappings
	node04:
		good:
			hdfs groups
				good : good ooxx

	结论：默认hdfs依赖操作系统上的用户和组


-------------------hdfs api 实操：

windows idea eclips  叫什么：  集成开发环境  ide 你不需要做太多，用它~！！！

	语义：开发hdfs的client
	权限：1）参考系统登录用户名；2）参考环境变量；3）代码中给出；
	HADOOP_USER_NAME  god
		这一步操作优先启动idea
	jdk版本：集群和开发环境jdk版本一致~！！

	maven：构建工具
		包含了依赖管理（pom）
		jar包有仓库的概念，互联网仓库全，大
			本地仓库，用过的会缓存
		打包、测试、清除、构建项目目录。。。。
		GAV定位。。。。
		https://mvnrepository.com/

	hdfs的pom：
		hadoop：（common，hdfs，yarn，mapreduce）


===================================================================


1，最终去开发MR计算程序
	*,HDFS和YARN  是俩概念
2，hadoop2.x  出现了一个yarn ： 资源管理  》  MR  没有后台常服务
	yarn模型：container  容器，里面会运行我们的AppMaster ，map/reduce Task
		解耦
		mapreduce on yarn
	架构：
		RM
		NM
	搭建：
			   NN	NN	JN	ZKFC	ZK	DN	RM	NM
		node01	    *		*	*
		node02		*	*	*	*	 *		*
		node03			*		*	 *	 *	*
		node04					*	 *	 *	*

	hadoop	1.x		2.x			3.x
	hdfs:	no ha		ha(向前兼容，没有过多的改NN，二是通过新增了角色 zkfc)
	yarn	no yarn		yarn  （不是新增角色，二是直接在RM进程中增加了HA的模块）


-----通过官网：
	mapred-site.xml  >  mapreduce on yarn
		    <property>
			<name>mapreduce.framework.name</name>
			<value>yarn</value>
		    </property>
	yarn-site.xml
	//shuffle  洗牌  M  -shuffle>  R
		    <property>
			<name>yarn.nodemanager.aux-services</name>
			<value>mapreduce_shuffle</value>
		    </property>

		 <property>
		   <name>yarn.resourcemanager.ha.enabled</name>
		   <value>true</value>
		 </property>
		 <property>
		   <name>yarn.resourcemanager.zk-address</name>
		   <value>node02:2181,node03:2181,node04:2181</value>
		 </property>

		 <property>
		   <name>yarn.resourcemanager.cluster-id</name>
		   <value>mashibing</value>
		 </property>

		 <property>
		   <name>yarn.resourcemanager.ha.rm-ids</name>
		   <value>rm1,rm2</value>
		 </property>
		 <property>
		   <name>yarn.resourcemanager.hostname.rm1</name>
		   <value>node03</value>
		 </property>
		 <property>
		   <name>yarn.resourcemanager.hostname.rm2</name>
		   <value>node04</value>
		 </property>


	流程：
	我hdfs等所有的都用root来操作的
		node01：
			cd $HADOOP_HOME/etc/hadoop
			cp mapred-site.xml.template   mapred-site.xml
			vi mapred-site.xml
			vi yarn-site.xml
			scp mapred-site.xml yarn-site.xml    node02:`pwd`
			scp mapred-site.xml yarn-site.xml    node03:`pwd`
			scp mapred-site.xml yarn-site.xml    node04:`pwd`
			vi slaves  //可以不用管，搭建hdfs时候已经改过了。。。
			start-yarn.sh
		node03~04:
			yarn-daemon.sh start resourcemanager
		http://node03:8088
		http://node04:8088
			This is standby RM. Redirecting to the current active RM: http://node03:8088/

-------MR 官方案例使用：wc
	实战：MR ON YARN 的运行方式：
		hdfs dfs -mkdir -p   /data/wc/input
		hdfs dfs -D dfs.blocksize=1048576  -put data.txt  /data/wc/input
		cd  $HADOOP_HOME
		cd share/hadoop/mapreduce
		hadoop jar  hadoop-mapreduce-examples-2.6.5.jar   wordcount   /data/wc/input   /data/wc/output
			1)webui:
			2)cli:
		hdfs dfs -ls /data/wc/output
			-rw-r--r--   2 root supergroup          0 2019-06-22 11:37 /data/wc/output/_SUCCESS  //标志成功的文件
			-rw-r--r--   2 root supergroup     788922 2019-06-22 11:37 /data/wc/output/part-r-00000  //数据文件
				part-r-00000
				part-m-00000
					r/m :  map+reduce   r   / map  m
		hdfs dfs -cat  /data/wc/output/part-r-00000
		hdfs dfs -get  /data/wc/output/part-r-00000  ./
	抛出一个问题：
		data.txt 上传会切割成2个block 计算完，发现数据是对的~！~？后边注意听源码分析~！~~










=====================================================================
MR  提交方式
源码

提交方式：
	1，开发-> jar  -> 上传到集群中的某一个节点 -> hadoop jar  ooxx.jar  ooxx  in out
	2，嵌入【linux，windows】（非hadoop jar）的集群方式  on yarn
		集群：M、R
		client -> RM -> AppMaster
		mapreduce.framework.name -> yarn   //决定了集群运行
		conf.set("mapreduce.app-submission.cross-platform","true");
		job.setJar("C:\\Users\\Administrator\\IdeaProjects\\msbhadoop\\target\\hadoop-hdfs-1.0-0.1.jar");
			//^推送jar包到hdfs
	3，local，单机  自测
		mapreduce.framework.name -> local
		conf.set("mapreduce.app-submission.cross-platform","true"); //windows上必须配
				1，在win的系统中部署我们的hadoop：
					C:\usr\hadoop-2.6.5\hadoop-2.6.5
				2，在我给你的资料中\hadoop-install\soft\bin  文件覆盖到 你部署的bin目录下
					还要将hadoop.dll  复制到  c:\windwos\system32\
				3，设置环境变量：HADOOP_HOME  C:\usr\hadoop-2.6.5\hadoop-2.6.5

		IDE -> 集成开发：
			hadoop最好的平台是linux
			部署hadoop，bin

参数个性化：
	GenericOptionsParser parser = new GenericOptionsParser(conf, args);  //工具类帮我们把-D 等等的属性直接set到conf，会留下commandOptions
        String[] othargs = parser.getRemainingArgs();

-----------------------------------------------------------------
源码的分析：（目的）  更好的理解你学的技术的细节 以及原理
	资源层yarn
	what？why？how？
	3个环节 <-  分布式计算  <- 追求：
					计算向数据移动
					并行度、分治
					数据本地化读取
	Client
		没有计算发生
		很重要：支撑了计算向数据移动和计算的并行度
		1，Checking the input and output specifications of the job.
		2，Computing the InputSplits for the job.  // split  ->并行度和计算向数据移动就可以实现了
		3，Setup the requisite accounting information for the DistributedCache of the job, if necessary.
		4，Copying the job's jar and configuration to the map-reduce system directory on the distributed file-system.
		5，Submitting the job to the JobTracker and optionally monitoring it's status

		MR框架默认的输入格式化类： TextInputFormat < FileInputFormat < InputFormat
								getSplits()

			minSize = 1
			maxSize = Long.Max
			blockSize = file
			splitSize = Math.max(minSize, Math.min(maxSize, blockSize));  //默认split大小等于block大小
				切片split是一个窗口机制：（调大split改小，调小split改大）
					如果我想得到一个比block大的split：

			if ((blkLocations[i].getOffset() <= offset < blkLocations[i].getOffset() + blkLocations[i].getLength()))
			split：解耦 存储层和计算层
				1，file
				2，offset
				3，length
				4，hosts    //支撑的计算向数据移动

	MapTask
		input ->  map  -> output
		input:(split+format)  通用的知识，未来的spark底层也是
			来自于我们的输入格式化类给我们实际返回的记录读取器对象
				TextInputFormat->LineRecordreader
							split: file , offset , length
							init():
								in = fs.open(file).seek(offset)
								除了第一个切片对应的map，之后的map都在init环节，
								从切片包含的数据中，让出第一行，并把切片的起始更新为切片的第二行。
								换言之，前一个map会多读取一行，来弥补hdfs把数据切割的问题~！
							nextKeyValue():
								1，读取数据中的一条记录对key，value赋值
								2，返回布尔值
							getCurrentKey():
							getCurrentValue():




		output：
			NewOutputCollector
				partitioner
				collector
					MapOutputBuffer:
						*：
							map输出的KV会序列化成字节数组，算出P，最中是3元组：K,V,P
							buffer是使用的环形缓冲区：
								1，本质还是线性字节数组
								2，赤道，两端方向放KV,索引
								3，索引：是固定宽度：16B：4个int
									a)P
									b)KS
									c)VS
									d)VL
								5,如果数据填充到阈值：80%，启动线程：
									快速排序80%数据，同时map输出的线程向剩余的空间写
									快速排序的过程：是比较key排序，但是移动的是索引
								6，最终，溢写时只要按照排序的索引，卸下的文件中的数据就是有序的
									注意：排序是二次排序（索引里有P，排序先比较索引的P决定顺序，然后在比较相同P中的Key的顺序）
										分区有序  ： 最后reduce拉取是按照分区的
										分区内key有序： 因为reduce计算是按分组计算，分组的语义（相同的key排在了一起）
								7，调优：combiner
									1，其实就是一个map里的reduce
										按组统计
									2，发生在哪个时间点：
										a)内存溢写数据之前排序之后
											溢写的io变少~！
										b)最终map输出结束，过程中，buffer溢写出多个小文件（内部有序）
											minSpillsForCombine = 3
											map最终会把溢写出来的小文件合并成一个大文件：
												避免小文件的碎片化对未来reduce拉取数据造成的随机读写
											也会触发combine
									3，combine注意
										必须幂等
										例子：
											1，求和计算
											1，平均数计算
												80：数值和，个数和
						init():
							spillper = 0.8
							sortmb = 100M
							sorter = QuickSort
							comparator = job.getOutputKeyComparator();
										1，优先取用户覆盖的自定义排序比较器
										2，保底，取key这个类型自身的比较器
							combiner ？reduce
								minSpillsForCombine = 3

							SpillThread
								sortAndSpill()
									if (combinerRunner == null)


	ReduceTask
		input ->  reduce  -> output
		map:run:	while (context.nextKeyValue())
					一条记录调用一次map
		reduce:run:	while (context.nextKey())
					一组数据调用一次reduce

		doc：
			1，shuffle：  洗牌（相同的key被拉取到一个分区），拉取数据
			2，sort：  整个MR框架中只有map端是无序到有序的过程，用的是快速排序
					reduce这里的所谓的sort其实
					你可以想成就是一个对着map排好序的一堆小文件做归并排序
				grouping comparator
				1970-1-22 33	bj
				1970-1-8  23	sh
					排序比较啥：年，月，温度，，且温度倒序
					分组比较器：年，月
			3，reduce：

		run：
			rIter = shuffle。。//reduce拉取回属于自己的数据，并包装成迭代器~！真@迭代器
				file(磁盘上)-> open -> readline -> hasNext() next()
				时时刻刻想：我们做的是大数据计算，数据可能撑爆内存~！
			comparator = job.getOutputValueGroupingComparator();
					1，取用户设置的分组比较器
					2，取getOutputKeyComparator();
						1，优先取用户覆盖的自定义排序比较器
						2，保底，取key这个类型自身的比较器
					#：分组比较器可不可以复用排序比较器
						什么叫做排序比较器：返回值：-1,0,1
						什么叫做分组比较器：返回值：布尔值，false/true
						排序比较器可不可以做分组比较器：可以的

					mapTask				reduceTask
									1，取用户自定义的分组比较器
					1，用户定义的排序比较器		2，用户定义的排序比较器
					2，取key自身的排序比较器	3，取key自身的排序比较器
					组合方式：
						1）不设置排序和分组比较器：
							map：取key自身的排序比较器
							reduce：取key自身的排序比较器
						2）设置了排序
							map：用户定义的排序比较器
							reduce：用户定义的排序比较器
						3）设置了分组
							map：取key自身的排序比较器
							reduce：取用户自定义的分组比较器
						4）设置了排序和分组
							map：用户定义的排序比较器
							reduce：取用户自定义的分组比较器
					做减法：结论，框架很灵活，给了我们各种加工数据排序和分组的方式

			ReduceContextImpl
				input = rIter  真@迭代器
				hasMore = true
				nextKeyIsSame = false
				iterable = ValueIterable
				iterator = ValueIterator

				ValueIterable
					iterator()
						return iterator;
				ValueIterator	假@迭代器  嵌套迭代器
					hasNext()
						return firstValue || nextKeyIsSame;
					next()
						nextKeyValue();

				nextKey()
					nextKeyValue()

				nextKeyValue()
					1，通过input取数据，对key和value赋值
					2，返回布尔值
					3，多取一条记录判断更新nextKeyIsSame
						窥探下一条记录是不是还是一组的！

				getCurrentKey()
					return key

				getValues()
					return iterable;

			**：
				reduceTask拉取回的数据被包装成一个迭代器
				reduce方法被调用的时候，并没有把一组数据真的加载到内存
					而是传递一个迭代器-values
					在reduce方法中使用这个迭代器的时候：
						hasNext方法判断nextKeyIsSame：下一条是不是还是一组
						next方法：负责调取nextKeyValue方法，从reduceTask级别的迭代器中取记录，
							并同时更新nextKeyIsSame
				以上的设计艺术：
					充分利用了迭代器模式：
						规避了内存数据OOM的问题
						且：之前不是说了框架是排序的
							所以真假迭代器他们只需要协作，一次I/O就可以线性处理完每一组数据~！
























========

普通方式讲知识点。。。
费曼学习法。。。

========





































rwx
001  1
010  2
100  4

rwx
111
rw-
110
rw- --- ---
110 000 000   600



































