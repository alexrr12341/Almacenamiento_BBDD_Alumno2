# Alumno 2:

## ORACLE:

### 1. Establece que los objetos que se creen en el TS1 (creado por Alumno 1) tengan un tamaño inicial de 200K, y que cada extensión sea del doble del tamaño que la anterior. El número máximo de extensiones debe ser de 3.
Vamos ahora a crear que tenga sus objetos un tamaño inicial de 200K, primero lo apagamos para operar con él.

```sql
Alter tablespace TS1 offline;
```

```sql
ALTER TABLESPACE TS1
DEFAULT STORAGE (
INITIAL 200K
NEXT 400K
PCTINCREASE 100
MINEXTENTS 1
MAXEXTENTS 3);

ALTER TABLESPACE TS1
*
ERROR en línea 1:
ORA-25143: la cláusula de almacenamiento por defecto no es compatible con la
política de asignación
```

A la hora de hacer el alter table nos salta un error debido a que el tablespace está por defecto hecho en local, no por diccionario, por lo que no podemos modificar las clausulas de almacenamiento.
Podemos observar que el tablespace system efectivamente está guardado en local.

```sql
SQL> SELECT tablespace_name, extent_management FROM dba_tablespaces where tablespace_name='SYSTEM';

TABLESPACE_NAME 	       EXTENT_MAN
------------------------------ ----------
SYSTEM			       LOCAL

```

	

### 2. Crea dos tablas en el tablespace recién creado e inserta un registro en cada una de ellas. Comprueba el espacio libre existente en el tablespace. Borra una de las tablas y comprueba si ha aumentado el espacio disponible en el tablespace. Explica la razón.

Primero vamos a poner el tablespace online
```sql
alter tablespace TS1 online;
```
Vamos a observar cual es el espacio libre del tablespace ahora que esta vacio.
![](/Tablespace1.png)

Ahora vamos a crear las dos tablas en el tablespace TS1
```sql
Create table Pruebin
(
	Prueba varchar2(20)
)
tablespace TS1;

Create table Pruebin2
(
	Prueba2 varchar2(20)
)
tablespace TS1;
```

Y vamos a insertar los datos en las dos tablas

```sql
insert into Pruebin values('Esto es una prueba');
insert into Pruebin2 values('Bytes se consumen');
```

Vamos a ver de vuelta si ha cambiado el espacio libre
![](/Tablespace2.png)

Vamos a borrar la tabla Pruebin y miraremos de vuelta a ver que ocurre
```sql
drop table Pruebin;
```

![](/Tablespace3.png)

Como podemos observar, se ha añadido una nueva línea de 65536 bytes, esto se debe a que en Oracle, los tablespaces se dividen en segmentos,cada segmento es un objeto del tablespace,por lo que esos bytes son los que se han liberado tras el drop de la tabla Pruebin.

### 3. Convierte a TS1 en un tablespace de sólo lectura. Intenta insertar registros en la tabla existente. ¿Qué ocurre?. Intenta ahora borrar la tabla. ¿Qué ocurre? ¿Porqué crees que pasa eso?

Vamos a convertir TS1 en un tablespace de lectura con este comando
```sql
alter tablespace TS1 read only;
```

Ahora vamos a intentar insertar un registro en la tabla Pruebin2
![](/Tablespace4.png)

Obviamente, no nos deja ya que una inserción de datos sería una escritura sobre el tablespace TS1.

Ahora vamos a intentar borrar la tabla Pruebin2.

![](/Tablespace5.png)

Esto es debido a que envia la información al diccionario de datos, y dicho tablespace que lo gestiona si tiene permisos de escritura, por lo que permite el borrado de la tabla.
       
### 4. Crea un espacio de tablas TS2 con dos ficheros en rutas diferentes de 1M cada uno no autoextensibles. Crea en el citado tablespace una tabla con la clausula de almacenamiento que quieras. Inserta registros hasta que se llene el tablespace. ¿Qué ocurre?

Vamos a crear el tablespace con los dos ficheros y no autoextensibles
```sql
Create tablespace TS2
Datafile '/home/oracle/TS2.dbf'
size 1M,
'/home/oracle/TS2-2.dbf'
size 1M
autoextend off;
```

Vamos ahora a crear una tabla con una clausula de almacenamiento por ejemplo una de Initial.
```sql
Create table Pruebon
(
	Prueba char(2000),
	Prueba2 char(2000),
	Prueba3 char(2000)
)
Storage
(
	Initial 10K
)
tablespace TS2;
```

En este caso usaremos el tipo de datos char ya que será el mas rápido para ocupar bytes, ya que siempre intentará rellenar todos los valores restantes

Ahora vamos a insertar varios datos hasta que superen los 2M.

