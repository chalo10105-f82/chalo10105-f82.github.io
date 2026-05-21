---
title: "Evidencia Clase 18 de febrero"
publishDate: 2026-04-28
isFeatured: true
---

```sql

--- Estructuras de control
--- En PL/SQL, Solo se tiene la estructura de control "if"

--- IF (expresion) THEN
        -- Instrucciones
---ELSIF (expresion) THEN
        -- Instrucciones
---ELSE
        --Instrucciones
---END IF;
---------------------------------------------------------------------------------------
--- GOTO ETIQUETAS.

---------------------------------------------------------------------------------------
---EJM:Calcule si el dia de hoy es primo o no, si es primo lance el mensaje "Es primo"

SET SERVEROUTPUT ON;
----------------------------------------------------------------------------------------
DECLARE
    vd_hoy INT;
BEGIN
    SELECT TO_NUMBER(TO_CHAR(SYSDATE, 'DD')) INTO vd_hoy FROM DUAL;
    IF vd_hoy = 2 THEN
        DBMS_OUTPUT.PUT_LINE(' ES PRIMO');
    ELSIF vd_hoy <= 1 OR MOD(vd_hoy, 2) = 0 THEN
        DBMS_OUTPUT.PUT_LINE(' NO ES PRIMO');
    ELSE
        DBMS_OUTPUT.PUT_LINE('HOLA PRIMO ');
    END IF;
END;
/
---------------------------------------------------------------------------------------------------------
DECLARE
    dia NUMBER := TO_NUMBER(TO_CHAR(SYSDATE,'DD'));
BEGIN
 
    IF dia IN (2,3,5,7,11,13,17,19,23,29,31) THEN
        GOTO primo;
    ELSE
        GOTO noprimo;
    END IF;
 
<<primo>>
    DBMS_OUTPUT.PUT_LINE('HOLA PRIMO');
    GOTO fin;
 
<<noprimo>>
    DBMS_OUTPUT.PUT_LINE('NO ES PRIMO');
 
<<fin>>
    NULL;
 
END;
/
---------------------------------------------------------------------------------------------------------
---IMPRIMIR LA SERIE FIBONACCI HASTA EL TOPE DE 100 CON EL LOOP.

DECLARE 
    vn_a NUMBER := 0;
    vn_b NUMBER := 1;
    vn_c NUMBER;
BEGIN
 
    loop 
    EXIT WHEN a > 100;
    DBMS_OUTPUT.PUT_LINE(a);
    vn_c := vn_a + vn_b;
    vn_a := vn_b;
    vn_b := vn_c;
    END LOOP;
END;
/
---------------------------------------------------------------------------------------------------------
---Minimo comun multiplo de un numero cualquiera
DECLARE
   a NUMBER := 12;
   b NUMBER := 18;
   mcd NUMBER;
   mcm NUMBER;
   x NUMBER;
   y NUMBER;
BEGIN
   x := a;
   y := b;
 
 
   WHILE y != 0 LOOP
      mcd := MOD(x, y);
      x := y;
      y := mcd;
   END LOOP;
 
   mcd := x;
 
 
   mcm := (a * b) / mcd;
 
   DBMS_OUTPUT.PUT_LINE('El MCM es: ' || mcm);
END;
/
---------------------------------------------------------------------------------------------------------
---Encontrar 2 numeros que al cuadrado den un numero x CON FOR

DECLARE
    vn_numerouno NUMBER := 64;
    vn_numeroraiz NUMBER := 0;
 
BEGIN
    FOR vn_numeroraiz IN 1..SQRT(vn_numerouno) LOOP
        IF ((vn_numeroraiz * vn_numeroraiz)= vn_numerouno) THEN 
            dbms_output.put_line(vn_numeroraiz);
        END IF;
    END LOOP;
END;
/

---------------------------------------------------------------------------------------------------------
---ESTRUCTURAS DE CONTROL
--- Iteradores o bluces
--- LOOP, WHILE, FOR
--- EL BUCLE LOOP, se repite tantas veces sea necesario hasta que se fuerza su salida con la instruccion EXIT
--- LOOP
---Instrucciones
---IF (expresion)THEN
---Instrucciones
---EXIT;
---END IF;
---END LOOP;

--- WHILE (expresion) LOOP 
---Instrucciones
---END LOOP;


---FOR

---BEGIN
---FOR X IN 1..50 LOOP
--- DBMS_OUTPUT.PUT_LINE(2*X);
---END LOOP;
---END;

---BEGIN
---FOR X IN REVERSE 1..50 LOOP
--- DBMS_OUTPUT.PUT_LINE(2*X);
---END LOOP;
---END;















