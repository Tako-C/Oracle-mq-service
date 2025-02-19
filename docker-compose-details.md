# Docker Compose Setup
<p align="center">
  <img src="https://www.svgrepo.com/show/353659/docker-icon.svg" width="150">
  <img src="https://cdn-icons-png.flaticon.com/128/16001/16001835.png" width="40">
  <img src="https://cdn-icons-png.flaticon.com/128/675/675729.png" width="100">
</p>

## Prerequisites
Before running the Docker Compose setup, ensure you have:
- [Docker](https://www.docker.com/) installed on your machine
- Access to the Oracle Container Registry (requires authentication)
- Sufficient system resources (memory and CPU) for Oracle Database

## Services
This `docker-compose.yml` defines the following services:

## Install Docker compose
```
sudo apt-get update
sudo apt-get install docker-compose-plugin
```
### 1. **Oracle Database (oracle-db)**
- **Image:** `container-registry.oracle.com/database/express`
- **Container Name:** `oracle-db`
- **Networks:** `oracle_network`
- **Ports:**
  - `1521:1521` (Oracle Database listener port)
  - `5500:5500` (Oracle EM Express port)
- **Environment Variables:**
  - `ORACLE_PWD=newPassword123` → Sets the database SYS/SYSTEM password
  - `ORACLE_ALLOW_REMOTE=true` → Enables remote connections
- **Volumes:**
  - `ords-config:/config` → Stores ORDS configuration files

### 2. **Oracle REST Data Services (ORDS)**
- **Image:** `container-registry.oracle.com/database/ords:latest`
- **Container Name:** `ords`
- **Depends On:** `oracle-db` (ensures ORDS starts after OracleDB)
- **Networks:** `oracle_network`
- **Ports:**
  - `8080:8080` (ORDS web service port)
- **Environment Variables:**
  - `ORDS_DB_HOST=oracle-db` → Connects ORDS to the Oracle database
  - `ORDS_DB_PORT=1521` → Matches OracleDB listener port
  - `ORDS_DB_SERVICE=xe` → Specifies the database service name
  - `ORDS_DB_USER=SYSTEM` → ORDS administrator user
  - `ORDS_DB_PASSWORD=newPassword123` → Password for SYSTEM user

### 3. **RabbitMQ (Message Broker)**
- **Image:** `rabbitmq:3-management`
- **Container Name:** `rabbitmq`
- **Networks:** `oracle_network`
- **Ports:**
  - `5672:5672` (RabbitMQ messaging port)
  - `15672:15672` (RabbitMQ management UI)
- **Environment Variables:**
  - `RABBITMQ_DEFAULT_USER=admin` → Default admin username
  - `RABBITMQ_DEFAULT_PASS=admin` → Default admin password

## Networks
- **oracle_network**: A bridge network that allows seamless communication between containers.

## Volumes
- **oracle-data**: Persistent storage for OracleDB data.
- **ords-config**: Stores ORDS configuration files.

## Deployment
To start the containers, run:
```sh
docker-compose up -d
```
To stop and remove containers, run:
```sh
docker-compose down
```
To view logs for a specific service, use:
```sh
docker logs -f oracle-db
```

## Accessing Services
- **OracleDB:** Connect using SQL Developer or SQL*Plus with:
  ```
  Host: localhost
  Port: 1521
  Service Name: xe
  Username: SYSTEM
  Password: newPassword123
  ```
- **ORDS:** Open [http://localhost:8080/ords](http://localhost:8080/ords)
- **RabbitMQ Management UI:** Open [http://localhost:15672](http://localhost:15672) and log in with `admin/admin`
