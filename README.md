Alumno 2:

ORACLE:

### 1. Establece que los objetos que se creen en el TS1 (creado por Alumno 1) tengan un tamaño inicial de 200K, y que cada extensión sea del doble del tamaño que la anterior. El número máximo de extensiones debe ser de 3.
Vamos ahora a crear que tenga sus objetos un tamaño inicial de 200K, primero lo apagamos para operar con él.

```
Alter tablespace TS1 offline;
```

```
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

```
SQL> SELECT tablespace_name, extent_management FROM dba_tablespaces where tablespace_name='SYSTEM';

TABLESPACE_NAME 	       EXTENT_MAN
------------------------------ ----------
SYSTEM			       LOCAL

```

	

### 2. Crea dos tablas en el tablespace recién creado e inserta un registro en cada una de ellas. Comprueba el espacio libre existente en el tablespace. Borra una de las tablas y comprueba si ha aumentado el espacio disponible en el tablespace. Explica la razón.

       
### 3. Convierte a TS1 en un tablespace de sólo lectura. Intenta insertar registros en la tabla existente. ¿Qué ocurre?. Intenta ahora borrar la tabla. ¿Qué ocurre? ¿Porqué crees que pasa eso?
       
### 4. Crea un espacio de tablas TS2 con dos ficheros en rutas diferentes de 1M cada uno no autoextensibles. Crea en el citado tablespace una tabla con la clausula de almacenamiento que quieras. Inserta registros hasta que se llene el tablespace. ¿Qué ocurre?

### 5. Hacer un procedimiento llamado MostrarUsuariosporTablespace que muestre por pantalla un listado de los tablespaces existentes con la lista de usuarios que tienen asignado cada uno de ellos por defecto y el número de los mismos, así:
```
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



```
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

```
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

```	
Create or replace procedure BuscarTabla(p_tabla varchar2)
is
	v_cuenta number: =0;
begin
	Select count(*) into v_cuenta
	from dba_indexes
	where table_name=p_tabla;
	if v_cuenta = 0 then
		raise_application_error(-20001,'Esa tabla no existe');
	end if;
end;
/
```

```
Create or replace procedure MostrarDetallesIndices(p_tabla varchar2)
is
	cursor c_indices is
	Select INDEX_NAME,INDEX_TYPE,TABLE_OWNER
	from dba_indexes
	where table_name=upper(p_tabla);
begin
	BuscarTabla(upper(p_tabla));
	dbms_output.put_line('Indices');
	dbms_output.put_line('-------');
	for v_indices in c_indices loop
		dbms_output.put_line('Nombre: '||v_indices.INDEX_NAME);
		dbms_output.put_line('Tipo: '||v_indices.INDEX_TYPE);
		dbms_output.put_line('Dueño: '||v_indices.TABLE_OWNER);
	end loop;
end;
/
```
       
Postgres:
       
### 7. Averigua si existe el concepto de segmento y el de extensión en Postgres, en qué consiste y las diferencias con los conceptos correspondientes de ORACLE.
       
MySQL:

### 8. Averigua si existe el concepto de espacio de tablas en MySQL y las diferencias con los tablespaces de ORACLE.

MongoDB:

### 9. Averigua si existe la posibilidad en MongoDB de decidir en qué archivo se almacena una colección.