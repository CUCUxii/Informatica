# Bases de datos SQL


----------------------------------------------------------------------

## Crear base de datos

 - Crear la base de datos -> *create table nombre (columna1 tipo(bytes), columna2 tipo(bytes));*

**Tipos de datos** -> int (numero); float(decimal); double(numero grande); date(año-mes-dia); time(hora-minuto-segundo); datetime(date + time); 
char(string de tamaño fijo); varchar(string de tamaño especificado); blob(objetis grandes como fotos, etc); text(texto largo)

**NOT NULL** -> El valor de esa columna no puede estar vació, 

```sql
MariaDB [(none)]> create database pruebas;
MariaDB [(none)]> use pruebas;
Database changed
MariaDB [pruebas]> CREATE TABLE peliculas(id int(2), nombre varchar(20), año varchar(4), pais varchar(10), PRIMARY KEY(id));  
MariaDB [pruebas]> CREATE TABLE bandas(id int NOT NULL AUTO_INCREMENT, nombre varchar(10) NOT NULL, PRIMARY KEY(id));
```
- Meter entradas en la base de datos -> El numero de columnas tiene que ser el mismo que el de entradas, si no, se metera un valor predeterminado.
```sql
MariaDB [pruebas]> INSERT INTO peliculas VALUES (1, 'Blade Runner', 1984, 'EE.UU');
MariaDB [pruebas]> INSERT INTO peliculas (id, nombre, año, pais) VALUES (2, 'Ghost in the shell', 1995, 'Japon');
```
- Modificar entrada 
```sql
MariaDB [pruebas]> INSERT INTO peliculas VALUES (3, 'Millenium', 2011, 'EE.UU');
MariaDB [pruebas]> UPDATE peliculas SET pais='Suecia' WHERE id=3;
MariaDB [pruebas]> INSERT INTO peliculas VALUES (4, 'Elysium', 1999, 'España');
MariaDB [pruebas]> UPDATE peliculas SET pais='España', año= WHERE id=4;
```
- Borrar entrada
```sql
MariaDB [pruebas]> DELETE FROM Peliculas WHERE ID=6;
```
- Crear columna, (todos los valores se pondran en NULL); Borrarla
```sql
MariaDB [pruebas]> ALTER TABLE peliculas ADD duracion int(3);
MariaDB [pruebas]> ALTER TABLE peliculas DROP COLUMN duracion;
MariaDB [pruebas]> ALTER TABLE peliculas RENAME nombre TO titulo;
```
----------------------------------------------------------------------

## Mostrar datos de la Database, consultas

> En SQL una base de datos se le conoce tabmien como table_schema

```sql
MariaDB [pruebas]> show databases;
prueba, information_schema, mysql, performance_schema
MariaDB [pruebas]> SHOW TABLES FROM pruebas; 
bandas, peliculas
MariaDB [pruebas]> SHOW COLUMNS FROM peliculas; 
id, nombre, año, pais
```
- SELECT

```sql
MariaDB [(none)]>  SELECT * FROM pruebas.peliculas;
+----+--------------------+------+--------+
| id | nombre             | año  | pais   |
+----+--------------------+------+--------+
|  1 | Blade Runner       | 1984 | EE.UU  |
|  2 | Ghost in the shell | 1995 | Japon  |
|  3 | Millenium          | 2011 | Suecia |
|  4 | Elyisum            | 2013 | EE.UU  |
|  5 | Jhonny Mnemonic    | 1995 | EE.UU  |
+----+--------------------+------+--------+
MariaDB [pruebas]> SELECT nombre FROM pruebas.peliculas;
Blade Runner, Ghost in the shell, Millenium, Elyisum, Jhonny Mnemonic 
MariaDB [pruebas]> SELECT tabla.columns FROM database.tabla; # En caso de haber varias tablas con el mismo nombre
MariaDB [pruebas]> SELECT nombre FROM peliculas WHERE id=4;
Elysium, 
MariaDB [pruebas]> SELECT nombre FROM peliculas WHERE id>2 AND pais='EE.UU';
Elysium, Jhonne Memmonic
MariaDB [pruebas]> SELECT nombre FROM peliculas WHERE id>2 AND (pais='EE.UU' OR pais='Suecia');
Millenium, Elysium, Jhonne Memmonic
MariaDB [pruebas]> SELECT nombre FROM peliculas WHERE pais IN ('EE.UU','Suecia');
Blade Runner, Millenium, Elysium, Jhonne Memmonic
MariaDB [pruebas]> SELECT nombre FROM peliculas WHERE pais NOT IN ('EE.UU','Suecia');
Ghost in the shell

```




