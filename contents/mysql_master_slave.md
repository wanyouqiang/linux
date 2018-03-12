《MySQL 读写分离与主从复制》
==================================

MySQL的读写分离保证了读与写分别位于不同的服务器，提高了读写性能，读与写的分离。当master库中的数据有改变后，将会进行主从复制同步数据到slave库中，这样的一个机制主要是通过读取二进制日志来实现的。MyCat是一个开源分布式中间件，可以用来配置MySql进行读取分离，然而也可以用MySQL官方提供的mysql-router/mysql-proxy来实现。

### master 的配置

* 修改MySQL的my.cnf配置文件

```
	bind-address=192.168.0.168(本机IP)
	log_bin=mysql-bin
	server_id=168(IP地址最后数字)
```

* 创建允许slave访问的用户并授权主从复制(replication)

```
	create user youqiang@192.168.0.169 identified by 'youqiang';

	grant replication slave on *.* to 'youqiang'@'192.168.0.169'
```

* 记录二进制日志名称及位置信息

```
	show master status;
```

* 导出主库所有信息到sql文件中

```
	mysqldump -uroot -p --all-databases --master-data > master.sql
```

* 发送master.sql到从库所在服务器上

```
	scp master.sql youqiangwan@192.168.0.169:~
```

### slave 的配置

* 修改MySQL的my.cnf配置文件

```
	bind-address=192.168.0.169
	log_bin=mysql-bin
	server_id=169
```

* 重启MySQL并导入master.sql文件,保持数据一致

```
	sudo systemctl restart mysql

	mysql -uroot -p < master.sql
```

* slave与master建立连接，实现主从复制

```
	change master to
	master_user='youqiang',
	master_password='youqiang',
	master_log_file='mysql-bin.000223',
	master_log_pos=233;

	start slave;

```

* 查询slave状态信息

```
	show slave status \G

	Slave_IO_Running:Yes
	Slave_SQL_Running:Yes
	如果以上两条为Yes则配置成功！
```

### MyCat 的配置

* JDK的安装与配置

* MyCat schema.xml的配置

```
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

	<schema name="mycatdb" checkSQLschema="true" sqlMaxLimit="100" dataNode="dn1"/>
		<!-- auto sharding by id (long) -->
	<!-- <dataNode name="dn1$0-743" dataHost="localhost1" database="db$0-743"
		/> -->
	<dataNode name="dn1" dataHost="localhost1" database="test" />
	<!--<dataNode name="dn4" dataHost="sequoiadb1" database="SAMPLE" />
	 <dataNode name="jdbc_dn1" dataHost="jdbchost" database="db1" />
	<dataNode	name="jdbc_dn2" dataHost="jdbchost" database="db2" />
	<dataNode name="jdbc_dn3" 	dataHost="jdbchost" database="db3" /> -->
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- can have multi write hosts -->
		<writeHost host="hostM1" url="192.168.0.110:3306" user="root"
				   password="root">
			<!-- can have multi read hosts -->
		<readHost host="hostS2" url="192.168.0.109:3306" user="root" password="root" />
                </writeHost>
		<!-- <writeHost host="hostM2" url="localhost:3316" user="root" password="123456"/> -->
	</dataHost>
	<!--
		<dataHost name="sequoiadb1" maxCon="1000" minCon="1" balance="0" dbType="sequoiadb" dbDriver="jdbc">
		<heartbeat> 		</heartbeat>
		 <writeHost host="hostM1" url="sequoiadb://1426587161.dbaas.sequoialab.net:11920/SAMPLE" user="jifeng" 	password="jifeng"></writeHost>
		 </dataHost>

	  <dataHost name="oracle1" maxCon="1000" minCon="1" balance="0" writeType="0" 	dbType="oracle" dbDriver="jdbc"> <heartbeat>select 1 from dual</heartbeat>
		<connectionInitSql>alter session set nls_date_format='yyyy-mm-dd hh24:mi:ss'</connectionInitSql>
		<writeHost host="hostM1" url="jdbc:oracle:thin:@127.0.0.1:1521:nange" user="base" 	password="123456" > </writeHost> </dataHost>

		<dataHost name="jdbchost" maxCon="1000" 	minCon="1" balance="0" writeType="0" dbType="mongodb" dbDriver="jdbc">
		<heartbeat>select 	user()</heartbeat>
		<writeHost host="hostM" url="mongodb://192.168.0.99/test" user="admin" password="123456" ></writeHost> </dataHost>

		<dataHost name="sparksql" maxCon="1000" minCon="1" balance="0" dbType="spark" dbDriver="jdbc">
		<heartbeat> </heartbeat>
		 <writeHost host="hostM1" url="jdbc:hive2://feng01:10000" user="jifeng" 	password="jifeng"></writeHost> </dataHost> -->

	<!-- <dataHost name="jdbchost" maxCon="1000" minCon="10" balance="0" dbType="mysql"
		dbDriver="jdbc"> <heartbeat>select user()</heartbeat> <writeHost host="hostM1"
		url="jdbc:mysql://localhost:3306" user="root" password="123456"> </writeHost>
		</dataHost> -->
</mycat:schema>

```

