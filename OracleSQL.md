# Oracle SQL

# PL/SQL

## Bloque Anonimo
* ver/salida de DBMS
* nueva ventana
```
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hola Mundo');
END;
```
* varios bloques anonimos se agrega /
```
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hola Mundo');
END;
/
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hola Dev');
END;
```

## Variables
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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

```
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

```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
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
```
DECLARE
    v_codigo producto.codigo%type := &codigo;    
BEGIN
    infoProducto(v_codigo);
END;
```
/
```
EXECUTE infoProducto(2);
EXEC infoProducto(5);
```

## Funcion
```
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
```
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
```
DROP PROCEDURE infoProducto;
```

## Parametro de Entrada y Salida
```
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
```
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
```
DROP FUNCTION obtenerProducto;
```

## Paquete
```
CREATE OR REPLACE PACKAGE productos AS
    PROCEDURE infoProducto(p_codigo producto.codigo%type);
    FUNCTION obtenerProducto(p_codigo producto.codigo%type) RETURN producto%rowtype;
END;
```
/
```
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
```
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
```
DROP PACKAGE productos;
```

## Cursores
```
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

```
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

```
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
```
```