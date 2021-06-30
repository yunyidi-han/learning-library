# Relocate a PDB by using DBCA in Silent Mode

## Introduction

Estimated Lab Time: 30 minutes

### Objectives

Learn how to do the following:

- Switch your PDB to ARCHIVELOG mode
- Relocate remote PDB from CDB
- Check that PDB is relocated

### Prerequisites

Be sure that the following tasks are completed before you start:

- Obtain an Oracle Cloud account.
- Create SSH keys.
- Sign in to Oracle Cloud Infrastructure.


## **STEP 1**: Set CDB1 and PDB1 to archivelog mode

1. Connect to the PDB container using SQL*Plus.  
    ```
    $ sqlplus /as sysdba

    SQL*Plus: Release 19.0.0.0.0 - Production on Tue Jun 1 14:52:31 2021
    Version 19.10.0.0.0

    Copyright (c) 1982, 2020, Oracle.  All rights reserved.


    Connected to:
    Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
    Version 19.10.0.0.0
    ```

2. Check the CDB log mode.
    ```
    SQL> select name, open_mode, log_mode from v$database;

    NAME      OPEN_MODE            LOG_MODE
    --------- -------------------- ------------
    CDB1      READ WRITE           NOARCHIVELOG
    ```

3.  Check your parameter values. A container cannot be in archive log mode if there
there are no values for each parameter.
    ```
    SQL> show parameter DB_RECOVERY_FILE        

    NAME                                 TYPE        VALUE
    ------------------------------------ ----------- ---------------------------
    db_recovery_file_dest                string      
    db_recovery_file_dest_size           big integer 0
    ```

4. Enter values for the parameters.
    ```
    SQL> alter system set db_recovery_file_dest_size=50G SCOPE=both;

    System altered.

    SQL> alter system set db_recovery_file_dest='/u01/app/oracle/recovery_area/CDB1' SCOPE=both;

    System altered.
    ```

5. Ensure your parameters contain values.
    ```
    SQL> select * from V$RECOVERY_FILE_DEST

    NAME
    --------------------------------------------------------------------------------
    SPACE_LIMIT SPACE_USED SPACE_RECLAIMABLE NUMBER_OF_FILES     CON_ID
    ----------- ---------- ----------------- --------------- ----------
    /u01/app/oracle/recovery_area/CDB1
                         0                 0               0          0

    SQL> show parameter DB_RECOVERY_FILE_DEST        

    NAME                                 TYPE        VALUE
    ------------------------------------ ----------- ---------------------------
    db_recovery_file_dest                string      /u01/app/oracle/recovery_area/CDB1
    db_recovery_file_dest_size           big integer 50G
    ```

6. Mount your database instance and change it to archive log mode.
    ```
    SQL> shutdown immediate
    Database closed.
    Database dismounted.
    ORACLE instance shut down.
    ```
    ```
    SQL> startup mount
    ORACLE instance started.

    Total System Global Area 9932110488 bytes
    Fixed Size                  9144984 bytes
    Variable Size            1509949440 bytes
    Database Buffers         8388608000 bytes
    Redo Buffers               24408064 bytes
    Database mounted.
    ```
    ```
    SQL> alter database archivelog;

    Database altered.

    SQL> alter database open;

    Database altered.
    ```
    ```
    SQL> archive log list
    Database log mode              Archive Mode
    Automatic archival             Enabled
    Archive destination            USE_DB_RECOVERY_FILE_DEST
    Oldest online log sequence     20
    Next log sequence to archive   22
    Current log sequence           22
    ```

7. Exit SQL*Plus
    ```
    SQL> exit
    ```

8. Login to CDB2 and repeat above steps from Step 1 - 7.
    ```
    $ . oraenv
    ORACLE_SID = [CDB1] ? CDB2
    ```

## **STEP 2**: Add the hr.sql script to the admin folder.
1. Make sure you are in the Oracle instance.
    ```
    $ sudo su - oracle
    Last login: Tue Jun  1 16:35:21 GMT 2021 on pts/0
    ```