![](/Tablespace6.png)

Cuando llega al límite de espacio, intenta extender el tablespace, pero como hemos deshabilitado esa opción, no nos deja insertar mas datos en el tablespace.

### 5. Hacer un procedimiento llamado MostrarUsuariosporTablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios que tienen asignado cada uno de ellos por defecto y el número de los mismos, así:
```sql
Tablespace xxxx:

	Usr1
	Usr2
	...

Total Usuarios Tablespace xxxx: n1

Tablespace yyyy:

	Usr1
	Usr2
	...

Total Usuarios Tablespace yyyy: n2
....
Total Usuarios BD: nn
```
No olvides incluir los tablespaces temporales y de undo.



```sql
Create or replace procedure Usuariosdeltablespace(p_tablespace varchar2,p_cuenta in out number)
is
	Cursor c_usuarios is
	Select username
	from dba_users
	where default_tablespace=p_tablespace;
	v_cuentausers number:=0;
begin
	for v_usuarios in c_usuarios loop
		dbms_output.put_line(v_usuarios.username);
		p_cuenta:=p_cuenta+1;
		v_cuentausers:=v_cuentausers+1;
	end loop;
	dbms_output.put_line('Total Usuarios Tablespace '||p_tablespace||': '||v_cuentausers);
end;
/
```

```sql
Create or replace procedure MostrarUsuariosporTablespace
is
	Cursor c_tablespaces is
	Select tablespace_name
	from dba_tablespaces;
	v_cuenta number:=0;
begin
	for v_tablespaces in c_tablespaces loop
		dbms_output.put_line('Tablespace '||v_tablespaces.tablespace_name||':');
		Usuariosdeltablespace(v_tablespaces.tablespace_name,v_cuenta);
	end loop;
	dbms_output.put_line('Total Usuarios BD: '||v_cuenta);
end;
/
```



### 6. Realiza un procedimiento llamado MostrarDetallesIndices que reciba el nombre de una tabla y muestre los detalles sobre los índices que hay definidos sobre las columnas de la misma.

```sql	
Create or replace procedure BuscarPropiedadesIndice(p_indice varchar2)
is
	cursor c_indicesinfo is
	Select INDEX_TYPE,TABLESPACE_NAME,UNIQUENESS,NUM_ROWS
	from dba_indexes
	where index_name=p_indice;
begin
	for v_indicesinfo in c_indicesinfo loop
		dbms_output.put_line('Tablespace: '||v_indicesinfo.TABLESPACE_NAME);
		dbms_output.put_line('Tipo de indice: '||v_indicesinfo.INDEX_TYPE);
		if v_indicesinfo.UNIQUENESS='UNIQUE' then
			dbms_output.put_line('Es un índice único');
		else
			dbms_output.put_line('No es un índice único');
		end if;
		dbms_output.put_line('Número de filas: '||v_indicesinfo.NUM_ROWS);
	end loop;
end;
/
```

```sql
Create or replace procedure BuscarTabla(p_tabla varchar2)
is
	v_cuenta number(1):=0;
begin
	Select count(*) into v_cuenta
	from dba_tables
	where table_name=p_tabla;
	if v_cuenta = 0 then
		raise_application_error(-20001,'No existe esa tabla');
	end if;
end;
/
```

```sql
Create or replace procedure MostrarDetallesIndices(p_tabla varchar2)
is
	cursor c_indices is
	Select INDEX_NAME,TABLE_OWNER,COLUMN_NAME
	from dba_ind_columns
	where table_name=upper(p_tabla);
begin
	BuscarTabla(upper(p_tabla));
	dbms_output.put_line('Indices');
	dbms_output.put_line('-------');
	for v_indices in c_indices loop
		dbms_output.put_line('Nombre: '||v_indices.INDEX_NAME);
		dbms_output.put_line('Dueño: '||v_indices.TABLE_OWNER);
		dbms_output.put_line('Columna: '||v_indices.COLUMN_NAME);
		BuscarPropiedadesIndice(v_indices.index_name);
		dbms_output.put_line(chr(10));
	end loop;
end;
/
```
       
## Postgres:
       
### 7. Averigua si existe el concepto de segmento y el de extensión en Postgres, en qué consiste y las diferencias con los conceptos correspondientes de ORACLE.
       
En Oracle, el concepto de segmento significa que es el espacio que reserva la base de datos en su datafile para que lo utilice sólo 1 objeto. Los objetos de la base de datos es simplemente un segmento dentro del datafile.
El uso efectivo de los segmentos requiere que el DBA conozca los objetos, que utiliza una aplicación, cómo los datos son introducidos en esos objetos y el modo en que serán recuperados.

