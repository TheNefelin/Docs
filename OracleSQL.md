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