---
title: "Evidencia Clase 16 de febrero"
publishDate: 2026-04-28
isFeatured: true
---

```sql

--- SQL lenguaje estructurado de consultas propio de las bases de datos relacionales.
--- PL/SQL SI es un LENGUAJE DE PROGRAMACION --> Solo usarlo cuando se agoten las opciones SQL.
--- APP <---SENTENCIA SQL (Hablando de las relacionales)---> BASE DE DATOS.
--- SQL Developer -----String-----> Base De Datos. 
-----------------------------------------------------------------------------------------------------------------------------------
---PL SQL-> Lenguaje procedimental, Propiedad de ORACLE, unidades de programa: programas que nosotros vamos hacer.
---PL SQL-> No es sensible a masyusculas.
---PL SQL-> Unidades lexicas: grupo de caracteres (5): Delimitadores, Identificadores, Literales, Comentarios, Expresiones.
---PL SQL-> Delimitadores: Operadores aritmeticos, logicos y relacionales. EJM: Suma
---PL SQL-> Identificador: Constantes,Cursores, variables, subprograma, excepciones, paquetes. EJM: v_frame, c_percent
---PL SQL-> Literal: Valor de tipo numerico
---Comentarios: -- o /* varias lineas */
--- Tipos De Datos:
---Number: Almacena numeros enteros o de tipo flotante, virtualmente de cualquier longitud.
---Boolean: TRUE - FALSE
---Date: Fechas internamente como datos numericos.
--- DECLARE No es necesario en todos los casos
--- BEGIN Totalmente necesario
--- Punto y coma es buena practica y da mas organizacion y precision.

SET SERVEROUTPUT ON; ---> CADA VEZ QUE ENTRAMOS A LA BASE DE DATOS.

Declare 
fecha timestamp;
BEGIN 
SELECT sysdate INTO fecha FROM dual;
dbms_output.put_line('La fecha es: '||fecha);
END;
-------------------------------------------------------------------------
BEGIN
dbms_output.put_line('Hola mundo');
END;
/
--> Bloque anonimo porque no le hemos puesto nombre a ese programa.
------------------------------------------------------------------------
--- PREFIJO PARA LAS VARIABLES
--- vn_xxxxx ---> Forma correcta de crear una variable numerica.
--- vd_xxxx ---> Forma correcta de crear una variable de tipo Date.
--- vv_xxxx ---> Forma correcta de crear una variable de tipo Varchar.
--- vdo_xxx ----> Forma correcta de crear una variable de tipo double.
-------------------------------------------------------------------------
--- PREFIJO PARA LAS CONSTANTES
--- cn_xxx ---> Forma correcta de crear una variable numerica.
--- cd_xxx ---> Forma correcta de crear una variable de tipo Date.
--- cv_xxx ---> Forma correcta de crear una variable de tipo Varchar
--- cdo_xxx ----> Forma correcta de crear una variable de tipo double.
---------------------------------------------------------------------------
--- prefijo para los procedimientos almacenados.
--- sp_xxx
---------------------------------------------------------------------------
--- prefijo para los parametros
--- param_xxx
-----------------------------------------------------------------------------
DECLARE
    vv_miPrimeraVariable VARCHAR2(50);
BEGIN
    vv_miPrimeraVariable := 'Hola mundo';
dbms_output.put_line(vv_miPrimeraVariable);
END;
/
-----------------------------------------------------------------------------
DECLARE
    vv_miPrimeraVariable VARCHAR2(50) := 'Hola mundo';
BEGIN
dbms_output.put_line(vv_miPrimeraVariable);
END;
/
----------------------------------------------------------------------------
DECLARE
    vv_nombre   VARCHAR2(50);
    vv_apellido VARCHAR2(50);
BEGIN
    SELECT
        first_name,
        last_name
    INTO
        vv_nombre,
        vv_apellido  ---> insertar variables
    FROM
        hr.employees
    WHERE
        employee_id = 110;

    dbms_output.put_line('El nombre del empleado es: '
                         || vv_nombre
                         || ' '
                         || vv_apellido); --->Lo que va dentro de las comillas se llama LITERAL
END;
/
----------------------------------------------------------------------------------
DECLARE
    vv_nombre HR.EMPLOYEES.FIRST_NAME%TYPE;
    vv_apellido HR.EMPLOYEES.LAST_NAME%TYPE;
BEGIN
    SELECT
        first_name,
        last_name
    INTO
        vv_nombre,
        vv_apellido  ---> insertar variables
    FROM
        hr.employees
    WHERE
        employee_id = 110;

    dbms_output.put_line('El nombre del empleado es: '
                         || vv_nombre
                         || ' '
                         || vv_apellido); --->Lo que va dentro de las comillas se llama LITERAL
END;
/
----------------------------------------------------------------------------------
DECLARE
    vv_empleado HR.EMPLOYEES%ROWTYPE;
BEGIN
    SELECT
        *
    INTO
        vv_EMPLEADO
    FROM
        hr.employees
    WHERE
        employee_id = 110;

    dbms_output.put_line('El nombre del empleado es: '
                         || vv_empleado.FIRST_NAME
                         || ' '
                         || vv_empleado.LAST_NAME); --->Lo que va dentro de las comillas se llama LITERAL
END;
/
---------------------------------------------------------------------------------------
DECLARE
    vv_empleado HR.EMPLOYEES%ROWTYPE;
BEGIN

    SELECT
        *
    INTO
        vv_EMPLEADO
    FROM
        hr.employees
    WHERE
        employee_id = (110,108);---> ESTO ASI NO FUNKA 

    dbms_output.put_line('El nombre del empleado es: '
                         || vv_empleado.FIRST_NAME
                         || ' '
                         || vv_empleado.LAST_NAME); --->Lo que va dentro de las comillas se llama LITERAL
END;
/
---------------------------------------------------------------------------------------
--- TIPOS DE ATRIBUTOS IMPORTANTES
--- %TYPE: HEREDA EL TIPO DE DATO DE LA TABLA
--- &ROWTYPE: HEREDA TODOS LOS TIPOS DE DATOS DE LA TABLA
------------------------------------------------------------------------------------
--- Jamas olvidar diccionario de datos para ser organizado en todo tipo de trabajos
--- CONTROL + F7 HACE LA INDETACION
--- || ES PARA CONCATENAR
-----------------------------------------------------------------------------------


















