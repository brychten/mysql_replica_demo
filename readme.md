# Mysql Master-Slave Demo 

Este repositorio contiene un ejemplo de configuración de MySQL en modalidad master-slave con 2 contenedores de Docker. El master atenderá en el puerto 3306 y el esclavo en el 3307 (en la configuración por defecto).
Se asume que se configura desde un comienzo, es decir que no se precisa backupear la base de datos y restorearla en el slave, aunque en la ppt queda como guía los pasos opcionales para hacerlo. 


**Pre requisitos**
* Docker
* Docker Desktop

## Pasos para configurar
### 0. Analizar el archivo docker-compose y cambiar el password al menos, de root :).
### 1. Levantar el compose
```
docker compose up -d
```

### 2. Otorgar permisos de *replication slave* al usuario *replication_user*

* a. Conectarse al contenedor master como root `mysql -uroot -p`

* b. Otorgar permisos al usuario `replication_user`
```
ALTER USER 'replication_user'@'%' IDENTIFIED WITH 'mysql_native_password' BY 'replication_password_123';
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%';
FLUSH PRIVILEGES;
```

* c. Chequear el estado de la base master `SHOW MASTER STATUS;`

### 3. Contectase desde *mysql-slave* al nodo master para configurar replicación

* a. Conectarse al contenedor slave como root `mysql -uroot -p`

* b. Configurar conexión con nodo master
```
CHANGE MASTER TO
  MASTER_HOST='mysql-master',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='replication_password_123',
  MASTER_LOG_FILE='mysql-bin.000003',
  MASTER_LOG_POS=850;
```

```
Deben sustituir MASTER_LOG_FILE y MASTER_LOG_POS por los valores que aparecen luego de ejecutar SHOW MASTER STATUS
```

* c. Inicial el nodo slave `START SLAVE;`

* d. Chequear el estado de la base slave `SHOW SLAVE STATUS\G`

### 4. Probar

* a. Prueben crear una base de datos llamada *ejemploBD* en el nodo master. 
* b. Creen una tabla en el nodo master: 
create table persona(
    ci varchar(10) primary key,
    nombre varchar(50),
    apellido varchar(50)
)
* c. Vayan a la consola del esclavo y verifiquen que la base y la tabla hayan sido creadas. 
* d. Inserten un registro en persona en master.
* e. Chequeen que el registro fue creado en el esclavo.
* f. Realicen otras pruebas con eliminaciones, updates, etc 
