在cas-overlay基础上, 根据文档配置的服务管理服务端。
管理员通过该服务，注册、编辑、删除、查看客户端服务。
cas文档：http://jasig.github.io/cas/4.0.x/index.html
cas构建基础：https://github.com/lansheng228/cas-overlay
构建命令：mvn clean package
将生成的war包，放到/var/lib/tomcat7/webapps/下， 重启tomcat。
进入/var/lib/tomcat7/webapps/下，执行：
rm ./cas/WEB-INF/lib/hibernate-commons-annotations-4.0.1.Final.jar 
rm ./cas/WEB-INF/lib/hibernate-core-4.1.0.Final.jar 
rm ./cas/WEB-INF/lib/hibernate-jpa-2.0-api-1.0.1.Final.jar
rm ./management/WEB-INF/lib/hibernate-commons-annotations-4.0.1.Final.jar 
rm ./management/WEB-INF/lib/hibernate-core-4.1.0.Final.jar 
rm ./management/WEB-INF/lib/hibernate-jpa-2.0-api-1.0.1.Final.jar
再次重启tomcat。
  
访问https://localhost:8443/management
账户: simba 密码：simba

分为两个war包：cas.war提供cas认证功能；management.war提供服务管理功能。
该配置中，所有服务存储在mysql数据库中。

在mysql数据库上新建方案cas,在cas上建立表RegisteredServiceImpl、rs_attributes、LOCKS、SERVICETICKET、TICKETGRANTINGTICKET

create table LOCKS
(
  APPLICATION_ID  varchar(255) not null,
  EXPIRATION_DATE date,
  UNIQUE_ID       varchar(255)
);

create table rs_attributes
(
  registeredserviceimpl_id integer not null,
  a_name                   varchar(255) not null,
  a_id                     integer not null
);

create table RegisteredServiceImpl
 (expression_type varchar(15) DEFAULT 'ant' not null,
  id bigint not null primary key auto_increment,
 allowedToProxy char(1) not null,
 anonymousAccess char(1) not null,
 description varchar(255),
 enabled char(1) not null,
 evaluation_order integer not null,
  ignoreAttributes char(1) not null,
 name varchar(255),
 serviceId varchar(255),
 ssoEnabled char(1) not null,
 required_handlers blob(255),
  theme varchar(255),
 username_attr varchar(256)
  );

create table SERVICETICKET (
 ID varchar(255) not null,
 NUMBER_OF_TIMES_USED integer,
 CREATION_TIME bigint,
 EXPIRATION_POLICY blob not null,
 LAST_TIME_USED bigint,
 PREVIOUS_LAST_TIME_USED bigint,
 FROM_NEW_LOGIN char(1) not null,
 TICKET_ALREADY_GRANTED char(1) not null,
 SERVICE blob not null,
 ticketGrantingTicket_ID varchar(255),
 primary key (ID));


create table TICKETGRANTINGTICKET (
 ID varchar(255) not null,
 NUMBER_OF_TIMES_USED bigint,
CREATION_TIME bigint,
EXPIRATION_POLICY blob not null,
 LAST_TIME_USED bigint,
PREVIOUS_LAST_TIME_USED bigint,
AUTHENTICATION blob not null,
 EXPIRED char(1) not null,
SERVICES_GRANTED_ACCESS_TO blob not null,
SUPPLEMENTAL_AUTHENTICATIONS blob not null,
ticketGrantingTicket_ID varchar(255),
primary key (ID));

alter table RS_ATTRIBUTES add constraint FK4322E153C595E1F foreign key (RegisteredServiceImpl_id)
references RegisteredServiceImpl;

alter table LOCKS
  add primary key (APPLICATION_ID);

alter table RS_ATTRIBUTES
  add primary key (REGISTEREDSERVICEIMPL_ID, A_ID);

插入测试数据:
  insert into RegisteredServiceImpl (allowedToProxy, anonymousAccess, description, enabled, evaluation_order, ignoreAttributes, name, required_handlers, serviceId, ssoEnabled, theme, username_attr, expression_type) values (0, 0, "dddd", 1, 10000002, 0, "syberos", null, "^(https|imaps)://.*", 1, "apereo", "", 'ant')

根据实际情况配置deployerConfigContext.xml中的mysql管理员用户名和密码。

