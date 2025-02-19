# Container Installation Guide  
<p align="center">
  <img src="https://www.rabbitmq.com/img/rabbitmq-logo-with-name.svg" width="270">
  <span style="font-size: 60px; font-weight: bold; margin: 0 15px;">+</span>
  <img src="https://www.oracle.com/a/ocom/img/rest.svg" width="100"
  style="margin: 0 15px;"
  >
  <span style="font-size: 60px; font-weight: bold; margin: 0 15px;">+</span>
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/50/Oracle_logo.svg/512px-Oracle_logo.svg.png?20210811183004" width="270">
</p>

## Overview  
This guide provides instructions for setting up **RabbitMQ, OracleDB, and ORDS** containers.  

### <img src="https://cdn-icons-png.flaticon.com/128/2687/2687160.png" width="40">  [RabbitMQ](https://www.rabbitmq.com/documentation.html)  
RabbitMQ is a **message broker** that enables applications to communicate asynchronously using the **AMQP protocol**. It is widely used for distributed systems, microservices, and event-driven architectures.  

### <img src="https://cdn-icons-png.flaticon.com/128/9672/9672242.png" width="40"> [Oracle Database (OracleDB)](https://docs.oracle.com/en/database/oracle/oracle-database/index.html)  
OracleDB is a **relational database management system (RDBMS)** known for its high performance, scalability, and enterprise-grade security. It is commonly used in mission-critical applications.  

### <img src="https://cdn-icons-png.flaticon.com/128/5064/5064666.png" width="40">  [Oracle REST Data Services (ORDS)](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/index.html)  
ORDS is a **middleware** that allows RESTful access to **Oracle Database** without writing complex backend code. It is often used to expose database resources as **REST APIs** and integrates with **APEX**.  

### <img src="https://cdn-icons-png.flaticon.com/128/1674/1674969.png" width="40"> [Oracle Application Express (APEX)](https://apex.oracle.com/en/learn/documentation/)  
APEX is a **low-code platform** for building web applications on top of **OracleDB**. It enables developers to create secure and scalable applications without extensive coding.  

---
<br/>

## 1. Download the `oracle-mq-service` folder
```
oracle-mq-service.git
```

## 2. Navigate to the `oracle-mq-service` folder
```sh
cd oracle-mq-service
```

## 3. The folder contains the following files:
- `docker-compose.yml`
- `create_Process_log.sql`
- `Readme.md`

## 4. Run `docker-compose` to create the containers
```sh
docker compose -f docker-compose.yml up -d
```
**TIP:** Command to remove the created `docker-compose`
```sh
docker compose -f docker-compose.yml down
```

## 5. Check container status
```sh
docker ps
```

## 6. Wait until all containers are fully started


<br/>

## 7. Configure OracleDB Container
### Access the container's Bash
```sh
docker exec -it oracle-db bash
```
### Switch to `su`
```sh
su
```
### Install `wget`
```sh
yum install -y wget
```
### Exit from `su`
```sh
Ctrl + D
```
### Download & install APEX
```sh
wget https://download.oracle.com/otn_software/apex/apex_24.2.zip
```
```sh
unzip apex_24.2.zip
```
```sh
cd apex
```
### Access SQL*Plus as `SYS`
```sh
sqlplus / as sysdba
```
### Set password for `SYSTEM`
```sh
ALTER USER SYSTEM IDENTIFIED BY newPassword123;
```
### Run the APEX installation script
```sh
@apexins.sql SYSAUX SYSAUX TEMP /i/
```
รอจนกว่าจะติดตั้งสำเร็จ
### Configure APEX Admin User
```sh
@apex_rest_config.sql
```
**Verify APEX installation**
```sh
SELECT * FROM apex_release;
```

---

## 8. Configure ORDS Container
### Access the `ORDS` container
```sh
docker exec -it ords bash
```
### Install ORDS
```sh
ords install
```
ORDS installation prompts:
```
Oracle REST Data Services - Interactive Install

  Enter a number to select the database connection type to use
    [1] Basic (host name, port, service name)
    [2] TNS (TNS alias, TNS directory)
    [3] Custom database URL
  Choose [1]: 1

  Enter the database host name [localhost]: oracle-db
  Enter the database listen port [1521]: 1521
  Enter the database service name [orcl]: xe

  Provide database user name with administrator privileges.
    Enter the administrator username: SYSTEM
  Enter the database password for SYSTEM: newPassword123
  Install, upgrade, validate or uninstall ORDS in the CDB requires you to login as SYS AS SYSDBA
```
### Start ORDS
```sh
ords serve --port 8081
```

