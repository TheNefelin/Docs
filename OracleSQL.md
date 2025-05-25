# Oracle SQL

# PL/SQL

## Bloque Anonimo
* ver/salida de DBMS
* nueva ventana
```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hola Mundo');
END;
```
* varios bloques anonimos se agrega /
```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hola Mundo');
END;
/
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hola Dev');
END;
```

## Variables
```sql
DECLARE
    v_num NUMBER(2) := 10;
    v_cadena VARCHAR(10) := 'Francisco';
    v_fecha DATE := SYSDATE;
    v_fecha2 DATE := TO_DATE('20250303', 'YYYYMMDD');

BEGIN
    v_num := 15;
    DBMS_OUTPUT.PUT_LINE('El valor de v_num es: ' || v_num);
    DBMS_OUTPUT.PUT_LINE('El valor de v_cadena es: ' || v_cadena);
    DBMS_OUTPUT.PUT_LINE('El valor de v_fecha es: ' || v_fecha);     
    DBMS_OUTPUT.PUT_LINE('El valor de v_fecha2 es: ' || v_fecha2);
END;
```

## Asignar Valor a Variables por Usuario
```sql
DECLARE
    v_op1 NUMBER(2) := &operando1;
    v_op2 NUMBER(2) := &operando3;
    v_suma NUMBER(3);
    
BEGIN
    v_suma := v_op1 + v_op2;
    
    DBMS_OUTPUT.PUT_LINE(v_op1 || ' + ' || v_op2 || ' = ' || v_suma);
END;
```

## Operadores Logicos
```
<
>
<=
>=
=
<> o !=
```
## Operadores Relacionales
```
AND
OR
```

## Operaciones
```sql
DECLARE
    v_n1 NUMBER(2) := 10;
    v_n2 NUMBER(2) := 2;
    
BEGIN
    DBMS_OUTPUT.PUT_LINE('Suma: ' || (v_n1 + v_n2));
    DBMS_OUTPUT.PUT_LINE('Resta: ' || (v_n1 - v_n2));
    DBMS_OUTPUT.PUT_LINE('Multipicacion: ' || (v_n1 * v_n2));
    DBMS_OUTPUT.PUT_LINE('Division: ' || (v_n1 / v_n2));
    DBMS_OUTPUT.PUT_LINE('Potencia: ' || (v_n1 ** v_n2));
END;
```

## Condiciones
* IF
```sql
DECLARE
    v_n1 NUMBER(2) := 1;
    v_n2 NUMBER(2) := 2;
    v_n3 NUMBER(2) := 3;
    
BEGIN

    IF v_n1 >= v_n2 AND v_n1 >= v_n3 THEN
        DBMS_OUTPUT.PUT_LINE(v_n1 || ' Es el Mayor');
    ELSIF v_n2 >= v_n3 THEN
        DBMS_OUTPUT.PUT_LINE(v_n2 || ' Es el Mayor');
    ELSE
        DBMS_OUTPUT.PUT_LINE(v_n3 || ' Es el Mayor');
    END IF;

END;
```

## Case
```sql
DECLARE
    v_dia NUMBER(1) := &dia;
    
BEGIN

    CASE v_dia
        WHEN 1 THEN
            DBMS_OUTPUT.PUT_LINE(v_dia || 'Es Lunes');
        WHEN 2 THEN
            DBMS_OUTPUT.PUT_LINE(v_dia || 'Es Martes');
        WHEN 3 THEN
            DBMS_OUTPUT.PUT_LINE(v_dia || 'Es Miercoles');
        WHEN 4 THEN
            DBMS_OUTPUT.PUT_LINE(v_dia || 'Es Jueves');
        WHEN 5 THEN
            DBMS_OUTPUT.PUT_LINE(v_dia || 'Es Viernes');
        WHEN 6 THEN
            DBMS_OUTPUT.PUT_LINE(v_dia || 'Es Sabado');
        WHEN 7 THEN
            DBMS_OUTPUT.PUT_LINE(v_dia || 'Es Domingo');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Debes Introducir un Valor Entre 1 y 7');
    END CASE;
    
END;
```

