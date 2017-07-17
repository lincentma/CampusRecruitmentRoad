# SSM框架整合理解

> 把IntelliJ IDEA+Maven+Spring + SpringMVC + MyBatis项目部署，框架流程梳理调试了一遍，加深自己的理解。



## 回顾SSM框架

**Spring**
Spring就像是整个项目中装配bean的大工厂，在配置文件中可以指定使用特定的参数去调用实体类的构造方法来实例化对象。
Spring的核心思想是IoC（控制反转），即不再需要程序员去显式地`new`一个对象，而是让Spring框架帮你来完成这一切。

**SpringMVC**
SpringMVC在项目中拦截用户请求，它的核心Servlet即DispatcherServlet承担中介或是前台这样的职责，将用户请求通过HandlerMapping去匹配Controller，Controller就是具体对应请求所执行的操作。SpringMVC相当于SSH框架中struts。

**mybatis**
mybatis是对jdbc的封装，它让数据库底层操作变的透明。mybatis的操作都是围绕一个sqlSessionFactory实例展开的。mybatis通过配置文件关联到各实体类的Mapper文件，Mapper文件中配置了每个类对数据库所需进行的sql语句映射。在每次与数据库交互时，通过sqlSessionFactory拿到一个sqlSession，再执行sql命令。

## SSM框架流程



