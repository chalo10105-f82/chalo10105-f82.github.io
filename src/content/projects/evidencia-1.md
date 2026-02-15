---
title: "Evidencia   Ejercicio 1"
description: "Cuantos empleados han pasado por mas de un cargo en la compañía"
isFeatured: true
---

```sql
SELECT CONCAT(e.first_name,' ' ,e.last_name) AS "Nombre Completo", 
       COUNT(jh.employee_id) + 1 AS "Total de Cargos" 
FROM hr.employees e 
JOIN hr.job_history jh ON e.employee_id = jh.employee_id 
GROUP BY 
       e.employee_id, 
       e.first_name, 
       e.last_name 
HAVING 
       COUNT(jh.employee_id) >= 1 
ORDER BY 
       "Total de Cargos" DESC;