## Bucles
```sql
DECLARE
    v_i NUMBER(2) := 1;
    
BEGIN

    DBMS_OUTPUT.PUT_LINE('While');
    WHILE (v_i <= 10)
    LOOP    
        DBMS_OUTPUT.PUT_LINE(v_i);
        v_i := v_i + 1;
    END LOOP;
    
END;

/
    
BEGIN

    DBMS_OUTPUT.PUT_LINE('For');
    FOR v_i IN 1..10
    LOOP    
        DBMS_OUTPUT.PUT_LINE(v_i);
    END LOOP;
 
    DBMS_OUTPUT.PUT_LINE('For Reverse');
    FOR v_i IN REVERSE 1..10
    LOOP    
        DBMS_OUTPUT.PUT_LINE(v_i);
    END LOOP;
    
END;

/

DECLARE
    v_i NUMBER(2) := 1;
    
BEGIN

    DBMS_OUTPUT.PUT_LINE('Loop');
    LOOP    
        DBMS_OUTPUT.PUT_LINE(v_i);
        EXIT WHEN v_i = 10;
        v_i := v_i + 1;
    END LOOP;
    
END;
```

## Arreglos
```sql
DECLARE
    TYPE alumnosarray IS VARRAY(3) OF VARCHAR2(20);
    v_alumnos alumnosarray := alumnosarray('Netero', 'Hisoka', 'Kuroro');
    
    TYPE notasarray IS VARRAY(3) OF NUMBER(2);
    v_notas notasarray := notasarray(10, 9, 8);
        
BEGIN
    
    FOR v_i IN 1..v_alumnos.COUNT 
    LOOP
        DBMS_OUTPUT.PUT_LINE('Alumno: ' || v_alumnos(v_i) || ', Notas: ' || v_notas(v_i));
    END LOOP;
    
END;
```

## Tablas
```sql
CREATE TABLE fabricante (
    codigo NUMBER PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL
);

CREATE TABLE producto (
    codigo NUMBER  PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    precio FLOAT NOT NULL,
    codigo_fabricante INT NOT NULL,
    CONSTRAINT fabr_prod_FK FOREIGN KEY (codigo_fabricante) REFERENCES fabricante(codigo)
);
```

```sql
INSERT INTO fabricante VALUES(1, 'Asus');
INSERT INTO fabricante VALUES(2, 'Lenovo');
INSERT INTO fabricante VALUES(3, 'Hewlett-Packard');
INSERT INTO fabricante VALUES(4, 'Samsung');
INSERT INTO fabricante VALUES(5, 'Seagate');
INSERT INTO fabricante VALUES(6, 'Crucial');
INSERT INTO fabricante VALUES(7, 'Gigabyte');
INSERT INTO fabricante VALUES(8, 'Huawei');
INSERT INTO fabricante VALUES(9, 'Xiaomi');
```

```sql
INSERT INTO producto VALUES(1, 'Disco duro SATA3 1TB', 86.99, 5);
INSERT INTO producto VALUES(2, 'Memoria RAM DDR4 8GB', 120, 6);
INSERT INTO producto VALUES(3, 'Disco SSD 1 TB', 150.99, 4);
INSERT INTO producto VALUES(4, 'GeForce GTX 1050Ti', 185, 7);
INSERT INTO producto VALUES(5, 'GeForce GTX 1080 Xtreme', 755, 6);
INSERT INTO producto VALUES(6, 'Monitor 24 LED Full HD', 202, 1);
INSERT INTO producto VALUES(7, 'Monitor 27 LED Full HD', 245.99, 1);
INSERT INTO producto VALUES(8, 'Portátil Yoga 520', 559, 2);
INSERT INTO producto VALUES(9, 'Portátil Ideapd 320', 444, 2);
INSERT INTO producto VALUES(10, 'Impresora HP Deskjet 3720', 59.99, 3);
INSERT INTO producto VALUES(11, 'Impresora HP Laserjet Pro M26nw', 180, 3);
```

