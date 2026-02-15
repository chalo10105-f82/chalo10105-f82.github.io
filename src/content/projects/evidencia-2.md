---
title: "Evidencia   Ejercicio 2"
description: "Identifique todos los empleados que vivan o trabajen en Europa y tengan rango entre un salario entre 4 mil dólares y 6 mil dólares .
Mostrar Columnas

 1.Nombre y apellidos una solo columna

 2.País al que pertenece

 3.Salario que tiene"
publishDate: 2026-02-14
isFeatured: true
---

```sql
SELECT
    INITCAP(E.FIRST_NAME || ' ' || E.LAST_NAME) AS "Nombre Completo",
    (C.COUNTRY_NAME)                      AS "Continente",
    E.SALARY                                   AS "Salario"
FROM HR.EMPLOYEES E
    JOIN HR.DEPARTMENTS D ON E.DEPARTMENT_ID = D.DEPARTMENT_ID
    JOIN HR.LOCATIONS L   ON D.LOCATION_ID = L.LOCATION_ID
    JOIN HR.COUNTRIES C   ON L.COUNTRY_ID = C.COUNTRY_ID
    JOIN HR.REGIONS R     ON C.REGION_ID = R.REGION_ID
WHERE R.REGION_NAME = 'Americas'
    AND E.SALARY BETWEEN 4000 AND 6000;
