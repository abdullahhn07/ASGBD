# Replicació de sistemes heterogenis

## 0. Preparació de l'escenari

Dos máquinas con Ubuntu Server 24.04:

servidor-mysql: con la versión más reciente de MySQL.
servidor-psql: con PostgreSQL 17 o 18.

```
isard@ahussain-servidor-mysql:~$ mysql --version
mysql  Ver 8.0.45-0ubuntu0.24.04.1 for Linux on x86_64 ((Ubuntu))
```
```
isard@ahussain-servidor-psql:~$ psql --version
psql (PostgreSQL) 18.1 (Ubuntu 18.1-1.pgdg24.04+2)
```

## 1. Enunciado

Se documenta todo el proceso de migración de la base de datos employees desde MySQL a PostgreSQL utilizando pgloader, con el objetivo de tener un tutorial de referencia y soporte para otros ejercicios. Se incluyen los pasos de preparación de los servidores, instalación de pgloader, creación de usuarios y bases de datos, ejecución de la migración y verificación de los datos importados correctamente.


## Primera Parte: Preparación

## Objetivo

Migrar los datos de la base de datos employees desde servidor-mysql a servidor-psql utilizando pgloadery Se documenta todo el proceso para usarlo como referencia futura y como soporte para otros ejercicios.

## Paso 1: Comprobar la base de datos employees en MySQL

Actualizamos la lista de paquetes y el sistema para evitar problemas de dependencias al compilar pgloader.

```
isard@ahussain-servidor-mysql:~$ sudo apt update && sudo apt upgrade -y
```

Des de la màquina servidor-mysql, entramos a MySQL con el comando ( sudo mysql -u root -p ).

```
isard@ahussain-servidor-mysql:~$ sudo mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.45-0ubuntu0.24.04.1 (Ubuntu)

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

Comprobamos que la base de datos existe con el comando ( SHOW DATABASES; )

```
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0,00 sec)
```
Entramos a la base de datos con el comando ( USE employees; ) y ( SHOW TABLES; )

```
mysql> USE employees;
Database changed
mysql> SHOW TABLES;
+----------------------+
| Tables_in_employees  |
+----------------------+
| current_dept_emp     |
| departments          |
| dept_emp             |
| dept_emp_latest_date |
| dept_manager         |
| employees            |
| salaries             |
| titles               |
+----------------------+
8 rows in set (0,00 sec)

mysql> 
```
Finalmente , comprobamos que contiene datos con el comando ( SELECT COUNT(*) FROM employees; )

```
mysql> SELECT COUNT(*) FROM employees;
+----------+
| COUNT(*) |
+----------+
|   300024 |
+----------+
1 row in set (0,02 sec)

mysql> 
```

## Paso 2: Descarga del repositorio oficial de pgloader


Instalamos las dependencias necesarias para compilar pgloader desde código fuente (SBCL, librerías de MySQL y PostgreSQL, herramientas de compilación).

```
isard@ahussain-servidor-mysql:~$ sudo apt install -y \

git build-essential curl \

sbcl unzip \

libsqlite3-dev libpq-dev \

libmysqlclient-dev

```

Clonamos el repositorio oficial de pgloader directamente en /opt/pgloader.

```
isard@ahussain-servidor-mysql:~$ sudo git clone https://github.com/dimitri/pgloader /opt/pgloader

Cloning into '/opt/pgloader'...

remote: Enumerating objects: 12050, done.

remote: Counting objects: 100% (215/215), done.

remote: Compressing objects: 100% (132/132), done.

remote: Total 12050 (delta 135), reused 83 (delta 83), pack-reused 11835 (from 4)

Receiving objects: 100% (12050/12050), 20.44 MiB | 11.29 MiB/s, done.

Resolving deltas: 100% (8417/8417), done.

isard@ahussain-servidor-mysql:~$ 
```

Compilamos pgloader usando el código fuente descargado sin necesidad de cambiar de directorio.

```
isaRd@ahussain-servidor-mysql:~$ make -C /opt/pgloader
```

Forzamos el uso de la versión compilada de pgloader creando un enlace simbólico en el PATH del sistema.

```
isard@ahussain-servidor-mysql:~$ sudo ln -sf /opt/pgloader/build/bin/pgloader /usr/local/bin/pgloader

isard@ahussain-servidor-mysql:~$
```

Verificamos que la versión de pgloader sea 3.6.7 o superior, compatible con MySQL 8.

```
isard@ahussain-servidor-mysql:~$ pgloader --version

pgloader version "3.6.d9ca38e"

compiled with SBCL 2.2.9.debian

isard@ahussain-servidor-mysql:~$ 
```

Comprobamos que la base de datos employees existe en el servidor MySQL antes de realizar la migración.

```
isard@ahussain-servidor-mysql:~$ sudo mysql -u root -p

Enter password: 

Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 34

Server version: 8.0.45-0ubuntu0.24.04.1 (Ubuntu)



Copyright (c) 2000, 2026, Oracle and/or its affiliates.



Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.



Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.



mysql> SHOW DATABASES;

+--------------------+

| Database           |

+--------------------+

| employees          |

| information_schema |

| mysql              |

| performance_schema |

| sys                |

+--------------------+

5 rows in set (0,02 sec)



mysql> 

```

```
mysql> USE employees;

Reading table information for completion of table and column names

You can turn off this feature to get a quicker startup with -A



Database changed

mysql> SHOW TABLES;

+----------------------+

| Tables_in_employees  |

+----------------------+

| current_dept_emp     |

| departments          |

| dept_emp             |

| dept_emp_latest_date |

| dept_manager         |

| employees            |

| salaries             |

| titles               |

+----------------------+

8 rows in set (0,00 sec)



mysql> SELECT COUNT(*) FROM employees;

+----------+

| COUNT(*) |

+----------+

|   300024 |

+----------+

1 row in set (0,05 sec)



mysql>
```



```



