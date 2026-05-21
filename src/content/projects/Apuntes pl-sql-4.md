---
title: "Evidencia Clase 25 de febrero"
publishDate: 2026-04-28
isFeatured: true
---

```sql

CREATE TABLE regions
    ( region_id      NUMBER 
       CONSTRAINT  region_id_nn NOT NULL 
    , region_name    VARCHAR2(25) 
    );

CREATE UNIQUE INDEX reg_id_pk
ON regions (region_id);

ALTER TABLE regions
ADD ( CONSTRAINT reg_id_pk
       		 PRIMARY KEY (region_id)
    ) ;
    
CREATE TABLE countries 
    ( country_id      CHAR(2) 
       CONSTRAINT  country_id_nn NOT NULL 
    , country_name    VARCHAR2(60) 
    , region_id       NUMBER 
    , CONSTRAINT     country_c_id_pk 
        	     PRIMARY KEY (country_id) 
    ) 
    ORGANIZATION INDEX; 

ALTER TABLE countries
ADD ( CONSTRAINT countr_reg_fk
        	 FOREIGN KEY (region_id)
          	  REFERENCES regions(region_id) 
    ) ;
    
CREATE TABLE locations
    ( location_id    NUMBER(4)
    , street_address VARCHAR2(40)
    , postal_code    VARCHAR2(12)
    , city       VARCHAR2(30)
	CONSTRAINT     loc_city_nn  NOT NULL
    , state_province VARCHAR2(25)
    , country_id     CHAR(2)
    ) ;

CREATE UNIQUE INDEX loc_id_pk
ON locations (location_id) ;

ALTER TABLE locations
ADD ( CONSTRAINT loc_id_pk
       		 PRIMARY KEY (location_id)
    , CONSTRAINT loc_c_id_fk
       		 FOREIGN KEY (country_id)
        	  REFERENCES countries(country_id) 
    ) ;
    
    
CREATE SEQUENCE locations_seq
 START WITH     3300
 INCREMENT BY   100
 MAXVALUE       9900
 NOCACHE
 NOCYCLE;
 
 CREATE TABLE departments
    ( department_id    NUMBER(4)
    , department_name  VARCHAR2(30)
	CONSTRAINT  dept_name_nn  NOT NULL
    , manager_id       NUMBER(6)
    , location_id      NUMBER(4)
    ) ;

CREATE UNIQUE INDEX dept_id_pk
ON departments (department_id) ;

ALTER TABLE departments
ADD ( CONSTRAINT dept_id_pk
       		 PRIMARY KEY (department_id)
    , CONSTRAINT dept_loc_fk
       		 FOREIGN KEY (location_id)
        	  REFERENCES locations (location_id)
     ) ;
     
     
     
CREATE SEQUENCE departments_seq
 START WITH     280
 INCREMENT BY   10
 MAXVALUE       9990
 NOCACHE
 NOCYCLE;
 
 
 CREATE TABLE jobs
    ( job_id         VARCHAR2(10)
    , job_title      VARCHAR2(35)
	CONSTRAINT     job_title_nn  NOT NULL
    , min_salary     NUMBER(6)
    , max_salary     NUMBER(6)
    ) ;

CREATE UNIQUE INDEX job_id_pk 
ON jobs (job_id) ;

ALTER TABLE jobs
ADD ( CONSTRAINT job_id_pk
      		 PRIMARY KEY(job_id)
    ) ;
    
    
CREATE TABLE employees
    ( employee_id    NUMBER(6)
    , first_name     VARCHAR2(20)
    , last_name      VARCHAR2(25)
	 CONSTRAINT     emp_last_name_nn  NOT NULL
    , email          VARCHAR2(25)
	CONSTRAINT     emp_email_nn  NOT NULL
    , phone_number   VARCHAR2(20)
    , hire_date      DATE
	CONSTRAINT     emp_hire_date_nn  NOT NULL
    , job_id         VARCHAR2(10)
	CONSTRAINT     emp_job_nn  NOT NULL
    , salary         NUMBER(8,2)
    , commission_pct NUMBER(2,2)
    , manager_id     NUMBER(6)
    , department_id  NUMBER(4)
    , CONSTRAINT     emp_salary_min
                     CHECK (salary > 0) 
    , CONSTRAINT     emp_email_uk
                     UNIQUE (email)
    ) ;

CREATE UNIQUE INDEX emp_emp_id_pk
ON employees (employee_id) ;


ALTER TABLE employees
ADD ( CONSTRAINT     emp_emp_id_pk
                     PRIMARY KEY (employee_id)
    , CONSTRAINT     emp_dept_fk
                     FOREIGN KEY (department_id)
                      REFERENCES departments
    , CONSTRAINT     emp_job_fk
                     FOREIGN KEY (job_id)
                      REFERENCES jobs (job_id)
    , CONSTRAINT     emp_manager_fk
                     FOREIGN KEY (manager_id)
                      REFERENCES employees
    ) ;

ALTER TABLE departments
ADD ( CONSTRAINT dept_mgr_fk
      		 FOREIGN KEY (manager_id)
      		  REFERENCES employees (employee_id)
    ) ;
    
    
    
CREATE SEQUENCE employees_seq
 START WITH     207
 INCREMENT BY   1
 NOCACHE
 NOCYCLE;
 
 CREATE TABLE job_history
    ( employee_id   NUMBER(6)
	 CONSTRAINT    jhist_employee_nn  NOT NULL
    , start_date    DATE
	CONSTRAINT    jhist_start_date_nn  NOT NULL
    , end_date      DATE
	CONSTRAINT    jhist_end_date_nn  NOT NULL
    , job_id        VARCHAR2(10)
	CONSTRAINT    jhist_job_nn  NOT NULL
    , department_id NUMBER(4)
    , CONSTRAINT    jhist_date_interval
                    CHECK (end_date > start_date)
    ) ;

CREATE UNIQUE INDEX jhist_emp_id_st_date_pk 
ON job_history (employee_id, start_date) ;

ALTER TABLE job_history
ADD ( CONSTRAINT jhist_emp_id_st_date_pk
      PRIMARY KEY (employee_id, start_date)
    , CONSTRAINT     jhist_job_fk
                     FOREIGN KEY (job_id)
                     REFERENCES jobs
    , CONSTRAINT     jhist_emp_fk
                     FOREIGN KEY (employee_id)
                     REFERENCES employees
    , CONSTRAINT     jhist_dept_fk
                     FOREIGN KEY (department_id)
                     REFERENCES departments
    ) ;
    
    
CREATE OR REPLACE VIEW emp_details_view
  (employee_id,
   job_id,
   manager_id,
   department_id,
   location_id,
   country_id,
   first_name,
   last_name,
   salary,
   commission_pct,
   department_name,
   job_title,
   city,
   state_province,
   country_name,
   region_name)
AS SELECT
  e.employee_id, 
  e.job_id, 
  e.manager_id, 
  e.department_id,
  d.location_id,
  l.country_id,
  e.first_name,
  e.last_name,
  e.salary,
  e.commission_pct,
  d.department_name,
  j.job_title,
  l.city,
  l.state_province,
  c.country_name,
  r.region_name
FROM
  employees e,
  departments d,
  jobs j,
  locations l,
  countries c,
  regions r
WHERE e.department_id = d.department_id
  AND d.location_id = l.location_id
  AND l.country_id = c.country_id
  AND c.region_id = r.region_id
  AND j.job_id = e.job_id 
WITH READ ONLY;


CREATE INDEX emp_department_ix
       ON employees (department_id);

CREATE INDEX emp_job_ix
       ON employees (job_id);

CREATE INDEX emp_manager_ix
       ON employees (manager_id);

CREATE INDEX emp_name_ix
       ON employees (last_name, first_name);

CREATE INDEX dept_location_ix
       ON departments (location_id);

CREATE INDEX jhist_job_ix
       ON job_history (job_id);

CREATE INDEX jhist_employee_ix
       ON job_history (employee_id);

CREATE INDEX jhist_department_ix
       ON job_history (department_id);

CREATE INDEX loc_city_ix
       ON locations (city);

CREATE INDEX loc_state_province_ix	
       ON locations (state_province);

CREATE INDEX loc_country_ix
       ON locations (country_id);
       

COMMENT ON TABLE regions 
IS 'Regions table that contains region numbers and names. references with the Countries table.';

COMMENT ON COLUMN regions.region_id
IS 'Primary key of regions table.';

COMMENT ON COLUMN regions.region_name
IS 'Names of regions. Locations are in the countries of these regions.';

COMMENT ON TABLE locations
IS 'Locations table that contains specific address of a specific office,
warehouse, and/or production site of a company. Does not store addresses /
locations of customers. references with the departments and countries tables. ';

COMMENT ON COLUMN locations.location_id
IS 'Primary key of locations table';

COMMENT ON COLUMN locations.street_address
IS 'Street address of an office, warehouse, or production site of a company.
Contains building number and street name';

COMMENT ON COLUMN locations.postal_code
IS 'Postal code of the location of an office, warehouse, or production site 
of a company. ';

COMMENT ON COLUMN locations.city
IS 'A not null column that shows city where an office, warehouse, or 
production site of a company is located. ';

COMMENT ON COLUMN locations.state_province
IS 'State or Province where an office, warehouse, or production site of a 
company is located.';

COMMENT ON COLUMN locations.country_id
IS 'Country where an office, warehouse, or production site of a company is
located. Foreign key to country_id column of the countries table.';



COMMENT ON TABLE departments
IS 'Departments table that shows details of departments where employees 
work. references with locations, employees, and job_history tables.';

COMMENT ON COLUMN departments.department_id
IS 'Primary key column of departments table.';

COMMENT ON COLUMN departments.department_name
IS 'A not null column that shows name of a department. Administration, 
Marketing, Purchasing, Human Resources, Shipping, IT, Executive, Public 
Relations, Sales, Finance, and Accounting. ';

COMMENT ON COLUMN departments.manager_id
IS 'Manager_id of a department. Foreign key to employee_id column of employees table. The manager_id column of the employee table references this column.';

COMMENT ON COLUMN departments.location_id
IS 'Location id where a department is located. Foreign key to location_id column of locations table.';


COMMENT ON TABLE job_history
IS 'Table that stores job history of the employees. If an employee 
changes departments within the job or changes jobs within the department, 
new rows get inserted into this table with old job information of the 
employee. Contains a complex primary key: employee_id+start_date.
References with jobs, employees, and departments tables.';

COMMENT ON COLUMN job_history.employee_id
  IS 'A not null column in the complex primary key employee_id+start_date. 
Foreign key to employee_id column of the employee table';

COMMENT ON COLUMN job_history.start_date
  IS 'A not null column in the complex primary key employee_id+start_date. 
Must be less than the end_date of the job_history table. (enforced by 
constraint jhist_date_interval)';

COMMENT ON COLUMN job_history.end_date
  IS 'Last day of the employee in this job role. A not null column. Must be 
greater than the start_date of the job_history table. 
(enforced by constraint jhist_date_interval)';

COMMENT ON COLUMN job_history.job_id
  IS 'Job role in which the employee worked in the past; foreign key to 
job_id column in the jobs table. A not null column.';

COMMENT ON COLUMN job_history.department_id
  IS 'Department id in which the employee worked in the past; foreign key to deparment_id column in the departments table';


COMMENT ON TABLE countries
  IS 'country table. References with locations table.';

COMMENT ON COLUMN countries.country_id
  IS 'Primary key of countries table.';

COMMENT ON COLUMN countries.country_name
  IS 'Country name';

COMMENT ON COLUMN countries.region_id
  IS 'Region ID for the country. Foreign key to region_id column in the departments table.';


COMMENT ON TABLE jobs
  IS 'jobs table with job titles and salary ranges.
References with employees and job_history table.';

COMMENT ON COLUMN jobs.job_id
  IS 'Primary key of jobs table.';

COMMENT ON COLUMN jobs.job_title
  IS 'A not null column that shows job title, e.g. AD_VP, FI_ACCOUNTANT';

COMMENT ON COLUMN jobs.min_salary
  IS 'Minimum salary for a job title.';

COMMENT ON COLUMN jobs.max_salary
  IS 'Maximum salary for a job title';


COMMENT ON TABLE employees
  IS 'employees table. References with departments,
jobs, job_history tables. Contains a self reference.';

COMMENT ON COLUMN employees.employee_id
  IS 'Primary key of employees table.';

COMMENT ON COLUMN employees.first_name
  IS 'First name of the employee. A not null column.';

COMMENT ON COLUMN employees.last_name
  IS 'Last name of the employee. A not null column.';

COMMENT ON COLUMN employees.email
  IS 'Email id of the employee';

COMMENT ON COLUMN employees.phone_number
  IS 'Phone number of the employee; includes country code and area code';

COMMENT ON COLUMN employees.hire_date
  IS 'Date when the employee started on this job. A not null column.';

COMMENT ON COLUMN employees.job_id
  IS 'Current job of the employee; foreign key to job_id column of the 
jobs table. A not null column.';

COMMENT ON COLUMN employees.salary
  IS 'Monthly salary of the employee. Must be greater 
than zero (enforced by constraint emp_salary_min)';

COMMENT ON COLUMN employees.commission_pct
  IS 'Commission percentage of the employee; Only employees in sales 
department elgible for commission percentage';

COMMENT ON COLUMN employees.manager_id
  IS 'Manager id of the employee; has same domain as manager_id in 
departments table. Foreign key to employee_id column of employees table. 
(useful for reflexive joins and CONNECT BY query)';

COMMENT ON COLUMN employees.department_id
  IS 'Department id where employee works; foreign key to department_id 
column of the departments table';

GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO afegonzalezg;
GRANT SELECT, INSERT, UPDATE, DELETE ON regions TO afegonzalezg;
GRANT SELECT, INSERT, UPDATE, DELETE ON countries TO afegonzalezg;
GRANT SELECT, INSERT, UPDATE, DELETE ON locations TO afegonzalezg;
GRANT SELECT, INSERT, UPDATE, DELETE ON departments TO afegonzalezg;
GRANT SELECT, INSERT, UPDATE, DELETE ON jobs TO afegonzalezg;
GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO afegonzalezg;
GRANT SELECT, INSERT, UPDATE, DELETE ON job_history TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON regions TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON countries TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON locations TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON departments TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON jobs TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON job_history TO sscalero;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_payroll_types TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_payroll_types TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_concepts TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_concepts TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_periods TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_periods TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_emp_contracts TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_emp_contracts TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_time_entries TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_time_entries TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_leave_requests TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_leave_requests TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_emp_events TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_emp_events TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_payslips TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_payslips TO afegonzalezg;

GRANT SELECT, INSERT, UPDATE, DELETE ON pay_payslip_lines TO sscalero;
GRANT SELECT, INSERT, UPDATE, DELETE ON pay_payslip_lines TO afegonzalezg;


BEGIN
  INSERT INTO regions VALUES
      ( 10
      , 'Europe'
      );

  INSERT INTO regions VALUES
      ( 20
      , 'Americas'
      );

  INSERT INTO regions VALUES
      ( 30
      , 'Asia'
      );

  INSERT INTO regions VALUES
      ( 40
      , 'Oceania'
      );

  INSERT INTO regions VALUES
      ( 50
      , 'Africa'
      );
END;
/

BEGIN
  INSERT INTO countries VALUES
      ( 'IT'
      , 'Italy'
      , 10
      );

  INSERT INTO countries VALUES
      ( 'JP'
      , 'Japan'
      , 30
      );

  INSERT INTO countries VALUES
      ( 'US'
      , 'United States of America'
      , 20
      );

  INSERT INTO countries VALUES
      ( 'CA'
      , 'Canada'
      , 20
      );

  INSERT INTO countries VALUES
      ( 'CN'
      , 'China'
      , 30
      );

  INSERT INTO countries VALUES
      ( 'IN'
      , 'India'
      , 30
      );

  INSERT INTO countries VALUES
      ( 'AU'
      , 'Australia'
      , 40
      );

  INSERT INTO countries VALUES
      ( 'ZW'
      , 'Zimbabwe'
      , 50
      );

  INSERT INTO countries VALUES
      ( 'SG'
      , 'Singapore'
      , 30
      );

  INSERT INTO countries VALUES
      ( 'GB'
      , 'United Kingdom of Great Britain and Northern Ireland'
      , 10
      );

  INSERT INTO countries VALUES
      ( 'FR'
      , 'France'
      , 10
      );

  INSERT INTO countries VALUES
      ( 'DE'
      , 'Germany'
      , 10
      );

  INSERT INTO countries VALUES
      ( 'ZM'
      , 'Zambia'
      , 50
      );

  INSERT INTO countries VALUES
      ( 'EG'
      , 'Egypt'
      , 50
      );

  INSERT INTO countries VALUES
      ( 'BR'
      , 'Brazil'
      , 20
      );

  INSERT INTO countries VALUES
      ( 'CH'
      , 'Switzerland'
      , 10
      );

  INSERT INTO countries VALUES
      ( 'NL'
      , 'Netherlands'
      , 10
      );

  INSERT INTO countries VALUES
      ( 'MX'
      , 'Mexico'
      , 20
      );

  INSERT INTO countries VALUES
      ( 'KW'
      , 'Kuwait'
      , 30
      );

  INSERT INTO countries VALUES
      ( 'IL'
      , 'Israel'
      , 30
      );

  INSERT INTO countries VALUES
      ( 'DK'
      , 'Denmark'
      , 10
      );

  INSERT INTO countries VALUES
      ( 'ML'
      , 'Malaysia'
      , 30
      );

  INSERT INTO countries VALUES
      ( 'NG'
      , 'Nigeria'
      , 50
      );

  INSERT INTO countries VALUES
      ( 'AR'
      , 'Argentina'
      , 20
      );

  INSERT INTO countries VALUES
      ( 'BE'
      , 'Belgium'
      , 10
      );
END;
/



BEGIN
  INSERT INTO locations VALUES
      ( 1000
      , '1297 Via Cola di Rie'
      , '00989'
      , 'Roma'
      , NULL
      , 'IT'
      );

  INSERT INTO locations VALUES
      ( 1100
      , '93091 Calle della Testa'
      , '10934'
      , 'Venice'
      , NULL
      , 'IT'
      );

  INSERT INTO locations VALUES
      ( 1200
      , '2017 Shinjuku-ku'
      , '1689'
      , 'Tokyo'
      , 'Tokyo Prefecture'
      , 'JP'
      );

  INSERT INTO locations VALUES
      ( 1300
      , '9450 Kamiya-cho'
      , '6823'
      , 'Hiroshima'
      , NULL
      , 'JP'
      );

  INSERT INTO locations VALUES
      ( 1400
      , '2014 Jabberwocky Rd'
      , '26192'
      , 'Southlake'
      , 'Texas'
      , 'US'
      );

  INSERT INTO locations VALUES
      ( 1500
      , '2011 Interiors Blvd'
      , '99236'
      , 'South San Francisco'
      , 'California'
      , 'US'
      );

  INSERT INTO locations VALUES
      ( 1600
      , '2007 Zagora St'
      , '50090'
      , 'South Brunswick'
      , 'New Jersey'
      , 'US'
      );

  INSERT INTO locations VALUES
      ( 1700
      , '2004 Charade Rd'
      , '98199'
      , 'Seattle'
      , 'Washington'
      , 'US'
      );

  INSERT INTO locations VALUES
      ( 1800
      , '147 Spadina Ave'
      , 'M5V 2L7'
      , 'Toronto'
      , 'Ontario'
      , 'CA'
      );

  INSERT INTO locations VALUES
      ( 1900
      , '6092 Boxwood St'
      , 'YSW 9T2'
      , 'Whitehorse'
      , 'Yukon'
      , 'CA'
      );

  INSERT INTO locations VALUES
      ( 2000
      , '40-5-12 Laogianggen'
      , '190518'
      , 'Beijing'
      , NULL
      , 'CN'
      );

  INSERT INTO locations VALUES
      ( 2100
      , '1298 Vileparle (E)'
      , '490231'
      , 'Bombay'
      , 'Maharashtra'
      , 'IN'
      );

  INSERT INTO locations VALUES
      ( 2200
      , '12-98 Victoria Street'
      , '2901'
      , 'Sydney'
      , 'New South Wales'
      , 'AU'
      );

  INSERT INTO locations VALUES
      ( 2300
      , '198 Clementi North'
      , '540198'
      , 'Singapore'
      , NULL
      , 'SG'
      );

  INSERT INTO locations VALUES
      ( 2400
      , '8204 Arthur St'
      , NULL
      , 'London'
      , NULL
      , 'GB'
      );

  INSERT INTO locations VALUES
      ( 2500
      , 'Magdalen Centre, The Oxford Science Park'
      , 'OX9 9ZB'
      , 'Oxford'
      , 'Oxford'
      , 'GB'
      );

  INSERT INTO locations VALUES
      ( 2600
      , '9702 Chester Road'
      , '09629850293'
      , 'Stretford'
      , 'Manchester'
      , 'GB'
      );

  INSERT INTO locations VALUES
      ( 2700
      , 'Schwanthalerstr. 7031'
      , '80925'
      , 'Munich'
      , 'Bavaria'
      , 'DE'
      );

  INSERT INTO locations VALUES
      ( 2800
      , 'Rua Frei Caneca 1360 '
      , '01307-002'
      , 'Sao Paulo'
      , 'Sao Paulo'
      , 'BR'
      );

  INSERT INTO locations VALUES
      ( 2900
      , '20 Rue des Corps-Saints'
      , '1730'
      , 'Geneva'
      , 'Geneve'
      , 'CH'
      );

  INSERT INTO locations VALUES
      ( 3000
      , 'Murtenstrasse 921'
      , '3095'
      , 'Bern'
      , 'BE'
      , 'CH'
      );

  INSERT INTO locations VALUES
      ( 3100
      , 'Pieter Breughelstraat 837'
      , '3029SK'
      , 'Utrecht'
      , 'Utrecht'
      , 'NL'
      );

  INSERT INTO locations VALUES
      ( 3200
      , 'Mariano Escobedo 9991'
      , '11932'
      , 'Mexico City'
      , 'Distrito Federal'
      , 'MX'
      );
END;
/




ALTER TABLE departments
  DISABLE CONSTRAINT dept_mgr_fk;

BEGIN
  INSERT INTO departments VALUES
      ( 10
      , 'Administration'
      , 200
      , 1700
      );

  INSERT INTO departments VALUES
      ( 20
      , 'Marketing'
      , 201
      , 1800
      );

  INSERT INTO departments VALUES
      ( 30
      , 'Purchasing'
      , 114
      , 1700
      );

  INSERT INTO departments VALUES
      ( 40
      , 'Human Resources'
      , 203
      , 2400
      );

  INSERT INTO departments VALUES
      ( 50
      , 'Shipping'
      , 121
      , 1500
      );

  INSERT INTO departments VALUES
      ( 60
      , 'IT'
      , 103
      , 1400
      );

  INSERT INTO departments VALUES
      ( 70
      , 'Public Relations'
      , 204
      , 2700
      );

  INSERT INTO departments VALUES
      ( 80
      , 'Sales'
      , 145
      , 2500
      );

  INSERT INTO departments VALUES
      ( 90
      , 'Executive'
      , 100
      , 1700
      );

  INSERT INTO departments VALUES
      ( 100
      , 'Finance'
      , 108
      , 1700
      );

  INSERT INTO departments VALUES
      ( 110
      , 'Accounting'
      , 205
      , 1700
      );

  INSERT INTO departments VALUES
      ( 120
      , 'Treasury'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 130
      , 'Corporate Tax'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 140
      , 'Control And Credit'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 150
      , 'Shareholder Services'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 160
      , 'Benefits'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 170
      , 'Manufacturing'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 180
      , 'Construction'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 190
      , 'Contracting'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 200
      , 'Operations'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 210
      , 'IT Support'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 220
      , 'NOC'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 230
      , 'IT Helpdesk'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 240
      , 'Government Sales'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 250
      , 'Retail Sales'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 260
      , 'Recruiting'
      , NULL
      , 1700
      );

  INSERT INTO departments VALUES
      ( 270
      , 'Payroll'
      , NULL
      , 1700
      );
END;
/

REM *************************** insert data into the JOBS table

Prompt ****** Populating JOBS table ....

BEGIN
  INSERT INTO jobs VALUES
      ( 'AD_PRES'
      , 'President'
      , 20080
      , 40000
      );
  INSERT INTO jobs VALUES
      ( 'AD_VP'
      , 'Administration Vice President'
      , 15000
      , 30000
      );

  INSERT INTO jobs VALUES
      ( 'AD_ASST'
      , 'Administration Assistant'
      , 3000
      , 6000
      );

  INSERT INTO jobs VALUES
      ( 'FI_MGR'
      , 'Finance Manager'
      , 8200
      , 16000
      );

  INSERT INTO jobs VALUES
      ( 'FI_ACCOUNT'
      , 'Accountant'
      , 4200
      , 9000
      );

  INSERT INTO jobs VALUES
      ( 'AC_MGR'
      , 'Accounting Manager'
      , 8200
      , 16000
      );

  INSERT INTO jobs VALUES
      ( 'AC_ACCOUNT'
      , 'Public Accountant'
      , 4200
      , 9000
      );
  INSERT INTO jobs VALUES
      ( 'SA_MAN'
      , 'Sales Manager'
      , 10000
      , 20080
      );

  INSERT INTO jobs VALUES
      ( 'SA_REP'
      , 'Sales Representative'
      , 6000
      , 12008
      );

  INSERT INTO jobs VALUES
      ( 'PU_MAN'
      , 'Purchasing Manager'
      , 8000
      , 15000
      );

  INSERT INTO jobs VALUES
      ( 'PU_CLERK'
      , 'Purchasing Clerk'
      , 2500
      , 5500
      );

  INSERT INTO jobs VALUES
      ( 'ST_MAN'
      , 'Stock Manager'
      , 5500
      , 8500
      );
  INSERT INTO jobs VALUES
      ( 'ST_CLERK'
      , 'Stock Clerk'
      , 2008
      , 5000
      );

  INSERT INTO jobs VALUES
      ( 'SH_CLERK'
      , 'Shipping Clerk'
      , 2500
      , 5500
      );

  INSERT INTO jobs VALUES
      ( 'IT_PROG'
      , 'Programmer'
      , 4000
      , 10000
      );

  INSERT INTO jobs VALUES
      ( 'MK_MAN'
      , 'Marketing Manager'
      , 9000
      , 15000
      );

  INSERT INTO jobs VALUES
      ( 'MK_REP'
      , 'Marketing Representative'
      , 4000
      , 9000
      );

  INSERT INTO jobs VALUES
      ( 'HR_REP'
      , 'Human Resources Representative'
      , 4000
      , 9000
      );

  INSERT INTO jobs VALUES
      ( 'PR_REP'
      , 'Public Relations Representative'
      , 4500
      , 10500
      );
END;
/


REM *************************** insert data into the EMPLOYEES table

Prompt ****** Populating EMPLOYEES table ....

BEGIN
  INSERT INTO employees VALUES
      ( 100
      , 'Steven'
      , 'King'
      , 'SKING'
      , '1.515.555.0100'
      , TO_DATE('17-06-2013', 'dd-MM-yyyy')
      , 'AD_PRES'
      , 24000
      , NULL
      , NULL
      , 90
      );

  INSERT INTO employees VALUES
      ( 101
      , 'Neena'
      , 'Yang'
      , 'NYANG'
      , '1.515.555.0101'
      , TO_DATE('21-09-2015', 'dd-MM-yyyy')
      , 'AD_VP'
      , 17000
      , NULL
      , 100
      , 90
      );

  INSERT INTO employees VALUES
      ( 102
      , 'Lex'
      , 'Garcia'
      , 'LGARCIA'
      , '1.515.555.0102'
      , TO_DATE('13-01-2011', 'dd-MM-yyyy')
      , 'AD_VP'
      , 17000
      , NULL
      , 100
      , 90
      );

  INSERT INTO employees VALUES
      ( 103
      , 'Alexander'
      , 'James'
      , 'AJAMES'
      , '1.590.555.0103'
      , TO_DATE('03-01-2016', 'dd-MM-yyyy')
      , 'IT_PROG'
      , 9000
      , NULL
      , 102
      , 60
      );

  INSERT INTO employees VALUES
      ( 104
      , 'Bruce'
      , 'Miller'
      , 'BMILLER'
      , '1.590.555.0104'
      , TO_DATE('21-05-2017', 'dd-MM-yyyy')
      , 'IT_PROG'
      , 6000
      , NULL
      , 103
      , 60
      );

  INSERT INTO employees VALUES
      ( 105
      , 'David'
      , 'Williams'
      , 'DWILLIAMS'
      , '1.590.555.0105'
      , TO_DATE('25-06-2015', 'dd-MM-yyyy')
      , 'IT_PROG'
      , 4800
      , NULL
      , 103
      , 60
      );

  INSERT INTO employees VALUES
      ( 106
      , 'Valli'
      , 'Jackson'
      , 'VJACKSON'
      , '1.590.555.0106'
      , TO_DATE('05-02-2016', 'dd-MM-yyyy')
      , 'IT_PROG'
      , 4800
      , NULL
      , 103
      , 60
      );

  INSERT INTO employees VALUES
      ( 107
      , 'Diana'
      , 'Nguyen'
      , 'DNGUYEN'
      , '1.590.555.0107'
      , TO_DATE('07-02-2017', 'dd-MM-yyyy')
      , 'IT_PROG'
      , 4200
      , NULL
      , 103
      , 60
      );

  INSERT INTO employees VALUES
      ( 108
      , 'Nancy'
      , 'Gruenberg'
      , 'NGRUENBE'
      , '1.515.555.0108'
      , TO_DATE('17-08-2012', 'dd-MM-yyyy')
      , 'FI_MGR'
      , 12008
      , NULL
      , 101
      , 100
      );

  INSERT INTO employees VALUES
      ( 109
      , 'Daniel'
      , 'Faviet'
      , 'DFAVIET'
      , '1.515.555.0109'
      , TO_DATE('16-08-2012', 'dd-MM-yyyy')
      , 'FI_ACCOUNT'
      , 9000
      , NULL
      , 108
      , 100
      );

  INSERT INTO employees VALUES
      ( 110
      , 'John'
      , 'Chen'
      , 'JCHEN'
      , '1.515.555.0110'
      , TO_DATE('28-09-2015', 'dd-MM-yyyy')
      , 'FI_ACCOUNT'
      , 8200
      , NULL
      , 108
      , 100
      );

  INSERT INTO employees VALUES
      ( 111
      , 'Ismael'
      , 'Sciarra'
      , 'ISCIARRA'
      , '1.515.555.0111'
      , TO_DATE('30-09-2015', 'dd-MM-yyyy')
      , 'FI_ACCOUNT'
      , 7700
      , NULL
      , 108
      , 100
      );

  INSERT INTO employees VALUES
      ( 112
      , 'Jose Manuel'
      , 'Urman'
      , 'JMURMAN'
      , '1.515.555.0112'
      , TO_DATE('07-03-2016', 'dd-MM-yyyy')
      , 'FI_ACCOUNT'
      , 7800
      , NULL
      , 108
      , 100
      );

  INSERT INTO employees VALUES
      ( 113
      , 'Luis'
      , 'Popp'
      , 'LPOPP'
      , '1.515.555.0113'
      , TO_DATE('07-12-2017', 'dd-MM-yyyy')
      , 'FI_ACCOUNT'
      , 6900
      , NULL
      , 108
      , 100
      );

  INSERT INTO employees VALUES
      ( 114
      , 'Den'
      , 'Li'
      , 'DLI'
      , '1.515.555.0114'
      , TO_DATE('07-12-2012', 'dd-MM-yyyy')
      , 'PU_MAN'
      , 11000
      , NULL
      , 100
      , 30
      );

  INSERT INTO employees VALUES
      ( 115
      , 'Alexander'
      , 'Khoo'
      , 'AKHOO'
      , '1.515.555.0115'
      , TO_DATE('18-05-2013', 'dd-MM-yyyy')
      , 'PU_CLERK'
      , 3100
      , NULL
      , 114
      , 30
      );

  INSERT INTO employees VALUES
      ( 116
      , 'Shelli'
      , 'Baida'
      , 'SBAIDA'
      , '1.515.555.0116'
      , TO_DATE('24-12-2015', 'dd-MM-yyyy')
      , 'PU_CLERK'
      , 2900
      , NULL
      , 114
      , 30
      );

  INSERT INTO employees VALUES
      ( 117
      , 'Sigal'
      , 'Tobias'
      , 'STOBIAS'
      , '1.515.555.0117'
      , TO_DATE('24-07-2015', 'dd-MM-yyyy')
      , 'PU_CLERK'
      , 2800
      , NULL
      , 114
      , 30
      );

  INSERT INTO employees VALUES
      ( 118
      , 'Guy'
      , 'Himuro'
      , 'GHIMURO'
      , '1.515.555.0118'
      , TO_DATE('15-11-2016', 'dd-MM-yyyy')
      , 'PU_CLERK'
      , 2600
      , NULL
      , 114
      , 30
      );

  INSERT INTO employees VALUES
      ( 119
      , 'Karen'
      , 'Colmenares'
      , 'KCOLMENA'
      , '1.515.555.0119'
      , TO_DATE('10-08-2017', 'dd-MM-yyyy')
      , 'PU_CLERK'
      , 2500
      , NULL
      , 114
      , 30
      );

  INSERT INTO employees VALUES
      ( 120
      , 'Matthew'
      , 'Weiss'
      , 'MWEISS'
      , '1.650.555.0120'
      , TO_DATE('18-07-2014', 'dd-MM-yyyy')
      , 'ST_MAN'
      , 8000
      , NULL
      , 100
      , 50
      );

  INSERT INTO employees VALUES
      ( 121
      , 'Adam'
      , 'Fripp'
      , 'AFRIPP'
      , '1.650.555.0121'
      , TO_DATE('10-04-2015', 'dd-MM-yyyy')
      , 'ST_MAN'
      , 8200
      , NULL
      , 100
      , 50
      );

  INSERT INTO employees VALUES
      ( 122
      , 'Payam'
      , 'Kaufling'
      , 'PKAUFLIN'
      , '1.650.555.0122'
      , TO_DATE('01-05-2013', 'dd-MM-yyyy')
      , 'ST_MAN'
      , 7900
      , NULL
      , 100
      , 50
      );

  INSERT INTO employees VALUES
      ( 123
      , 'Shanta'
      , 'Vollman'
      , 'SVOLLMAN'
      , '1.650.555.0123'
      , TO_DATE('10-10-2015', 'dd-MM-yyyy')
      , 'ST_MAN'
      , 6500
      , NULL
      , 100
      , 50
      );

  INSERT INTO employees VALUES
      ( 124
      , 'Kevin'
      , 'Mourgos'
      , 'KMOURGOS'
      , '1.650.555.0124'
      , TO_DATE('16-11-2017', 'dd-MM-yyyy')
      , 'ST_MAN'
      , 5800
      , NULL
      , 100
      , 50
      );

  INSERT INTO employees VALUES
      ( 125
      , 'Julia'
      , 'Nayer'
      , 'JNAYER'
      , '1.650.555.0125'
      , TO_DATE('16-07-2015', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 3200
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 126
      , 'Irene'
      , 'Mikkilineni'
      , 'IMIKKILI'
      , '1.650.555.0126'
      , TO_DATE('28-09-2016', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2700
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 127
      , 'James'
      , 'Landry'
      , 'JLANDRY'
      , '1.650.555.0127'
      , TO_DATE('14-01-2017', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2400
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 128
      , 'Steven'
      , 'Markle'
      , 'SMARKLE'
      , '1.650.555.0128'
      , TO_DATE('08-03-2018', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2200
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 129
      , 'Laura'
      , 'Bissot'
      , 'LBISSOT'
      , '1.650.555.0129'
      , TO_DATE('20-08-2015', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 3300
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 130
      , 'Mozhe'
      , 'Atkinson'
      , 'MATKINSO'
      , '1.650.555.0130'
      , TO_DATE('30-10-2015', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2800
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 131
      , 'James'
      , 'Marlow'
      , 'JAMRLOW'
      , '1.650.555.0131'
      , TO_DATE('16-02-2015', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2500
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 132
      , 'TJ'
      , 'Olson'
      , 'TJOLSON'
      , '1.650.555.0132'
      , TO_DATE('10-04-2017', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2100
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 133
      , 'Jason'
      , 'Mallin'
      , 'JMALLIN'
      , '1.650.555.0133'
      , TO_DATE('14-06-2014', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 3300
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 134
      , 'Michael'
      , 'Rogers'
      , 'MROGERS'
      , '1.650.555.0134'
      , TO_DATE('26-08-2016', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2900
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 135
      , 'Ki'
      , 'Gee'
      , 'KGEE'
      , '1.650.555.0135'
      , TO_DATE('12-12-2017', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2400
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 136
      , 'Hazel'
      , 'Philtanker'
      , 'HPHILTAN'
      , '1.650.555.0136'
      , TO_DATE('06-02-2018', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2200
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 137
      , 'Renske'
      , 'Ladwig'
      , 'RLADWIG'
      , '1.650.555.0137'
      , TO_DATE('14-07-2013', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 3600
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 138
      , 'Stephen'
      , 'Stiles'
      , 'SSTILES'
      , '1.650.555.0138'
      , TO_DATE('26-10-2015', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 3200
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 139
      , 'John'
      , 'Seo'
      , 'JSEO'
      , '1.650.555.0139'
      , TO_DATE('12-02-2016', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2700
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 140
      , 'Joshua'
      , 'Patel'
      , 'JPATEL'
      , '1.650.555.0140'
      , TO_DATE('06-04-2016', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2500
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 141
      , 'Trenna'
      , 'Rajs'
      , 'TRAJS'
      , '1.650.555.0141'
      , TO_DATE('17-10-2013', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 3500
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 142
      , 'Curtis'
      , 'Davies'
      , 'CDAVIES'
      , '1.650.555.0142'
      , TO_DATE('29-01-2015', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 3100
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 143
      , 'Randall'
      , 'Matos'
      , 'RMATOS'
      , '1.650.555.0143'
      , TO_DATE('15-03-2016', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2600
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 144
      , 'Peter'
      , 'Vargas'
      , 'PVARGAS'
      , '1.650.555.0144'
      , TO_DATE('09-07-2016', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 2500
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 145
      , 'John'
      , 'Singh'
      , 'JSINGH'
      , '44.1632.960000'
      , TO_DATE('01-10-2014', 'dd-MM-yyyy')
      , 'SA_MAN'
      , 14000
      , .4
      , 100
      , 80
      );

  INSERT INTO employees VALUES
      ( 146
      , 'Karen'
      , 'Partners'
      , 'KPARTNER'
      , '44.1632.960001'
      , TO_DATE('05-01-2015', 'dd-MM-yyyy')
      , 'SA_MAN'
      , 13500
      , .3
      , 100
      , 80
      );

  INSERT INTO employees VALUES
      ( 147
      , 'Alberto'
      , 'Errazuriz'
      , 'AERRAZUR'
      , '44.1632.960002'
      , TO_DATE('10-03-2015', 'dd-MM-yyyy')
      , 'SA_MAN'
      , 12000
      , .3
      , 100
      , 80
      );

  INSERT INTO employees VALUES
      ( 148
      , 'Gerald'
      , 'Cambrault'
      , 'GCAMBRAU'
      , '44.1632.960003'
      , TO_DATE('15-10-2017', 'dd-MM-yyyy')
      , 'SA_MAN'
      , 11000
      , .3
      , 100
      , 80
      );

  INSERT INTO employees VALUES
      ( 149
      , 'Eleni'
      , 'Zlotkey'
      , 'EZLOTKEY'
      , '44.1632.960004'
      , TO_DATE('29-01-2018', 'dd-MM-yyyy')
      , 'SA_MAN'
      , 10500
      , .2
      , 100
      , 80
      );

  INSERT INTO employees VALUES
      ( 150
      , 'Sean'
      , 'Tucker'
      , 'STUCKER'
      , '44.1632.960005'
      , TO_DATE('30-01-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 10000
      , .3
      , 145
      , 80
      );

  INSERT INTO employees VALUES
      ( 151
      , 'David'
      , 'Bernstein'
      , 'DBERNSTE'
      , '44.1632.960006'
      , TO_DATE('24-03-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 9500
      , .25
      , 145
      , 80
      );

  INSERT INTO employees VALUES
      ( 152
      , 'Peter'
      , 'Hall'
      , 'PHALL'
      , '44.1632.960007'
      , TO_DATE('20-08-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 9000
      , .25
      , 145
      , 80
      );

  INSERT INTO employees VALUES
      ( 153
      , 'Christopher'
      , 'Olsen'
      , 'COLSEN'
      , '44.1632.960008'
      , TO_DATE('30-03-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 8000
      , .2
      , 145
      , 80
      );

  INSERT INTO employees VALUES
      ( 154
      , 'Nanette'
      , 'Cambrault'
      , 'NCAMBRAU'
      , '44.1632.960009'
      , TO_DATE('09-12-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7500
      , .2
      , 145
      , 80
      );

  INSERT INTO employees VALUES
      ( 155
      , 'Oliver'
      , 'Tuvault'
      , 'OTUVAULT'
      , '44.1632.960010'
      , TO_DATE('23-11-2017', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7000
      , .15
      , 145
      , 80
      );

  INSERT INTO employees VALUES
      ( 156
      , 'Janette'
      , 'King'
      , 'JKING'
      , '44.1632.960011'
      , TO_DATE('30-01-2014', 'dd-MM-yyyy')
      , 'SA_REP'
      , 10000
      , .35
      , 146
      , 80
      );

  INSERT INTO employees VALUES
      ( 157
      , 'Patrick'
      , 'Sully'
      , 'PSULLY'
      , '44.1632.960012'
      , TO_DATE('04-03-2014', 'dd-MM-yyyy')
      , 'SA_REP'
      , 9500
      , .35
      , 146
      , 80
      );

  INSERT INTO employees VALUES
      ( 158
      , 'Allan'
      , 'McEwen'
      , 'AMCEWEN'
      , '44.1632.960013'
      , TO_DATE('01-08-2014', 'dd-MM-yyyy')
      , 'SA_REP'
      , 9000
      , .35
      , 146
      , 80
      );

  INSERT INTO employees VALUES
      ( 159
      , 'Lindsey'
      , 'Smith'
      , 'LSMITH'
      , '44.1632.960014'
      , TO_DATE('10-03-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 8000
      , .3
      , 146
      , 80
      );

  INSERT INTO employees VALUES
      ( 160
      , 'Louise'
      , 'Doran'
      , 'LDORAN'
      , '44.1632.960015'
      , TO_DATE('15-12-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7500
      , .3
      , 146
      , 80
      );

  INSERT INTO employees VALUES
      ( 161
      , 'Sarath'
      , 'Sewall'
      , 'SSEWALL'
      , '44.1632.960016'
      , TO_DATE('03-11-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7000
      , .25
      , 146
      , 80
      );

  INSERT INTO employees VALUES
      ( 162
      , 'Clara'
      , 'Vishney'
      , 'CVISHNEY'
      , '44.1632.960017'
      , TO_DATE('11-11-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 10500
      , .25
      , 147
      , 80
      );

  INSERT INTO employees VALUES
      ( 163
      , 'Danielle'
      , 'Greene'
      , 'DGREENE'
      , '44.1632.960018'
      , TO_DATE('19-03-2017', 'dd-MM-yyyy')
      , 'SA_REP'
      , 9500
      , .15
      , 147
      , 80
      );

  INSERT INTO employees VALUES
      ( 164
      , 'Mattea'
      , 'Marvins'
      , 'MMARVINS'
      , '44.1632.960019'
      , TO_DATE('24-01-2018', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7200
      , .10
      , 147
      , 80
      );

  INSERT INTO employees VALUES
      ( 165
      , 'David'
      , 'Lee'
      , 'DLEE'
      , '44.1632.960020'
      , TO_DATE('23-02-2018', 'dd-MM-yyyy')
      , 'SA_REP'
      , 6800
      , .1
      , 147
      , 80
      );

  INSERT INTO employees VALUES
      ( 166
      , 'Sundar'
      , 'Ande'
      , 'SANDE'
      , '44.1632.960021'
      , TO_DATE('24-03-2018', 'dd-MM-yyyy')
      , 'SA_REP'
      , 6400
      , .10
      , 147
      , 80
      );

  INSERT INTO employees VALUES
      ( 167
      , 'Amit'
      , 'Banda'
      , 'ABANDA'
      , '44.1632.960022'
      , TO_DATE('21-04-2018', 'dd-MM-yyyy')
      , 'SA_REP'
      , 6200
      , .10
      , 147
      , 80
      );

  INSERT INTO employees VALUES
      ( 168
      , 'Lisa'
      , 'Ozer'
      , 'LOZER'
      , '44.1632.960023'
      , TO_DATE('11-03-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 11500
      , .25
      , 148
      , 80
      );

  INSERT INTO employees VALUES
      ( 169
      , 'Harrison'
      , 'Bloom'
      , 'HBLOOM'
      , '44.1632.960024'
      , TO_DATE('23-03-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 10000
      , .20
      , 148
      , 80
      );

  INSERT INTO employees VALUES
      ( 170
      , 'Tayler'
      , 'Fox'
      , 'TFOX'
      , '44.1632.960025'
      , TO_DATE('24-01-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 9600
      , .20
      , 148
      , 80
      );

  INSERT INTO employees VALUES
      ( 171
      , 'William'
      , 'Smith'
      , 'WSMITH'
      , '44.1632.960026'
      , TO_DATE('23-02-2017', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7400
      , .15
      , 148
      , 80
      );

  INSERT INTO employees VALUES
      ( 172
      , 'Elizabeth'
      , 'Bates'
      , 'EBATES'
      , '44.1632.960027'
      , TO_DATE('24-03-2017', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7300
      , .15
      , 148
      , 80
      );

  INSERT INTO employees VALUES
      ( 173
      , 'Sundita'
      , 'Kumar'
      , 'SKUMAR'
      , '44.1632.960028'
      , TO_DATE('21-04-2018', 'dd-MM-yyyy')
      , 'SA_REP'
      , 6100
      , .10
      , 148
      , 80
      );

  INSERT INTO employees VALUES
      ( 174
      , 'Ellen'
      , 'Abel'
      , 'EABEL'
      , '44.1632.960029'
      , TO_DATE('11-05-2014', 'dd-MM-yyyy')
      , 'SA_REP'
      , 11000
      , .30
      , 149
      , 80
      );

  INSERT INTO employees VALUES
      ( 175
      , 'Alyssa'
      , 'Hutton'
      , 'AHUTTON'
      , '44.1632.960030'
      , TO_DATE('19-03-2015', 'dd-MM-yyyy')
      , 'SA_REP'
      , 8800
      , .25
      , 149
      , 80
      );

  INSERT INTO employees VALUES
      ( 176
      , 'Jonathon'
      , 'Taylor'
      , 'JTAYLOR'
      , '44.1632.960031'
      , TO_DATE('24-03-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 8600
      , .20
      , 149
      , 80
      );

  INSERT INTO employees VALUES
      ( 177
      , 'Jack'
      , 'Livingston'
      , 'JLIVINGS'
      , '44.1632.960032'
      , TO_DATE('23-04-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 8400
      , .20
      , 149
      , 80
      );

  INSERT INTO employees VALUES
      ( 178
      , 'Kimberely'
      , 'Grant'
      , 'KGRANT'
      , '44.1632.960033'
      , TO_DATE('24-05-2017', 'dd-MM-yyyy')
      , 'SA_REP'
      , 7000
      , .15
      , 149
      , NULL
      );

  INSERT INTO employees VALUES
      ( 179
      , 'Charles'
      , 'Johnson'
      , 'CJOHNSON'
      , '44.1632.960034'
      , TO_DATE('04-01-2018', 'dd-MM-yyyy')
      , 'SA_REP'
      , 6200
      , .10
      , 149
      , 80
      );

  INSERT INTO employees VALUES
      ( 180
      , 'Winston'
      , 'Taylor'
      , 'WTAYLOR'
      , '1.650.555.0145'
      , TO_DATE('24-01-2016', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3200
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 181
      , 'Jean'
      , 'Fleaur'
      , 'JFLEAUR'
      , '1.650.555.0146'
      , TO_DATE('23-02-2016', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3100
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 182
      , 'Martha'
      , 'Sullivan'
      , 'MSULLIVA'
      , '1.650.555.0147'
      , TO_DATE('21-06-2017', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 2500
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 183
      , 'Girard'
      , 'Geoni'
      , 'GGEONI'
      , '1.650.555.0148'
      , TO_DATE('03-02-2018', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 2800
      , NULL
      , 120
      , 50
      );

  INSERT INTO employees VALUES
      ( 184
      , 'Nandita'
      , 'Sarchand'
      , 'NSARCHAN'
      , '1.650.555.0149'
      , TO_DATE('27-01-2014', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 4200
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 185
      , 'Alexis'
      , 'Bull'
      , 'ABULL'
      , '1.650.555.0150'
      , TO_DATE('20-02-2015', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 4100
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 186
      , 'Julia'
      , 'Dellinger'
      , 'JDELLING'
      , '1.650.555.0151'
      , TO_DATE('24-06-2016', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3400
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 187
      , 'Anthony'
      , 'Cabrio'
      , 'ACABRIO'
      , '1.650.555.0152'
      , TO_DATE('07-02-2017', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3000
      , NULL
      , 121
      , 50
      );

  INSERT INTO employees VALUES
      ( 188
      , 'Kelly'
      , 'Chung'
      , 'KCHUNG'
      , '1.650.555.0153'
      , TO_DATE('14-06-2015', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3800
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 189
      , 'Jennifer'
      , 'Dilly'
      , 'JDILLY'
      , '1.650.555.0154'
      , TO_DATE('13-08-2015', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3600
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 190
      , 'Timothy'
      , 'Venzl'
      , 'TVENZL'
      , '1.650.555.0155'
      , TO_DATE('11-07-2016', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 2900
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 191
      , 'Randall'
      , 'Perkins'
      , 'RPERKINS'
      , '1.650.555.0156'
      , TO_DATE('19-12-2017', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 2500
      , NULL
      , 122
      , 50
      );

  INSERT INTO employees VALUES
      ( 192
      , 'Sarah'
      , 'Bell'
      , 'SBELL'
      , '1.650.555.0157'
      , TO_DATE('04-02-2014', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 4000
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 193
      , 'Britney'
      , 'Everett'
      , 'BEVERETT'
      , '1.650.555.0158'
      , TO_DATE('03-03-2015', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3900
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 194
      , 'Samuel'
      , 'McLeod'
      , 'SMCLEOD'
      , '1.650.555.0159'
      , TO_DATE('01-07-2016', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3200
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 195
      , 'Vance'
      , 'Jones'
      , 'VJONES'
      , '1.650.555.0160'
      , TO_DATE('17-03-2017', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 2800
      , NULL
      , 123
      , 50
      );

  INSERT INTO employees VALUES
      ( 196
      , 'Alana'
      , 'Walsh'
      , 'AWALSH'
      , '1.650.555.0161'
      , TO_DATE('24-04-2016', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3100
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 197
      , 'Kevin'
      , 'Feeney'
      , 'KFEENEY'
      , '1.650.555.0162'
      , TO_DATE('23-05-2016', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 3000
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 198
      , 'Donald'
      , 'OConnell'
      , 'DOCONNEL'
      , '1.650.555.0163'
      , TO_DATE('21-06-2017', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 2600
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 199
      , 'Douglas'
      , 'Grant'
      , 'DGRANT'
      , '1.650.555.0164'
      , TO_DATE('13-01-2018', 'dd-MM-yyyy')
      , 'SH_CLERK'
      , 2600
      , NULL
      , 124
      , 50
      );

  INSERT INTO employees VALUES
      ( 200
      , 'Jennifer'
      , 'Whalen'
      , 'JWHALEN'
      , '1.515.555.0165'
      , TO_DATE('17-09-2013', 'dd-MM-yyyy')
      , 'AD_ASST'
      , 4400
      , NULL
      , 101
      , 10
      );

  INSERT INTO employees VALUES
      ( 201
      , 'Michael'
      , 'Martinez'
      , 'MMARTINE'
      , '1.515.555.0166'
      , TO_DATE('17-02-2014', 'dd-MM-yyyy')
      , 'MK_MAN'
      , 13000
      , NULL
      , 100
      , 20
      );

  INSERT INTO employees VALUES
      ( 202
      , 'Pat'
      , 'Davis'
      , 'PDAVIS'
      , '1.603.555.0167'
      , TO_DATE('17-08-2015', 'dd-MM-yyyy')
      , 'MK_REP'
      , 6000
      , NULL
      , 201
      , 20
      );

  INSERT INTO employees VALUES
      ( 203
      , 'Susan'
      , 'Jacobs'
      , 'SJACOBS'
      , '1.515.555.0168'
      , TO_DATE('07-06-2012', 'dd-MM-yyyy')
      , 'HR_REP'
      , 6500
      , NULL
      , 101
      , 40
      );

  INSERT INTO employees VALUES
      ( 204
      , 'Hermann'
      , 'Brown'
      , 'HBROWN'
      , '1.515.555.0169'
      , TO_DATE('07-06-2012', 'dd-MM-yyyy')
      , 'PR_REP'
      , 10000
      , NULL
      , 101
      , 70
      );

  INSERT INTO employees VALUES
      ( 205
      , 'Shelley'
      , 'Higgins'
      , 'SHIGGINS'
      , '1.515.555.0170'
      , TO_DATE('07-06-2012', 'dd-MM-yyyy')
      , 'AC_MGR'
      , 12008
      , NULL
      , 101
      , 110
      );

  INSERT INTO employees VALUES
      ( 206
      , 'William'
      , 'Gietz'
      , 'WGIETZ'
      , '1.515.555.0171'
      , TO_DATE('07-06-2012', 'dd-MM-yyyy')
      , 'AC_ACCOUNT'
      , 8300
      , NULL
      , 205
      , 110
      );
END;
/

REM ********* insert data into the JOB_HISTORY table

Prompt ****** Populating JOB_HISTORY table ....

BEGIN
  INSERT INTO job_history
  VALUES (102
       , TO_DATE('13-01-2011', 'dd-MM-yyyy')
       , TO_DATE('24-07-2016', 'dd-MM-yyyy')
       , 'IT_PROG'
       , 60);

  INSERT INTO job_history
  VALUES (101
       , TO_DATE('21-09-2007', 'dd-MM-yyyy')
       , TO_DATE('27-10-2011', 'dd-MM-yyyy')
       , 'AC_ACCOUNT'
       , 110);

  INSERT INTO job_history
  VALUES (101
       , TO_DATE('28-10-2011', 'dd-MM-yyyy')
       , TO_DATE('15-03-2015', 'dd-MM-yyyy')
       , 'AC_MGR'
       , 110);

  INSERT INTO job_history
  VALUES (201
       , TO_DATE('17-02-2014', 'dd-MM-yyyy')
       , TO_DATE('19-12-2017', 'dd-MM-yyyy')
       , 'MK_REP'
       , 20);

  INSERT INTO job_history
  VALUES  (114
      , TO_DATE('24-03-2016', 'dd-MM-yyyy')
      , TO_DATE('31-12-2017', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 50
      );

  INSERT INTO job_history
  VALUES  (122
      , TO_DATE('01-01-2017', 'dd-MM-yyyy')
      , TO_DATE('31-12-2017', 'dd-MM-yyyy')
      , 'ST_CLERK'
      , 50
      );

  INSERT INTO job_history
  VALUES  (200
      , TO_DATE('17-09-2005', 'dd-MM-yyyy')
      , TO_DATE('17-06-2011', 'dd-MM-yyyy')
      , 'AD_ASST'
      , 90
      );

  INSERT INTO job_history
  VALUES  (176
      , TO_DATE('24-03-2016', 'dd-MM-yyyy')
      , TO_DATE('31-12-2016', 'dd-MM-yyyy')
      , 'SA_REP'
      , 80
      );

  INSERT INTO job_history
  VALUES  (176
      , TO_DATE('01-01-2017', 'dd-MM-yyyy')
      , TO_DATE('31-12-2017', 'dd-MM-yyyy')
      , 'SA_MAN'
      , 80
      );

  INSERT INTO job_history
  VALUES  (200
      , TO_DATE('01-07-2012', 'dd-MM-yyyy')
      , TO_DATE('31-12-2016', 'dd-MM-yyyy')
      , 'AC_ACCOUNT'
      , 90
      );
END;
/

COMMIT;

REM enable integrity constraint to DEPARTMENTS

ALTER TABLE departments
  ENABLE CONSTRAINT dept_mgr_fk;
  
  
  -- 1) Tipos de nómina
CREATE TABLE pay_payroll_types (--> crear los tipos, reales que sean minimo 3
  payroll_type_id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  code            VARCHAR2(20) UNIQUE NOT NULL,  -- MENSUAL, QUINCENAL
  description     VARCHAR2(100) NOT NULL
);

INSERT INTO pay_payroll_types (code, description)
VALUES ('MENSUAL', 'Pago de nómina mensual');

INSERT INTO pay_payroll_types (code, description)
VALUES ('QUINCENAL', 'Pago de nómina cada quincena');

INSERT INTO pay_payroll_types (code, description)
VALUES ('SEMANAL', 'Pago de nómina semanal');

-- 2) Conceptos
CREATE TABLE pay_concepts (--> preguntar a la IA, defnir conceptos a pagar, minimo 80 que sean reales
  concept_id      NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  code            VARCHAR2(30) UNIQUE NOT NULL,   -- HED, HEN, VAC, SAL_BASE
  name            VARCHAR2(100) NOT NULL,
  concept_type    VARCHAR2(10)  NOT NULL CHECK (concept_type IN ('DEVENGO','DEDUCCION')),
  calc_method     VARCHAR2(15)  NOT NULL CHECK (calc_method IN ('FIJO','PORCENTAJE','POR_HORA','POR_DIA')),
  default_rate    NUMBER(12,4),                   -- % o tarifa por hora/día
  is_active       CHAR(1) DEFAULT 'Y' CHECK (is_active IN ('Y','N'))
);


DECLARE
  PROCEDURE upsert_concept(
    p_code         IN VARCHAR2,
    p_name         IN VARCHAR2,
    p_concept_type IN VARCHAR2,
    p_calc_method  IN VARCHAR2,
    p_default_rate IN NUMBER,
    p_is_active    IN CHAR
  ) IS
  BEGIN
    MERGE INTO pay_concepts c
    USING (
      SELECT
        p_code         AS code,
        p_name         AS name,
        p_concept_type AS concept_type,
        p_calc_method  AS calc_method,
        p_default_rate AS default_rate,
        p_is_active    AS is_active
      FROM dual
    ) s
    ON (c.code = s.code)
    WHEN MATCHED THEN
      UPDATE SET
        c.name         = s.name,
        c.concept_type = s.concept_type,
        c.calc_method  = s.calc_method,
        c.default_rate = s.default_rate,
        c.is_active    = s.is_active
    WHEN NOT MATCHED THEN
      INSERT (code, name, concept_type, calc_method, default_rate, is_active)
      VALUES (s.code, s.name, s.concept_type, s.calc_method, s.default_rate, s.is_active);
  END;
BEGIN
  -- ===================== DEVENGOS =====================
  upsert_concept('SAL_BASE', 'Salario base mensual', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('HORA_NORMAL', 'Hora ordinaria', 'DEVENGO', 'POR_HORA', 1.0000, 'Y');
  upsert_concept('HED', 'Hora extra diurna (125%)', 'DEVENGO', 'POR_HORA', 1.2500, 'Y');
  upsert_concept('HEN', 'Hora extra nocturna (175%)', 'DEVENGO', 'POR_HORA', 1.7500, 'Y');
  upsert_concept('HEDDF', 'Hora extra diurna dominical/festiva (200%)', 'DEVENGO', 'POR_HORA', 2.0000, 'Y');
  upsert_concept('HENDF', 'Hora extra nocturna dominical/festiva (250%)', 'DEVENGO', 'POR_HORA', 2.5000, 'Y');
  upsert_concept('REC_NOCT', 'Recargo nocturno (35% adicional)', 'DEVENGO', 'POR_HORA', 0.3500, 'Y');
  upsert_concept('REC_DOMFEST', 'Recargo dominical/festivo (75% adicional)', 'DEVENGO', 'POR_HORA', 0.7500, 'Y');

  upsert_concept('COMISION_VENTAS', 'Comisión por ventas', 'DEVENGO', 'PORCENTAJE', 0.0500, 'Y');
  upsert_concept('COMISION_COBRANZA', 'Comisión por cobranza', 'DEVENGO', 'PORCENTAJE', 0.0200, 'Y');
  upsert_concept('COMISION_OBJETIVOS', 'Comisión por cumplimiento de objetivos', 'DEVENGO', 'PORCENTAJE', 0.1000, 'Y');

  upsert_concept('BONO_DESEMPENO', 'Bono de desempeño', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_PRODUCTIVIDAD', 'Bono de productividad', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_ASISTENCIA', 'Bono por asistencia perfecta', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_PUNTUALIDAD', 'Bono por puntualidad', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_CALIDAD', 'Bono por calidad', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_SST', 'Bono por seguridad y salud en el trabajo', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_NAVIDAD', 'Bono navideño', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_REFERIDO', 'Bono por referido', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_RECONOCIMIENTO', 'Bono de reconocimiento', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_IDEAS', 'Bono por innovación/ideas', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_IDIOMA', 'Bono por idioma', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_RELOCALIZACION', 'Bono por relocalización', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_RESULTADOS_TRIM', 'Bono por resultados trimestral', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_RESULTADOS_ANUAL', 'Bono por resultados anual', 'DEVENGO', 'FIJO', NULL, 'Y');

  upsert_concept('PRIMA_SERVICIOS', 'Prima de servicios (acumulación 8.33%)', 'DEVENGO', 'PORCENTAJE', 0.0833, 'Y');
  upsert_concept('CESANTIAS_MES', 'Cesantías (acumulación 8.33%)', 'DEVENGO', 'PORCENTAJE', 0.0833, 'Y');
  upsert_concept('INT_CESANTIAS_MES', 'Intereses a cesantías (1% mensual)', 'DEVENGO', 'PORCENTAJE', 0.0100, 'Y');
  upsert_concept('PRIMA_VACACIONES', 'Prima de vacaciones', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('VAC_DISFRUTE', 'Vacaciones en disfrute', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('VAC_COMPENSADAS', 'Vacaciones compensadas', 'DEVENGO', 'FIJO', NULL, 'Y');

  upsert_concept('AUX_TRANSPORTE', 'Auxilio de transporte', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_ALIMENTACION', 'Auxilio de alimentación', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_MOVIL', 'Auxilio de telefonía/móvil', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_INTERNET', 'Auxilio de internet', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_VIVIENDA', 'Auxilio de vivienda', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('SUBSIDIO_FAMILIAR', 'Subsidio familiar', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('SUBSIDIO_HIJO', 'Subsidio por hijo', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('SUBSIDIO_ESCOLAR', 'Subsidio escolar', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('SUB_ALIM_DIA', 'Subsidio de alimentación por día', 'DEVENGO', 'POR_DIA', NULL, 'Y');

  upsert_concept('VIATICOS_DIA', 'Viáticos (por día)', 'DEVENGO', 'POR_DIA', NULL, 'Y');
  upsert_concept('VIATICOS_NO_SAL', 'Viáticos no constitutivos salariales', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('REEMB_GASTOS', 'Reembolso de gastos', 'DEVENGO', 'FIJO', NULL, 'Y');

  upsert_concept('PAGO_TIEMPO_COMP', 'Pago por tiempo compensado', 'DEVENGO', 'POR_HORA', 1.0000, 'Y');
  upsert_concept('PAGO_CAPACITACION', 'Pago por horas de capacitación', 'DEVENGO', 'POR_HORA', 1.0000, 'Y');

  upsert_concept('PRIMA_ANTIGUEDAD', 'Prima de antigüedad (2%)', 'DEVENGO', 'PORCENTAJE', 0.0200, 'Y');
  upsert_concept('PRIMA_TECNICA', 'Prima técnica', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('PRIMA_EXTRAORD', 'Prima extraordinaria', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('GRATIFICACION', 'Gratificación especial', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('PROPINA', 'Propinas', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AJUSTE_POSITIVO', 'Ajuste positivo de nómina', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('DIFERENCIAL_SAL', 'Diferencial salarial', 'DEVENGO', 'FIJO', NULL, 'Y');

  upsert_concept('AUX_LENTES', 'Auxilio para lentes', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_MEDICO', 'Auxilio médico', 'DEVENGO', 'FIJO', NULL, 'Y');

  upsert_concept('INC_GENERAL_EPS', 'Incapacidad general EPS (pago empleador/EPS)', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('INC_LABORAL_ARL', 'Incapacidad laboral ARL', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('LIC_REMUN', 'Licencia remunerada', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('LIC_MATERNIDAD', 'Licencia de maternidad', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('LIC_PATERNIDAD', 'Licencia de paternidad', 'DEVENGO', 'FIJO', NULL, 'Y');

  upsert_concept('RETROACTIVO', 'Pago retroactivo de nómina', 'DEVENGO', 'FIJO', NULL, 'Y');

  -- Prestaciones/beneficios adicionales
  upsert_concept('BONO_FIN_ANO', 'Bono fin de año', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_FIDELIDAD', 'Bono de fidelidad', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_PROYECTO', 'Bono por proyecto', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_NOCTURNO', 'Bono por turno nocturno', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_TURNO', 'Bono por turno', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AYUDA_ESCOLAR', 'Ayuda escolar', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_TRANSP_EXTRA', 'Auxilio de transporte extra', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('RECOLOCACION', 'Reconocimiento por relocalización', 'DEVENGO', 'FIJO', NULL, 'Y');

  -- ===================== DEDUCCIONES =====================
  upsert_concept('SALUD_EE', 'Aporte a salud (empleado 4%)', 'DEDUCCION', 'PORCENTAJE', 0.0400, 'Y');
  upsert_concept('PENSION_EE', 'Aporte a pensión (empleado 4%)', 'DEDUCCION', 'PORCENTAJE', 0.0400, 'Y');
  upsert_concept('FONDO_SOL', 'Fondo de solidaridad pensional (1%)', 'DEDUCCION', 'PORCENTAJE', 0.0100, 'Y');
  upsert_concept('RETEFUENTE', 'Retención en la fuente', 'DEDUCCION', 'PORCENTAJE', 0.1000, 'Y');
  upsert_concept('SINDICATO', 'Cuota sindical', 'DEDUCCION', 'PORCENTAJE', 0.0100, 'Y');
  upsert_concept('COOPERATIVA', 'Aporte a cooperativa', 'DEDUCCION', 'PORCENTAJE', 0.0200, 'Y');
  upsert_concept('FONDO_EMPLEADOS', 'Cuota fondo de empleados', 'DEDUCCION', 'PORCENTAJE', 0.0200, 'Y');
  upsert_concept('APORTE_VOL_PEN', 'Aporte voluntario a pensión', 'DEDUCCION', 'PORCENTAJE', 0.0300, 'Y');
  upsert_concept('APORTE_VOL_SAL', 'Aporte voluntario a salud', 'DEDUCCION', 'PORCENTAJE', 0.0100, 'Y');

  upsert_concept('LIBRANZA', 'Libranza', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('PRESTAMO_EMP', 'Cuota préstamo a empleado', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('EMBARGO_JUD', 'Embargo judicial', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('ADELANTO_NOM', 'Descuento por anticipo de nómina', 'DEDUCCION', 'FIJO', NULL, 'Y');

  upsert_concept('CUOTA_PARQ', 'Cuota de parqueadero', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('COMEDOR_EMP', 'Comedor empresarial', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('SEGURO_VIDA', 'Seguro de vida', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('SEGURO_MEDICO', 'Seguro médico', 'DEDUCCION', 'FIJO', NULL, 'Y');

  upsert_concept('PERMISO_NO_REMUN', 'Permisos no remunerados (por día)', 'DEDUCCION', 'POR_DIA', 1.0000, 'Y');
  upsert_concept('AUSENTISMO_NO_PAGO', 'Ausentismo sin salario (por día)', 'DEDUCCION', 'POR_DIA', 1.0000, 'Y');
  upsert_concept('TARDANZA', 'Descuento por tardanza (por hora)', 'DEDUCCION', 'POR_HORA', 1.0000, 'Y');

  upsert_concept('DEV_VIATICOS', 'Devolución de viáticos', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DEV_COMISION', 'Devolución de comisiones', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_HERRAMIENTAS', 'Descuento por herramientas', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_DANOS', 'Descuento por daños', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_UNIFORME', 'Descuento por uniforme', 'DEDUCCION', 'FIJO', NULL, 'Y');

  upsert_concept('SS_ADICIONAL', 'Deducción seguridad social adicional', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('RETENCION_JUD', 'Retención judicial adicional', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_SERVICIOS', 'Descuento por servicios (gimnasio, etc.)', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_ALMUERZO', 'Descuento por almuerzo', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_TRANS_EMP', 'Descuento transporte empresarial', 'DEDUCCION', 'FIJO', NULL, 'Y');

  upsert_concept('AJUSTE_NEGATIVO', 'Ajuste negativo de nómina', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DIFERENCIAL_NEG', 'Diferencial negativo', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_FALTANTE_CAJA', 'Descuento por faltante de caja', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_DIAS_SIN_SUELDO', 'Días sin sueldo (por día)', 'DEDUCCION', 'POR_DIA', 1.0000, 'Y');
  upsert_concept('DEDUCCION_OTROS', 'Otras deducciones', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_TECNOLOGIA', 'Descuento por compra de tecnología', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DESC_SEGURO_HOGAR', 'Descuento seguro de hogar', 'DEDUCCION', 'FIJO', NULL, 'Y');

  -- Extras para llegar a 80+ y cubrir casos comunes
  upsert_concept('SEGURO_MIXTO', 'Seguro mixto (plan corporativo)', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('APORTE_CAJA', 'Aporte a caja (empleado)', 'DEDUCCION', 'PORCENTAJE', 0.0100, 'Y');
  upsert_concept('DESCUENTO_PRESTACIONES', 'Descuento por prestaciones', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('COOPERATIVA_LIBRANZA', 'Cooperativa - libranza', 'DEDUCCION', 'FIJO', NULL, 'Y');
  upsert_concept('DONACION', 'Donación voluntaria', 'DEDUCCION', 'FIJO', NULL, 'Y');

  upsert_concept('BONO_BIENESTAR', 'Bono de bienestar', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_RETENCION', 'Bono de retención', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('BONO_FORMACION', 'Bono por formación', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_HERRAMIENTAS', 'Auxilio de herramientas', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_TELETRABAJO', 'Auxilio de teletrabajo', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_GASOLINA', 'Auxilio de gasolina', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_PARQUEADERO', 'Auxilio de parqueadero', 'DEVENGO', 'FIJO', NULL, 'Y');
  upsert_concept('AUX_UNIFORME', 'Auxilio de uniforme', 'DEVENGO', 'FIJO', NULL, 'Y');

  COMMIT;
END;
/

-- 3) Periodos
CREATE TABLE pay_periods (--> guardar peridos hacia atras, vamos a guardar los 12 meses de atras, 52 semanas, 12 mensual 24 quincenal
  period_id    NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  payroll_type_id NUMBER NOT NULL REFERENCES pay_payroll_types(payroll_type_id),
  period_code  VARCHAR2(20) NOT NULL,             -- 2026-02, 2026Q1-01, etc.
  date_start   DATE NOT NULL,
  date_end     DATE NOT NULL,
  status       VARCHAR2(10) DEFAULT 'ABIERTO' CHECK (status IN ('ABIERTO','CERRADO')),
  CONSTRAINT uk_pay_period UNIQUE (payroll_type_id, period_code),
  CONSTRAINT ck_pay_period_dates CHECK (date_end >= date_start)
);

DECLARE
  v_mensual_id    pay_payroll_types.payroll_type_id%TYPE;
  v_quincenal_id  pay_payroll_types.payroll_type_id%TYPE;
  v_semanal_id    pay_payroll_types.payroll_type_id%TYPE;

  v_start   DATE;
  v_end     DATE;
  v_code    VARCHAR2(20);

  v_base_month        DATE;
  v_last_week_start   DATE;
  v_today             DATE;

  v_q                 NUMBER;
  v_year              VARCHAR2(4);
  v_month             NUMBER;
  v_month_in_quarter  NUMBER;
  v_half_index        NUMBER;

  -- Inserta y omite si ya existe (respetando uk_pay_period)
  PROCEDURE ins_period(p_type_id NUMBER, p_code VARCHAR2, p_start DATE, p_end DATE) IS
  BEGIN
    INSERT INTO pay_periods (payroll_type_id, period_code, date_start, date_end, status)
    VALUES (p_type_id, p_code, p_start, p_end, 'ABIERTO');
  EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
      NULL; -- ya existe, no hacer nada
  END;
BEGIN
  -- Buscar IDs de tipos de nómina
  SELECT payroll_type_id INTO v_mensual_id   FROM pay_payroll_types WHERE code = 'MENSUAL';
  SELECT payroll_type_id INTO v_quincenal_id FROM pay_payroll_types WHERE code = 'QUINCENAL';
  SELECT payroll_type_id INTO v_semanal_id   FROM pay_payroll_types WHERE code = 'SEMANAL';

  ---------------------------------------------------------------------------
  -- 1) 12 meses completos hacia atrás (sin incluir el mes en curso)
  ---------------------------------------------------------------------------
  v_base_month := TRUNC(ADD_MONTHS(SYSDATE, -1), 'MM'); -- inicio del mes pasado
  FOR i IN 0 .. 11 LOOP
    v_start := ADD_MONTHS(v_base_month, -i);
    v_end   := LAST_DAY(v_start);
    v_code  := TO_CHAR(v_start, 'YYYY-MM'); -- p.ej. 2025-11
    ins_period(v_mensual_id, v_code, v_start, v_end);
  END LOOP;

  ---------------------------------------------------------------------------
  -- 2) 52 semanas completas hacia atrás (lunes a domingo)
  --    No incluye la semana en curso.
  ---------------------------------------------------------------------------
  v_last_week_start := TRUNC(SYSDATE, 'IW') - 7; -- lunes de la semana anterior
  FOR i IN 0 .. 51 LOOP
    v_start := v_last_week_start - 7 * i; -- lunes
    v_end   := v_start + 6;               -- domingo
    v_code  := TO_CHAR(v_start, 'IYYY') || '-W' || TO_CHAR(v_start, 'IW'); -- p.ej. 2026-W05
    ins_period(v_semanal_id, v_code, v_start, v_end);
  END LOOP;

  ---------------------------------------------------------------------------
  -- 3) 24 quincenas completas hacia atrás
  --    Quincena 1: día 1 al 15 | Quincena 2: día 16 al fin de mes
  --    Código: YYYYQq-xx (xx = 01..06 dentro del trimestre)
  ---------------------------------------------------------------------------
  v_today := TRUNC(SYSDATE);

  -- Determinar la última quincena completa finalizada
  IF TO_NUMBER(TO_CHAR(v_today, 'DD')) > 15 THEN
    v_end := TRUNC(v_today, 'MM') + 14;        -- 15 del mes actual
  ELSE
    v_end := LAST_DAY(ADD_MONTHS(v_today, -1)); -- fin del mes anterior
  END IF;

  FOR i IN 1 .. 24 LOOP
    -- Determinar inicio según el fin de la quincena
    IF TO_NUMBER(TO_CHAR(v_end, 'DD')) = 15 THEN
      v_start := TRUNC(v_end, 'MM');          -- 1 al 15
    ELSE
      v_start := TRUNC(v_end, 'MM') + 15;     -- 16 a fin de mes
    END IF;

    -- Calcular índice de quincena dentro del trimestre (01..06)
    v_q     := TO_NUMBER(TO_CHAR(v_start, 'Q'));      -- 1..4
    v_year  := TO_CHAR(v_start, 'YYYY');
    v_month := TO_NUMBER(TO_CHAR(v_start, 'MM'));
    v_month_in_quarter := v_month - ((v_q - 1) * 3);  -- 1..3
    v_half_index := (v_month_in_quarter - 1) * 2
                    + CASE WHEN TO_NUMBER(TO_CHAR(v_start, 'DD')) = 1 THEN 1 ELSE 2 END;

    v_code  := v_year || 'Q' || v_q || '-' || LPAD(v_half_index, 2, '0'); -- p.ej. 2026Q1-03
    ins_period(v_quincenal_id, v_code, v_start, v_end);

    -- Retroceder a la quincena anterior
    IF TO_NUMBER(TO_CHAR(v_end, 'DD')) = 15 THEN
      v_end := LAST_DAY(ADD_MONTHS(v_end, -1)); -- fin del mes anterior
    ELSE
      v_end := TRUNC(v_end, 'MM') + 14;         -- 15 del mismo mes
    END IF;
  END LOOP;

  COMMIT;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RAISE_APPLICATION_ERROR(
      -20001,
      'Faltan tipos de nómina requeridos (MENSUAL, QUINCENAL o SEMANAL) en PAY_PAYROLL_TYPES.'
    );
END;
/



-- 4) Contratos (condiciones por empleado)
CREATE TABLE pay_emp_contracts (
  contract_id     NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  employee_id     NUMBER NOT NULL REFERENCES employees(employee_id),
  payroll_type_id NUMBER NOT NULL REFERENCES pay_payroll_types(payroll_type_id),
  base_salary     NUMBER(12,2) NOT NULL,
  hours_per_month NUMBER(6,2) DEFAULT 240,        -- para calcular valor hora
  start_date      DATE NOT NULL,
  end_date        DATE,
  status          VARCHAR2(10) DEFAULT 'ACTIVO' CHECK (status IN ('ACTIVO','INACTIVO')),
  CONSTRAINT ck_contract_dates CHECK (end_date IS NULL OR end_date >= start_date)
);


-- 5) Registro de horas (horas extra/recargos)
CREATE TABLE pay_time_entries (
  time_entry_id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  employee_id   NUMBER NOT NULL REFERENCES employees(employee_id),
  work_date     DATE   NOT NULL,
  concept_id    NUMBER NOT NULL REFERENCES pay_concepts(concept_id), -- ej HED/HEN/DOM
  hours_qty     NUMBER(6,2) NOT NULL CHECK (hours_qty > 0),
  notes         VARCHAR2(200)
);



-- 6) Vacaciones / licencias
CREATE TABLE pay_leave_requests (
  leave_id     NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  employee_id  NUMBER NOT NULL REFERENCES employees(employee_id),
  leave_type   VARCHAR2(15) NOT NULL CHECK (leave_type IN ('VACACIONES','INCAPACIDAD','LICENCIA')),
  date_start   DATE NOT NULL,
  date_end     DATE NOT NULL,
  days_qty     NUMBER(6,2),
  status       VARCHAR2(12) DEFAULT 'APROBADA' CHECK (status IN ('SOLICITADA','APROBADA','RECHAZADA')),
  CONSTRAINT ck_leave_dates CHECK (date_end >= date_start)
);

-- 7) Novedades (bonos, descuentos, préstamos, etc.)
CREATE TABLE pay_emp_events (
  event_id     NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  employee_id  NUMBER NOT NULL REFERENCES employees(employee_id),
  period_id    NUMBER NOT NULL REFERENCES pay_periods(period_id),
  concept_id   NUMBER NOT NULL REFERENCES pay_concepts(concept_id),
  qty          NUMBER(10,2) DEFAULT 1,             -- unidades/días/horas si aplica
  amount       NUMBER(12,2),                       -- monto directo si aplica
  rate         NUMBER(12,4),                       -- % o tarifa si aplica
  notes        VARCHAR2(200)
);



-- 8) Cabecera de nómina (desprendible)
CREATE TABLE pay_payslips (
  payslip_id   NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  period_id    NUMBER NOT NULL REFERENCES pay_periods(period_id),
  employee_id  NUMBER NOT NULL REFERENCES employees(employee_id),
  calc_date    DATE DEFAULT SYSDATE,
  gross_total  NUMBER(12,2) DEFAULT 0,
  ded_total    NUMBER(12,2) DEFAULT 0,
  net_total    NUMBER(12,2) DEFAULT 0,
  status       VARCHAR2(12) DEFAULT 'CALCULADO' CHECK (status IN ('CALCULADO','APROBADO','PAGADO')),
  CONSTRAINT uk_payslip UNIQUE (period_id, employee_id)
);



-- 9) Detalle por concepto
CREATE TABLE pay_payslip_lines (
  line_id      NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  payslip_id   NUMBER NOT NULL REFERENCES pay_payslips(payslip_id) ON DELETE CASCADE,
  concept_id   NUMBER NOT NULL REFERENCES pay_concepts(concept_id),
  qty          NUMBER(10,2),
  unit_value   NUMBER(12,4),
  line_total   NUMBER(12,2) NOT NULL,
  CONSTRAINT ck_line_total CHECK (line_total <> 0)
);