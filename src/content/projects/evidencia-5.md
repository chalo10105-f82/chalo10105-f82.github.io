---
title: "Evidencia comparacion Cursor"
publishDate: 2026-04-28
isFeatured: true
---

```sql

SET SERVEROUTPUT ON;

DECLARE

  vv_total_nomina  NUMBER(12,2) := 0;
  vv_contador      NUMBER       := 0;
  vv_inicio        TIMESTAMP;
  vv_fin           TIMESTAMP;
  vv_segundos      NUMBER;


  vv_nombre_completo  VARCHAR2(50);
  vv_departamento     VARCHAR2(30);
  vv_cargo            VARCHAR2(35);
  vv_salario          NUMBER(8,2);


  CURSOR vv_cursor_empleados IS
    SELECT e.first_name || ' ' || e.last_name AS nombre_completo,
           d.department_name                   AS departamento,
           j.job_title                         AS cargo,
           e.salary                            AS salario
    FROM   employees   e,
           departments d,
           jobs        j
    WHERE  e.department_id = d.department_id
    AND    e.job_id        = j.job_id
    ORDER BY e.department_id, e.salary DESC;

BEGIN
  vv_inicio := SYSTIMESTAMP;


  OPEN vv_cursor_empleados;

  LOOP
    FETCH vv_cursor_empleados
      INTO vv_nombre_completo,
           vv_departamento,
           vv_cargo,
           vv_salario;

    EXIT WHEN vv_cursor_empleados%NOTFOUND;

    vv_contador     := vv_contador + 1;
    vv_total_nomina := vv_total_nomina + vv_salario;

    DBMS_OUTPUT.PUT_LINE(
      LPAD(vv_contador, 3)              || ' | ' ||
      RPAD(vv_nombre_completo, 22)      || ' | ' ||
      RPAD(vv_departamento, 20)         || ' | ' ||
      RPAD(vv_cargo, 30)                || ' | ' ||
      TO_CHAR(vv_salario, '999,999')
    );
  END LOOP;

  CLOSE vv_cursor_empleados;

  vv_fin      := SYSTIMESTAMP;
  vv_segundos := EXTRACT(SECOND FROM (vv_fin - vv_inicio));

  DBMS_OUTPUT.PUT_LINE('----------------------------------------');
  DBMS_OUTPUT.PUT_LINE('Total empleados : ' || vv_contador);
  DBMS_OUTPUT.PUT_LINE('Total nomina    : ' || TO_CHAR(vv_total_nomina, '999,999,999'));
  DBMS_OUTPUT.PUT_LINE('Tiempo (seg)    : ' || vv_segundos);

EXCEPTION
  WHEN OTHERS THEN
    IF vv_cursor_empleados%ISOPEN THEN
      CLOSE vv_cursor_empleados;
    END IF;
    DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/



---implicito
SET SERVEROUTPUT ON;

DECLARE
  vv_nombre_completo  VARCHAR2(50);
  vv_departamento     VARCHAR2(30);
  vv_cargo            VARCHAR2(35);
  vv_salario          NUMBER(8,2);
  vv_total_nomina     NUMBER(12,2) := 0;
  vv_contador         NUMBER       := 0;
  vv_inicio           TIMESTAMP;
  vv_fin              TIMESTAMP;
  vv_segundos         NUMBER;

BEGIN
  vv_inicio := SYSTIMESTAMP;

  FOR vv_reg IN (
    SELECT e.first_name || ' ' || e.last_name AS nombre_completo,
           d.department_name                   AS departamento,
           j.job_title                         AS cargo,
           e.salary                            AS salario
    FROM   employees   e,
           departments d,
           jobs        j
    WHERE  e.department_id = d.department_id
    AND    e.job_id        = j.job_id
    ORDER BY e.department_id, e.salary DESC
  ) LOOP

    vv_contador    := vv_contador + 1;
    vv_total_nomina := vv_total_nomina + vv_reg.salario;

    DBMS_OUTPUT.PUT_LINE(
      LPAD(vv_contador, 3)              || ' | ' ||
      RPAD(vv_reg.nombre_completo, 22)  || ' | ' ||
      RPAD(vv_reg.departamento, 20)     || ' | ' ||
      RPAD(vv_reg.cargo, 30)            || ' | ' ||
      TO_CHAR(vv_reg.salario, '999,999')
    );
  END LOOP;

  vv_fin      := SYSTIMESTAMP;
  vv_segundos := EXTRACT(SECOND FROM (vv_fin - vv_inicio));

  DBMS_OUTPUT.PUT_LINE('----------------------------------------');
  DBMS_OUTPUT.PUT_LINE('Total empleados : ' || vv_contador);
  DBMS_OUTPUT.PUT_LINE('Total nomina    : ' || TO_CHAR(vv_total_nomina, '999,999,999'));
  DBMS_OUTPUT.PUT_LINE('Tiempo (seg)    : ' || vv_segundos);

EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/