# Bases de datos SQL

\[Indice]

 - [¿Que es SQL?](#que-es-sql)
 - [Instalación y uso](#instalacion-y-uso)
 - [Crear base de datos](#crear-base-de-datos)
 - [Consultar datos](#consultar-datos)
 - [Funciones](#funciones)
 - [Tabla information schema](#tabla-information-schema)
 - [Los archivos .sql](#los-archivos-sql)

----------------------------------------------------------------------

## Que es SQL

**SQL** se traduce como *"Structured Query Languaje"* o lenguaje de consultas estruturado, es decir, es una manera de clasificar datos creando lo que 
se llama base de datos. Estas bases de datos tienen cada una tablas, que son como las clasicas tablas excel, donde cada columna es un atributo (ej nombre, edad, sueldo...) y cada fila es un dato con esos atributos(ej empleado). Con simples consultas se pueden acceder a esos datos ordenandolos, filtrandolos, hasta modificandolos...

SQL es un estilo de lenguaje para estas bases de datos, dentro de SQL hay diferentes lenguajes como MySQL, SQLite...

----------------------------------------------------------------------

## Instalacion y uso (Linux)

```console
[usaurio@linux]─[~]:$ sudo apt install mariadb-client mariadb-server
[usaurio@linux]─[~]:$ sudo service mysql start
[usaurio@linux]─[~]:$ lsof -i:3306                                                                                                                       
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
mariadbd 1588 mysql   19u  IPv4  23975      0t0  TCP localhost:mysql (LISTEN)
[usaurio@linux]─[~]:$ sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 35
MariaDB [(none)]> exit;
Bye
[usaurio@linux]─[~]:$
```

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
MariaDB [pruebas]> DELETE FROM bandas WHERE nombre='Maluma';
```
- Crear columna, (todos los valores se pondran en NULL); Borrarla
```sql
MariaDB [pruebas]> ALTER TABLE peliculas ADD duracion int(3);
MariaDB [pruebas]> ALTER TABLE peliculas DROP COLUMN duracion;
MariaDB [pruebas]> ALTER TABLE peliculas RENAME nombre TO titulo;
```
----------------------------------------------------------------------

## Consultar datos 

> En SQL una base de datos se le conoce tabmien como table_schema

- **SHOW**
```sql
MariaDB [pruebas]> show databases;
prueba, information_schema, mysql, performance_schema
MariaDB [pruebas]> SHOW TABLES FROM pruebas; 
bandas, peliculas
MariaDB [pruebas]> SHOW COLUMNS FROM peliculas; 
id, nombre, año, pais
```
- **SELECT**

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
- **UNION SELECT** -> conbinar varias tablas 

```sql
MariaDB [pruebas]> SELECT nombre FROM peliculas WHERE pais='Suecia' UNION SELECT nombre from bandas;
Millenium, Nirvana, Scorpions, Metallica
MariaDB [pruebas]> SELECT id,nombre FROM peliculas WHERE pais='Suecia' UNION SELECT id,nombre from bandas;
+----+-----------+
| id | nombre    |
+----+-----------+
|  3 | Millenium |
|  1 | Nirvana   |
|  2 | Scorpions |
|  3 | Metallica |
+----+-----------+
```
- **LIMIT** -> por cual empieza, cuantos resultados  
```sql
MariaDB [pruebas]> SELECT nombre FROM peliculas LIMIT 2,3;
Millenium, Elysium, Jhonne Mnemonic
```
- **ORDER BY** -> ordena la columna especificada por orden alfabetico
```sql
MariaDB [pruebas]> SELECT nombre FROM peliculas ORDER BY nombre;
Blade Runner, Elyisum, Ghost in the shell, Jhonny Mnemonic, Milenium      
```
- **CONCAT** -> Une resultados de dos columnas en una sola, pero esta pegado, para separarlo por ":" se po e 0x3a (: en hexadecimal)
```sql
MariaDB [pruebas]> SELECT concat(nombre,0x3a,pais) FROM peliculas ORDER BY nombre;
Blade Runner:EE.UU, Elyisum:EE.UU, Ghost in the shell:Japon, Jhonny Mnemonic:EE.UU, Milenium:Suecia   
```
 - Subconsultas
```sql
MariaDB [pruebas]> SELECT nombre FROM peliculas WHERE año>(SELECT AVG(año) FROM peliculas);
Millenium, Elysium
```
- **DISTINCT** 
```sql
MariaDB [pruebas]> SELECT DISTINCT pais FROM peliculas;
EE.UU, Japón, Suecia
```

----------------------------------------------------------------------

## Funciones

```sql
MariaDB [pruebas]> SELECT UPPER(concat(nombre,0x3a,pais)) FROM peliculas ORDER BY nombre;
BLADE RUNNER:EE.UU, ELYISUM:EE.UU, GHOST IN THE SHELL:JAPON, JHONNY MNEMONIC:EE.UU, MILENIUM:SUECIA   
MariaDB [pruebas]> SELECT LOWER(nombre) FROM peliculas ORDER BY nombre LIMIT 2,1;
ghost in the shell
MariaDB [pruebas]> SELECT MIN(año) FROM peliculas UNION SELECT MAX(año) FROM peliculas UNION SELECT AVG(año) FROM peliculas;
1984, 2013, 1998
MariaDB [pruebas]> SELECT DISTINCT UPPER(substr(pais,1,2)) FROM peliculas;
EE, JA, SU
```

----------------------------------------------------------------------

## Tabla information schema

Es una database especial que contiene los nombres de tablas, bases de datos y columnas del sistema

```sql
MariaDB [pruebas]> SELECT schema_name FROM information_schema.schemata;
information_schema, mysql, performance_schema, ejercicios
MariaDB [pruebas]> SELECT table_name FROM information_schema.tables WHERE table_schema='prueba';
peliculas, bandas
MariaDB [pruebas]> SELECT column_name FROM information_schema.columns WHERE table_schema='prueba' AND table_name='peliculas';
id, nombre, año, pais, 
```
----------------------------------------------------------------------

## Los archivos sql

- Exportar
```console
[usaurio@linux]─[~]:$ sudo mysqldump prueba > ~/Documentos/ejercicios.sql
[usaurio@linux]─[~]:$ ls
ejercicios.sql
```
- Importar 
```console
[usaurio@linux]─[~]:$ sudo mysql
MariaDB [(none)]> show databases;
information_schema, mysql, performance_schema
MariaDB [(none)]> create database prueba;
MariaDB [(none)]> exit
[usaurio@linux]─[~]:$sudo mysqldump prueba < ~/Documentos/ejercicios.sql
-- Dump completed on 2022-05-09 12:33:36
[usaurio@linux]─[~]:$ sudo mysql
MariaDB [(none)]> show databases;
information_schema, mysql, performance_schema, prueba
```






