# Docker

[Descargar Docker](https://docs.docker.com/)  
[Descargar imágenes de Docker](https://hub.docker.com/)

## Comandos básicos
```cmd
docker ps
docker ps -a
docker container start <CONTAINER ID>
docker container start <CONTAINER ID> -d
docker container stop <CONTAINER ID>
docker container rm <CONTAINER ID>
docker logs <CONTAINER ID>
```

## Imágenes públicas
- Descargar una imagen.
- Crear un contenedor.
- Iniciar un contenedor.
- `-e`: variables de entorno.
- `-p`: mapeo de puertos (`PuertoHost:PuertoContenedor`, por ejemplo, `8080:80`).
```cmd
docker pull <IMAGE>
docker container create -e <ENVIRONMENT> <IMAGE>
docker container start <CONTAINER ID>
```

## SQL Server
- [Descargar imagen](https://hub.docker.com/r/microsoft/mssql-server)
- [Descargar cliente](https://learn.microsoft.com/en-us/ssms/download-sql-server-management-studio-ssms)
```cmd
docker pull mcr.microsoft.com/mssql/server
```
```cmd
docker container create -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=mysecretpassword" -e "MSSQL_PID=Developer" -p 1433:1433 --name SQLServer mcr.microsoft.com/mssql/server
```
```cmd
docker container start <CONTAINER ID>
```
- Crear un usuario de SQL Server.
```sql
SELECT 
	NAME AS LoginName, 
	TYPE_DESC AS AccountType, 
	create_date, 
	modify_date,
	TYPE
FROM sys.server_principals
WHERE TYPE IN ('S', 'U', 'G');
GO
CREATE LOGIN testing WITH PASSWORD = 'testing', CHECK_POLICY = OFF;
GO
CREATE DATABASE db_testing
GO
USE db_testing
GO
CREATE USER testing FOR LOGIN testing;
GO
EXEC sp_addrolemember 'db_owner', 'testing';
```

## MySQL
- [Descargar imagen](https://hub.docker.com/_/mysql)
- [Descargar cliente](https://www.mysql.com/products/workbench/)
```cmd
docker pull mysql
```
```cmd
docker run --name MySQL -e MYSQL_ROOT_PASSWORD=mysecretpassword -p 3306:3306 -d mysql
```
- Crear un usuario de MySQL.
```sql
CREATE DATABASE db_testing;
USE db_testing;

CREATE USER 'testing'@'%' IDENTIFIED BY 'testing';
GRANT CREATE, ALTER, DROP ON db_testing.* TO 'testing'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON db_testing.* TO 'testing'@'%';
GRANT REFERENCES ON db_testing.* TO 'testing'@'%';
```

## PostgreSQL
- [Descargar imagen](https://hub.docker.com/_/postgres)
- [Descargar cliente](https://www.pgadmin.org/download/pgadmin-4-windows/)
```cmd
docker pull postgres
```
```cmd
docker run --name PostgreSQL -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres
```
- Crear un usuario de PostgreSQL.
```sql
CREATE USER testing WITH PASSWORD 'testing';
CREATE DATABASE db_testing OWNER testing;
```

## Oracle Database Express
- [Registro de imágenes de Oracle](https://container-registry.oracle.com/)
- [Cliente SQL Developer](https://www.oracle.com/cl/database/sqldeveloper/)
```cmd
docker pull container-registry.oracle.com/database/express:latest
```
```cmd
docker run --name OracleDB -p 1521:1521 -e ORACLE_PWD=mysecretpassword -d container-registry.oracle.com/database/express:latest
```
- Crear un usuario de Oracle.
```sql
ALTER SESSION SET "_ORACLE_SCRIPT"=TRUE;
CREATE USER testing IDENTIFIED BY testing;
GRANT ALL PRIVILEGES TO testing;
```

## Azurite
- Emulador de Azure.
```cmd
docker run -p 10000:10000 -p 10001:10001 -p 10002:10002 mcr.microsoft.com/azure-storage/azurite
```
```cmd
docker run --name Azurite-Emulator -p 10000:10000 -p 10001:10001 -p 10002:10002 mcr.microsoft.com/azure-storage/azurite
```

## Dockerfile

### Java 21 + MySQL + Tomcat 10 + `.war`
- `Dockerfile`
```dockerfile
FROM tomcat:10.1-jdk21

WORKDIR /usr/local/tomcat/webapps/

COPY target/BootcApp-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
```
- `docker-compose.yml`
```yml
services:
  mysql:
    image: mysql
    container_name: mysql_container
    environment:
      MYSQL_ROOT_PASSWORD: testing
      MYSQL_DATABASE: testing
      MYSQL_USER: testing
      MYSQL_PASSWORD: testing
    ports:
      - '3306:3306'

  app:
    build: .
    container_name: java_app_container
    ports:
      - '8080:8080'
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/testing
      SPRING_DATASOURCE_USERNAME: testing
      SPRING_DATASOURCE_PASSWORD: testing
```

- Construir y ejecutar.
```cmd
docker compose up --build
```

> [!WARNING]  
> Si aparecen errores durante el despliegue, revisa los logs del contenedor.

- `application.properties`:
```sh
mvn clean
mvn install
```

```properties
# Configuración del servidor
server.servlet.context-path=
server.port=8080
```

## Docker init
```cmd
docker init
```

## Dockerfile
- Crear un archivo `Dockerfile`.
- `build`: construir la imagen.
- `-t`: nombre de la imagen.
- `.`: ruta del `Dockerfile`.
```cmd
docker build -t welcome-to-docker .
```

## Varios servicios en un contenedor
- Crear `compose.yaml`.
```yaml
services:
  todo-app:
    ...

  todo-database:
    ...
```
- Ejecutar el comando para construir la imagen desde Compose.
```cmd
docker compose up -d
```
- Recargar el host durante el desarrollo con Docker.
```cmd
docker compose watch
```
- Persistir la base de datos en disco agregando volúmenes al archivo Compose.
```yaml
services:
  todo-database:
    volumes: 
      - database:/data/db

volumes:
  database:
```
