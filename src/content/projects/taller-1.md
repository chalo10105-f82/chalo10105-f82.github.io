---
title: "Evidencia Taller 1 Completo"
description: "Contexto del caso La empresa quiere aplicar un ajuste salarial extraordinario. No se puede hacer de forma masiva sin revisar reglas de negocio.El objetivo es intervenir solo a los empleados correctos y dejar evidencia."
publishDate: 2026-04-28
isFeatured: true
---

```sql
-- =====================================================================
-- 03_template_entrega_taller1_v2.sql
-- Taller aplicado 1 - SQL avanzado + Transacciones (ACID) aplicado
-- Plantilla de entrega para estudiantes
--
-- IMPORTANTE:
-- 1. Trabajar únicamente sobre las tablas T1_% y AUDIT_SALARY_ADJUSTMENTS_T1
-- 2. NO modificar la estructura del entorno entregado por el docente
-- 3. NO eliminar secciones de esta plantilla
-- 4. Reemplazar únicamente los bloques indicados como "ESCRIBA AQUÍ"
-- 5. Usar la variante asignada por el docente (1, 2, 3 o 4)
-- 6. Usar un tag único de ejecución final, por ejemplo: P03_FINAL
-- =====================================================================

SET SERVEROUTPUT ON
SET FEEDBACK ON

-- ============================================================
-- 0. ENCABEZADO OBLIGATORIO
-- Complete toda esta información antes de ejecutar el script.
-- ============================================================
-- Integrante 1: Gonzalo Ayala Almeciga
-- Integrante 2: David millan castaneda 
-- Curso: bases de datos 2
-- Fecha: 08/04/2026
-- Variante asignada por el docente (1, 2, 3 o 4): 4
-- Tag de ejecución final (ejemplo: P03_FINAL): MillanGonzalo_FINAL

DEFINE p_variant_id = 4
DEFINE p_execution_tag = 'MillanGonzalo_FINAL'

PROMPT ===== 0. VERIFICACIÓN DE LA VARIANTE ASIGNADA =====
SELECT
    variant_id,
    variant_name,
    excluded_department_id,
    min_years_service,
    recent_job_history_months,
    gap_high_threshold_pct,
    gap_mid_threshold_pct,
    raise_high_pct,
    raise_mid_pct,
    raise_low_pct,
    max_salary_vs_avg_pct,
    notes
FROM t1_variants
WHERE variant_id = &p_variant_id;

-- ============================================================
-- GUÍA RÁPIDA DE OBJETOS DISPONIBLES
-- Use estos nombres reales de tablas y columnas.
-- ============================================================
-- Tabla principal de empleados: T1_EMPLOYEES
-- Columnas más importantes:
--   employee_id, first_name, last_name, email, phone_number,
--   hire_date, job_id, salary, commission_pct, manager_id, department_id
--
-- Tabla de departamentos: T1_DEPARTMENTS
-- Columnas más importantes:
--   department_id, department_name, manager_id, location_id
--
-- Tabla de historial laboral: T1_JOB_HISTORY
-- Columnas más importantes:
--   employee_id, start_date, end_date, job_id, department_id
--
-- Tabla de auditoría: AUDIT_SALARY_ADJUSTMENTS_T1
-- Columnas:
--   audit_id, execution_tag, variant_id, employee_id, department_id,
--   salary_before, salary_after, pct_gap_to_avg_before, rule_applied,
--   executed_by, executed_at, notes
--
-- Tabla de variantes: T1_VARIANTS
-- Columnas:
--   variant_id, variant_name, excluded_department_id, min_years_service,
--   recent_job_history_months, gap_high_threshold_pct,
--   gap_mid_threshold_pct, raise_high_pct, raise_mid_pct,
--   raise_low_pct, max_salary_vs_avg_pct, notes

-- ============================================================
-- GUÍA RÁPIDA DE TÉRMINOS QUE DEBE USAR EN SU SOLUCIÓN
-- ============================================================
-- CTE:
--   Una CTE es una consulta temporal escrita con WITH.
--   Sirve para dividir una consulta grande en partes más claras.
--
--   Ejemplo:
--   WITH dept_stats AS (
--       SELECT department_id, AVG(salary) avg_salary
--       FROM t1_employees
--       GROUP BY department_id
--   )
--   SELECT *
--   FROM dept_stats;
--
-- Función analítica:
--   Es una función como ROW_NUMBER, RANK o DENSE_RANK.
--   Sirve para calcular posiciones o comparaciones sin perder el detalle.
--
--   Ejemplo:
--   DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC)
--
-- JOIN:
--   Es la unión entre tablas relacionadas, por ejemplo empleados y departamentos.
--
-- Subconsulta:
--   Es una consulta dentro de otra consulta.
--
-- SAVEPOINT:
--   Es un punto de restauración dentro de una transacción.
--   Permite devolver la operación a un punto intermedio con ROLLBACK TO.

-- ============================================================
-- 1. CONSULTA DIAGNÓSTICA
-- OBJETIVO:
-- Analizar la información antes de actualizar salarios.
--
-- SU CONSULTA DEBE MOSTRAR, COMO MÍNIMO, ESTAS COLUMNAS:
--   employee_id
--   first_name
--   last_name
--   job_id
--   manager_id
--   department_id
--   department_name
--   salary
--   hire_date
--   years_service
--   dept_avg_salary
--   dept_max_salary
--   dept_employee_count
--   pct_gap_to_avg
--   recent_job_history_flag
--   salary_rank_in_department
--
-- QUÉ SIGNIFICA CADA COLUMNA:
--   years_service: años de antigüedad del empleado
--   dept_avg_salary: promedio salarial del departamento
--   dept_max_salary: salario más alto del departamento
--   dept_employee_count: cantidad de empleados del departamento
--   pct_gap_to_avg: porcentaje que le falta al salario del empleado para llegar
--                   al promedio del departamento
--   recent_job_history_flag: SI o NO, según si tuvo historial reciente
--   salary_rank_in_department: posición salarial dentro del departamento
--
-- IMPORTANTE:
-- - Puede usar una o varias CTE
-- - Debe usar al menos una función analítica
-- - Debe unir como mínimo T1_EMPLOYEES con T1_DEPARTMENTS
-- - Debe revisar T1_JOB_HISTORY para detectar historial reciente
-- ============================================================

PROMPT ===== 1. CONSULTA DIAGNÓSTICA =====

-- ESCRIBA AQUÍ SU CONSULTA DIAGNÓSTICA PRINCIPAL

WITH v AS (
    SELECT *
    FROM t1_variants
    WHERE variant_id = &p_variant_id
),
dept_stats AS (
    SELECT
        e.department_id,
        AVG(e.salary) AS dept_avg_salary,
        MAX(e.salary) AS dept_max_salary,
        COUNT(*) AS dept_employee_count
    FROM t1_employees e
    GROUP BY e.department_id
),
recent_history AS (
    SELECT DISTINCT j.employee_id
    FROM t1_job_history j
    CROSS JOIN v
    WHERE j.end_date >= ADD_MONTHS(TRUNC(SYSDATE), -v.recent_job_history_months)
),
diagnostic_base AS (
    SELECT
        e.employee_id,
        e.first_name,
        e.last_name,
        e.job_id,
        e.manager_id,
        e.department_id,
        d.department_name,
        e.salary,
        e.hire_date,
        ROUND(MONTHS_BETWEEN(TRUNC(SYSDATE), e.hire_date) / 12, 2) AS years_service,
        ROUND(ds.dept_avg_salary, 2) AS dept_avg_salary,
        ds.dept_max_salary,
        ds.dept_employee_count,
        ROUND(
            CASE
                WHEN ds.dept_avg_salary > 0 THEN
                    ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100
                ELSE 0
            END
        , 2) AS pct_gap_to_avg,
        CASE
            WHEN rh.employee_id IS NOT NULL THEN 'SI'
            ELSE 'NO'
        END AS recent_job_history_flag,
        DENSE_RANK() OVER (
            PARTITION BY e.department_id
            ORDER BY e.salary DESC
        ) AS salary_rank_in_department
    FROM t1_employees e
    LEFT JOIN t1_departments d
        ON e.department_id = d.department_id
    LEFT JOIN dept_stats ds
        ON e.department_id = ds.department_id
    LEFT JOIN recent_history rh
        ON e.employee_id = rh.employee_id
)
SELECT
    employee_id,
    first_name,
    last_name,
    job_id,
    manager_id,
    department_id,
    department_name,
    salary,
    hire_date,
    years_service,
    dept_avg_salary,
    dept_max_salary,
    dept_employee_count,
    pct_gap_to_avg,
    recent_job_history_flag,
    salary_rank_in_department
FROM diagnostic_base
ORDER BY department_id, salary_rank_in_department, employee_id;
-- Debe devolver las columnas mínimas exigidas arriba.



-- COMENTARIO OBLIGATORIO:
-- Esta consulta diagnóstica consolida la información laboral y salarial de cada
-- empleado antes del ajuste, relacionándola con su departamento, antigüedad,
-- historial reciente y posición salarial interna. Con esto se identifican
-- diferencias frente al promedio del área y se obtiene una base objetiva para
-- decidir quién podría ser elegible y quién debe ser excluido.

-- Explique en 3 a 5 líneas qué demuestra su consulta diagnóstica y por qué
-- le sirve para decidir qué empleados pueden ser elegibles.

-- ============================================================
-- 2. DECISIÓN DE POBLACIÓN ELEGIBLE
-- OBJETIVO:
-- Determinar qué empleados sí califican, cuáles no califican y por qué.
--
-- SU CONSULTA DEBE MOSTRAR, COMO MÍNIMO, ESTAS COLUMNAS:
--   employee_id
--   first_name
--   last_name
--   department_id
--   department_name
--   salary
--   years_service
--   dept_avg_salary
--   dept_max_salary
--   dept_employee_count
--   pct_gap_to_avg
--   recent_job_history_flag
--   manager_or_exec_flag
--   eligibility_flag
--   exclusion_reason
--   adjustment_pct
--   rule_applied
--
-- QUÉ SIGNIFICA CADA COLUMNA:
--   manager_or_exec_flag: SI o NO, según si es gerente principal o alta dirección
--   eligibility_flag: ELEGIBLE o NO_ELEGIBLE
--   exclusion_reason: motivo de exclusión, por ejemplo:
--                     SIN_DEPARTAMENTO, HISTORIAL_RECIENTE,
--                     ANTIGUEDAD_INSUFICIENTE, MANAGER_O_DIRECTIVO,
--                     DEPTO_EXCLUIDO, DEPTO_MENOR_A_3, SALARIO_NO_APLICA
--   adjustment_pct: porcentaje de ajuste que le corresponde
--   rule_applied: regla aplicada, por ejemplo AJUSTE_ALTO, AJUSTE_MEDIO, AJUSTE_BAJO
--
-- IMPORTANTE:
-- - Debe tomar en cuenta la variante asignada por el docente
-- - Debe usar los valores de T1_VARIANTS según &p_variant_id
-- - Debe quedar visible por qué una persona sí o no entra al proceso
-- ============================================================

PROMPT ===== 2. DECISIÓN DE ELEGIBLES =====

-- ESCRIBA AQUÍ SU CONSULTA DE DECISIÓN DE ELEGIBLES
WITH v AS (
    SELECT *
    FROM t1_variants
    WHERE variant_id = &p_variant_id
),
dept_stats AS (
    SELECT
        e.department_id,
        AVG(e.salary) AS dept_avg_salary,
        MAX(e.salary) AS dept_max_salary,
        COUNT(*) AS dept_employee_count
    FROM t1_employees e
    GROUP BY e.department_id
),
recent_history AS (
    SELECT DISTINCT j.employee_id
    FROM t1_job_history j
    CROSS JOIN v
    WHERE j.end_date >= ADD_MONTHS(TRUNC(SYSDATE), -v.recent_job_history_months)
),
manager_list AS (
    SELECT DISTINCT manager_id AS employee_id
    FROM t1_employees
    WHERE manager_id IS NOT NULL
),
diagnostic_base AS (
    SELECT
        e.employee_id,
        e.first_name,
        e.last_name,
        e.job_id,
        e.manager_id,
        e.department_id,
        d.department_name,
        e.salary,
        e.hire_date,
        ROUND(MONTHS_BETWEEN(TRUNC(SYSDATE), e.hire_date) / 12, 2) AS years_service,
        ROUND(ds.dept_avg_salary, 2) AS dept_avg_salary,
        ds.dept_max_salary,
        ds.dept_employee_count,
        ROUND(
            CASE
                WHEN ds.dept_avg_salary > 0 THEN
                    ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100
                ELSE 0
            END
        , 2) AS pct_gap_to_avg,
        CASE
            WHEN rh.employee_id IS NOT NULL THEN 'SI'
            ELSE 'NO'
        END AS recent_job_history_flag,
        CASE
            WHEN ml.employee_id IS NOT NULL
                 OR e.job_id LIKE '%MAN%'
                 OR e.job_id LIKE '%MGR%'
                 OR e.job_id LIKE 'AD_%'
            THEN 'SI'
            ELSE 'NO'
        END AS manager_or_exec_flag
    FROM t1_employees e
    LEFT JOIN t1_departments d
        ON e.department_id = d.department_id
    LEFT JOIN dept_stats ds
        ON e.department_id = ds.department_id
    LEFT JOIN recent_history rh
        ON e.employee_id = rh.employee_id
    LEFT JOIN manager_list ml
        ON e.employee_id = ml.employee_id
),
eligibility AS (
    SELECT
        db.employee_id,
        db.first_name,
        db.last_name,
        db.department_id,
        db.department_name,
        db.salary,
        db.years_service,
        db.dept_avg_salary,
        db.dept_max_salary,
        db.dept_employee_count,
        db.pct_gap_to_avg,
        db.recent_job_history_flag,
        db.manager_or_exec_flag,
        CASE
            WHEN db.department_id IS NULL THEN 'NO_ELEGIBLE'
            WHEN db.recent_job_history_flag = 'SI' THEN 'NO_ELEGIBLE'
            WHEN db.years_service < v.min_years_service THEN 'NO_ELEGIBLE'
            WHEN db.manager_or_exec_flag = 'SI' THEN 'NO_ELEGIBLE'
            WHEN db.department_id = v.excluded_department_id THEN 'NO_ELEGIBLE'
            WHEN db.dept_employee_count < 3 THEN 'NO_ELEGIBLE'
            WHEN db.pct_gap_to_avg <= 0 THEN 'NO_ELEGIBLE'
            ELSE 'ELEGIBLE'
        END AS eligibility_flag,
        CASE
            WHEN db.department_id IS NULL THEN 'SIN_DEPARTAMENTO'
            WHEN db.recent_job_history_flag = 'SI' THEN 'HISTORIAL_RECIENTE'
            WHEN db.years_service < v.min_years_service THEN 'ANTIGUEDAD_INSUFICIENTE'
            WHEN db.manager_or_exec_flag = 'SI' THEN 'MANAGER_O_DIRECTIVO'
            WHEN db.department_id = v.excluded_department_id THEN 'DEPTO_EXCLUIDO'
            WHEN db.dept_employee_count < 3 THEN 'DEPTO_MENOR_A_3'
            WHEN db.pct_gap_to_avg <= 0 THEN 'SALARIO_NO_APLICA'
            ELSE 'APLICA'
        END AS exclusion_reason,
        CASE
            WHEN db.department_id IS NULL THEN 0
            WHEN db.recent_job_history_flag = 'SI' THEN 0
            WHEN db.years_service < v.min_years_service THEN 0
            WHEN db.manager_or_exec_flag = 'SI' THEN 0
            WHEN db.department_id = v.excluded_department_id THEN 0
            WHEN db.dept_employee_count < 3 THEN 0
            WHEN db.pct_gap_to_avg >= v.gap_high_threshold_pct THEN v.raise_high_pct
            WHEN db.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN v.raise_mid_pct
            WHEN db.pct_gap_to_avg > 0 THEN v.raise_low_pct
            ELSE 0
        END AS adjustment_pct,
        CASE
            WHEN db.department_id IS NULL THEN 'NO_APLICA'
            WHEN db.recent_job_history_flag = 'SI' THEN 'NO_APLICA'
            WHEN db.years_service < v.min_years_service THEN 'NO_APLICA'
            WHEN db.manager_or_exec_flag = 'SI' THEN 'NO_APLICA'
            WHEN db.department_id = v.excluded_department_id THEN 'NO_APLICA'
            WHEN db.dept_employee_count < 3 THEN 'NO_APLICA'
            WHEN db.pct_gap_to_avg >= v.gap_high_threshold_pct THEN 'AJUSTE_ALTO'
            WHEN db.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN 'AJUSTE_MEDIO'
            WHEN db.pct_gap_to_avg > 0 THEN 'AJUSTE_BAJO'
            ELSE 'NO_APLICA'
        END AS rule_applied
    FROM diagnostic_base db
    CROSS JOIN v
)
SELECT
    employee_id,
    first_name,
    last_name,
    department_id,
    department_name,
    salary,
    years_service,
    dept_avg_salary,
    dept_max_salary,
    dept_employee_count,
    pct_gap_to_avg,
    recent_job_history_flag,
    manager_or_exec_flag,
    eligibility_flag,
    exclusion_reason,
    adjustment_pct,
    rule_applied
FROM eligibility
ORDER BY eligibility_flag DESC, department_id, employee_id;
-- Debe devolver las columnas mínimas exigidas arriba.



-- COMENTARIO OBLIGATORIO:
-- En esta consulta se aplican las reglas definidas en la variante seleccionada,
-- evaluando exclusiones por departamento, antigüedad, historial reciente,
-- perfil gerencial/directivo y tamaño del departamento. Solo se consideran
-- elegibles los empleados con brecha salarial positiva y que cumplen todas
-- las restricciones del caso, asignando además el porcentaje de ajuste aplicable.

-- Explique en 3 a 5 líneas cómo aplicó la variante y por qué su población
-- elegible sí cumple las reglas del caso.

-- ============================================================
-- 3. PREVALIDACIÓN ANTES DE LA TRANSACCIÓN
-- OBJETIVO:
-- Mostrar qué pasaría antes de ejecutar el cambio real.
--
-- DEBE MOSTRAR, COMO MÍNIMO:
-- A. Un resumen con estas columnas:
--    total_eligible_employees
--    total_salary_before
--    total_salary_after
--    total_increment
--
-- B. Un detalle de empleados elegibles con estas columnas:
--    employee_id
--    department_id
--    salary_before
--    salary_after
--    adjustment_pct
--    rule_applied
--
-- C. Un control de topes por departamento con estas columnas:
--    department_id
--    department_name
--    dept_avg_salary
--    dept_max_salary
--    max_allowed_salary_by_variant
--
-- QUÉ SIGNIFICA:
--   total_salary_before: suma de salarios antes del ajuste
--   total_salary_after: suma de salarios proyectados después del ajuste
--   total_increment: incremento total proyectado
--   max_allowed_salary_by_variant: salario máximo permitido según la variante
-- ============================================================

PROMPT ===== 3. PREVALIDACIÓN =====

-- ESCRIBA AQUÍ SU CONSULTA O SUS CONSULTAS DE PREVALIDACIÓN

PROMPT --- 3A. RESUMEN ---
WITH v AS (
    SELECT *
    FROM t1_variants
    WHERE variant_id = &p_variant_id
),
dept_stats AS (
    SELECT
        e.department_id,
        AVG(e.salary) AS dept_avg_salary,
        MAX(e.salary) AS dept_max_salary,
        COUNT(*) AS dept_employee_count
    FROM t1_employees e
    GROUP BY e.department_id
),
recent_history AS (
    SELECT DISTINCT j.employee_id
    FROM t1_job_history j
    CROSS JOIN v
    WHERE j.end_date >= ADD_MONTHS(TRUNC(SYSDATE), -v.recent_job_history_months)
),
manager_list AS (
    SELECT DISTINCT manager_id AS employee_id
    FROM t1_employees
    WHERE manager_id IS NOT NULL
),
eligible_population AS (
    SELECT
        e.employee_id,
        e.department_id,
        d.department_name,
        e.salary AS salary_before,
        ROUND(ds.dept_avg_salary, 2) AS dept_avg_salary,
        ds.dept_max_salary,
        ds.dept_employee_count,
        ROUND(
            CASE
                WHEN ds.dept_avg_salary > 0 THEN
                    ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100
                ELSE 0
            END
        , 2) AS pct_gap_to_avg,
        CASE
            WHEN rh.employee_id IS NOT NULL THEN 'SI'
            ELSE 'NO'
        END AS recent_job_history_flag,
        CASE
            WHEN ml.employee_id IS NOT NULL
                 OR e.job_id LIKE '%MAN%'
                 OR e.job_id LIKE '%MGR%'
                 OR e.job_id LIKE 'AD_%'
            THEN 'SI'
            ELSE 'NO'
        END AS manager_or_exec_flag,
        ROUND(MONTHS_BETWEEN(TRUNC(SYSDATE), e.hire_date) / 12, 2) AS years_service
    FROM t1_employees e
    LEFT JOIN t1_departments d
        ON e.department_id = d.department_id
    LEFT JOIN dept_stats ds
        ON e.department_id = ds.department_id
    LEFT JOIN recent_history rh
        ON e.employee_id = rh.employee_id
    LEFT JOIN manager_list ml
        ON e.employee_id = ml.employee_id
),
final_eligible AS (
    SELECT
        ep.*,
        CASE
            WHEN ep.department_id IS NULL THEN 0
            WHEN ep.recent_job_history_flag = 'SI' THEN 0
            WHEN ep.years_service < v.min_years_service THEN 0
            WHEN ep.manager_or_exec_flag = 'SI' THEN 0
            WHEN ep.department_id = v.excluded_department_id THEN 0
            WHEN ep.dept_employee_count < 3 THEN 0
            WHEN ep.pct_gap_to_avg >= v.gap_high_threshold_pct THEN v.raise_high_pct
            WHEN ep.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN v.raise_mid_pct
            WHEN ep.pct_gap_to_avg > 0 THEN v.raise_low_pct
            ELSE 0
        END AS adjustment_pct,
        CASE
            WHEN ep.department_id IS NULL THEN 'NO_APLICA'
            WHEN ep.recent_job_history_flag = 'SI' THEN 'NO_APLICA'
            WHEN ep.years_service < v.min_years_service THEN 'NO_APLICA'
            WHEN ep.manager_or_exec_flag = 'SI' THEN 'NO_APLICA'
            WHEN ep.department_id = v.excluded_department_id THEN 'NO_APLICA'
            WHEN ep.dept_employee_count < 3 THEN 'NO_APLICA'
            WHEN ep.pct_gap_to_avg >= v.gap_high_threshold_pct THEN 'AJUSTE_ALTO'
            WHEN ep.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN 'AJUSTE_MEDIO'
            WHEN ep.pct_gap_to_avg > 0 THEN 'AJUSTE_BAJO'
            ELSE 'NO_APLICA'
        END AS rule_applied,
        ROUND(ep.dept_avg_salary * (v.max_salary_vs_avg_pct / 100), 2) AS max_allowed_salary_by_variant
    FROM eligible_population ep
    CROSS JOIN v
    WHERE ep.department_id IS NOT NULL
      AND ep.recent_job_history_flag = 'NO'
      AND ep.years_service >= v.min_years_service
      AND ep.manager_or_exec_flag = 'NO'
      AND ep.department_id <> v.excluded_department_id
      AND ep.dept_employee_count >= 3
      AND ep.pct_gap_to_avg > 0
),
projection AS (
    SELECT
        employee_id,
        department_id,
        department_name,
        salary_before,
        ROUND(salary_before * (1 + adjustment_pct / 100), 2) AS salary_after,
        adjustment_pct,
        rule_applied,
        dept_avg_salary,
        dept_max_salary,
        max_allowed_salary_by_variant
    FROM final_eligible
)
SELECT
    COUNT(*) AS total_eligible_employees,
    ROUND(SUM(salary_before), 2) AS total_salary_before,
    ROUND(SUM(salary_after), 2) AS total_salary_after,
    ROUND(SUM(salary_after - salary_before), 2) AS total_increment
FROM projection;

PROMPT --- 3B. DETALLE DE ELEGIBLES ---
WITH v AS (
    SELECT *
    FROM t1_variants
    WHERE variant_id = &p_variant_id
),
dept_stats AS (
    SELECT
        e.department_id,
        AVG(e.salary) AS dept_avg_salary,
        MAX(e.salary) AS dept_max_salary,
        COUNT(*) AS dept_employee_count
    FROM t1_employees e
    GROUP BY e.department_id
),
recent_history AS (
    SELECT DISTINCT j.employee_id
    FROM t1_job_history j
    CROSS JOIN v
    WHERE j.end_date >= ADD_MONTHS(TRUNC(SYSDATE), -v.recent_job_history_months)
),
manager_list AS (
    SELECT DISTINCT manager_id AS employee_id
    FROM t1_employees
    WHERE manager_id IS NOT NULL
),
eligible_population AS (
    SELECT
        e.employee_id,
        e.department_id,
        d.department_name,
        e.salary AS salary_before,
        ROUND(ds.dept_avg_salary, 2) AS dept_avg_salary,
        ds.dept_max_salary,
        ds.dept_employee_count,
        ROUND(
            CASE
                WHEN ds.dept_avg_salary > 0 THEN
                    ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100
                ELSE 0
            END
        , 2) AS pct_gap_to_avg,
        CASE
            WHEN rh.employee_id IS NOT NULL THEN 'SI'
            ELSE 'NO'
        END AS recent_job_history_flag,
        CASE
            WHEN ml.employee_id IS NOT NULL
                 OR e.job_id LIKE '%MAN%'
                 OR e.job_id LIKE '%MGR%'
                 OR e.job_id LIKE 'AD_%'
            THEN 'SI'
            ELSE 'NO'
        END AS manager_or_exec_flag,
        ROUND(MONTHS_BETWEEN(TRUNC(SYSDATE), e.hire_date) / 12, 2) AS years_service
    FROM t1_employees e
    LEFT JOIN t1_departments d
        ON e.department_id = d.department_id
    LEFT JOIN dept_stats ds
        ON e.department_id = ds.department_id
    LEFT JOIN recent_history rh
        ON e.employee_id = rh.employee_id
    LEFT JOIN manager_list ml
        ON e.employee_id = ml.employee_id
),
projection AS (
    SELECT
        ep.employee_id,
        ep.department_id,
        ep.salary_before,
        ROUND(
            ep.salary_before * (
                1 + (
                    CASE
                        WHEN ep.pct_gap_to_avg >= v.gap_high_threshold_pct THEN v.raise_high_pct
                        WHEN ep.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN v.raise_mid_pct
                        WHEN ep.pct_gap_to_avg > 0 THEN v.raise_low_pct
                        ELSE 0
                    END
                ) / 100
            ),
            2
        ) AS salary_after,
        CASE
            WHEN ep.pct_gap_to_avg >= v.gap_high_threshold_pct THEN v.raise_high_pct
            WHEN ep.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN v.raise_mid_pct
            WHEN ep.pct_gap_to_avg > 0 THEN v.raise_low_pct
            ELSE 0
        END AS adjustment_pct,
        CASE
            WHEN ep.pct_gap_to_avg >= v.gap_high_threshold_pct THEN 'AJUSTE_ALTO'
            WHEN ep.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN 'AJUSTE_MEDIO'
            WHEN ep.pct_gap_to_avg > 0 THEN 'AJUSTE_BAJO'
            ELSE 'NO_APLICA'
        END AS rule_applied
    FROM eligible_population ep
    CROSS JOIN v
    WHERE ep.department_id IS NOT NULL
      AND ep.recent_job_history_flag = 'NO'
      AND ep.years_service >= v.min_years_service
      AND ep.manager_or_exec_flag = 'NO'
      AND ep.department_id <> v.excluded_department_id
      AND ep.dept_employee_count >= 3
      AND ep.pct_gap_to_avg > 0
)
SELECT
    employee_id,
    department_id,
    salary_before,
    salary_after,
    adjustment_pct,
    rule_applied
FROM projection
ORDER BY department_id, employee_id;

PROMPT --- 3C. CONTROL DE TOPES ---
WITH v AS (
    SELECT *
    FROM t1_variants
    WHERE variant_id = &p_variant_id
),
dept_stats AS (
    SELECT
        e.department_id,
        d.department_name,
        ROUND(AVG(e.salary), 2) AS dept_avg_salary,
        MAX(e.salary) AS dept_max_salary
    FROM t1_employees e
    LEFT JOIN t1_departments d
        ON e.department_id = d.department_id
    GROUP BY e.department_id, d.department_name
)
SELECT
    department_id,
    department_name,
    dept_avg_salary,
    dept_max_salary,
    ROUND(dept_avg_salary * (v.max_salary_vs_avg_pct / 100), 2) AS max_allowed_salary_by_variant
FROM dept_stats
CROSS JOIN v
ORDER BY department_id;



-- Debe mostrar el resumen, el detalle y el control de topes.



-- ============================================================
-- 4. EJECUCIÓN TRANSACCIONAL
-- OBJETIVO:
-- Ejecutar la actualización real y registrar la auditoría.
--
-- DEBE INCLUIR OBLIGATORIAMENTE:
-- 1. SAVEPOINT
-- 2. UPDATE o MERGE para actualizar salarios
-- 3. INSERT a AUDIT_SALARY_ADJUSTMENTS_T1
-- 4. Validación intermedia
-- 5. COMMIT o ROLLBACK TO SAVEPOINT
--
-- IMPORTANTE:
-- - La auditoría debe usar el valor &p_execution_tag
-- - La auditoría debe usar el valor &p_variant_id
-- - Debe usar la secuencia AUDIT_SALARY_ADJ_T1_SEQ.NEXTVAL
-- ============================================================

PROMPT ===== 4. EJECUCIÓN TRANSACCIONAL =====

SAVEPOINT sv_before_adjustment;

-- 4.1 ACTUALIZACIÓN DE SALARIOS
-- ESCRIBA AQUÍ SU UPDATE O MERGE
MERGE INTO t1_employees e
USING (
    WITH v AS (
        SELECT *
        FROM t1_variants
        WHERE variant_id = &p_variant_id
    ),
    dept_stats AS (
        SELECT
            emp.department_id,
            AVG(emp.salary) AS dept_avg_salary,
            MAX(emp.salary) AS dept_max_salary,
            COUNT(*) AS dept_employee_count
        FROM t1_employees emp
        GROUP BY emp.department_id
    ),
    recent_history AS (
        SELECT DISTINCT j.employee_id
        FROM t1_job_history j
        CROSS JOIN v
        WHERE j.end_date >= ADD_MONTHS(TRUNC(SYSDATE), -v.recent_job_history_months)
    ),
    manager_list AS (
        SELECT DISTINCT manager_id AS employee_id
        FROM t1_employees
        WHERE manager_id IS NOT NULL
    ),
    eligible_population AS (
        SELECT
            emp.employee_id,
            emp.department_id,
            emp.salary AS salary_before,
            ROUND(ds.dept_avg_salary, 2) AS dept_avg_salary,
            ds.dept_max_salary,
            ds.dept_employee_count,
            ROUND(
                CASE
                    WHEN ds.dept_avg_salary > 0 THEN
                        ((ds.dept_avg_salary - emp.salary) / ds.dept_avg_salary) * 100
                    ELSE 0
                END
            , 2) AS pct_gap_to_avg,
            CASE
                WHEN rh.employee_id IS NOT NULL THEN 'SI'
                ELSE 'NO'
            END AS recent_job_history_flag,
            CASE
                WHEN ml.employee_id IS NOT NULL
                     OR emp.job_id LIKE '%MAN%'
                     OR emp.job_id LIKE '%MGR%'
                     OR emp.job_id LIKE 'AD_%'
                THEN 'SI'
                ELSE 'NO'
            END AS manager_or_exec_flag,
            ROUND(MONTHS_BETWEEN(TRUNC(SYSDATE), emp.hire_date) / 12, 2) AS years_service
        FROM t1_employees emp
        LEFT JOIN dept_stats ds
            ON emp.department_id = ds.department_id
        LEFT JOIN recent_history rh
            ON emp.employee_id = rh.employee_id
        LEFT JOIN manager_list ml
            ON emp.employee_id = ml.employee_id
    ),
    final_eligible AS (
        SELECT
            ep.employee_id,
            ep.department_id,
            ep.salary_before,
            ep.pct_gap_to_avg,
            CASE
                WHEN ep.pct_gap_to_avg >= v.gap_high_threshold_pct THEN v.raise_high_pct
                WHEN ep.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN v.raise_mid_pct
                WHEN ep.pct_gap_to_avg > 0 THEN v.raise_low_pct
                ELSE 0
            END AS adjustment_pct,
            CASE
                WHEN ep.pct_gap_to_avg >= v.gap_high_threshold_pct THEN 'AJUSTE_ALTO'
                WHEN ep.pct_gap_to_avg >= v.gap_mid_threshold_pct THEN 'AJUSTE_MEDIO'
                WHEN ep.pct_gap_to_avg > 0 THEN 'AJUSTE_BAJO'
                ELSE 'NO_APLICA'
            END AS rule_applied,
            ROUND(ep.dept_avg_salary * (v.max_salary_vs_avg_pct / 100), 2) AS allowed_max_salary
        FROM eligible_population ep
        CROSS JOIN v
        WHERE ep.department_id IS NOT NULL
          AND ep.recent_job_history_flag = 'NO'
          AND ep.years_service >= v.min_years_service
          AND ep.manager_or_exec_flag = 'NO'
          AND ep.department_id <> v.excluded_department_id
          AND ep.dept_employee_count >= 3
          AND ep.pct_gap_to_avg > 0
    )
    SELECT
        employee_id,
        department_id,
        salary_before,
        pct_gap_to_avg,
        adjustment_pct,
        rule_applied,
        allowed_max_salary,
        ROUND(salary_before * (1 + adjustment_pct / 100), 2) AS salary_after
    FROM final_eligible
) src
ON (e.employee_id = src.employee_id)
WHEN MATCHED THEN
UPDATE SET e.salary = src.salary_after;
-- Debe actualizar únicamente empleados ELEGIBLES.



-- 4.2 INSERCIÓN EN AUDITORÍA
-- Debe llenar estas columnas de AUDIT_SALARY_ADJUSTMENTS_T1:
--   audit_id               -> usar AUDIT_SALARY_ADJ_T1_SEQ.NEXTVAL
--   execution_tag          -> usar &p_execution_tag
--   variant_id             -> usar &p_variant_id
--   employee_id            -> id del empleado ajustado
--   department_id          -> departamento del empleado
--   salary_before          -> salario antes del ajuste
--   salary_after           -> salario después del ajuste
--   pct_gap_to_avg_before  -> brecha porcentual antes del ajuste
--   rule_applied           -> regla aplicada
--   executed_by            -> USER
--   executed_at            -> SYSDATE
--   notes                  -> comentario libre

INSERT INTO audit_salary_adjustments_t1 (
    audit_id,
    execution_tag,
    variant_id,
    employee_id,
    department_id,
    salary_before,
    salary_after,
    pct_gap_to_avg_before,
    rule_applied,
    executed_by,
    executed_at,
    notes
)
-- ESCRIBA AQUÍ SU SELECT O VALUES PARA INSERTAR LA AUDITORÍA
INSERT INTO audit_salary_adjustments_t1 (
    audit_id,
    execution_tag,
    variant_id,
    employee_id,
    department_id,
    salary_before,
    salary_after,
    pct_gap_to_avg_before,
    rule_applied,
    executed_by,
    executed_at,
    notes
)
WITH v AS (
    SELECT *
    FROM t1_variants
    WHERE variant_id = &p_variant_id
),
dept_stats AS (
    SELECT
        e.department_id,
        AVG(e.salary) AS dept_avg_salary,
        COUNT(*) AS dept_employee_count
    FROM t1_employees e
    GROUP BY e.department_id
),
recent_history AS (
    SELECT DISTINCT j.employee_id
    FROM t1_job_history j
    CROSS JOIN v
    WHERE j.end_date >= ADD_MONTHS(TRUNC(SYSDATE), -v.recent_job_history_months)
),
manager_list AS (
    SELECT DISTINCT manager_id AS employee_id
    FROM t1_employees
    WHERE manager_id IS NOT NULL
),
eligible AS (
    SELECT
        e.employee_id,
        e.department_id,
        e.salary AS salary_after,
        ROUND(
            e.salary / (1 + (
                CASE
                    WHEN ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100 >= v.gap_high_threshold_pct THEN v.raise_high_pct
                    WHEN ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100 >= v.gap_mid_threshold_pct THEN v.raise_mid_pct
                    ELSE v.raise_low_pct
                END
            ) / 100),
        2) AS salary_before,
        ROUND(
            ((ds.dept_avg_salary - (
                e.salary / (1 + (
                    CASE
                        WHEN ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100 >= v.gap_high_threshold_pct THEN v.raise_high_pct
                        WHEN ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100 >= v.gap_mid_threshold_pct THEN v.raise_mid_pct
                        ELSE v.raise_low_pct
                    END
                ) / 100)
            )) / ds.dept_avg_salary) * 100
        , 2) AS pct_gap_to_avg,
        CASE
            WHEN ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100 >= v.gap_high_threshold_pct THEN 'AJUSTE_ALTO'
            WHEN ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100 >= v.gap_mid_threshold_pct THEN 'AJUSTE_MEDIO'
            WHEN ((ds.dept_avg_salary - e.salary) / ds.dept_avg_salary) * 100 > 0 THEN 'AJUSTE_BAJO'
            ELSE 'NO_APLICA'
        END AS rule_applied
    FROM t1_employees e
    LEFT JOIN dept_stats ds ON e.department_id = ds.department_id
    LEFT JOIN recent_history rh ON e.employee_id = rh.employee_id
    LEFT JOIN manager_list ml ON e.employee_id = ml.employee_id
    CROSS JOIN v
    WHERE e.department_id IS NOT NULL
      AND rh.employee_id IS NULL
      AND ml.employee_id IS NULL
      AND e.department_id <> v.excluded_department_id
      AND ds.dept_employee_count >= 3
)
SELECT
    audit_salary_adj_t1_seq.NEXTVAL,
    '&p_execution_tag',
    &p_variant_id,
    employee_id,
    department_id,
    salary_before,
    salary_after,
    pct_gap_to_avg,
    rule_applied,
    USER,
    SYSDATE,
    'Ajuste salarial aplicado'
FROM eligible;

-- 4.3 VALIDACIÓN INTERMEDIA
-- Debe mostrar, como mínimo, estas columnas:
--   employee_id
--   department_id
--   current_salary
--   original_salary
--   allowed_max_salary
--   validation_status
--
-- validation_status debe indicar si cumple o no cumple.

PROMPT ===== 4.3 VALIDACIÓN INTERMEDIA =====

-- ESCRIBA AQUÍ SU CONSULTA DE VALIDACIÓN INTERMEDIA

WITH v AS (
    SELECT *
    FROM t1_variants
    WHERE variant_id = &p_variant_id
),
dept_stats AS (
    SELECT
        e.department_id,
        ROUND(AVG(e.salary), 2) AS dept_avg_salary
    FROM t1_employees e
    GROUP BY e.department_id
)
SELECT
    e.employee_id,
    e.department_id,
    e.salary AS current_salary,
    a.salary_before AS original_salary,
    ROUND(ds.dept_avg_salary * (v.max_salary_vs_avg_pct / 100), 2) AS allowed_max_salary,
    CASE
        WHEN e.salary <= ROUND(ds.dept_avg_salary * (v.max_salary_vs_avg_pct / 100), 2)
        THEN 'CUMPLE'
        ELSE 'NO_CUMPLE'
    END AS validation_status
FROM t1_employees e
JOIN audit_salary_adjustments_t1 a
    ON e.employee_id = a.employee_id
    AND a.execution_tag = '&p_execution_tag'
LEFT JOIN dept_stats ds
    ON e.department_id = ds.department_id
CROSS JOIN v
ORDER BY e.department_id, e.employee_id;

-- 4.4 CONTROL TRANSACCIONAL
-- Debe demostrar UNO de estos escenarios:
-- A. COMMIT si toda la validación es correcta
-- B. ROLLBACK TO SAVEPOINT si detecta incumplimientos
--
-- ESCRIBA AQUÍ SU DECISIÓN TRANSACCIONAL Y AGREGUE UN COMENTARIO
DECLARE
    v_invalid_count NUMBER;
BEGIN
    SELECT COUNT(*)
    INTO v_invalid_count
    FROM (
        WITH v AS (
            SELECT *
            FROM t1_variants
            WHERE variant_id = &p_variant_id
        ),
        dept_stats AS (
            SELECT department_id, AVG(salary) AS dept_avg_salary
            FROM t1_employees
            GROUP BY department_id
        )
        SELECT e.employee_id
        FROM t1_employees e
        JOIN audit_salary_adjustments_t1 a
            ON e.employee_id = a.employee_id
            AND a.execution_tag = '&p_execution_tag'
        JOIN dept_stats ds
            ON e.department_id = ds.department_id
        CROSS JOIN v
        WHERE e.salary > (ds.dept_avg_salary * (v.max_salary_vs_avg_pct / 100))
    );

    IF v_invalid_count = 0 THEN
        COMMIT;
        DBMS_OUTPUT.PUT_LINE('COMMIT ejecutado correctamente.');
    ELSE
        BEGIN
            ROLLBACK TO sv_before_adjustment;
            DBMS_OUTPUT.PUT_LINE('ROLLBACK ejecutado: ' || v_invalid_count || ' errores.');
        EXCEPTION
            WHEN OTHERS THEN
                DBMS_OUTPUT.PUT_LINE('No se pudo hacer ROLLBACK al SAVEPOINT.');
        END;
    END IF;
END;
/



-- explicando por qué hizo COMMIT o por qué hizo ROLLBACK.

-- Se realiza la validación de los salarios ajustados frente al límite máximo permitido
-- por la variante. Si todos los empleados cumplen la restricción, se confirma la
-- transacción con COMMIT garantizando la persistencia de los cambios.
-- En caso de detectar incumplimientos, se ejecuta ROLLBACK TO SAVEPOINT para
-- revertir los cambios y mantener la consistencia de los datos.

-- ============================================================
-- 5. VALIDACIÓN POSTERIOR
-- OBJETIVO:
-- Demostrar el resultado final de la transacción.
--
-- DEBE MOSTRAR, COMO MÍNIMO, ESTAS 4 SALIDAS:
--
-- SALIDA 1. Empleados impactados
-- Columnas mínimas:
--   employee_id, first_name, last_name, department_id,
--   salary_before, salary_after, execution_tag


EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        125 Julia                Nayer                                50          3488      3697,28 MillanGonzalo_FINAL           
        126 Irene                Mikkilineni                          50          2943      3207,87 MillanGonzalo_FINAL           
        127 James                Landry                               50          2616      2851,44 MillanGonzalo_FINAL           
        128 Steven               Markle                               50          2398      2613,82 MillanGonzalo_FINAL           
        129 Laura                Bissot                               50          3498      3707,88 MillanGonzalo_FINAL           
        130 Mozhe                Atkinson                             50          3052      3326,68 MillanGonzalo_FINAL           
        131 James                Marlow                               50          2725      2970,25 MillanGonzalo_FINAL           
        132 TJ                   Olson                                50          2289      2495,01 MillanGonzalo_FINAL           
        133 Jason                Mallin                               50          3498      3707,88 MillanGonzalo_FINAL           
        134 Michael              Rogers                               50          3161      3445,49 MillanGonzalo_FINAL           
        135 Ki                   Gee                                  50          2616      2851,44 MillanGonzalo_FINAL           

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        136 Hazel                Philtanker                           50          2398      2613,82 MillanGonzalo_FINAL           
        137 Renske               Ladwig                               50       3704,85         3816 MillanGonzalo_FINAL           
        138 Stephen              Stiles                               50          3488      3697,28 MillanGonzalo_FINAL           
        139 John                 Seo                                  50          2943      3207,87 MillanGonzalo_FINAL           
        140 Joshua               Patel                                50          2725      2970,25 MillanGonzalo_FINAL           
        141 Trenna               Rajs                                 50       3502,97      3713,15 MillanGonzalo_FINAL           
        142 Curtis               Davies                               50       3474,63      3683,11 MillanGonzalo_FINAL           
        143 Randall              Matos                                50          2834      3089,06 MillanGonzalo_FINAL           
        144 Peter                Vargas                               50          2725      2970,25 MillanGonzalo_FINAL           
        180 Winston              Taylor                               50       5728,16         5900 MillanGonzalo_FINAL           
        181 Jean                 Fleaur                               50       5436,89         5600 MillanGonzalo_FINAL           

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        182 Martha               Sullivan                             50          2725      2970,25 MillanGonzalo_FINAL           
        183 Girard               Geoni                                50          3052      3326,68 MillanGonzalo_FINAL           
        184 Nandita              Sarchand                             50       4077,67         4200 MillanGonzalo_FINAL           
        185 Alexis               Bull                                 50       3980,58         4100 MillanGonzalo_FINAL           
        186 Julia                Dellinger                            50          3502      3712,12 MillanGonzalo_FINAL           
        187 Anthony              Cabrio                               50          3270       3564,3 MillanGonzalo_FINAL           
        188 Kelly                Chung                                50       3689,32         3800 MillanGonzalo_FINAL           
        189 Jennifer             Dilly                                50       3704,85         3816 MillanGonzalo_FINAL           
        190 Timothy              Venzl                                50          3161      3445,49 MillanGonzalo_FINAL           
        191 Randall              Perkins                              50          2725      2970,25 MillanGonzalo_FINAL           
        192 Sarah                Bell                                 50        3883,5         4000 MillanGonzalo_FINAL           

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        193 Britney              Everett                              50       3786,41         3900 MillanGonzalo_FINAL           
        194 Samuel               McLeod                               50          3488      3697,28 MillanGonzalo_FINAL           
        195 Vance                Jones                                50          3052      3326,68 MillanGonzalo_FINAL           
        196 Alana                Walsh                                50       3474,63      3683,11 MillanGonzalo_FINAL           
        197 Kevin                Feeney                               50          3270       3564,3 MillanGonzalo_FINAL           
        198 Donald               OConnell                             50          2834      3089,06 MillanGonzalo_FINAL           
        199 Douglas              Grant                                50          2834      3089,06 MillanGonzalo_FINAL           
        104 Bruce                Miller                               60          6000         6180 MillanGonzalo_FINAL           
        105 David                Williams                             60          5232      5702,88 MillanGonzalo_FINAL           
        106 Valli                Jackson                              60          5232      5702,88 MillanGonzalo_FINAL           
        107 Diana                Nguyen                               60          4578      4990,02 MillanGonzalo_FINAL           

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        150 Sean                 Tucker                               80       9708,74        10000 MillanGonzalo_FINAL           
        151 David                Bernstein                            80        9223,3         9500 MillanGonzalo_FINAL           
        152 Peter                Hall                                 80       8737,86         9000 MillanGonzalo_FINAL           
        153 Christopher          Olsen                                80       8726,99       8988,8 MillanGonzalo_FINAL           
        154 Nanette              Cambrault                            80          8175       8665,5 MillanGonzalo_FINAL           
        155 Oliver               Tuvault                              80          7630       8316,7 MillanGonzalo_FINAL           
        156 Janette              King                                 80       9708,74        10000 MillanGonzalo_FINAL           
        157 Patrick              Sully                                80        9223,3         9500 MillanGonzalo_FINAL           
        158 Allan                McEwen                               80       8737,86         9000 MillanGonzalo_FINAL           
        159 Lindsey              Smith                                80       8726,99       8988,8 MillanGonzalo_FINAL           
        160 Louise               Doran                                80          8175       8665,5 MillanGonzalo_FINAL           

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        161 Sarath               Sewall                               80          7630       8316,7 MillanGonzalo_FINAL           
        162 Clara                Vishney                              80      10194,17        10500 MillanGonzalo_FINAL           
        163 Danielle             Greene                               80        9223,3         9500 MillanGonzalo_FINAL           
        164 Mattea               Marvins                              80       8070,11      8554,32 MillanGonzalo_FINAL           
        165 David                Lee                                  80          7412      8079,08 MillanGonzalo_FINAL           
        166 Sundar               Ande                                 80          6976      7603,84 MillanGonzalo_FINAL           
        167 Amit                 Banda                                80          6758      7366,22 MillanGonzalo_FINAL           
        168 Lisa                 Ozer                                 80      11165,05        11500 MillanGonzalo_FINAL           
        169 Harrison             Bloom                                80       9708,74        10000 MillanGonzalo_FINAL           
        170 Tayler               Fox                                  80       9320,39         9600 MillanGonzalo_FINAL           
        171 William              Smith                                80       8294,28      8791,94 MillanGonzalo_FINAL           

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        172 Elizabeth            Bates                                80        8182,2      8673,13 MillanGonzalo_FINAL           
        173 Sundita              Kumar                                80          6649      7247,41 MillanGonzalo_FINAL           
        174 Ellen                Abel                                 80       8637,46      8896,58 MillanGonzalo_FINAL           
        175 Alyssa               Hutton                               80          8800         9064 MillanGonzalo_FINAL           
        177 Jack                 Livingston                           80          7412      8079,08 MillanGonzalo_FINAL           
        179 Charles              Johnson                              80          6649      7247,41 MillanGonzalo_FINAL           
        109 Daniel               Faviet                              100      15631,07        16100 MillanGonzalo_FINAL           
        110 John                 Chen                                100      11456,31        11800 MillanGonzalo_FINAL           
        111 Ismael               Sciarra                             100      11407,77        11750 MillanGonzalo_FINAL           
        112 Jose Manuel          Urman                               100          6758      7366,22 MillanGonzalo_FINAL           
        113 Luis                 Popp                                100          7085      7722,65 MillanGonzalo_FINAL           

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER EXECUTION_TAG                 
----------- -------------------- ------------------------- ------------- ------------- ------------ ------------------------------
        303 Sara                 Luna                                290          3193      3288,79 MillanGonzalo_FINAL           
        305 Valeria              Nieto                               290       3398,06         3500 MillanGonzalo_FINAL           
        306 Yuri                 Mora                                290       2660,55         2900 MillanGonzalo_FINAL           

80 filas seleccionadas. 


--
-- SALIDA 2. Resumen económico final
-- Columnas mínimas:
--   total_rows_audited, total_salary_before, total_salary_after, total_increment

TOTAL_ROWS_AUDITED TOTAL_SALARY_BEFORE TOTAL_SALARY_AFTER TOTAL_INCREMENT
------------------ ------------------- ------------------ ---------------
                80            452241,7          477038,86        24797,16
--
-- SALIDA 3. Validación de topes
-- Columnas mínimas:
--   employee_id, department_id, salary_after, allowed_max_salary, top_limit_status


EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        125            50      3697,28            4675,12 CUMPLE   
        126            50      3207,87            4675,12 CUMPLE   
        127            50      2851,44            4675,12 CUMPLE   
        128            50      2613,82            4675,12 CUMPLE   
        129            50      3707,88            4675,12 CUMPLE   
        130            50      3326,68            4675,12 CUMPLE   
        131            50      2970,25            4675,12 CUMPLE   
        132            50      2495,01            4675,12 CUMPLE   
        133            50      3707,88            4675,12 CUMPLE   
        134            50      3445,49            4675,12 CUMPLE   
        135            50      2851,44            4675,12 CUMPLE   

EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        136            50      2613,82            4675,12 CUMPLE   
        137            50         3816            4675,12 CUMPLE   
        138            50      3697,28            4675,12 CUMPLE   
        139            50      3207,87            4675,12 CUMPLE   
        140            50      2970,25            4675,12 CUMPLE   
        141            50      3713,15            4675,12 CUMPLE   
        142            50      3683,11            4675,12 CUMPLE   
        143            50      3089,06            4675,12 CUMPLE   
        144            50      2970,25            4675,12 CUMPLE   
        180            50         5900            4675,12 NO_CUMPLE
        181            50         5600            4675,12 NO_CUMPLE

EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        182            50      2970,25            4675,12 CUMPLE   
        183            50      3326,68            4675,12 CUMPLE   
        184            50         4200            4675,12 CUMPLE   
        185            50         4100            4675,12 CUMPLE   
        186            50      3712,12            4675,12 CUMPLE   
        187            50       3564,3            4675,12 CUMPLE   
        188            50         3800            4675,12 CUMPLE   
        189            50         3816            4675,12 CUMPLE   
        190            50      3445,49            4675,12 CUMPLE   
        191            50      2970,25            4675,12 CUMPLE   
        192            50         4000            4675,12 CUMPLE   

EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        193            50         3900            4675,12 CUMPLE   
        194            50      3697,28            4675,12 CUMPLE   
        195            50      3326,68            4675,12 CUMPLE   
        196            50      3683,11            4675,12 CUMPLE   
        197            50       3564,3            4675,12 CUMPLE   
        198            50      3089,06            4675,12 CUMPLE   
        199            50      3089,06            4675,12 CUMPLE   
        104            60         6180            7515,04 CUMPLE   
        105            60      5702,88            7515,04 CUMPLE   
        106            60      5702,88            7515,04 CUMPLE   
        107            60      4990,02            7515,04 CUMPLE   

EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        150            80        10000           10967,08 CUMPLE   
        151            80         9500           10967,08 CUMPLE   
        152            80         9000           10967,08 CUMPLE   
        153            80       8988,8           10967,08 CUMPLE   
        154            80       8665,5           10967,08 CUMPLE   
        155            80       8316,7           10967,08 CUMPLE   
        156            80        10000           10967,08 CUMPLE   
        157            80         9500           10967,08 CUMPLE   
        158            80         9000           10967,08 CUMPLE   
        159            80       8988,8           10967,08 CUMPLE   
        160            80       8665,5           10967,08 CUMPLE   

EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        161            80       8316,7           10967,08 CUMPLE   
        162            80        10500           10967,08 CUMPLE   
        163            80         9500           10967,08 CUMPLE   
        164            80      8554,32           10967,08 CUMPLE   
        165            80      8079,08           10967,08 CUMPLE   
        166            80      7603,84           10967,08 CUMPLE   
        167            80      7366,22           10967,08 CUMPLE   
        168            80        11500           10967,08 NO_CUMPLE
        169            80        10000           10967,08 CUMPLE   
        170            80         9600           10967,08 CUMPLE   
        171            80      8791,94           10967,08 CUMPLE   

EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        172            80      8673,13           10967,08 CUMPLE   
        173            80      7247,41           10967,08 CUMPLE   
        174            80      8896,58           10967,08 CUMPLE   
        175            80         9064           10967,08 CUMPLE   
        177            80      8079,08           10967,08 CUMPLE   
        179            80      7247,41           10967,08 CUMPLE   
        109           100        16100           13990,21 NO_CUMPLE
        110           100        11800           13990,21 CUMPLE   
        111           100        11750           13990,21 CUMPLE   
        112           100      7366,22           13990,21 CUMPLE   
        113           100      7722,65           13990,21 CUMPLE   

EMPLOYEE_ID DEPARTMENT_ID SALARY_AFTER ALLOWED_MAX_SALARY TOP_LIMIT
----------- ------------- ------------ ------------------ ---------
        303           290      3288,79            3864,17 CUMPLE   
        305           290         3500            3864,17 CUMPLE   
        306           290         2900            3864,17 CUMPLE   

80 filas seleccionadas. 


--
-- SALIDA 4. Auditoría generada
-- Columnas mínimas:
--   audit_id, execution_tag, variant_id, employee_id, department_id,
--   salary_before, salary_after, rule_applied, executed_by, executed_at


  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       193 MillanGonzalo_FINAL                     4         104            60          6000         6180                  4,99 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       180 MillanGonzalo_FINAL                     4         105            60          5232      5702,88                 17,15 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       237 MillanGonzalo_FINAL                     4         106            60          5232      5702,88                 17,15 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       161 MillanGonzalo_FINAL                     4         107            60          4578      4990,02                 27,51 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       173 MillanGonzalo_FINAL                     4         109           100      15631,07        16100 -32,96                NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       186 MillanGonzalo_FINAL                     4         110           100      11456,31        11800                  2,55 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       225 MillanGonzalo_FINAL                     4         111           100      11407,77        11750                  2,97 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       216 MillanGonzalo_FINAL                     4         112           100          6758      7366,22                 42,52 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       166 MillanGonzalo_FINAL                     4         113           100          7085      7722,65                 39,74 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       167 MillanGonzalo_FINAL                     4         125            50          3488      3697,28                 11,22 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       218 MillanGonzalo_FINAL                     4         126            50          2943      3207,87                 25,09 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       169 MillanGonzalo_FINAL                     4         127            50          2616      2851,44                 33,41 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       219 MillanGonzalo_FINAL                     4         128            50          2398      2613,82                 38,96 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       187 MillanGonzalo_FINAL                     4         129            50          3498      3707,88                 10,96 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       220 MillanGonzalo_FINAL                     4         130            50          3052      3326,68                 22,31 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       175 MillanGonzalo_FINAL                     4         131            50          2725      2970,25                 30,64 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       230 MillanGonzalo_FINAL                     4         132            50          2289      2495,01                 41,74 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       226 MillanGonzalo_FINAL                     4         133            50          3498      3707,88                 10,96 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       170 MillanGonzalo_FINAL                     4         134            50          3161      3445,49                 19,54 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       164 MillanGonzalo_FINAL                     4         135            50          2616      2851,44                 33,41 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       176 MillanGonzalo_FINAL                     4         136            50          2398      2613,82                 38,96 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       206 MillanGonzalo_FINAL                     4         137            50       3704,85         3816                   5,7 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       236 MillanGonzalo_FINAL                     4         138            50          3488      3697,28                 11,22 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       231 MillanGonzalo_FINAL                     4         139            50          2943      3207,87                 25,09 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       204 MillanGonzalo_FINAL                     4         140            50          2725      2970,25                 30,64 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       195 MillanGonzalo_FINAL                     4         141            50       3502,97      3713,15                 10,84 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       212 MillanGonzalo_FINAL                     4         142            50       3474,63      3683,11                 11,56 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       177 MillanGonzalo_FINAL                     4         143            50          2834      3089,06                 27,86 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       223 MillanGonzalo_FINAL                     4         144            50          2725      2970,25                 30,64 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       238 MillanGonzalo_FINAL                     4         150            80       9708,74        10000 -5,35                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       214 MillanGonzalo_FINAL                     4         151            80        9223,3         9500 -0,08                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       217 MillanGonzalo_FINAL                     4         152            80       8737,86         9000                  5,19 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       228 MillanGonzalo_FINAL                     4         153            80       8726,99       8988,8                  5,31 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       233 MillanGonzalo_FINAL                     4         154            80          8175       8665,5                  11,3 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       202 MillanGonzalo_FINAL                     4         155            80          7630       8316,7                 17,21 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       213 MillanGonzalo_FINAL                     4         156            80       9708,74        10000 -5,35                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       209 MillanGonzalo_FINAL                     4         157            80        9223,3         9500 -0,08                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       168 MillanGonzalo_FINAL                     4         158            80       8737,86         9000                  5,19 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       181 MillanGonzalo_FINAL                     4         159            80       8726,99       8988,8                  5,31 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       240 MillanGonzalo_FINAL                     4         160            80          8175       8665,5                  11,3 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       163 MillanGonzalo_FINAL                     4         161            80          7630       8316,7                 17,21 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       183 MillanGonzalo_FINAL                     4         162            80      10194,17        10500 -10,61                NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       182 MillanGonzalo_FINAL                     4         163            80        9223,3         9500 -0,08                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       229 MillanGonzalo_FINAL                     4         164            80       8070,11      8554,32                 12,43 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       234 MillanGonzalo_FINAL                     4         165            80          7412      8079,08                 19,57 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       165 MillanGonzalo_FINAL                     4         166            80          6976      7603,84                 24,31 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       221 MillanGonzalo_FINAL                     4         167            80          6758      7366,22                 26,67 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       184 MillanGonzalo_FINAL                     4         168            80      11165,05        11500 -21,15                NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       210 MillanGonzalo_FINAL                     4         169            80       9708,74        10000 -5,35                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       215 MillanGonzalo_FINAL                     4         170            80       9320,39         9600 -1,13                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       222 MillanGonzalo_FINAL                     4         171            80       8294,28      8791,94                    10 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       185 MillanGonzalo_FINAL                     4         172            80        8182,2      8673,13                 11,22 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       162 MillanGonzalo_FINAL                     4         173            80          6649      7247,41                 27,85 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       199 MillanGonzalo_FINAL                     4         174            80       8637,46      8896,58                  6,28 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       235 MillanGonzalo_FINAL                     4         175            80          8800         9064                  4,51 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       200 MillanGonzalo_FINAL                     4         177            80          7412      8079,08                 19,57 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       191 MillanGonzalo_FINAL                     4         179            80          6649      7247,41                 27,85 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       178 MillanGonzalo_FINAL                     4         180            50       5728,16         5900 -45,8                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       224 MillanGonzalo_FINAL                     4         181            50       5436,89         5600 -38,39                NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       239 MillanGonzalo_FINAL                     4         182            50          2725      2970,25                 30,64 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       171 MillanGonzalo_FINAL                     4         183            50          3052      3326,68                 22,31 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       188 MillanGonzalo_FINAL                     4         184            50       4077,67         4200 -3,79                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       189 MillanGonzalo_FINAL                     4         185            50       3980,58         4100 -1,32                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       207 MillanGonzalo_FINAL                     4         186            50          3502      3712,12                 10,86 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       197 MillanGonzalo_FINAL                     4         187            50          3270       3564,3                 16,77 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       196 MillanGonzalo_FINAL                     4         188            50       3689,32         3800                  6,09 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       172 MillanGonzalo_FINAL                     4         189            50       3704,85         3816                   5,7 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       192 MillanGonzalo_FINAL                     4         190            50          3161      3445,49                 19,54 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       205 MillanGonzalo_FINAL                     4         191            50          2725      2970,25                 30,64 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       232 MillanGonzalo_FINAL                     4         192            50        3883,5         4000                  1,15 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       227 MillanGonzalo_FINAL                     4         193            50       3786,41         3900                  3,62 AJUSTE_BAJO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       203 MillanGonzalo_FINAL                     4         194            50          3488      3697,28                 11,22 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       198 MillanGonzalo_FINAL                     4         195            50          3052      3326,68                 22,31 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       190 MillanGonzalo_FINAL                     4         196            50       3474,63      3683,11                 11,56 AJUSTE_MEDIO                                                                                         GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       211 MillanGonzalo_FINAL                     4         197            50          3270       3564,3                 16,77 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       179 MillanGonzalo_FINAL                     4         198            50          2834      3089,06                 27,86 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       208 MillanGonzalo_FINAL                     4         199            50          2834      3089,06                 27,86 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

  AUDIT_ID EXECUTION_TAG                  VARIANT_ID EMPLOYEE_ID DEPARTMENT_ID SALARY_BEFORE SALARY_AFTER PCT_GAP_TO_AVG_BEFORE RULE_APPLIED                                                                                         EXECUTED_BY                                                                                          EXECUTED NOTES                                                                                                                                                                                                                                                                                                                                                                                                           
---------- ------------------------------ ---------- ----------- ------------- ------------- ------------ --------------------- ---------------------------------------------------------------------------------------------------- ---------------------------------------------------------------------------------------------------- -------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
       201 MillanGonzalo_FINAL                     4         303           290          3193      3288,79                  1,67 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       174 MillanGonzalo_FINAL                     4         305           290       3398,06         3500 -4,65                 NO_APLICA                                                                                            GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        
       194 MillanGonzalo_FINAL                     4         306           290       2660,55         2900                 18,07 AJUSTE_ALTO                                                                                          GAAYALA                                                                                              27/04/26 Ajuste salarial aplicado                                                                                                                                                                                                                                                                                                                                                                                        

80 filas seleccionadas. 




--
-- IMPORTANTE:
-- Todas las validaciones posteriores deben filtrar por &p_execution_tag
-- ============================================================

PROMPT ===== 5. VALIDACIÓN POSTERIOR =====

-- SALIDA 1. EMPLEADOS IMPACTADOS



-- SALIDA 2. RESUMEN ECONÓMICO FINAL



-- SALIDA 3. VALIDACIÓN DE TOPES



-- SALIDA 4. AUDITORÍA GENERADA



-- ============================================================
-- 6. JUSTIFICACIÓN TÉCNICA
-- Responder dentro del script, en comentarios.
-- Cada respuesta debe tener entre 3 y 6 líneas.
-- ============================================================

-- ATOMICIDAD:
-- Explique cómo su solución demuestra atomicidad.
--
-- RESPUESTA:
--La solución demuestra atomicidad al ejecutar todo el proceso
-- (ajuste salarial, auditoría y validación) dentro de una misma
-- unidad de transacción. Si ocurre algún incumplimiento en las
-- validaciones, se realiza un ROLLBACK al SAVEPOINT, evitando
-- que los cambios queden parcialmente aplicados.
-- CONSISTENCIA:
-- Explique cómo su solución asegura que los datos quedan válidos
-- después de la operación.
--
-- RESPUESTA:
-- Se asegura la consistencia mediante reglas de negocio claras,
-- como límites salariales por promedio departamental y filtros
-- de elegibilidad. Además, la validación posterior verifica que
-- ningún salario supere el máximo permitido, garantizando que
-- los datos finales cumplen las restricciones definidas.
-- AISLAMIENTO:
-- Explique cómo se comportaría su transacción frente a otras sesiones.
--
-- RESPUESTA:
-- La transacción se ejecuta de forma aislada, por lo que otras
-- sesiones no ven los cambios hasta que se realiza el COMMIT.
-- Esto evita lecturas inconsistentes o intermedias mientras se
-- aplican los ajustes y se registran en la auditoría.
-- DURABILIDAD:
-- Explique qué garantiza la persistencia del cambio una vez confirmado.
--
-- RESPUESTA:
-- Una vez ejecutado el COMMIT, los cambios quedan almacenados
-- permanentemente en la base de datos. Oracle garantiza que la
-- información persiste incluso ante fallos del sistema, gracias
-- a sus mecanismos internos de redo logs y recuperación.

-- USO DE SAVEPOINT / ROLLBACK:
-- Explique qué riesgo controló y por qué ese punto de restauración
-- era necesario.
--
-- RESPUESTA:
-- El SAVEPOINT permite definir un punto seguro antes de aplicar
-- los ajustes salariales. Si la validación detecta errores, se
-- hace ROLLBACK a este punto, evitando inconsistencias y
-- permitiendo revertir solo esta operación sin afectar otras.
PROMPT ===== Fin de plantilla =====