## 9. Set Up RabbitMQ and Create a Queue
### Access the `RabbitMQ` container
```sh
docker exec -it rabbitmq bash
```
### Create a queue in RabbitMQ
```sh
apt-get update ;apt-get install -y curl
```
### Enable RabbitMQ Management Plugin
```sh
rabbitmq-plugins enable rabbitmq_management
```

## 10. Allow Network Access
### Create Network Access
```sql
BEGIN
  DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
    host => 'rabbitmq', 
    ace => xs$ace_type(privilege_list => xs$name_list('connect', 'resolve'),
                       principal_name => 'APEX_240200', -- Change to your SCHEMA
                       principal_type => xs_acl.ptype_db)
  );
END;
/
COMMIT;
```
### Check Network Access
```sql
SELECT host, acl FROM dba_network_acls;
```
### Tip:
```sql
SELECT acl, principal, privilege, is_grant 
FROM dba_network_acl_privileges 
WHERE acl = 'NETWORK_ACL_2E666DFBA3620A91E063030012ACF9FE';
```
```sql
SELECT host, acl FROM dba_network_acls WHERE host = 'rabbitmq';
```
### Create external network access
```sql
BEGIN
  DBMS_NETWORK_ACL_ADMIN.CREATE_ACL(
    acl         => 'apex_acl_thddns.xml',
    description => 'Allow APEX to access external network',
    principal   => 'APEX_240200',
    is_grant    => TRUE,
    privilege   => 'connect'
  );

  DBMS_NETWORK_ACL_ADMIN.ADD_PRIVILEGE(
    acl       => 'apex_acl_thddns.xml',
    principal => 'APEX_240200',
    is_grant  => TRUE,
    privilege => 'resolve'
  );

  DBMS_NETWORK_ACL_ADMIN.ASSIGN_ACL(
    acl  => 'apex_acl_thddns.xml',
    host => 'zyxmanetwork.thddns.net',
    lower_port => 5054,
    upper_port => 5054
  );
END;
/
COMMIT;
```
```sql
SELECT host, acl FROM dba_network_acls WHERE host = 'zyxmanetwork.thddns.net';
```

## 11. Create Procedure 
### Within the same host
```sql
CREATE OR REPLACE PROCEDURE READ_FROM_QUEUE IS
    l_url        VARCHAR2(4000);
    l_response   CLOB;
    l_payload    VARCHAR2(4000);
BEGIN
    l_url := 'http://rabbitmq:15672/api/queues/%2F/my_queue/get';
    l_response := apex_web_service.make_rest_request(
        p_url         => l_url,
        p_http_method => 'POST',
        p_body        => '{"count":1, "ackmode":"ack_requeue_false", "encoding":"auto"}',
        p_username    => 'admin',
        p_password    => 'admin'
    );
    DBMS_OUTPUT.PUT_LINE('Response: ' || l_response);
END;
/
COMMIT;
```
### Execute to retrieve a queue from RabbitMQ
```sql
EXEC READ_FROM_QUEUE; 
```
```sql
BEGIN
    READ_FROM_QUEUE;
END;
/
``` 
### From an external host
```sql
CREATE OR REPLACE PROCEDURE READ_FROM_QUEUE_FROM_OUT_HOST IS
    l_url        VARCHAR2(4000);
    l_response   CLOB;
    l_payload    VARCHAR2(4000);
BEGIN
    l_url := 'http://zyxmanetwork.thddns.net:5054/api/queues/%2F/test_queue/get';
    l_response := apex_web_service.make_rest_request(
        p_url         => l_url,
        p_http_method => 'POST',
        p_body        => '{"count":1, "ackmode":"ack_requeue_false", "encoding":"auto"}',
        p_username    => 'guest',
        p_password    => 'guest'
    );
    DBMS_OUTPUT.PUT_LINE('Response: ' || l_response);
END;
/
COMMIT;
```
### Execute to retrieve a queue from RabbitMQ
```sql
EXEC READ_FROM_QUEUE_FROM_OUT_HOST;
```