## Select Into
```sql
DECLARE
    v_total NUMBER(8);
    v_suma_productos NUMBER(8,2);
    
BEGIN
    
    SELECT 
        COUNT(*), SUM(precio) 
    INTO 
        v_total,
        v_suma_productos
    FROM producto;
    
    DBMS_OUTPUT.PUT_LINE('Total de Productos: ' || v_total);
    DBMS_OUTPUT.PUT_LINE('Suma de Precios: ' || v_suma_productos);
    
END;
```

## Atributo Type
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo; 
    v_nombre producto.nombre%type;
    
BEGIN
    
    SELECT nombre INTO v_nombre
    FROM producto
    WHERE codigo = v_codigo;
    
    DBMS_OUTPUT.PUT_LINE('Nombre del Producto con codigo ' || v_codigo || ': ' || v_nombre);
    
END;
```

## Atributo RowType
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;
    v_producto producto%rowtype;
    
BEGIN
    
    SELECT * INTO v_producto
    FROM producto
    WHERE codigo = v_codigo;
    
    DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || v_codigo);
    DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
    DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
    DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);

END;
```

## Excepcioness
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;
    v_producto producto%rowtype;
    
BEGIN
    
    SELECT * INTO v_producto
    FROM producto
    WHERE codigo = v_codigo;
    
    DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || v_codigo);
    DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
    DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
    DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
 
EXCEPTION 
    WHEN no_data_found THEN
        DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || v_codigo);
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || v_codigo);
    
END;
```

## Excepcioness propias
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;
    v_producto producto%rowtype;
    
    limite_precio EXCEPTION;
    
BEGIN
    
    SELECT * INTO v_producto
    FROM producto
    WHERE codigo = v_codigo;
    
    IF v_producto.precio >= 100 THEN
        RAISE limite_precio;
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || v_codigo);
    DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
    DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
    DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
 
EXCEPTION 
    WHEN no_data_found THEN
        DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || v_codigo);
    WHEN limite_precio THEN
        DBMS_OUTPUT.PUT_LINE('Se ha sperado el Limite');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || v_codigo);
    
END;
```

## Excepcioness RAISE_APPLICATION_ERROR
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;
    v_producto producto%rowtype;
    
    limite_precio EXCEPTION;
    PRAGMA EXCEPTION_INIT (limite_precio, -20999);
    
BEGIN
    
    SELECT * INTO v_producto
    FROM producto
    WHERE codigo = v_codigo;
    
    IF v_producto.precio >= 100 THEN
        RAISE_APPLICATION_ERROR(-20999, 'Se ha sperado el Limite');
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || v_codigo);
    DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
    DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
    DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
 
EXCEPTION 
    WHEN no_data_found THEN
        DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || v_codigo);
    WHEN limite_precio THEN
        DBMS_OUTPUT.PUT_LINE(sqlcode);
        DBMS_OUTPUT.PUT_LINE(sqlerrm);
        
END;
```

## Procedimiento
```sql
CREATE OR REPLACE PROCEDURE infoProducto(p_codigo producto.codigo%type)
AS
    v_producto producto%rowtype;
BEGIN
    SELECT * INTO v_producto
    FROM producto
    WHERE codigo = p_codigo;
    
    IF v_producto.precio >= 100 THEN
        RAISE_APPLICATION_ERROR(-20999, 'Se ha sperado el Limite');
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || p_codigo);
    DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
    DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
    DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
EXCEPTION 
    WHEN no_data_found THEN
        DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || p_codigo);
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || p_codigo);   
END;
```
/
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;    
BEGIN
    infoProducto(v_codigo);
END;
```
/
```sql
EXECUTE infoProducto(2);
EXEC infoProducto(5);
```

