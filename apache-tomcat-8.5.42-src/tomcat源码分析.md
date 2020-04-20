# Tocmat底层原理解析

## 一. tomcat源码编译过程

1. 去官网下载tomcat官网源码8.5.42

   [https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.42/src/apache-tomcat-8.5.42-src.zip](tomcat官网)

2. 打开idea创建一个空项目，并且把下载好的tomcat源码解压到新创建的空项目的目录下。

   ![](https://raw.githubusercontent.com/shihaoyan/myimages/master/1.png)

3. 进入到源码目录，创建home文件夹，并且把源码目录下的conf文件夹和webapps文件夹移动到新创建的home文件夹下

   ![](https://raw.githubusercontent.com/shihaoyan/myimages/master/2.png)

4. 因为需要把这个项目转化成maven项目，所以需要在源码目录下创建pom.xml文件，内容如下

   ```
   <?xml version="1.0" encoding="utf-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <groupId>org.apache.tomcat</groupId>
       <artifactId>apache-tomcat-8.5.42-src</artifactId>
       <name>Tomcat8.5</name>
       <version>8.5</version>
       <build>
           <finalName>Tomcat8.5</finalName>
           <sourceDirectory>java</sourceDirectory>
           <resources>
               <resource>
                   <directory>java</directory>
               </resource>
           </resources>
           <plugins>
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-compiler-plugin</artifactId>
                   <version>2.3</version>
                   <configuration>
                       <encoding>UTF-8</encoding>
                       <source>1.8</source>
                       <target>1.8</target>
                   </configuration>
               </plugin>
           </plugins>
       </build>
       <dependencies>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
               <scope>test</scope>
           </dependency>
           <dependency>
               <groupId>ant</groupId>
               <artifactId>ant</artifactId>
               <version>1.7.0</version>
           </dependency>
           <dependency>
               <groupId>wsdl4j</groupId>
               <artifactId>wsdl4j</artifactId>
               <version>1.6.2</version>
           </dependency>
           <dependency>
               <groupId>javax.xml</groupId>
               <artifactId>jaxrpc</artifactId>
               <version>1.1</version>
           </dependency>
           <dependency>
               <groupId>org.eclipse.jdt.core.compiler</groupId>
               <artifactId>ecj</artifactId>
               <version>4.5.1</version>
           </dependency>
           <dependency>
               <groupId>org.easymock</groupId>
               <artifactId>easymock</artifactId>
               <version>3.4</version>
           </dependency>
       </dependencies>
   </project>
   ```

5. 在idea中打开项目，并且导入maven项目（maven一定要设置成如果仓库中没有jar去外网下载）。

6. 添加启动类Main类，是在org.apache.catalina.startup.Bootstrap#main这个类中

7. 设置启动参数添加一个启动配置 Application，命名为BootStrap，选择主启动类，设置VM参数。

   ![](https://raw.githubusercontent.com/shihaoyan/myimages/master/3.png)

   启动参数

   -Dcatalina.home=D:/Java/IdeaProject/tomcat-8.5.42/apache-tomcat-8.5.42-src/home
   -Dcatalina.base=D:/Java/IdeaProject/tomcat-8.5.42/apache-tomcat-8.5.42-src/home
   -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
   -Djava.util.logging.config.fil=D:/Java/IdeaProject/tomcat-8.5.42/apache-tomcat-8.5.42-src/home/conf/logging.properties

8. 启动tomcat（这个时候可能会出现异常信息，我的做法是把webapps下面的examples全部删除就好使了），通过浏览器访问tocmat

9. 访问会出错。

   ![](https://raw.githubusercontent.com/shihaoyan/myimages/master/4.png)

10. 出现的原因是因为jsp解析器只能手动启动，所以我们需要自己去启动jsp解析器

    （当编译tomcat7的时候不会出现这个问题，也就不需要添加这行代码）

    org.apache.catalina.startup.ContextConfig这个类的configureStart 方法下面webConfig();下添加一行代码

    ```java
    context.addServletContainerInitializer(new JasperInitializer(),null);
    ```

    ![](https://raw.githubusercontent.com/shihaoyan/myimages/master/5.png)

    

    11. 重新运行并访问，启动成功。(当编译tomcat7的时候会出现问题，需要注释掉一下的代码)

        ![](images\6.png)

    org.apache.catalina.tribes.tipis.AbstractReplicatedMap#keySet
    
    ------
    
    

## 二. web应用部署

### 2.1 部署的三种方式

1. 直接把war包扔到webapps下（但是要设置tomcat自动解压war包,Host主机标签中）

   unpackWARs="true"

    如果想要context为空，即以http://localhost:8080/ 形式访问，只要将WAR包重命名为ROOT.WAR或者将文件夹重命名为ROOT

   webapps其实是Host节点appBase属性的值，相对路径是相对于$CATALINA_BASE的，即$CATALINA_BASE/webapps，也可以配置为其他的值，或者一个绝对路径，这样那个目录下的WAR包或者文件夹都会在Tomcat启动时被自动发布

2. 可以直接在conf\Catalina\localhost目录下配置为一个xml文件，默认文件名就是项目名

   ```xml
   <Context path="/项目名" docBase="项目的根目录"></Context>
   ```

3. 可以在host节点下配置context（当项目启动的时候，解析server.xml文件的时候就已经启动应用了）

   `<Context path="/test" docBase="test"**/>**`其中path就是Context，如果要配置根目录，只有设置path=””；docBase就是文件夹名称或者是WAR包名，如果是相对路径，则是相对于它所在Host节点的appBase

   不要将docBase指向webapps下的某个WAR或者文件夹，这样可能会导致应用被多次发布;这种方式发布应用，需要重启Tomcat才能生效

   这种方式其实是在$CATALINA_BASE/conf/{enginename}/{hostname}，默认是$CATALINA_BASE/conf/Catalina/localhost下面添加一个{context}.xml，这样就使用这个xml的文件名作为项目path：http://127.0.0.1:8080/{context}/访问，如果{context}有多层，则用#间隔，例如a#b#c.xml，就用http://127.0.0.1:8080/a/b/c/访问

   path不用指定，同样也不能把war包放在{hostname}的{appBase}下，不用重启Tomcat部署就能生效

### 2.2 tomcat其他部署

1. 如果想只启动一个tomcat,使用不同端口提供服务，只要增加Service节点并相应改动相关值

   <Service name="Catalina1">
   　　<Connector connectionTimeout="20000" port="8082" protocol="HTTP/1.1" 				redirectPort="8443"/>
   　　<Engine defaultHost="localhost" name="Catalina1">
   　　　　<Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
   　　　　　　<Context path="/test" docBase="test"/>
   　　　　</Host>
   　　</Engine>
   </Service>

2. 如果一个站点配置多个应用，可以同过增加host虚拟主机的方式实现

   <Host appBase="aaa" autoDeploy="true" name="www.aaa.com" unpackWARs="true"
   	xmlNamespaceAware="false" xmlValidation="false">
   			<Context path="/test" docBase="test"/>
   </Host>

   <Host appBase="bbb" autoDeploy="true" name="www.bbb.com" unpackWARs="true"
   	xmlNamespaceAware="false" xmlValidation="false">
   			<Context path="/test" docBase="test"/>
   </Host>

## 三. tomcat的体系结构

### 3.1 tomcat的体系结构

问题1：Tomcat是 一个Servlet容器？

​	`class Tomcat{`

​		`Connector connector;			//连接器,请求 ，处理请求。`

​		`List<Servlet> servlets;		//servlet容器`

​	`}`

servlet======server+applet

java applet 	--客户端

网络请求---Linux---Servlet



Servlet-----小程序



servlet

Context

Host

Engine

![](https://raw.githubusercontent.com/shihaoyan/myimages/master/QQ%E6%88%AA%E5%9B%BE20200110095038.png)



### 3.2 tomcat架构平视图

![](https://raw.githubusercontent.com/shihaoyan/myimages/master/QQ截图20200111131302.png)

#### 3.2.1 数据解析过程











### 3.2 tomcat架构俯视图

![](images\QQ截图20200111132854.png)