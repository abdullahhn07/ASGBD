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