## Funcion
```sql
CREATE OR REPLACE FUNCTION obtenerProducto(p_codigo producto.codigo%type)
RETURN producto%rowtype
AS
    v_producto producto%rowtype;
BEGIN
    SELECT * INTO v_producto
    FROM producto
    WHERE codigo = p_codigo;
    
    RETURN v_producto;
EXCEPTION 
    WHEN no_data_found THEN
        DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || p_codigo);
        RETURN NULL;
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || p_codigo);
        RETURN NULL;
END;
```
/
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;
    v_producto producto%rowtype;
BEGIN
    v_producto := obtenerProducto(v_codigo);
    
    IF v_producto.codigo IS NOT NULL THEN
        DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || v_codigo);
        DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
        DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
        DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
    END IF;
END;
```
```sql
DROP PROCEDURE infoProducto;
```

## Parametro de Entrada y Salida
```sql
CREATE OR REPLACE PROCEDURE infoProducto(p_codigo producto.codigo%type, p_producto OUT producto%rowtype)
AS
BEGIN
    SELECT * INTO p_producto
    FROM producto
    WHERE codigo = p_codigo;
EXCEPTION 
    WHEN no_data_found THEN
        DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || p_codigo);
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || p_codigo);
END;
```
/
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;
    v_producto producto%rowtype;
BEGIN
    infoProducto(v_codigo, v_producto);
    
    IF v_producto.codigo IS NOT NULL THEN
        DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || v_codigo);
        DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
        DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
        DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
    END IF;
END;
```
```sql
DROP FUNCTION obtenerProducto;
```

## Paquete
```sql
CREATE OR REPLACE PACKAGE productos AS
    PROCEDURE infoProducto(p_codigo producto.codigo%type);
    FUNCTION obtenerProducto(p_codigo producto.codigo%type) RETURN producto%rowtype;
END;
```
/
```sql
CREATE OR REPLACE PACKAGE BODY productos AS

    PROCEDURE infoProducto(p_codigo producto.codigo%type)
    AS
        v_producto producto%rowtype;
    BEGIN
        SELECT * INTO v_producto
        FROM producto
        WHERE codigo = p_codigo;
        
        DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || p_codigo);
        DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
        DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
        DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
    EXCEPTION 
        WHEN no_data_found THEN
            DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || p_codigo);
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('Error: ' || p_codigo);   
    END;
    
    FUNCTION obtenerProducto(p_codigo producto.codigo%type) RETURN producto%rowtype
    AS
        v_producto producto%rowtype;
    BEGIN
        SELECT * INTO v_producto
        FROM producto
        WHERE codigo = p_codigo;
    
        RETURN v_producto;
    EXCEPTION 
        WHEN no_data_found THEN
            DBMS_OUTPUT.PUT_LINE('No Existe el producto: ' || p_codigo);
            RETURN NULL;
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('Error: ' || p_codigo);
            RETURN NULL;        
    END;        

END;
```
/
```sql
DECLARE
    v_codigo producto.codigo%type := &codigo;
    v_producto producto%rowtype;
BEGIN
    DBMS_OUTPUT.PUT_LINE('PROCEDIMIENTO');
    productos.infoproducto(v_codigo);
    
    DBMS_OUTPUT.PUT_LINE('FUNCION'); 
    v_producto := productos.obtenerproducto(v_codigo);
    
    IF v_producto.codigo IS NOT NULL THEN
        DBMS_OUTPUT.PUT_LINE('Informacion del producto con codigo: ' || v_codigo);
        DBMS_OUTPUT.PUT_LINE('Nombre: ' || v_producto.nombre);
        DBMS_OUTPUT.PUT_LINE('Precio: ' || v_producto.precio);
        DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
    END IF;    
END;
```
```sql
DROP PACKAGE productos;
```