El segmento de por sí está construido por extensiones, que son conjuntos de bloques en Oracle.
Una vez que una extensión en un segmento no puede almacenar más datos, el segmento obtendrá otra extensión.

En Postgres este concepto no existe como en Oracle.

Un tablespace es la asignación de un directorio, en el cual creará archivos para los segmentos. Cuando en postgres se crea un segmento, se crea un archivo de datos dentro del directorio asignado al tablespace. A este archivo no se le puede indicar el tamaño ni el nombre directamente, no es compartido por otras tablas.

Y respecto a la extensión no existe tal concepto en Postgres como lo conocemos en Oracle, solo como librerías o módulos que le agregan funcionalidades específicas (Se deben instalar con create extension)
La única referencia al sistema de almacenamiento es que la unidad mínima de almacenamiento se denomina pagina o bloque. Un bloque en postgres ocupa por defecto 8 kilobytes.

## MySQL:

### 8. Averigua si existe el concepto de espacio de tablas en MySQL y las diferencias con los tablespaces de ORACLE.

MariaDB tiene varios motores de almacenamiento, entre ellos está (MYISAM,InnoDB,etc.), en este caso, el mas conocido es InnoDB. Dicho motor nos permite controlar la lógica del almacenamiento físico y acceso a los datos, por lo que podemos editar nosotros dichas clausulas.

InnoDB tiene las siguientes características:
```
-Es transaccional
-Tiene fácil recuperación de datos
-Tiene rollback
-Los datos se guardan en discos
-Necesita bastante espacio de disco y bastante RAM
-Tiene restricciones de Foreign Key
```

La forma de crear un tablespace por InnoDB sería:

```sql
create tablespace {Nombre tablespace}
	add datafile '{Nombre de archivo}'
	use logfile group logfile_group
	[extent_size [=] extent_size]
	[initial_size [=] initial_size]
	[autoextend_size [=] autoextend_size]
	[max_size [=] max_size]
	[nodegroup [=] nodegroup_id]
	[wait]	
	[comment [=] comment_text]
	[engine [=] engine_name]
```

También están los motores de almacenamiento como myISAM que se utiliza para espacios de tablas que no requieran mucho espacio de disco, ram, etc. Es en sí mas simple que el InnoDB.

Y también están los motores tipo cluster como NDB.

La diferencia con Oracle es que este sólo contiene un tipo de motor de almacenamiento y están pensados para una sola base de datos, al contrario que MariaDB

## MongoDB:

### 9. Averigua si existe la posibilidad en MongoDB de decidir en qué archivo se almacena una colección.

En mongodb el almacenamiento se realiza en /var/lib/mongodb como podemos observar en el siguiente ejemplo
```
root@mongo:/var/lib/mongodb# ls
collection-0-1648952780993419213.wt   collection-2--6345544394620420526.wt  index-1-6127850418206265964.wt   index-5-1648952780993419213.wt  _mdb_catalog.wt  WiredTigerLAS.wt
collection-0-6127850418206265964.wt   collection-4-1648952780993419213.wt   index-1--6345544394620420526.wt  index-6-1648952780993419213.wt  mongod.lock      WiredTiger.lock
collection-0--6345544394620420526.wt  collection-7-1648952780993419213.wt   index-3-1648952780993419213.wt   index-8-1648952780993419213.wt  sizeStorer.wt    WiredTiger.turtle
collection-2-1648952780993419213.wt   diagnostic.data			    index-3-6127850418206265964.wt   index-9-1648952780993419213.wt  storage.bson     WiredTiger.wt
collection-2-6127850418206265964.wt   index-1-1648952780993419213.wt	    index-3--6345544394620420526.wt  journal			     WiredTiger
```

Si queremos que mongo utilice un nuevo path para almacenar una coleccion en otro sitio, debemos utilizar la herramienta mongod para que apunte las colecciones en otra parte

```
root@mongo:/home/vagrant# mkdir mongo

mongod --dbpath /home/vagrant/mongo --fork --logpath /home/vagrant/mongo/log
```


Ahora vamos a ver si se ha guardado la nueva información en la ruta

```
root@mongo:/home/vagrant/mongo# ls
collection-0-7741059389973239158.wt  diagnostic.data		     index-5-7741059389973239158.wt  log		      mongod.lock    WiredTiger        WiredTiger.turtle
collection-2-7741059389973239158.wt  index-1-7741059389973239158.wt  index-6-7741059389973239158.wt  log.2020-02-04T09-42-14  sizeStorer.wt  WiredTigerLAS.wt  WiredTiger.wt
collection-4-7741059389973239158.wt  index-3-7741059389973239158.wt  journal			     _mdb_catalog.wt	      storage.bson   WiredTiger.lock
```
