
SQL> CREATE TABLE CUSTOMERS (
  2      CUSTOMER_ID    NUMBER(8)    NOT NULL,
  3      CUSTOMER_SSN   VARCHAR2(9),
  4      FIRST_NAME     VARCHAR2(20),
  5      LAST_NAME      VARCHAR2(20),
  6      SALES_REP_ID   NUMBER(4),
  7      ADDR_LINE      VARCHAR2(80),
  8      CITY           VARCHAR2(30),
  9      STATE          VARCHAR2(30),
 10      ZIP_CODE       VARCHAR2(9),
 11      CTL_INS_DTTM   DATE,
 12      CTL_UPD_DTTM   DATE,
 13      CTL_REC_USER   VARCHAR2(30),
 14      CTL_REC_STAT   VARCHAR2(1),
 15      PRIMARY KEY (CUSTOMER_ID)
 16  );

Table created.

SQL> CREATE TABLE CUSTOMERS_HISTORY (
  2      CUSTOMER_ID    NUMBER(8)    NOT NULL,
  3      CUSTOMER_SSN   VARCHAR2(9),
  4      FIRST_NAME     VARCHAR2(20),
  5      LAST_NAME      VARCHAR2(20),
  6      SALES_REP_ID   NUMBER(4),
  7      ADDR_LINE      VARCHAR2(80),
  8      CITY           VARCHAR2(30),
  9      STATE          VARCHAR2(30),
 10      ZIP_CODE       VARCHAR2(9),
 11      CTL_INS_DTTM   DATE,
 12      CTL_UPD_DTTM   DATE,
 13      CTL_UPD_USER   VARCHAR2(30),
 14      CTL_REC_STAT   VARCHAR2(1),
 15      HST_INS_DTTM   DATE,
 16      HST_OPR_TYPE   VARCHAR2(20)
 17  );

Table created.

SQL> CREATE OR REPLACE TRIGGER TRG_CUSTOMERS_BIUR
  2  BEFORE INSERT OR UPDATE OR DELETE ON CUSTOMERS
  3  FOR EACH ROW
  4  DECLARE
  5      V_HST_OPR_TYPE CUSTOMERS_HISTORY.HST_OPR_TYPE%TYPE;
  6  BEGIN
  7      IF INSERTING THEN
  8          :NEW.CTL_INS_DTTM := SYSDATE;
  9          :NEW.CTL_UPD_DTTM := NULL;
 10          :NEW.CTL_REC_STAT := 'N';
 11          :NEW.CTL_REC_USER := USER;
 12          V_HST_OPR_TYPE := 'INSERT';
 13
 14          INSERT INTO CUSTOMERS_HISTORY (
 15              CUSTOMER_ID, CUSTOMER_SSN, FIRST_NAME, LAST_NAME,
 16              SALES_REP_ID, ADDR_LINE, CITY, STATE, ZIP_CODE,
 17              CTL_INS_DTTM, CTL_UPD_DTTM, CTL_UPD_USER, CTL_REC_STAT,
 18              HST_INS_DTTM, HST_OPR_TYPE
 19          ) VALUES (
 20              :NEW.CUSTOMER_ID, :NEW.CUSTOMER_SSN, :NEW.FIRST_NAME, :NEW.LAST_NAME,
 21              :NEW.SALES_REP_ID, :NEW.ADDR_LINE, :NEW.CITY, :NEW.STATE, :NEW.ZIP_CODE,
 22              :NEW.CTL_INS_DTTM, :NEW.CTL_UPD_DTTM, :NEW.CTL_REC_USER, :NEW.CTL_REC_STAT,
 23              SYSDATE, V_HST_OPR_TYPE
 24          );
 25
 26      ELSIF UPDATING THEN
 27          :NEW.CTL_UPD_DTTM := SYSDATE;
 28          :NEW.CTL_REC_USER := USER;
 29          V_HST_OPR_TYPE := 'UPDATE';
 30
 31          INSERT INTO CUSTOMERS_HISTORY (
 32              CUSTOMER_ID, CUSTOMER_SSN, FIRST_NAME, LAST_NAME,
 33              SALES_REP_ID, ADDR_LINE, CITY, STATE, ZIP_CODE,
 34              CTL_INS_DTTM, CTL_UPD_DTTM, CTL_UPD_USER, CTL_REC_STAT,
 35              HST_INS_DTTM, HST_OPR_TYPE
 36          ) VALUES (
 37              :NEW.CUSTOMER_ID, :NEW.CUSTOMER_SSN, :NEW.FIRST_NAME, :NEW.LAST_NAME,
 38              :NEW.SALES_REP_ID, :NEW.ADDR_LINE, :NEW.CITY, :NEW.STATE, :NEW.ZIP_CODE,
 39              :NEW.CTL_INS_DTTM, :NEW.CTL_UPD_DTTM, :NEW.CTL_REC_USER, :NEW.CTL_REC_STAT,
 40              SYSDATE, V_HST_OPR_TYPE
 41          );
 42
 43      ELSIF DELETING THEN
 44          V_HST_OPR_TYPE := 'DELETE';
 45
 46          INSERT INTO CUSTOMERS_HISTORY (
 47              CUSTOMER_ID, CUSTOMER_SSN, FIRST_NAME, LAST_NAME,
 48              SALES_REP_ID, ADDR_LINE, CITY, STATE, ZIP_CODE,
 49              CTL_INS_DTTM, CTL_UPD_DTTM, CTL_UPD_USER, CTL_REC_STAT,
 50              HST_INS_DTTM, HST_OPR_TYPE
 51          ) VALUES (
 52              :OLD.CUSTOMER_ID, :OLD.CUSTOMER_SSN, :OLD.FIRST_NAME, :OLD.LAST_NAME,
 53              :OLD.SALES_REP_ID, :OLD.ADDR_LINE, :OLD.CITY, :OLD.STATE, :OLD.ZIP_CODE,
 54              :OLD.CTL_INS_DTTM, :OLD.CTL_UPD_DTTM, :OLD.CTL_REC_USER, :OLD.CTL_REC_STAT,
 55              SYSDATE, V_HST_OPR_TYPE
 56          );
 57      END IF;
 58  END;
 59  /