## Cursores
```sql
DECLARE
    CURSOR c_productos IS
        SELECT * 
        FROM producto
        ORDER BY precio;
        
    v_producto producto%rowtype;
BEGIN

    OPEN c_productos;
        LOOP
            FETCH c_productos INTO v_producto;
            EXIT WHEN c_productos%notfound;
            
            DBMS_OUTPUT.PUT_LINE('Informacion prodcuto con codigo ' || v_producto.codigo);
            DBMS_OUTPUT.PUT_LINE('nombre' || v_producto.nombre);
            DBMS_OUTPUT.PUT_LINE('Precio' || v_producto.precio);
            DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
            DBMS_OUTPUT.PUT_LINE('');            
        END LOOP;
    CLOSE c_productos;

END;
```

```sql
DECLARE
    CURSOR c_productos IS
        SELECT * 
        FROM producto
        ORDER BY precio;
        
    v_producto producto%rowtype;
    
    CURSOR c_productos_fabricante IS
        SELECT
            p.codigo,
            p.nombre,
            p.precio,
            f.nombre AS fabricante
        FROM producto p, fabricante f
        WHERE p.codigo_fabricante = f.codigo
        ORDER BY p.precio;
BEGIN

    DBMS_OUTPUT.PUT_LINE('Cursor c_productos');
    OPEN c_productos;
        LOOP
            FETCH c_productos INTO v_producto;
            EXIT WHEN c_productos%notfound;
            
            DBMS_OUTPUT.PUT_LINE('Informacion prodcuto con codigo ' || v_producto.codigo);
            DBMS_OUTPUT.PUT_LINE('nombre' || v_producto.nombre);
            DBMS_OUTPUT.PUT_LINE('Precio' || v_producto.precio);
            DBMS_OUTPUT.PUT_LINE('Fabricante: ' || v_producto.codigo_fabricante);
            DBMS_OUTPUT.PUT_LINE('');            
        END LOOP;
    CLOSE c_productos;

    DBMS_OUTPUT.PUT_LINE('Cursor c_productos_fabricante');
    FOR registro IN c_productos_fabricante 
        LOOP
            DBMS_OUTPUT.PUT_LINE('Informacion prodcuto con codigo ' || registro.codigo);
            DBMS_OUTPUT.PUT_LINE('nombre' || registro.nombre);
            DBMS_OUTPUT.PUT_LINE('Precio' || registro.precio);
            DBMS_OUTPUT.PUT_LINE('Fabricante: ' || registro.fabricante);
            DBMS_OUTPUT.PUT_LINE('');  
        END LOOP;
END;
```

```sql
DECLARE
    CURSOR c_productos_fabricante(p_cod_fab NUMBER) IS
        SELECT * 
        FROM producto
        WHERE codigo_fabricante = p_cod_fab
        ORDER BY precio;
        
    v_codigo_fabricante fabricante.codigo%type := &codigo;
    v_producto producto%rowtype;
    
BEGIN

    OPEN c_productos_fabricante(v_codigo_fabricante);
        LOOP
            FETCH c_productos_fabricante INTO v_producto;
            EXIT WHEN c_productos_fabricante%notfound;
            
            DBMS_OUTPUT.PUT_LINE('Informacion prodcuto con codigo ' || v_producto.codigo);
            DBMS_OUTPUT.PUT_LINE('nombre' || v_producto.nombre);
            DBMS_OUTPUT.PUT_LINE('Precio' || v_producto.precio);
            DBMS_OUTPUT.PUT_LINE(''); 
        END LOOP;
    CLOSE c_productos_fabricante;
END;
```

