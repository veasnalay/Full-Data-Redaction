# Full-Data-Redaction
Oracle Data Redaction is a security feature that dynamically masks sensitive data in query results without modifying the actual data stored in the database. Redaction occurs at query execution time, just before data is returned to the user or application.

1- list redacte policies
  col POLICY_DESCRIPTION for a17
  col EXPRESSION for a17
  col POLICY_NAME for a17
  col OBJECT_NAME for a17
  col OBJECT_OWNER for a17
  SELECT * FROM redaction_policies;

2- list employees tables
  select * from hr.employees

3- create use for testing query 
  CREATE USER rduser IDENTIFIED BY rduser;
  GRANT create session TO rduser;
  GRANT select ON hr.employees TO rduser;

4- list detail of employees with id 201
  SQL> SELECT employee_id, last_name, salary, commission_pct FROM hr.employees WHERE employee_id = 201;
  
  EMPLOYEE_ID LAST_NAME                     SALARY COMMISSION_PCT
  ----------- ------------------------- ---------- --------------
          201 Hartstein                      13000
  
  SQL>

5- apply redact policies 'full' 
  -- expression 1=1 : redaction is always performed because the expression always evaluates to true
  BEGIN
  DBMS_REDACT.ADD_POLICY
  (object_schema => 'HR',
  object_name => 'EMPLOYEES',
  policy_name => 'EMP_POLICY',
  column_name => 'SALARY',
  function_type => DBMS_REDACT.FULL,
  expression => '1=1');
  END;
  /

6- check redact detail 
  col COLUMN_NAME for a17
  col FUNCTION_PARAMETERS for a17
  col OBJECT_OWNER for a17
  
  SQL> SELECT object_owner, object_name, column_name, function_type, function_parameters FROM redaction_columns;
  
  OBJECT_OWNER      OBJECT_NAME                       COLUMN_NAME       FUNCTION_TYPE               FUNCTION_PARAMETE
  ----------------- --------------------------------- ----------------- --------------------------- -----------------
  HR                EMPLOYEES                         SALARY            FULL REDACTION

7- list redacte policies
  col POLICY_DESCRIPTION for a17
  col EXPRESSION for a17
  col POLICY_NAME for a17
  col OBJECT_NAME for a17
  col OBJECT_OWNER for a17
  SQL> SELECT * FROM redaction_policies;
  OBJECT_OWNER      OBJECT_NAME                       POLICY_NAME       EXPRESSION        ENA POLICY_DESCRIPTION
  ----------------- --------------------------------- ----------------- ----------------- --- ----------------------
  HR                EMPLOYEES                         EMP_POLICY        1=1               YES


8- test result with other schema

  SQL> connect rduser
  Enter password:
  Connected.
  SQL> SELECT employee_id, last_name, salary, commission_pct FROM hr.employees WHERE employee_id = 201;
  
  EMPLOYEE_ID LAST_NAME                     SALARY COMMISSION_PCT
  ----------- ------------------------- ---------- --------------
          201 Hartstein                          0
  
  SQL>
  
  *************************************************************************************
  * we saw colume SALARY = 0 mean that it is fake value 
  *************************************************************************************

9- check result for sys schema 
  SQL> connect sys as sysdba
  Enter password:
  Connected.
  SQL> SELECT employee_id, last_name, salary, commission_pct FROM hr.employees WHERE employee_id = 201;
  
  EMPLOYEE_ID LAST_NAME                     SALARY COMMISSION_PCT
  ----------- ------------------------- ---------- --------------
          201 Hartstein                      13000
  
  SQL>
  
  *************************************************************************************
  * we saw colume SALARY = 13000 mean that it correct value
  *************************************************************************************

10- test case : try update salary to 15000
  SQL> connect hr
  Enter password:
  Connected.
  SQL> SELECT employee_id, last_name, salary, commission_pct FROM hr.employees WHERE employee_id = 201;
  
  EMPLOYEE_ID LAST_NAME                     SALARY COMMISSION_PCT
  ----------- ------------------------- ---------- --------------
          201 Hartstein                          0

  SQL> update employees set salary=15000 where employee_id=201; >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> update salary to 15000
  
  1 row updated.
  
  SQL> commit;
  
  Commit complete.

  SQL> SELECT employee_id, last_name, salary, commission_pct FROM hr.employees WHERE employee_id = 201;
  
  EMPLOYEE_ID LAST_NAME                     SALARY COMMISSION_PCT
  ----------- ------------------------- ---------- --------------
          201 Hartstein                          0
  
  SQL>


11- check value from sys schema 
  SQL> connect sys as sysdba
  Enter password:
  Connected.
  SQL>  SELECT employee_id, last_name, salary, commission_pct FROM hr.employees WHERE employee_id = 201;
  
  EMPLOYEE_ID LAST_NAME                     SALARY COMMISSION_PCT
  ----------- ------------------------- ---------- --------------
          201 Hartstein                      15000
  
  SQL>

**********************************/ Noted \***************************************************
* we saw colume SALARY = 15000 
* the salary value updated from 13000 to 15000 


* mean that other user can edit the redact column 
* but can not see the real value except sys schema
*************************************************************************************
