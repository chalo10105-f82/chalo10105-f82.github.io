---
title: "Evidencia Clase 4 de Marzo"
publishDate: 2026-04-28
isFeatured: true
---

```sql

--- primero llenar pay emp contracts
--- sacar id payrol
--- if para dividir en 
--- 
INSERT INTO pay_emp_contracts (
    employee_id,
    payroll_type_id,
    base_salary,
    hours_per_month,
    start_date,
    status
)
    SELECT
        e.employee_id,
        CASE
            WHEN e.employee_id <= 2000 THEN
                (
                    SELECT
                        payroll_type_id
                    FROM
                        pay_payroll_types
                    WHERE
                        code = 'MENSUAL'
                )
            WHEN e.employee_id <= 4000 THEN
                (
                    SELECT
                        payroll_type_id
                    FROM
                        pay_payroll_types
                    WHERE
                        code = 'QUINCENAL'
                )
            ELSE
                (
                    SELECT
                        payroll_type_id
                    FROM
                        pay_payroll_types
                    WHERE
                        code = 'SEMANAL'
                )
        END,
        e.salary,
        240,
        sysdate,
        'ACTIVO'
    FROM
        employees e;

CREATE OR REPLACE PROCEDURE ingresar_nomina (
    p_employee_id IN NUMBER,
    p_period_code IN VARCHAR2
) IS

    vv_period_id       pay_periods.period_id%TYPE;
    vv_salary          pay_emp_contracts.base_salary%TYPE;
    vv_payroll_type_id pay_emp_contracts.payroll_type_id%TYPE;
    vv_period_salary   NUMBER(12, 2);
    vv_salud_rate      pay_concepts.default_rate%TYPE;
    vv_pension_rate    pay_concepts.default_rate%TYPE;
    vv_salud           NUMBER(12, 2);
    vv_pension         NUMBER(12, 2);
    vv_payslip_id      pay_payslips.payslip_id%TYPE;
    vv_sal_base_id     pay_concepts.concept_id%TYPE;
    vv_salud_id        pay_concepts.concept_id%TYPE;
    vv_pension_id      pay_concepts.concept_id%TYPE;
    vv_payroll_code    pay_payroll_types.code%TYPE;
BEGIN

    --  periodo
    SELECT
        period_id
    INTO vv_period_id
    FROM
        pay_periods
    WHERE
        period_code = p_period_code;

    --  contrato activo
    SELECT
        c.base_salary,
        c.payroll_type_id,
        t.code
    INTO
        vv_salary,
        vv_payroll_type_id,
        vv_payroll_code
    FROM
             pay_emp_contracts c
        JOIN pay_payroll_types t ON c.payroll_type_id = t.payroll_type_id
    WHERE
            c.employee_id = p_employee_id
        AND c.status = 'ACTIVO';

    -- Calcular salario
    IF vv_payroll_code = 'MENSUAL' THEN
        vv_period_salary := vv_salary;
    ELSIF vv_payroll_code = 'QUINCENAL' THEN
        vv_period_salary := vv_salary / 2;
    ELSIF vv_payroll_code = 'SEMANAL' THEN
        vv_period_salary := vv_salary / 4;
    ELSE
        raise_application_error(-20001, 'Tipo de nómina inválido');
    END IF;

    -- conceptos y tasas en una sola consulta cada uno
    SELECT
        concept_id,
        default_rate
    INTO
        vv_salud_id,
        vv_salud_rate
    FROM
        pay_concepts
    WHERE
        code = 'SALUD_EE';

    SELECT
        concept_id,
        default_rate
    INTO
        vv_pension_id,
        vv_pension_rate
    FROM
        pay_concepts
    WHERE
        code = 'PENSION_EE';

    SELECT
        concept_id
    INTO vv_sal_base_id
    FROM
        pay_concepts
    WHERE
        code = 'SAL_BASE';

    --  Calcular deducciones
    vv_salud := vv_period_salary * vv_salud_rate;
    vv_pension := vv_period_salary * vv_pension_rate;

    --  Insertar cabecera
    INSERT INTO pay_payslips (
        period_id,
        employee_id,
        gross_total,
        ded_total,
        net_total
    ) VALUES ( vv_period_id,
               p_employee_id,
               0,
               0,
               0 ) RETURNING payslip_id INTO vv_payslip_id;

    -- Insertar salario base
    INSERT INTO pay_payslip_lines (
        payslip_id,
        concept_id,
        qty,
        unit_value,
        line_total
    ) VALUES ( vv_payslip_id,
               vv_sal_base_id,
               1,
               vv_period_salary,
               vv_period_salary );

    --  Insertar salud
    INSERT INTO pay_payslip_lines (
        payslip_id,
        concept_id,
        qty,
        unit_value,
        line_total
    ) VALUES ( vv_payslip_id,
               vv_salud_id,
               1,
               - vv_salud,
               - vv_salud );

    --  Insertar pensión
    INSERT INTO pay_payslip_lines (
        payslip_id,
        concept_id,
        qty,
        unit_value,
        line_total
    ) VALUES ( vv_payslip_id,
               vv_pension_id,
               1,
               - vv_pension,
               - vv_pension );

    -- Actualizar totales
    UPDATE pay_payslips
    SET
        gross_total = vv_period_salary,
        ded_total = vv_salud + vv_pension,
        net_total = vv_period_salary - ( vv_salud + vv_pension )
    WHERE
        payslip_id = vv_payslip_id;

    COMMIT;
END ingresar_nomina;
/

CREATE OR REPLACE PROCEDURE generar_nomina_total IS
BEGIN
    FOR vv_period IN (
        SELECT
            period_code
        FROM
            pay_periods
        ORDER BY
            period_code
    ) LOOP
        FOR vv_emp IN (
            SELECT
                employee_id
            FROM
                pay_emp_contracts
            WHERE
                status = 'ACTIVO'
        ) LOOP
            BEGIN
                ingresar_nomina(vv_emp.employee_id, vv_period.period_code);
            EXCEPTION
                WHEN OTHERS THEN
                    NULL;
            END;
        END LOOP;
    END LOOP;
END generar_nomina_total;
/

BEGIN
    generar_nomina_total;
END;
/



--- retornar un valor varchar y recibe un nombre
CREATE OR REPLACE 
FUNCTION fn_saludar_A_Alguien (vv_saludar VARCHAR2)
RETURN VARCHAR2
IS
    vv_resultado VARCHAR2(30):='Hola chiquitas,soy: ';
BEGIN
return (vv_resultado || vv_saludar );
END;
/

DECLARE 
mensa VARCHAR2(30);
BEGIN
mensa := fn_saludar_A_Alguien('Messi');
 dbms_output.put_line(mensa);
END;
/
--- 

--- retornar integral

--- paquetes - sintaxis (Especificacion)
--- CREATE OR REPLACE PACKAGE pck_same
--- PROCEDURE calcularNomina -> Esto es publico
--- FUNCTION calcularDivisas -> Esto es publico
-- END;
-- /

--- paquetes - BODY
--- CREATE OR REPLACE PACKAGE BODY pck_same

--- PROCEDURE calcularNomina -> Esto es publico
--
--
--- END;
--- /
--- FUNCTION calcularDivisas -> Esto es publico 
--
--
--- END;
--- END;


--- FUNCION RETORNA UN SOLO VALOR
--- PROCEDIMIENTO DEVUELVE PARAMETROS O NADA

--- Triggers 