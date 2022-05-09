# Bases de datos SQL




## Crear base de datos

 - Crear la base de datos -> *create table nombre (columna1 tipo(bytes), columna2 tipo(bytes));*

**Tipos de datos** -> int (numero); float(decimal); double(numero grande); date(año-mes-dia); time(hora-minuto-segundo); datetime(date + time); 
char(string de tamaño fijo); varchar(string de tamaño especificado); blob(objetis grandes como fotos, etc); text(texto largo)

```sql
MariaDB [(none)]> create database pruebas;
MariaDB [(none)]> use pruebas;
Database changed
MariaDB [(none)]> CREATE TABLE peliculas(id int(2), nombre varchar(20), año varchar(4), pais varchar(10), PRIMARY KEY(id));  
```
- Meter entradas en la base de datos -> El numero de columnas tiene que ser el mismo que el de entradas, si no, se metera un valor predeterminado.
```sql
MariaDB [(none)]> INSERT INTO peliculas VALUES (1, 'Blade Runner', 1984, 'EE.UU');
MariaDB [(none)]> INSERT INTO peliculas (id, nombre, año, pais) VALUES (2, 'Ghost in the shell', 1995, 'Japon');
```
- Modificar entrada 
```sql
MariaDB [(none)]> INSERT INTO peliculas VALUES (3, 'Millenium', 2011, 'EE.UU');
MariaDB [(none)]> UPDATE peliculas SET pais='Suecia' WHERE id=3;
MariaDB [(none)]> INSERT INTO peliculas VALUES (4, 'Elysium', 1999, 'España');
MariaDB [(none)]> UPDATE peliculas SET pais='España', año= WHERE id=4;
```
