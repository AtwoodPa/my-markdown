# Disconf
Distributed Configuration Management Platform(分布式配置管理平台)
专注于各种「分布式系统配置管理」的「通用组件」和「通用平台」, 提供统一的「配置管理服务」

功能
1、托管配置
通过简单的注解类方式 托管配置。托管后，本地不需要此配置文件，统一从配置中心服务获取。
当配置被更新后，注解类的数据自动同步。
```java
@Service
@DisconfFile(filename = "redis.properties")
public class JedisConfig {

    // 代表连接地址
    private String host;

    // 代表连接port
    private int port;

    /**
     * 地址, 分布式文件配置
     *
     * @return
     */
    @DisconfFileItem(name = "redis.host", associateField = "host")
    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    /**
     * 端口, 分布式文件配置
     *
     * @return
     */
    @DisconfFileItem(name = "redis.port", associateField = "port")
    public int getPort() {
        return port;
    }

    public void setPort(int port) {
        this.port = port;
    }
}
```
2、配置更新回调
不仅注解类自动同步，并且其它类也需要做些变化
在回调类上加注解：
```java
@DisconfUpdateService(classes = { JedisConfig.class })
或者
@DisconfUpdateService(confFileKeys = { "redis.properties" })
```
3、支持基于XML的配置文件托管
除了支持基于注解式的配置文件，我们还支持 基于XML无代码侵入式的：
```xml
application.properties
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <context:component-scan base-package="org.pp"/>

    <aop:aspectj-autoproxy proxy-target-class="true"/>

    <!-- 使用disconf必须添加以下配置 -->
    <bean id="disconfMgrBean" class="com.baidu.disconf.client.DisconfMgrBean"
          destroy-method="destroy">
        <property name="scanPackage" value="org.pp.disconf.demo"/>
    </bean>
    <bean id="disconfMgrBean2" class="com.baidu.disconf.client.DisconfMgrBeanSecond"
          init-method="init" destroy-method="destroy">
    </bean>

    <!-- 使用托管方式的disconf配置(无代码侵入, 配置更改会自动reload)-->
    <bean id="configproperties_disconf"
          class="com.baidu.disconf.client.addons.properties.ReloadablePropertiesFactoryBean">
        <property name="locations">
            <list>
                <value>classpath:/autoconfig.properties</value>
                <value>classpath:/autoconfig2.properties</value>
                <value>classpath:/myserver_slave.properties</value>
                <value>classpath:/testJson.json</value>
                <value>testXml2.xml</value>
            </list>
        </property>
    </bean>

    <bean id="propertyConfigurer"
          class="com.baidu.disconf.client.addons.properties.ReloadingPropertyPlaceholderConfigurer">
        <property name="ignoreResourceNotFound" value="true"/>
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
        <property name="propertiesArray">
            <list>
                <ref bean="configproperties_disconf"/>
            </list>
        </property>
    </bean>

    <!-- 使用托管方式的disconf配置(无代码侵入, 配置更改不会自动reload)-->
    <bean id="configproperties_no_reloadable_disconf"
          class="com.baidu.disconf.client.addons.properties.ReloadablePropertiesFactoryBean">
        <property name="locations">
            <list>
                <value>myserver.properties</value>
            </list>
        </property>
    </bean>

    <bean id="propertyConfigurerForProject1"
          class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="ignoreResourceNotFound" value="true"/>
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
        <property name="propertiesArray">
            <list>
                <ref bean="configproperties_no_reloadable_disconf"/>
            </list>
        </property>
    </bean>

    <bean id="autoService" class="org.pp.disconf.demo.service.AutoService">
        <property name="auto" value="${auto=100}"/>
    </bean>

    <bean id="autoService2" class="org.pp.disconf.demo.service.AutoService2">
        <property name="auto2" value="${auto2}"/>
    </bean>

</beans>
```
4、支持配置项
变量支持分布式配置
```java
@DisconfItem(key = key)
public Double getMoneyInvest() {
    return moneyInvest;
}
```

5、支持静态变量
```java
@DisconfFile(filename = "static.properties")
public class StaticConfig {

    protected static final Logger LOGGER = LoggerFactory.getLogger(StaticConfig.class);

    private static int staticVar;

    @DisconfFileItem(name = "staticVar", associateField = "staticVar")
    public static int getStaticVar() {
        return staticVar;
    }

    public static void setStaticVar(int staticVar) {
        StaticConfig.staticVar = staticVar;
        LOGGER.info("i' m here: setting static class variable");
    }

}
```

6、过滤要进行托管的配置
- 不想托管全部的配置文件，有一些配置使用本地的，可以通过ignore忽略

# 忽略哪些分布式配置，用逗号分隔

```xml
disconf.ignore=jdbc-mysql.properties
```
7、Web配置平台控制
在Web平台上支持：
- 上传、更新 您的配置文件、配置项（有邮件通知），并且实现动态推送
- 批量下载配置文件，查看ZK上部署情况
- 查看 此配置的影响范围： 哪些机器在使用，各机器上的配置内容各是什么，并且自动校验 一致性
- 支持 自动化校验配置一致性
- 简单权限控制