## Trigger
```sql
CREATE OR REPLACE TRIGGER estado_operacion_before
BEFORE INSERT OR UPDATE OR DELETE ON producto
BEGIN

    IF INSERTING THEN
        DBMS_OUTPUT.PUT_LINE('Insertando Productos...'); 
    ELSIF UPDATING THEN
        DBMS_OUTPUT.PUT_LINE('Actualizando Productos...'); 
    ELSIF DELETING THEN
        DBMS_OUTPUT.PUT_LINE('Eliminando Productos...'); 
    END IF;

END;

/

CREATE OR REPLACE TRIGGER estado_operacion_after
AFTER INSERT OR UPDATE OR DELETE ON producto
BEGIN

    IF INSERTING THEN
        DBMS_OUTPUT.PUT_LINE('Producto/s Insertando/s'); 
    ELSIF UPDATING THEN
        DBMS_OUTPUT.PUT_LINE('Producto/s Actualizando/s'); 
    ELSIF DELETING THEN
        DBMS_OUTPUT.PUT_LINE('Producto/s Eliminando/s'); 
    END IF;

END;
```
```sql
CREATE OR REPLACE TRIGGER validacion_producto
BEFORE INSERT OR UPDATE ON producto FOR EACH ROW
DECLARE
    v_num_fabricantes NUMBER(4);
BEGIN

    IF :NEW.precio < 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'El precio no puede ser negativo' );
    END IF;

    SELECT COUNT(*) INTO v_num_fabricantes
    FROM fabricante
    WHERE codigo = :NEW.codigo_fabricante;

    IF v_num_fabricantes = 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'El fabricante no Existe' );
    END IF;

END;

/

INSERT INTO producto (codigo, nombre, precio, codigo_fabricante)
VALUES (12, 'Nuevo producto', 100, 1);
```
```sql
CREATE OR REPLACE TRIGGER cambio_fabricante
BEFORE UPDATE OF codigo_fabricante ON producto FOR EACH ROW
BEGIN

    IF :NEW.codigo_fabricante <> :OLD.codigo_fabricante THEN
        :NEW.precio := :OLD.precio + 10;
    END IF;

END;

/

UPDATE producto
SET codigo_fabricante = 5
WHERE codigo = 1;
```
```
DELETE FROM producto WHERE codigo = 12;
```

