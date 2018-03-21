Created Zookeeper Cluster By Docker
Zookeeper集群的搭建
执行docker-compose up 之后创建三个zk实例
通过 echo stat | nc 127.0.0.1 2181 命令可查看ZK服务器运行状态
Zookeeper version: 3.4.11-37e277162d567b55a07d1755f0b31c32e93c01a0, built on 11/01/2017 18:06 GMT
Clients:
 /12.18.16.1:57084[0](queued=0,recved=1,sent=0)
 /18.10.80.14:37712[1](queued=0,recved=37,sent=37)

Latency min/avg/max: 0/0/20
Received: 40
Sent: 39
Connections: 2
Outstanding: 0
Zxid: 0x100000004
Mode: follower
Node count: 5

另一服务器运行dubbo-admin，配置dubbo.properties文件即可（zookeeper://ip:2181）
Prodiver和Consumer配置
Prodiver配置文件：
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans  
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://code.alibabatech.com/schema/dubbo  
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
	<dubbo:application name="provider-app" />
	<dubbo:registry protocol="zookeeper" address="zookeeper://ip:2183?backup=ip:2182,ip:2184" />
	<dubbo:protocol name="dubbo" port="20880" />
	<dubbo:service interface="com.fpx.dubbo.DemoService" ref="demoService" />       <!-- 和本地bean一样实现服务 -->
	<bean id="demoService" class="com.fpx.dubbo.DemoServiceImpl" />
</beans>

Consumer配置文件：
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://code.alibabatech.com/schema/dubbo  
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd  
        ">  
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->  
    <dubbo:application name="consumer-app" />       <!-- 使用multicast广播注册中心暴露发现服务地址 -->  
    <dubbo:registry  protocol="zookeeper" address="zookeeper://ip:2183?backup=ip:2182,ip:2184" />         <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->  
    <dubbo:reference id="demoService" interface="com.fpx.dubbo.DemoService"/>  
</beans>