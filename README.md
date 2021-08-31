# Spring Cloud+Nacos+Feign整合微信支付登录接口实现分布式医院挂号系统

## 整体架构 

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0b6e9f19eb94e1a9f4048b723c5147b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTA2MjI5OQ==,size_16,color_FFFFFF,t_70#pic_center)

## 业务流程图

![在这里插入图片描述](https://img-blog.csdnimg.cn/35f4415969db4a7a95e5ed51cae9a141.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTA2MjI5OQ==,size_16,color_FFFFFF,t_70#pic_center)

## **技术栈**   

 _使用Maven构建的SpringCloud项目 采用微服务架构 使用Redis MongoDB MyBatis-Pus RabbitMQ等技术实现医院挂号相关功能 并集成了微信登录、支付接口实现微信登录及订单支付功能_

## 包结构

- _common包 包含项目中通用的一些配置和工具类(全局异常处理 统一数据返回类型)(Swager2 Redis配置类)_
- _hospital-manage 医院端服务 实现医院接口的设置及科室、排班等相关操作(需与医院服务验证通过后才能实现相关操作)_
- _model 包含项目中所有实体类的定义(Enums DO VO)_
- *service 项目中心服务的服务包 包含项目下所有的微服务(用户 医院:8201(医院设置 科室 排班 ...) 数据字典:8202 ...)* 


```java
● yygh_parent[文件根目录]
	● common[项目通用工具类配置类]
		◎ common_util[全局异常处理 统一数据返回类型...]
		◎ service_util[Swagger2配置 Redis配置 MD5工具类...]
	● hospital_manage[医院端服务 包含医院接口设置 科室 排班等相关操作]
	● model[包含项目下所有的实体定义 Enums Do Vo...]
		◎ enums
		◎ model
		◎ vo
	● service[服务类包 包含所有微服务 用户 医院:8201 数据字典:8202]
		◎ service_cmn[数据字典相关服务 查询数据字典查]
		◎ service_hosp[医院相关服务 CRUD医院设置 上传查询医院 上传查询删除科室.排班 ]
		◎ service_user
	● service_client[]
		◎ service_cmn_client[service_cmn服务调用]
```

## 对应技术的配置与实现

    Nacos:8848 Redis:6379 MongoDB:27017 前后台跨越:@CrossOrigin注解 Nginx前端负载均衡

- ### 服务注册Nacos与服务调用Feign

  * ###### 准备

        下载nacos文件包 解压启动 在文件目录下 sh startup.sh -m standalone(MacOSX)

  * ###### 项目应用

        1.添加依赖
        <dependency>
        	<groupId>com.alibaba.cloud</groupId>
        	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        2.配置文件(yml)中添加配置
        Spring:
        	cloud:
        		nacos:
        			discovery:
        				server-addr: http://localhost:8848
        3.在启动类添加@EnableDiscoveryClient注解
        4.创建Feign接口 添加FeignClient注解 加入Controller中的方法@PathVariable添加命名

- ### MyBatis-Plus ( GitHub [MyBatis-Plus Demo](https://github.com/Sooocen/Spring-Boot-MyBatis-Plus-demo-) 地址 )

  * ###### 配置

        在config包配置文件内配置mybatisplusConfig的分页插件
        //分页查询插件
        @Bean
        public PaginationInterceptor paginationInterceptor(){
        	return new PaginationInterceptor();
        }

- ### Redis + Cache缓存

      各模块下pom.xml文件引入依赖 
      在common下service_util中创建Config文件 
      添加Configuration EnableCaching注解 在yml文件中进行配置

  * ###### 配置

        Spring:
        	redis:
        		host: localhost
        		port: 6379
        		database: 0
        		timeout: 1800000
        		lettuce:
        			pool:
        				max-active: 20
        				max-wait: 1
        				max-idle: 5

  * ###### 启用服务

        Terminal命令 redis-server (MacOSX)

 - ### MongoDB

      在对应微服务包下添加Repository包并创建对应类的Repository 继承MongoRepository<XXXX,String> 方法名遵循Spring Data的标准(XXXX为实体类)

   * ###### 配置

         Spring:	data:		mongodb:			uri: mongodb://localhost:27017/yygh_hosp

   * ###### 使用

         在ServiceImpl中调用相应类的Repository进行MongoDB操作(方法名遵循Spring Data的标准)

   * ###### 启动

         Terminal命令 brew services start mongodb-community@4.4(MacOSX)

- ### Nginx

      前端解决微服务架构 无法访问多个后端端口问题 下载安装Nginx在nginx文件/路径下nginx.conf中配置对应后端路由

  * ###### 前端反向路由

        修改前端config/dev.env.js中全局端口为9001

  * ###### 配置

        server {	listen       9001;	server_name  localhost;	location ~ /hosp/ {		proxy_pass http://localhost:8201;     }	location ~ /cmn/ {		proxy_pass http://localhost:8202;	 }}

# 核心业务

# 技术点Demo汇总

- [x] [Spring Boot RabbitMQ Demo](https://github.com/Sooocen/Spring-Boot-RabbitMQ-demo)
- [x] [Spring Boot MyBatis-Plus Demo](https://github.com/Sooocen/Spring-Boot-MyBatis-Plus-demo-)
- [x] [Spring Boot SSO Demo(Spring Security+Oauth2+JWT)](https://github.com/Sooocen/Spring-Boot-SSO-Demo-Security-Oauth2-JWT-)

# 写在最后

####   **项目为[尚硅谷](https://www.bilibili.com/video/BV1V5411K7rT)出品,此文章为个人学习过程中的心得和学习笔记,仅作为个人学习记录分享.侵删**