Trigger created.

SQL> INSERT INTO CUSTOMERS (
  2      CUSTOMER_ID, CUSTOMER_SSN, FIRST_NAME, LAST_NAME,
  3      SALES_REP_ID, ADDR_LINE, CITY, STATE, ZIP_CODE
  4  ) VALUES (
  5      201340, '969996970', 'Jeffrey', 'Antoine',
  6      6459, '9939 Moreno St.', 'Champagne', 'SD', '43172'
  7  );

1 row created.

SQL>
SQL> INSERT INTO CUSTOMERS (
  2      CUSTOMER_ID, CUSTOMER_SSN, FIRST_NAME, LAST_NAME,
  3      SALES_REP_ID, ADDR_LINE, CITY, STATE, ZIP_CODE
  4  ) VALUES (
  5      801349, '716647546', 'Cordell', 'Ayres',
  6      2200, '37 Noyes Street', 'Narod', 'NC', '15199'
  7  );

1 row created.

SQL> SELECT
  2      CUSTOMER_ID AS CUS_ID,
  3      FIRST_NAME,
  4      LAST_NAME,
  5      TO_CHAR(CTL_INS_DTTM, 'DD-MON-YYYY HH24:MI:SS') AS CTL_INS_DTTM,
  6      TO_CHAR(CTL_UPD_DTTM, 'DD-MON-YYYY HH24:MI:SS') AS CTL_UPD_DTTM,
  7      CTL_REC_USER,
  8      CTL_REC_STAT
  9  FROM CUSTOMERS;

    CUS_ID FIRST_NAME           LAST_NAME
---------- -------------------- --------------------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
CTL_REC_USER                   C
------------------------------ -
    201340 Jeffrey              Antoine
11-APR-2025 11:25:06
SYSTEM                         N

    801349 Cordell              Ayres
11-APR-2025 11:25:06
SYSTEM                         N

    CUS_ID FIRST_NAME           LAST_NAME
