---
title: "Evidencia Clase 23 de febrero"
publishDate: 2026-04-28
isFeatured: true
---

```sql

--- SUBPROGRAMAS ya viven dentro del motor de base de datos, se le asigna un nombre unico por esquema, a trabajar: (procedimientos almacendaos, funciones, triggers, subprogramas en bloques anonimos)
--- [corchetes cuadrados es opcional]
--- <Etiquetas>
--- obligatorio para la clase ponerle el nombre del procedimiento al END;
--- || significa excluyente en las presentaciones
--- procedimientos almacenados: Es como un "metodo" en java pero acà no devuelve ningun valor, devuelven paràmetros
--- valores quemados se les dice LITERALES
--- / -> Se pone al final para indicar "Pase al siguiente paso"

/*
Autor: Thomas Duarte Cano
Fecha: 23/02/2026
Descripcion: Este sp genera un texto saludando a alguien.
*/
--- DESPUES DE CADA CREATE, SELECT TODO

--- CREATE OR REPLACE
--- PROCEDURE Actualiza_saldo(cuenta Number, new_saldo Number)
--- IS
---     Declaracion de variables locales
--- BEGIN
---     Sentencias
--- UPDATE SALDOS_CUENTAS
--- SET SALDO = new_saldo,
--- FX

SET SERVEROUTPUT ON;

--- crar hola mundo pero recibiendo el parametro varchar2 y el nombre de una persona que admire

CREATE OR REPLACE PROCEDURE sp_mensaje (param_nombre IN VARCHAR2)
/*
Autor: Thomas Duarte Cano
Fecha: 23/02/2026
Descripcion: Este sp genera un texto saludando a alguien.
*/
IS
    vv_mensaje VARCHAR2(100);
BEGIN
    vv_mensaje := 'Hola mundo, el es campeon del mundo: ' || param_nombre || ' y cristiano llorando';
    dbms_output.put_line(vv_mensaje);
END sp_mensaje;
/

-------------------------------------------------------------------------------------
--Esto es para invocar el procedimiento almacenado

BEGIN
    sp_mensaje ('Messi'); 
END;
-----------------------------------------------------------------------------------

DROP PROCEDURE messi; --- borrar

--------------------------------------------------------------------------------------------------------------------
--- EJERCICIO CON VALOR EN DEFAULT 
CREATE OR REPLACE PROCEDURE sp_mensajeConDefault (param_nombre IN VARCHAR2 DEFAULT 'Messi')
/*
Autor: Thomas Duarte Cano
Fecha: 23/02/2026
Descripcion: Este sp genera un texto saludando a alguien usando valor en default.
*/
IS
    vv_mensaje VARCHAR2(100);
BEGIN
    vv_mensaje := 'Hola mundo, el es campeon del mundo: ' || param_nombre || ' y cristiano llorando';
    dbms_output.put_line(vv_mensaje);
END sp_mensajeConDefault;
/

BEGIN
    sp_mensajeConDefault (); 
END;
/
--- Si se pone algo en los parentesis se imprrime el valor de la invocacion
---------------------------------------------------------------------------------------------
SELECT NEXT_DAY (LAST_DAY(SYSDATE)-7,'VIERNES')---> ULTIMO VIERNES DEL MES
FROM DUAL;

SELECT NEXT_DAY (LAST_DAY(SYSDATE)-7,'VIERNES')---> ENCONTRAR LA FORMA DE EVALUARLO EN OTRO MES COMO SEPTIEMBRE.
FROM DUAL;


BEGIN
    sp_mensajeConDefaultYFecha (); 
END;
/











