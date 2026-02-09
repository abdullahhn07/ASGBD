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

Instalamos las dependencias necesarias para compilar pgloader desde código fuente (SBCL, librerías de MySQL y PostgreSQL, herramientas de compilación).

```
isard@ahussain-servidor-mysql:~$ sudo apt install -y \

git build-essential curl \

sbcl unzip \

libsqlite3-dev libpq-dev \

libmysqlclient-dev

Leyendo lista de paquetes... Hecho

Creando árbol de dependencias... Hecho

Leyendo la información de estado... Hecho

git ya está en su versión más reciente (1:2.43.0-1ubuntu7.3).

build-essential ya está en su versión más reciente (12.10ubuntu1).

curl ya está en su versión más reciente (8.5.0-2ubuntu10.6).

sbcl ya está en su versión más reciente (2:2.2.9-1ubuntu2).

unzip ya está en su versión más reciente (6.0-28ubuntu4.1).

libsqlite3-dev ya está en su versión más reciente (3.45.1-1ubuntu2.5).

libpq-dev ya está en su versión más reciente (18.1-1.pgdg24.04+2).

Los paquetes indicados a continuación se instalaron de forma automática y ya no son necesarios.

  linux-headers-6.8.0-90 linux-headers-6.8.0-90-generic linux-image-6.8.0-90-generic linux-modules-6.8.0-90-generic

  linux-modules-extra-6.8.0-90-generic linux-tools-6.8.0-90 linux-tools-6.8.0-90-generic

Utilice «sudo apt autoremove» para eliminarlos.

Se instalarán los siguientes paquetes adicionales:

  libmysqlclient21 libzstd-dev

Se instalarán los siguientes paquetes NUEVOS:

  libmysqlclient-dev libmysqlclient21 libzstd-dev

0 actualizados, 3 nuevos se instalarán, 0 para eliminar y 3 no actualizados.

Se necesita descargar 3.209 kB de archivos.

Se utilizarán 17,2 MB de espacio de disco adicional después de esta operación.

Des:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libmysqlclient21 amd64 8.0.45-0ubuntu0.24.04.1 [1.255 kB]

Des:2 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libzstd-dev amd64 1.5.5+dfsg2-2build1.1 [364 kB]

Des:3 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libmysqlclient-dev amd64 8.0.45-0ubuntu0.24.04.1 [1.591 kB]

Descargados 3.209 kB en 1s (2.651 kB/s)       

Seleccionando el paquete libmysqlclient21:amd64 previamente no seleccionado.

(Leyendo la base de datos ... 185036 ficheros o directorios instalados actualmente.)

Preparando para desempaquetar .../libmysqlclient21_8.0.45-0ubuntu0.24.04.1_amd64.deb ...

Desempaquetando libmysqlclient21:amd64 (8.0.45-0ubuntu0.24.04.1) ...

Seleccionando el paquete libzstd-dev:amd64 previamente no seleccionado.

Preparando para desempaquetar .../libzstd-dev_1.5.5+dfsg2-2build1.1_amd64.deb ...

Desempaquetando libzstd-dev:amd64 (1.5.5+dfsg2-2build1.1) ...

Seleccionando el paquete libmysqlclient-dev previamente no seleccionado.

Preparando para desempaquetar .../libmysqlclient-dev_8.0.45-0ubuntu0.24.04.1_amd64.deb ...

Desempaquetando libmysqlclient-dev (8.0.45-0ubuntu0.24.04.1) ...

Configurando libmysqlclient21:amd64 (8.0.45-0ubuntu0.24.04.1) ...

Configurando libzstd-dev:amd64 (1.5.5+dfsg2-2build1.1) ...

Configurando libmysqlclient-dev (8.0.45-0ubuntu0.24.04.1) ...

Procesando disparadores para man-db (2.12.0-4build2) ...

Procesando disparadores para libc-bin (2.39-0ubuntu8.7) ...

Scanning processes...                                                                                                            

Scanning linux images...                                                                                                         



Pending kernel upgrade!

Running kernel version:

  6.8.0-94-generic

Diagnostics:

  The currently running kernel version is not the expected kernel version 6.8.0-100-generic.



Restarting the system to load the new kernel will not be handled automatically, so you should consider rebooting.



No services need to be restarted.



No containers need to be restarted.



No user sessions are running outdated binaries.



No VM guests are running outdated hypervisor (qemu) binaries on this host.

isard@ahussain-servidor-mysql:~$ 

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

Creamos la base de datos employees en PostgreSQL que será el destino de la migración desde MySQL.

```
isard@ahussain-servidor-psql:~$ sudo -u postgres psql

[sudo] password for isard: 

psql (18.1 (Ubuntu 18.1-1.pgdg24.04+2))

Digite «help» para obtener ayuda.



postgres=# CREATE DATABASE employees;

CREATE DATABASE

postgres=#
```

Usamos pgloader para migrar todas las tablas, datos y estructuras de la base de datos employees desde MySQL hacia PostgreSQL.