## Stored Procedure
- Output object
```sql
CREATE OR REPLACE TYPE ResultObj AS OBJECT (
    IsSuccess NUMBER(1),
    Message VARCHAR2(255),
    Data VARCHAR2(255)
);
```
- SP Insert
```sql
CREATE OR REPLACE PROCEDURE sp_hunter_insert(
    p_name IN hunter.name%TYPE,
    p_age IN hunter.age%TYPE,
    p_origin IN hunter.origin%TYPE,
    p_result OUT ResultObj
)
AS
    v_id_hunter hunter.id_hunter%TYPE;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Execute sp_hunter_insert'); 

    IF p_name IS NULL OR p_age IS NULL OR p_origin IS NULL THEN
        p_result := ResultObj(0, 'Name, age, and origin are required', NULL);
        RETURN;
    END IF;
    
    IF p_age <= 0 THEN
        p_result := ResultObj(0, 'Age must be greater than 0', NULL);
        RETURN;
    END IF;    

    INSERT INTO hunter (name, age, origin)
    VALUES (p_name, p_age, p_origin)
    RETURNING id_hunter INTO v_id_hunter;
    
    COMMIT;
    
    p_result := ResultObj(1, 'Hunter inserted successfully', TO_CHAR(v_id_hunter));
EXCEPTION
    WHEN OTHERS THEN
        p_result := ResultObj(0, SQLERRM, NULL);
        ROLLBACK;    
END;

/

DECLARE
    v_result ResultObj;
BEGIN
    sp_hunter_insert('Prueba', 100, 'Chile', v_result);
    DBMS_OUTPUT.PUT_LINE('Resultado:');
    DBMS_OUTPUT.PUT_LINE('IsSucces: ' || v_result.IsSuccess);
    DBMS_OUTPUT.PUT_LINE('Message: ' || v_result.Message);
    DBMS_OUTPUT.PUT_LINE('Data: ' || v_result.Data);
END;
```
- SP Delete
```sql
CREATE OR REPLACE PROCEDURE sp_hunter_delete(
    p_id_hunter hunter.id_hunter%TYPE,
    p_result OUT ResultObj
)
AS
    v_count NUMBER;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Execute sp_hunter_delete');

    IF p_id_hunter IS NULL THEN
        p_result := ResultObj(0, 'Parameter id_hunter cannot be NULL', NULL);
        RETURN;
    END IF;

    SELECT COUNT(1) INTO v_count FROM hunter_nen WHERE id_hunter = p_id_hunter;
    IF v_count > 0 THEN
        p_result := ResultObj(0, 'This hunter has dependencies in hunter_nen', null);
        RETURN;
    END IF;

    SELECT COUNT(1) INTO v_count FROM hunter WHERE id_hunter = p_id_hunter;
    IF v_count = 0 THEN
        p_result := ResultObj(0, 'Hunter does not exist', NULL);
        RETURN;
    END IF;

    DELETE FROM hunter
    WHERE id_hunter = p_id_hunter;
    
    COMMIT;
    
    p_result := ResultObj(1, 'Hunter deleted successfully', null);
EXCEPTION
    WHEN OTHERS THEN
        p_result := ResultObj(0, SQLERRM, NULL);
        ROLLBACK;    
END;

/

DECLARE
    v_result ResultObj;
BEGIN
    sp_hunter_delete(NULL, v_result);
    DBMS_OUTPUT.PUT_LINE('Resultado:');
    DBMS_OUTPUT.PUT_LINE('IsSucces: ' || v_result.IsSuccess);
    DBMS_OUTPUT.PUT_LINE('Message: ' || v_result.Message);
    DBMS_OUTPUT.PUT_LINE('Data: ' || v_result.Data);
END;

/

DECLARE
    v_result ResultObj;
BEGIN
    sp_hunter_delete(99999, v_result);
    DBMS_OUTPUT.PUT_LINE('Resultado:');
    DBMS_OUTPUT.PUT_LINE('IsSucces: ' || v_result.IsSuccess);
    DBMS_OUTPUT.PUT_LINE('Message: ' || v_result.Message);
    DBMS_OUTPUT.PUT_LINE('Data: ' || v_result.Data);
END;
```
- SP Update
```sql
CREATE OR REPLACE PROCEDURE sp_hunter_update(
    p_id_hunter IN hunter.id_hunter%TYPE,
    p_name IN hunter.name%TYPE,
    p_age IN hunter.age%TYPE,
    p_origin IN hunter.origin%TYPE,
    p_result OUT ResultObj
)
AS
    v_count NUMBER;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Execute sp_hunter_update'); 

    IF p_id_hunter IS NULL OR p_name IS NULL OR p_age IS NULL OR p_origin IS NULL THEN
        p_result := ResultObj(0, 'Id_Hunter, name, age, and origin are required', NULL);
        RETURN;
    END IF;
    
    IF p_age <= 0 THEN
        p_result := ResultObj(0, 'Age must be greater than 0', NULL);
        RETURN;
    END IF;    

    SELECT COUNT(1) INTO v_count FROM hunter WHERE id_hunter = p_id_hunter;
    IF v_count = 0 THEN
        p_result := ResultObj(0, 'Hunter does not exist', NULL);
        RETURN;
    END IF;

    UPDATE hunter SET
        name = p_name,
        age = p_age,
        origin = p_origin
    WHERE id_hunter = p_id_hunter;

    COMMIT;
    
    p_result := ResultObj(1, 'Hunter updated successfully', TO_CHAR(p_id_hunter));
EXCEPTION
    WHEN OTHERS THEN
        p_result := ResultObj(0, SQLERRM, NULL);
        ROLLBACK;    
END;

/

DECLARE
    v_result ResultObj;
BEGIN
    sp_hunter_update(21, 'Netero', 150, 'Japon', v_result);
    DBMS_OUTPUT.PUT_LINE('Resultado:');
    DBMS_OUTPUT.PUT_LINE('IsSucces: ' || v_result.IsSuccess);
    DBMS_OUTPUT.PUT_LINE('Message: ' || v_result.Message);
    DBMS_OUTPUT.PUT_LINE('Data: ' || v_result.Data);
END;
```
- SP Get All
```sql
CREATE OR REPLACE PROCEDURE sp_hunter_get_all(
    p_cursor OUT SYS_REFCURSOR,
    p_result OUT ResultObj
)
AS
BEGIN
    DBMS_OUTPUT.PUT_LINE('Execute sp_hunter_get_all');

    OPEN p_cursor FOR
        SELECT id_hunter, name, age, origin FROM hunter ORDER BY id_hunter;

    p_result := ResultObj(1, 'All hunters retrieved successfully', NULL);
EXCEPTION
    WHEN OTHERS THEN
        p_result := ResultObj(0, SQLERRM, NULL);
        p_cursor := NULL;
END;

/

DECLARE
    v_cursor SYS_REFCURSOR;
    v_result ResultObj;
    v_id_hunter hunter.id_hunter%TYPE;
    v_name hunter.name%TYPE;
    v_age hunter.age%TYPE;
    v_origin hunter.origin%TYPE;
BEGIN
    sp_hunter_get_all(p_cursor => v_cursor, p_result => v_result);
    
    DBMS_OUTPUT.PUT_LINE('Resultado:');
    DBMS_OUTPUT.PUT_LINE('IsSucces: ' || v_result.IsSuccess);
    DBMS_OUTPUT.PUT_LINE('Message: ' || v_result.Message);
    DBMS_OUTPUT.PUT_LINE('Data: ' || v_result.Data);
    
    LOOP
        FETCH v_cursor INTO v_id_hunter, v_name, v_age, v_origin;
        EXIT WHEN v_cursor%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE('ID: ' || v_id_hunter || ', Name: ' || v_name || ', Age: ' || v_age || ', Origin: ' || v_origin);
    END LOOP;

    CLOSE v_cursor;
END;
```
- SP Get By Id
```sql
CREATE OR REPLACE PROCEDURE sp_hunter_get_by_id(
    p_id_hunter IN hunter.id_hunter%TYPE,
    p_cursor OUT SYS_REFCURSOR,
    p_result OUT ResultObj
)
AS
    v_count NUMBER;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Execute sp_hunter_get_by_id');

    IF p_id_hunter IS NULL THEN
        p_result := ResultObj(0, 'Id_Hunter is required', NULL);
        RETURN;
    END IF;

    SELECT COUNT(*) INTO v_count FROM hunter WHERE id_hunter = p_id_hunter;
    IF v_count = 0 THEN
        p_result := ResultObj(0, 'Hunter does not exist', NULL);
        RETURN;
    END IF;

    OPEN p_cursor FOR
        SELECT id_hunter, name, age, origin FROM hunter WHERE id_hunter = p_id_hunter;

    p_result := ResultObj(1, 'Hunter found', NULL);
EXCEPTION
    WHEN OTHERS THEN
        p_result := ResultObj(0, SQLERRM, NULL);
        p_cursor := NULL;
END;

/

DECLARE
    v_cursor SYS_REFCURSOR;
    v_result ResultObj;
    v_id_hunter hunter.id_hunter%TYPE := 1;  -- ID a buscar
    v_name hunter.name%TYPE;
    v_age hunter.age%TYPE;
    v_origin hunter.origin%TYPE;
BEGIN
    sp_hunter_get_by_id(p_id_hunter => v_id_hunter, p_cursor => v_cursor, p_result => v_result);
    
    DBMS_OUTPUT.PUT_LINE('Resultado:');
    DBMS_OUTPUT.PUT_LINE('IsSucces: ' || v_result.IsSuccess);
    DBMS_OUTPUT.PUT_LINE('Message: ' || v_result.Message);
    DBMS_OUTPUT.PUT_LINE('Data: ' || v_result.Data);
    
   LOOP
        FETCH v_cursor INTO v_id_hunter, v_name, v_age, v_origin;
        EXIT WHEN v_cursor%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE('ID: ' || v_id_hunter || ', Name: ' || v_name || ', Age: ' || v_age || ', Origin: ' || v_origin);
    END LOOP;

    CLOSE v_cursor;
END;
```