---------- -------------------- --------------------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
CTL_REC_USER                   C
------------------------------ -


SQL> SELECT
  2      CUSTOMER_ID AS CUS_ID,
  3      FIRST_NAME,
  4      LAST_NAME,
  5      TO_CHAR(CTL_INS_DTTM,
  6  ::contentReference[oaicite:7]{index=7}
  7
SQL>
SQL> SELECT
  2      CUSTOMER_ID AS CUS_ID,
  3      ADDR_LINE,
  4      TO_CHAR(CTL_INS_DTTM, 'DD-MON-YYYY HH24:MI:SS') AS CTL_INS_DTTM,
  5      TO_CHAR(CTL_UPD_DTTM, 'DD-MON-YYYY HH24:MI:SS') AS CTL_UPD_DTTM,
  6      CTL_UPD_USER,
  7      CTL_REC_STAT
  8  FROM
  9      CUSTOMERS
 10  WHERE
 11      CUSTOMER_ID = 201340;
    CTL_UPD_USER,
    *
ERROR at line 6:
ORA-00904: "CTL_UPD_USER": invalid identifier


SQL> SELECT
  2      CUSTOMER_ID AS CUS_ID,
  3      FIRST_NAME,
  4      LAST_NAME,
  5      TO_CHAR(CTL_INS_DTTM;
    TO_CHAR(CTL_INS_DTTM
                       *
ERROR at line 5:
ORA-00907: missing right parenthesis


SQL> SELECT
  2      CUSTOMER_ID AS CUS_ID,
  3      FIRST_NAME,
  4      LAST_NAME,
  5      TO_CHAR(CTL_INS_DTTM;
    TO_CHAR(CTL_INS_DTTM
                       *
ERROR at line 5:
ORA-00907: missing right parenthesis


SQL> SELECT
  2      CUSTOMER_ID AS CUS_ID,
  3      FIRST_NAME,
  4      LAST_NAME,
  5      TO_CHAR(CTL_INS_DTTM,
  6  ::contentReference[oaicite:7]{index=7}
  7
SQL> SELECT
  2      CUSTOMER_ID AS CUS_ID,
  3      CUSTOMER_SSN,
  4      FIRST_NAME,
  5      LAST_NAME,
  6      SALES_REP_ID,
  7      ADDR_LINE,
  8      CITY,
  9      STATE,
 10      ZIP_CODE,
 11      TO_CHAR(CTL_INS_DTTM, 'DD-MON-YYYY HH24:MI:SS') AS CTL_INS_DTTM,
 12      TO_CHAR(CTL_UPD_DTTM, 'DD-MON-YYYY HH24:MI:SS') AS CTL_UPD_DTTM,
 13      CTL_UPD_USER AS USERNAME,
 14      CTL_REC_STAT,
 15      TO_CHAR(HST_INS_DTTM, 'DD-MON-YYYY HH24:MI:SS') AS HST_INS_DTTM,
 16      HST_OPR_TYPE AS OPER
 17  FROM
 18      CUSTOMERS_HISTORY
 19  ORDER BY
 20      HST_INS_DTTM;

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
    801349 716647546 Cordell              Ayres                 2200

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
37 Noyes Street

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
Narod                          NC                      15199

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
11-APR-2025 11:25:06

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
SYSTEM                         N 11-APR-2025 11:25:06

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
INSERT

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------


    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
    201340 969996970 Jeffrey              Antoine                       6459

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
9939 Moreno St.

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
Champagne                      SD                      43172

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
11-APR-2025 11:25:06

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
SYSTEM                         N 11-APR-2025 11:25:06

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------
INSERT

    CUS_ID CUSTOMER_ FIRST_NAME           LAST_NAME             SALES_REP_ID
---------- --------- -------------------- -------------------- ------------
ADDR_LINE
--------------------------------------------------------------------------------
CITY                           STATE                   ZIP_CODE
------------------------------ ------------------------------ ---------
CTL_INS_DTTM                  CTL_UPD_DTTM
----------------------------- -----------------------------
USERNAME                       C HST_INS_DTTM
------------------------------ - -----------------------------
OPER
--------------------


SQL>
















