![SpringMVC处理流程](http://images2015.cnblogs.com/blog/713721/201603/713721-20160302144740705-313885038.png)



## SSM框架搭建

### 创建Maven的Web项目

1. 通过IntelliJ IDEA创建maven项目：
   - 选中Createfrom archetype，选择maven-archetype-webapp
   - 在Properties中添加一个参数 archetypeCatalog=internal，提高maven项目构建速度
2. SSH框架Web项目框架
   - main：
     - 创建java文件夹：项目代码
     - resources文件夹：
       1. mapping文件夹：数据库表xml
       2. xml配置文件
   - webapp：
     - WEB-INF：
       - 创建jsp文件夹：不同显示页面
       - web.xml:配置文件
3. Tomcat启动项目
   - 为项目配置Tomcat

### 配置各种XML

1. pom.xml——引入项目所需要的jar包

   - spring核心依赖
   - mybatis依赖
   - mybatis-spring整合包依赖
   - mysql驱动依赖
   - 其他依赖：
     - 日志相关：log4j、slf4j
     - 连接池相关：commons-dbcp、c3p0、Druid
     - Json相关：fastjson
     - 其他：jstl
   - PS：此外还有SpringBoot可以简化xml中的配置项数量。SpringBoot完全抛弃了繁琐的XML文件配置方式，而是替代性地用注解方式来实现。 
     - 参考文章：[IDEA下从零开始搭建SpringBoot工程](http://blog.csdn.net/u013248535/article/details/55100979)
     - 调试过程中的错误有很大一部分是所引的jar没有在pom.xml配置，这部分需要仔细细致。
     - 关于jar包的版本号的修改，可以在<properties></properties>标签中用变量保存版本号，<dependencies></dependencies>中具体的jar包的版本用变量代替，方便后续修改。

2. web.xml

   - 这是整个web项目的配置文件。

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://java.sun.com/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
             http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
            version="3.0">

     <display-name>cloudmusic_ssm_demo</display-name>

     <context-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:spring-mybatis.xml</param-value>
     </context-param>

     <context-param>
       <param-name>log4jConfigLocation</param-name>
       <param-value>classpath:log4j.properties</param-value>
     </context-param>

     <!-- 编码过滤器 -->
     <filter>
       <filter-name>encodingFilter</filter-name>
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
       <init-param>
         <param-name>encoding</param-name>
         <param-value>UTF-8</param-value>
       </init-param>
     </filter>
     <filter-mapping>
       <filter-name>encodingFilter</filter-name>
       <url-pattern>/*</url-pattern>
     </filter-mapping>

     <!-- spring监听器 -->
     <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
     </listener>

     <!-- 防止spring内存溢出监听器，比如quartz -->
     <listener>
       <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
     </listener>


     <!-- spring mvc servlet-->
     <servlet>
       <servlet-name>SpringMVC</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:spring-mvc.xml</param-value>
       </init-param>
       <load-on-startup>1</load-on-startup>
       <async-supported>true</async-supported>
     </servlet>
     
     <servlet-mapping>
       <servlet-name>SpringMVC</servlet-name>
       <!-- 此处也可以配置成 *.do 形式 -->
       <url-pattern>/</url-pattern>
     </servlet-mapping>

     <welcome-file-list>
       <welcome-file>/index.jsp</welcome-file>
     </welcome-file-list>

     <!-- session配置 -->
     <session-config>
       <session-timeout>15</session-timeout>
     </session-config>

   </web-app>
   ```

   - <servlet>中的配置，加载**SpringMVC**的配置文件。
     - SpringMVC具有统一的入口DispatcherServlet，所有的请求都通过DispatcherServlet。DispatcherServlet是前置控制器，配置在web.xml文件中的。拦截匹配的请求，Servlet拦截匹配规则要自已定义，把拦截下来的请求，依据某某规则分发到目标Controller来处理。
     - 拦截所有的请求，并加载所有的ssm配置文件（路径为classpath:spring-mvc.xml）
   - 在web.xml中使用contextConfigLocation参数定义要装入的**Spring**配置文件。
     - 加载路径为classpath:spring-mybatis.xml文件
   - 参考文章： [SSM:spring+springmvc+mybatis框架中的XML配置文件功能详细解释](http://blog.csdn.net/yijiemamin/article/details/51156189)

3. spring-mvc.xml

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context-4.0.xsd
                           http://www.springframework.org/schema/mvc
                           http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

       <!-- 自动扫描  @Controller-->
       <context:component-scan base-package="com.ssm.demo.controller"/>

       <!--避免IE执行AJAX时，返回JSON出现下载文件 -->
       <bean id="mappingJacksonHttpMessageConverter"
             class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
           <property name="supportedMediaTypes">
               <list>
                   <value>text/html;charset=UTF-8</value>
               </list>
           </property>
       </bean>
       <!-- 启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->
       <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
           <property name="messageConverters">
               <list>
                   <ref bean="mappingJacksonHttpMessageConverter"/> <!-- JSON转换器 -->
               </list>
           </property>
       </bean>


       <!-- 定义跳转的文件的前后缀 ，视图模式配置 -->
       <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="/WEB-INF/jsp/" />
           <property name="suffix" value=".jsp"/>
       </bean>

       <!-- 文件上传配置 -->
       <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
           <!-- 默认编码 -->
           <property name="defaultEncoding" value="UTF-8"/>
           <!-- 上传文件大小限制为31M，31*1024*1024 -->
           <property name="maxUploadSize" value="32505856"/>
           <!-- 内存中的最大值 -->
           <property name="maxInMemorySize" value="4096"/>
       </bean>
   </beans>
   ```

   - controller注入：使用组件扫描方式，扫描包下面所有的Controller，可以使用注解来指定访问路径。
   - Spring 所有功能都在 Bean 的基础上演化而来，所以必须事先将 Controller 变成 Bean。配置了一个 AnnotationMethodHandlerAdapter，它负责根据 Bean 中的 Spring MVC 注解对 Bean 进行加工处理，使这些 Bean 变成控制器并映射特定的 URL 请求。
   - 视图解析：在Controller中设置视图名的时候会自动加上前缀和后缀。

4. spring-mybatis.xml：Spring与MyBatis的整合配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context-3.1.xsd
                           http://www.springframework.org/schema/tx
                           http://www.springframework.org/schema/tx/spring-tx.xsd">

       <!-- 自动扫描 -->
       <context:component-scan base-package="com.ssm.demo"/>

       <!-- 第一种方式：加载一个properties文件 -->
       <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
           <property name="location" value="classpath:jdbc.properties"/>
       </bean>


       <!-- 第二种方式：加载多个properties文件
       <bean id="configProperties" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
           <property name="locations">
               <list>
                   <value>classpath:jdbc.properties</value>
                   <value>classpath:common.properties</value>
               </list>
           </property>
           <property name="fileEncoding" value="UTF-8"/>
       </bean>
       <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
           <property name="properties" ref="configProperties"/>
       </bean>
       -->

       <!-- 配置数据源 -->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
             destroy-method="close">
           <property name="driverClassName" value="${driverClass}"/>
           <property name="url" value="${jdbcUrl}"/>
           <property name="username" value="${username}"/>
           <property name="password" value="${password}"/>
           <!-- 初始化连接大小 -->
           <property name="initialSize" value="${initialSize}"></property>
           <!-- 连接池最大数量 -->
           <property name="maxActive" value="${maxActive}"></property>
           <!-- 连接池最大空闲 -->
           <property name="maxIdle" value="${maxIdle}"></property>
           <!-- 连接池最小空闲 -->
           <property name="minIdle" value="${minIdle}"></property>
           <!-- 获取连接最大等待时间 -->
           <property name="maxWait" value="${maxWait}"></property>
       </bean>

       <!-- mybatis和spring完美整合，不需要mybatis的配置映射文件 -->
       <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
           <property name="dataSource" ref="dataSource"/>
           <!-- 自动扫描mapping.xml文件 -->
           <property name="mapperLocations" value="classpath:mapping/*.xml"></property>
       </bean>

       <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
       <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
           <property name="basePackage" value="com.ssm.demo.dao"/>
           <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
       </bean>


       <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->
       <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
           <property name="dataSource" ref="dataSource"/>
       </bean>

       <!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->
       <tx:annotation-driven transaction-manager="transactionManager"/>
   </beans>
   ```

   - 自动扫描，自动注入，配置数据库
     - 自动扫描,将标注Spring注解的类自动转化Bean，同时完成Bean的注入
     - 加载数据资源属性文件
     - 配置数据源（三种方式，采用DBCP）
     - 配置sessionfactory
     - 装配Dao接口
     - 声明式事务管理 
     -  注解事务切面 
   - Mapper.xml映射文件中定义了操作数据库的sql，每一个sql是一个statement，映射文件是myBatis的核心。

5. jdbc.properties：JDBC属性文件

   ```properties
   driverClass=com.mysql.jdbc.Driver
   jdbcUrl=jdbc:mysql://localhost:3306/db_ssm?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull
   username=root
   password=147789

   #定义初始连接数
   initialSize=0
   #定义最大连接数
   maxActive=20
   #定义最大空闲
   maxIdle=20
   #定义最小空闲
   minIdle=1
   #定义最长等待时间
   maxWait=60000
   ```





### 创建业务流程

> 以数据库查询表内容为例



> 持久层：DAO层（mapper）**做数据持久层的工作**，负责与数据库进行联络的一些任务都封装在此，

> - DAO层的设计首先是设计DAO的接口，
> - 然后在Spring的配置文件中定义此接口的实现类，
> - 然后就可在模块中调用此接口来进行数据业务的处理，而不用关心此接口的具体实现类是哪个类，显得结构非常清晰，
> - DAO层的数据源配置，以及有关数据库连接的参数都在Spring的配置文件中进行配置。

> 业务层：Service层  **主要负责业务模块的逻辑应用设计**。

> - 首先设计接口，再设计其实现的类
> - 接着再在Spring的配置文件中配置其实现的关联。这样我们就可以在应用中调用Service接口来进行业务处理。
> - Service层的业务实现，具体要调用到已定义的DAO层的接口，
> - 封装Service层的业务逻辑有利于通用的业务逻辑的独立性和重复利用性，程序显得非常简洁。

> 表现层：Controller层（Handler层）**负责具体的业务模块流程的控制**

> - 在此层里面要调用Service层的接口来控制业务流程，
> - 控制的配置也同样是在Spring的配置文件里面进行，针对具体的业务流程，会有不同的控制器，我们具体的设计过程中可以将流程进行抽象归纳，设计出可以重复利用的子单元流程模块，这样不仅使程序结构变得清晰，也大大减少了代码量。

> 模型层：Model层 **主要存放实体类**

项目代码结构：

- controller：

  - “@RequestMapping”请求路径映射，如果标注在某个controller的类级别上，则表明访问此类路径下的方法都要加上其配置的路径；最常用是标注在方法上，表明哪个具体的方法来接受处理某次请求。
  - 调用service层方法
  - spring mvc 支持如下的返回方式：ModelAndView, Model, ModelMap, Map,View, String, void。本文返回的是String，通过model进行使用。
    - 参考文章：[SpringMVC返回（return）方式详解](https://my.oschina.net/zhdkn/blog/316530)

- service：建立service接口和实现类

  - impl:接口对应实现类：
    - 调用Dao层的数据库操作以及model层的实体类

- dao

  - 定义接口中的方法
  - **一个Dao对应一个对应的mapper文件，实现Dao对应的定义的接口方法**

- mapping：

  - mapper.xml：实现dao中接口定义的方法

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
  <mapper namespace="com.ssm.demo.dao.UserDao">

      <resultMap id="UserBaseMap" type="com.ssm.demo.model.User">
          <id column="id" property="id" jdbcType="BIGINT"/>
          <result column="user_name" property="userName" jdbcType="VARCHAR"/>
          <result column="user_phone" property="userPhone" jdbcType="VARCHAR"/>
          <result column="user_email" property="userEmail" jdbcType="VARCHAR"/>
          <result column="user_pwd" property="userPwd" jdbcType="VARCHAR"/>
          <result column="pwd_salt" property="pwdSalt" jdbcType="VARCHAR"/>
          <result column="create_time" property="createTime" jdbcType="DATE"/>
          <result column="modify_time" property="modifyTime" jdbcType="DATE"/>
          <result column="is_delete" property="isDelete" jdbcType="SMALLINT"></result>
      </resultMap>

      <select id="selectUserById" parameterType="java.lang.Long" resultMap="UserBaseMap">
          SELECT * FROM t_user
          WHERE id = #{userId}
      </select>

      <select id="selectUserByPhoneOrEmail" resultMap="UserBaseMap">
          SELECT * FROM t_user
          WHERE user_email = #{emailOrPhone} OR user_phone = #{emailOrPhone}
          AND user_state = #{state}
      </select>

      <select id="selectAllUser" resultMap="UserBaseMap">
          SELECT * FROM t_user
      </select>

  </mapper>
  ```

  - namespace:当前库表映射文件的命名空间，唯一的不能重复
  - 映射实体类的数据类型 id：resultMap的唯一标识
  - column:库表的字段名 property:实体类里的属性名
  - id：当前sql的唯一标识  
  - parameterType：输入参数的数据类型   
  - 返回值的数据类型：resultMap适合使用返回值是自定义实体类的情况 ； resultType适合使用返回值的数据类型是非自定义的，即jdk的提供的类型。
  - {}:用来接受参数的，如果是传递一个参数#{id}内容任意，如果是多个参数就有一定的规则,采用的是预编译的形式select

- model

  - 实体属性——对应表中的元组的属性
  - getter和setter方法



> DataBase ===> Entity ===> Mapper.xml ===> Mapper.[Java](http://lib.csdn.net/base/java) ===> Service.java ===> Controller.java ===> Jsp.

## 参考文章

[SSM框架整合（IntelliJ IDEA + maven + Spring + SpringMVC + MyBatis）](http://blog.csdn.net/gallenzhang/article/details/51932152)



## SSM框架感受

> 本质上的MVC，xml配置、注解，以及mapper的映射，让开发更加简洁和思路清晰



## 





