---
title: Springboot配置Mybatis-Generator代码生成器
date: 2024-06-04 21:27:03
tags: [springboot, mybatis]
---

## 前言

公司项目用的mybatis，而不是mybatis-plus，需要自己去写xml。

太累了啊。

只能配置个代码生成器稍微减轻一下负担

## 正文

pom.xml配置mybatis-generator插件

```xml
<!-- mybatis generator 自动生成代码插件 -->
<plugins>
    <plugin>
        <groupId>org.mybatis.generator</groupId>
        <artifactId>mybatis-generator-maven-plugin</artifactId>
        <version>1.4.1</version>
        <configuration>
            <!-- 配置生成的配置文件的位置（如果此处不配置该值，则在运行命令里需要添加执行文件路径） -->
            <configurationFile>generateSqlConfig/generatorConfig.xml</configurationFile>
            <verbose>true</verbose>
            <overwrite>true</overwrite>
        </configuration>
    </plugin>
</plugins>
```

在项目根目录（跟src同级）创建配置文件`generateSqlConfig/generatorConfig.xml` 

注意，这里要配置以下信息

- 数据库驱动包路径
- 你的数据连接信息
- 生成文件路径
- 需要生成的数据库表

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 数据库驱动:选择你的本地硬盘上面的数据库驱动包-->
    <classPathEntry location="C:\mysql-connector-j-8.2.0.jar"/>
    <context id="POSTGRESQLTables" targetRuntime="MyBatis3">
        <property name="mergeable" value="false"></property>
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接URL，用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver" connectionURL="jdbc:mysql://127.0.0.1:3306/database?serverTimezone=GMT%2B8"
                        userId="your_username" password="your_password">
        </jdbcConnection>
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>
        <!-- 生成模型的包名和位置-->
        <javaModelGenerator targetPackage="com.example.model" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!-- 生成映射文件的包名和位置-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!-- 生成mapper文件的包名和位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.example.mapper" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName 是实体类名-->
        <table tableName="tb_detection_spec_classify" domainObjectName="DetectionSpecClassifyPO">
            <property name="useActualColumnNames" value="false" />
        </table>
        <table tableName="tb_detection_spec" domainObjectName="DetectionSpec">
            <property name="useActualColumnNames" value="false" />
        </table>
        <table tableName="tb_detection_option" domainObjectName="DetectionOptionPO">
            <property name="useActualColumnNames" value="false" />
        </table>
    </context>
</generatorConfiguration>
```

在命令行执行`mvn mybatis-generator:generate -e` ，或者在idea中添加运行配置，在“运行”中输入“`mybatis-generator:generate -e`”

![IDEA配置示例](/images/20240604-001.png)

## 总结

这样生成的代码虽然能用，但还是不够理想，有几个问题：

1. 像创建时间、修改时间每个表都有的字段应该要放到基类里面，让PO继承基类的，但生成器生成的代码没法指定继承
2. 没法根据表字段注释生成po属性注释
3. 生成的PO会带有getter/setter方法，但我更喜欢用lombok注解

听说这三个问题可以通过mybatis-generator的插件解决，还得再研究下