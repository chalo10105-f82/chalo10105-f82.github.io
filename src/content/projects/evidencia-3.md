---
title: "Evidencia   Ejercicio 3"
description: "Proyectar orden jerarquico de los cargos de los empleados, mostrar el nombre del empelado y sus jefes y extraer emails de los dos (Las primeras 3 letras, Luego rellenar 6 asteriscos a la izquierda)"
publishDate: 2026-02-14
isFeatured: true
---

```sql
SELECT
    LPAD(' ', LEVEL * 2) || INITCAP(e.FIRST_NAME) || ' ' || INITCAP(e.LAST_NAME) AS empleado,
    INITCAP(m.FIRST_NAME) || ' ' || INITCAP(m.LAST_NAME) AS jefe,
    RPAD('*', LENGTH(e.EMAIL) - 3, '*') || SUBSTR(e.EMAIL, -3) AS email_empleado,
    CASE
        WHEN m.EMAIL IS NOT NULL THEN
            RPAD('*', LENGTH(m.EMAIL) - 3, '*') || SUBSTR(m.EMAIL, -3)
        ELSE NULL
    END AS email_jefe
FROM HR.EMPLOYEES e
LEFT JOIN HR.EMPLOYEES m
    ON e.MANAGER_ID = m.EMPLOYEE_ID
START WITH e.MANAGER_ID IS NULL
CONNECT BY PRIOR e.EMPLOYEE_ID = e.MANAGER_ID
ORDER SIBLINGS BY e.EMPLOYEE_ID;