2. Change to the /labs directory. Check that the 19cNewFeatures.zip is in the directory.
    ```
    $ cd ~/labs
    $ ls
    19cnf.zip    admin  db.rsp  DW          HA           OBEs  SEC
                          DB     DIAG    envprep.sh  multitenant  PERF  Videos
    ```

3. Change to the admin directory. Check to see if the hr.sql script is in the directory.
    ```
    $cd admin
    $ls
    cleanup_PDBs.sh     create_PDB19.sql      hr_code.sql     hr_idx.sql      listener.ora        startup_mount.sql     cleanup_PDBs.sql      create_PDB1.sh      hr_comnt.sql      hr_main_new.sql     profile.sql     startup.sql     cleanup_trace_ORCL.sh     create_PDB1.sql     hr_cre.sql      hr_main.sql     recreate_ORCL.sh      tnsnames.ora      close_TDE.sql     flashback.sql     hr_drop_new.sql     hr_popul.sql      shutdown.sql      create_CDB19.sh     hr_analz.sql      hr_drop.sql     **hr.sql**      sqlnet.ora
    ```

    >**Note**: If the hr.sql script is already in the directory, you can skip to Step 5.<

4. Insert the hr.sql script file in the admin
    1. Open this [hr.sql](https://docs.oracle.com/en/database/oracle/oracle-database/19/clone-pdbs-using-dbca-silent-mode/files/hr.sql) script in a separate tab.

    2. In the cloud shell, create the hr.sql file in the admin directory.
    ```
    $ vi hr.sql
    ```

    3. Once the vi editory opens, copy and paste the hr.sql script from the link
    into the newly created hr.sql file in the vi editor. It should look something
    like below.
    ```
    CREATE USER hr IDENTIFIED BY Ora4U_1234;

    GRANT dba, unlimited tablespace TO hr;
    ALTER SESSION SET CURRENT_SCHEMA=HR;

    ALTER SESSION SET NLS_LANGUAGE=American;
    ALTER SESSION SET NLS_TERRITORY=America;

    CREATE TABLE employees
    ( employee_id    NUMBER(6)
    , first_name     VARCHAR2(20)
    , last_name      VARCHAR2(25)
    , email          VARCHAR2(25)
    , phone_number   VARCHAR2(20)
    , hire_date      DATE
    , job_id         VARCHAR2(10)
    , salary         NUMBER(8,2)
    , commission_pct NUMBER(2,2)
    , manager_id     NUMBER(6)
    , department_id  NUMBER(4)) ;
    **script continues below**
    ```

    4. Exit the vi editor.
    ```
    Press Esc
    Input :q
    ```

    5. Check that hr.sql is now in the admin directory.
    ```
    $ ls
    ```

    6. Now that the hr.sql script is the directory, it must be granted permission to execute.
    ```
    $ chmod -R +x ~/labs
    ```

## **STEP 3**: Prepare the PDB Before Relocatiion
1. Login into the CDB1. If already logged into CDB1, skip to Step 2.
    ```
    $ . oraenv
    ORACLE_SID = [ORCL] ? CDB1
    ```

2. Connect to the PDB1 using SQL*Plus.
    ```
    $ sqlplus /as sysdba
    SQL> alter pluggable database PDB1 open;

    Pluggable database altered.

    SQL> alter session set container = PDB1;

    Session altered.
    ```

3. Verify that PDB1 contains the HR.EMPLOYEES table.
    ```
    SQL> SELECT count(*) FROM hr.employees;

      COUNT(*)
    ----------
          107
    ```

4. Connect to ORCL as SYS.
    ```
    SQL> CONNECT sys@CDB1 AS SYSDBA
    Enter password: Ora4U_1234
    ```

5. Create a common user in CDB1, used in the database link automatically created to connect to CDB1 during the relocation operation.
    ```
    SQL> CREATE USER c##remote_user IDENTIFIED BY Ora4U_1234 CONTAINER=ALL;
    ```

6. Grant the privileges.
    ```
    SQL> GRANT create session, create pluggable database, sysoper TO c##remote_user CONTAINER=ALL;
    ```

7. Quit session.
    ```
    SQL> exit
    ```

## **STEP 4**: Use DBCA to Relocate a Remote PDB
>In this section, you use DBCA in silent mode to relocate PDB1 from CDB1 as PDB1_IN_CDB2 in CDB2.<

1. Launch DBCA in silent mode to relocate PDB1 from CDB1 as PDB1_IN_CDB2 in CDB2.
    ```
    $ dbca -silent -relocatePDB -remotePDBName PDB1 -remoteDBConnString CDB1 -sysDBAUserName SYSTEM -sysDBAPassword Ora4U_1234 -remoteDBSYSDBAUserName SYS -remoteDBSYSDBAUserPassword Ora4U_1234 -dbLinkUsername c##remote_user -dbLinkUserPassword Ora4U_1234 -sourceDB CDB2 -pdbName PDB1_IN_CDB2
    ```

## **STEP 5**: Check that the PDB is Relocated
1. Connect to CDB2 as SYS. Check that PDB1 is relocated in CDB2.
    ```
    $ sqlplus sys@CDB2 AS SYSDBA
    Enter password: Ora4U_1234
    ```
    ```
    SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
    ------ ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB1_IN_CDB2                   READ WRITE NO
    ```

2. Check that PDB1_IN_CDB2 contains the HR.EMPLOYEES table as in PDB1.
    ```
    SQL> CONNECT hr@PDB1_IN_CDB2
    Enter password: Ora4U_1234
    ```
    ```
    SQL> SELECT count(*) FROM employees;

    COUNT(*)
    ----------
           107
    ```

3. Connect to ORCL as SYS. Check that PDB1 does not exist in ORCL anymore.
    ```
    SQL> CONNECT sys@ORCL AS SYSDBA
    Enter password: Ora4U_1234
    ```
    ```
    SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
    ------ ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
    ```

## **STEP 6**: Clean up the Cloned PDB
1. Connect to CDB2 as SYS.
    ```
    SQL> CONNECT sys@CDB2 AS SYSDBA
    Enter password: Ora4U_1234
    ```

2. Close PDB1_IN_CDB2.
    ```
    SQL> ALTER PLUGGABLE DATABASE PDB1_IN_CDB2 CLOSE;
    ```

3. Drop PDB1_IN_CDB2.
    ```
    DROP PLUGGABLE DATABASE PDB1_IN_CDB2 INCLUDING DATAFILES;
    ```

4. Quit the session.
    ```
    SQL> EXIT
    ```

## **STEP 7**: Disable archivelog mode for CDB1 and CDB2.
1. Set the environment variables for your CDB.
    ```
    $ . oraenv
    ORACLE_SID = [oracle] ? CDB1
    ```

2. Execute the following statements to disable ARCHIVELOG mode on the database.
    ```
    $ sqlplus / as sysdba

    SQL> SHUTDOWN IMMEDIATE;

    SQL> STARTUP MOUNT;

    SQL> ALTER DATABASE noarchivelog;

    SQL> ALTER DATABASE open;

    SQL> SELECT log_mode FROM v$database;
    ```

3. Switch to CDB2 and repeat Step 2.
    ```
    SQL> EXIT

    $ . oraenv
    ORACLE_SID = [CDB1] ? CDB2
    ```

## Learn More

- [New Features in Oracle Database 19c](https://docs.oracle.com/en/database/oracle/oracle-database/19/newft/preface.html#GUID-E012DF0F-432D-4C03-A4C8-55420CB185F3)


## Acknowledgements

- **Author**- Dominique Jeunot
- **Last Updated By/Date** - Kherington Barley, Austin Specialist Hub, June 2021