* MyCat server.xml的配置

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- - - Licensed under the Apache License, Version 2.0 (the "License"); 
	- you may not use this file except in compliance with the License. - You 
	may obtain a copy of the License at - - http://www.apache.org/licenses/LICENSE-2.0 
	- - Unless required by applicable law or agreed to in writing, software - 
	distributed under the License is distributed on an "AS IS" BASIS, - WITHOUT 
	WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. - See the 
	License for the specific language governing permissions and - limitations 
	under the License. -->
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
	<system>
	<property name="nonePasswordLogin">0</property> <!-- 0为需要密码登陆、1为不需要密码登陆 ,默认为0，设置为1则需要指定默认账户-->
	<property name="useHandshakeV10">1</property>
	<property name="useSqlStat">0</property>  <!-- 1为开启实时统计、0为关闭 -->
	<property name="useGlobleTableCheck">0</property>  <!-- 1为开启全加班一致性检测、0为关闭 -->

		<property name="sequnceHandlerType">2</property>
	<property name="subqueryRelationshipCheck">false</property> <!-- 子查询中存在关联查询的情况下,检查关联字段中是否有分片字段 .默认 false -->
      <!--  <property name="useCompression">1</property>--> <!--1为开启mysql压缩协议-->
        <!--  <property name="fakeMySQLVersion">5.6.20</property>--> <!--设置模拟的MySQL版本号-->
	<!-- <property name="processorBufferChunk">40960</property> -->
	<!-- 
	<property name="processors">1</property> 
	<property name="processorExecutor">32</property> 
	 -->
        <!--默认为type 0: DirectByteBufferPool | type 1 ByteBufferArena | type 2 NettyBufferPool -->
		<property name="processorBufferPoolType">0</property>
		<!--默认是65535 64K 用于sql解析时最大文本长度 -->
		<!--<property name="maxStringLiteralLength">65535</property>-->
		<!--<property name="sequnceHandlerType">0</property>-->
		<!--<property name="backSocketNoDelay">1</property>-->
		<!--<property name="frontSocketNoDelay">1</property>-->
		<!--<property name="processorExecutor">16</property>-->
		<!--
			<property name="serverPort">8066</property> <property name="managerPort">9066</property> 
			<property name="idleTimeout">300000</property> <property name="bindIp">0.0.0.0</property> 
			<property name="frontWriteQueueSize">4096</property> <property name="processors">32</property> -->
		<!--分布式事务开关，0为不过滤分布式事务，1为过滤分布式事务（如果分布式事务内只涉及全局表，则不过滤），2为不过滤分布式事务,但是记录分布式事务日志-->
		<property name="handleDistributedTransactions">0</property>
		
			<!--
			off heap for merge/order/group/limit      1开启   0关闭
		-->
		<property name="useOffHeapForMerge">1</property>

		<!--
			单位为m
		-->
        <property name="memoryPageSize">64k</property>

		<!--
			单位为k
		-->
		<property name="spillsFileBufferSize">1k</property>

		<property name="useStreamOutput">0</property>

		<!--
			单位为m
		-->
		<property name="systemReserveMemorySize">384m</property>


		<!--是否采用zookeeper协调切换  -->
		<property name="useZKSwitch">false</property>

		<!-- XA Recovery Log日志路径 -->
		<!--<property name="XARecoveryLogBaseDir">./</property>-->

		<!-- XA Recovery Log日志名称 -->
		<!--<property name="XARecoveryLogBaseName">tmlog</property>-->

	</system>
	
	<!-- 全局SQL防火墙设置 -->
	<!--白名单可以使用通配符%或着*-->
	<!--例如<host host="127.0.0.*" user="root"/>-->
	<!--例如<host host="127.0.*" user="root"/>-->
	<!--例如<host host="127.*" user="root"/>-->
	<!--例如<host host="1*7.*" user="root"/>-->
	<!--这些配置情况下对于127.0.0.1都能以root账户登录-->
	<!--
	<firewall>
	   <whitehost>
	      <host host="1*7.0.0.*" user="root"/>
	   </whitehost>
       <blacklist check="false">
       </blacklist>
	</firewall>
	-->

	 <user name="root">
		<property name="password">123456</property>
	        <property name="schemas">mycatdb</property>
		
		<!-- 表级 DML 权限设置 -->
		<!-- 		
		<privileges check="false">
			<schema name="TESTDB" dml="0110" >
				<table name="tb01" dml="0000"></table>
				<table name="tb02" dml="1111"></table>
			</schema>
		</privileges>		
		 -->
        </user>

	<user name="youqiang">
		<property name="password">youqiang</property>
		<property name="schemas">mycatdb</property>
	</user>

</mycat:server>

```

* MyCat的启用

/usr/local/mycat/bin/mycat start
/usr/local/mycat/bin/mycat stop
/usr/local/mycat/bin/mycat console

* MyCat常用端口

管理端口(9066)
数据端口(8066)

* 客户端连接

```
	mysql -h127.0.0.1 -uyouqiang -pyouqiang -P8066 -Dmycatdb

```