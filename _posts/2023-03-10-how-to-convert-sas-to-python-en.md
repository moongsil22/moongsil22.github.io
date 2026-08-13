---
layout: post
tags: [posts-en]
title: How to Convert SAS to Python
description: >
  A collection of patterns I used often when migrating data preprocessing logic from SAS to Python (Oracle connectivity, import/export, column handling, transpose, joins, conditional columns, macro replacement, and more).
---

Notes on patterns I ended up using repeatedly while migrating business-analysis data preprocessing logic from SAS to Python. Grouped in the order I actually ran into them in practice — from Oracle connectivity, to import/export, column manipulation, transpose, joins, conditional column creation, and finally replacing SAS macros with Python.

#### 0. Oracle Connection, Library References, and Variable Declarations

#### SAS 

Before connecting to Oracle, you first need the Oracle client installed and a `tnsnames.ora` file configured.

~~~sas
FILENAME PGM_PATH "&SAS_PGM_PTH./";
%INCLUDE PGM_PATH(common.sas);
LIBNAME ORADB ORACLE USER="test" PW="123456" PATH="TESTDB" SCHEMA="TEST" ORACLE_73=NO UPDATE_ISOLATION_LEVEL=READCOMMITTED DBINDEX=YES DBMAX_TEST=32767;

%LET DATA_DIR = "C:\DATA";
libname saslib "&DATA_DIR.\saslib";
~~~

#### Python

Requires cx_Oracle. -> pip install cx_Oracle
Create oracleTest.py first, then use it. -> import oracleTest 

oracleTest.py
~~~python
import cx_Oracle as co
import pandas as pd
from _datetime import datetime


def query_OracleSQL(query):

    start_tm = datetime.now()

    #   DB Connection
    dsn_tns = co.makedsn("192.168.0.1", "1521", service_name="TEST")
    conn = co.connect(user="test", password="123456", dsn=dsn_tns)

    # Get a dataframe
    query_result = pd.read_sql(query, conn)

    # Close connection
    conn.close()

    end_tm = datetime.now()
    print('START: ', str(start_tm))
    print('END: ', str(end_tm))
    print('ELAP: ', str(end_tm - start_tm))

    return query_result
~~~

~~~python
import oracleTest
import pandas as pd
import pandasql as ps
import numpy as np
from _datetime import datetime
import re


time = '2023'
query = f"select code, formula from TEST.AA where time='{time}'"

df1 = oracleTest.query_OracleSQL(query)
~~~


#### 1. Dataset import From CSV, EXCEL, SQL query, Existing Dataset

##### 1.1 CSV 

###### SAS
~~~sas
# from CSV 

FILENAME aaa "path/aaa.csv";

DATA WORK.aaa;
  INFILE aaa LRECL=100 ENCODING="UTF-8" delimiter = ';' MISSOVER DSD;
LENGTH
  NO $ 20
  AMOUNT 8
  PRODUCT_CODE1 8
  PRODUCT_CODE2 8;
LABEL
  NO = "ID"
  AMOUNT = "Sales Amount"
  PRODUCT_CODE1 = "Product 1"
  PRODUCT_CODE2 = "Product 2";
INFORMAT
  NO = $CHAR32.
  AMOUNT = BEST15.
  PRODUCT_CODE1 = BEST15.
  PRODUCT_CODE2 = BEST15.;
INPUT
  NO = $CHAR32.
  AMOUNT = BEST15.
  PRODUCT_CODE1 = BEST15.
  PRODUCT_CODE2 = BEST15.;  
RUN;  
~~~

###### python
~~~python
import pandas as pd

T_001 = pd.read_csv(r"C:\Users\BOK\Desktop\extract-by-step\STEP1.T_001.csv", sep=',',
names = ['no', 'name', 'age'],
header=None,
dtype = {"no":str,
         "name":str,
	 "age":str
	 },
low_memory=False)

T_001.columns = map(lambda x: x.upper(), T_027P100.columns)
~~~



##### 1.2 EXCEL 

###### SAS

~~~sas
PROC IMPORT OUT=WORK.DATA
DATAFILE="C:\excel\file_name.xlsx"
DBMS=XLSX REPLACE;
RANGE="SHEET1$A1:B5";
GETNAMES=YES;
RUN;
~~~

###### Python

~~~python
import pandas as pd
FML = pd.read_excel(r"C:\data\step2_error_check.xlsx", sheet_name='FML',
                        names=['KEY_SEQ', 'FML'])

RULE = pd.read_excel(r"C:\data\step2_error_check.xlsx", sheet_name='RULE',
                         names=['KEY_SEQ', 'CONDITION', 'ACNT_CODE', 'FORMULA'])

~~~

##### 1.3 ORACLE/QUERY TABLE 

###### SAS
With PROC SQL you can load and use SAS library/WORK/Oracle datasets. Besides creating tables, DML is also possible.
~~~sas
PROC SQL;
CREATE TABLE BBB AS
SELECT USER_NAME
FROM ORADB.BBB
WHERE TIME="&TIME"
ORDER BY UPJ;
QUIT;
RUN;
~~~

###### PYTHON
With pandasql you can load and use a dataframe. Besides creating tables, DML is also possible.
~~~python
import pandasql as ps
query = f"select no, name from df1"
df2 = ps.sqldf(query)
~~~

##### 1.4 Creating a New Dataset From an Existing One

###### SAS
~~~sas
DATA NEWAA(KEEP=NAME CODE DROP=CNT RENAME=(NO=NEW_NO));
SET AAA(where=(time='2019'));
~~~
###### Python
~~~python

no_list = ['1','3','7']

df2 = df1.copy() # use copy() to avoid mutating the original data
df2 = df2.loc[(~df2['no'].isin(no_list)) & (df2['name'] !='KANG')]

~~~

#### 2. File Export (내보내기)

##### SAS

1) dat
~~~sas
DATA _NULL_;
SET saslib.AA;
FILE OUT_PATH(&OUT) MOD;
IF FIRST.NO THEN DO;
  PUT @20 "&TITLE";
  PUT /"&ls_t1_line";
  PUT @2 "Data File" @40 "Record Count";
  PUT "&ls_t1_line"/;
END;
  PUT @2 source @40 CNT;
RUN;
~~~

2)CSV
~~~sas
data _null_;
  set work.aa end=EFIEOD;
  %let _EFIERR_=0;
  %let _EFIREC_=0;
  file "outpath/xxx..csv" delimiter=',' DROPOVER DSD lrecl=10000;
  format NO $2;
  if _n_ = 1 then /* first line: column names */
  do;
    put
    'NO,'CODE','A1','A2','A3'
  end;
  do;
    EFIOUT + 1;
    put NO $ @;
    put CODE $ @;
    put A1-A3;
    ;
  end;
  if _ERROR_ then call symputx('_EFIERR_',1);
  if EFIEOD then call symputx('_EFIREC',EFIOUT);
run;  
~~~

3)EXCEL
~~~sas
data _null_;
  set work.aa;
  file "outpath/xxx..xls" MOD delimiter='09'X DSD lrecl=10000;
  if _n_ = 1 then do;
   PUT /"&ls_t1_line";
   PUT "Type" '09'X "In Use" '09'X "Count" ;
   PUT "&ls_t1_line"/;
  end;
  PUT NO USE_YN CNT
run;  
~~~
4)
~~~sas
PROC EXPORT DATA=WORK.AAA
OUTFILE="C:\data\aaa.csv"
DBMS=CSV;
RUN;
~~~

##### Python
~~~python
df.to_csv('./aa.csv')
df.to_excel('./aa.xls')
~~~


#### 3. DROP Columns

##### SAS
Usable in DATA, SET, and OUT blocks.
~~~sas
DATA aaa(DROP=AMOUNT NO);
RUN;

DATA aaa;
DROP AMOUNT;
RUN;
~~~

#### 4. RENAME Columns

##### SAS
Usable in DATA, SET, and OUT blocks.
~~~sas
DATA AA(RENAME=(OLD_AMOUNT=NEW_AMOUNT OLD_NO=NEW_NO));
RUN;


DATA AA;
SET AA;
NEW_AMOUNT = OLD_AMOUNT;
NEW_NO = OLD_NO;
RUN;
~~~

##### Python
~~~python
df.rename(columns={'old_name':'new_name', 'old_no':'new_no'}, inplace=True)
~~~


#### 5. SORT

##### SAS
~~~sas
PROC SORT DATA=aaa; BY NO;
RUN;
~~~
##### Python
~~~python
df.sort_values(by=['no','name'], inplace=True)
~~~

#### 6. Transpose (rows -> columns)
ASIS
NO CODE1 CODE2
1 1000 2000

TOBE
NO AMOUNT PRODUCT_CODE
1 1000 CODE1
1 2000 CODE2

##### SAS
~~~sas
PROC TRANSPOSE DATA=aaa
OUT = aaa_TR(RENAME=(COL1=AMOUNT)) NAME=PRODUCT_CODE;
BY NO;
RUN;
~~~

##### Python
~~~python
df_tr = pd.melt(data=df, id_vars=['NO'], value_vars=['CODE1', 'CODE2'], var_name='PRODUCT_CODE', value_name='AMOUNT')
df_tr.sort_values(by=['NO'], inplace=True)
~~~

#### 7. Transpose (columns -> rows)
ASIS
NO AMOUNT PRODUCT_CODE
1 1000 CODE1
1 2000 CODE2

TOBE
NO CODE1 CODE2
1 1000 2000

##### SAS
~~~sas
PROC TRANSPOSE DATA=aaa
NAME = PRODUCT_CODE
LABEL = PRODUCT_NAME LET OUT = aaa_TR;
ID PRODUCT_CODE;
BY NO;
RUN;
~~~

##### Python
Note: with pivot_table, if the index column contains NaN values, that row won't appear in the output, so missing values in the index column need to be handled first.
~~~python
df_tr = pd.pivot_table(data=df, index=['NO'], columns='PRODUCT_CODE', values='AMOUNT')
df_tr.reset_index(inplace=True)
~~~

#### 8. Filling Missing Numeric Values With 0

##### SAS
~~~sas
DATA aaa;
SET aaa;
  ARRAY VALUE{*} _numeric_;
  do i=1 to dim(VALUE);
    IF VALUE(i) EQ . THEN VALUE(i)=0;
  end;
RUN;  
~~~

##### Python
~~~python
df.update(df.select_dtypes(include=[np.number]).fillna(0))
~~~

#### 9. Deletion

##### Deleting rows

###### SAS
~~~sas
PROC SQL;
DELETE FROM AAA
WHERE NO = '1';
QUIT;
RUN;
~~~

###### Python
~~~python
df.drop(df[df['NO'] =='1'].index, inplace=True)
~~~

##### Deleting columns

###### SAS
~~~sas
DATA AA(DROP=NO NAME);
RUN;
~~~

###### Python
~~~python
df.drop(['NO', 'NAME'], axis=1, inplace=True)
~~~

#### 10. Concatenating Datasets Vertically

##### SAS
~~~sas
PROC APPEND BASE=NEW_DATA DATA=AAA FORCE; RUN;
~~~

##### Python
~~~python
#1. append
df_new = df_new.append(df_aaa)
#2. concat
df_new = pd.concat([df_aaa, df_bbb], axis=0)
~~~

#### 11. Joining Datasets Horizontally (left merge)

##### SAS
~~~sas
DATA AB;
MERGE A(IN=T1) B(IN=T2);
BY NO;
IF T1=1 OR T2=0;
~~~

##### Python
~~~python
df_ab = df_a.merge(right=df_b, how='left', on=['NO'])
~~~

#### 12. Removing Duplicates
Besides this approach, you can also use DISTINCT in a query.
##### SAS
~~~sas
PROC SORT DATA=aa NODUPKEY;BY NO;
RUN;
~~~
##### Python
~~~python
df.drup_duplicates(subset=['NO','NAME'], inplace=True, keep='first')
~~~

#### 13. Adding a New Column Based on a Condition
Besides this approach, you can also use CASE WHEN in a query.
Example 1)
##### SAS
~~~sas
DATA AAA;
SET AAA;
IF (REGION="A") AND AGE >= 50 THEN DO;
  GROUP="1";
END;
ELSE IF (REGION="A") AND AGE < 50 THEN DO;
  GROUP="2";
END;
IF NAME="ABC" THEN DO;
GROUP="99";
END;
RUN;
~~~
##### Python
~~~python
df_aaa.loc[(df_aaa['REGION'] == 'A') & (df_aaa['AGE'] >= 50),'GROUP'] = '1'
df_aaa.loc[(df_aaa['REGION'] == 'A') & (df_aaa['AGE'] < 50),'GROUP'] = '2'
df_aaa.loc[(df_aaa['NAME'] == 'ABC'),'GROUP'] = '99'
~~~
Example 2)
##### SAS
~~~sas
DATA AAA;
SET AAA;

IF SUBSTSR(END_DATE,5,2)>="01" AND SUBSTSR(END_DATE,5,2)<="05" THEN CODE = "1";
IF SUBSTSR(END_DATE,5,2)>="06" AND SUBSTSR(END_DATE,5,2)<="11" THEN CODE = "2";
IF SUBSTSR(END_DATE,5,2)>="12" THEN CODE = "3";

IF LENGTH(TRIM(END_DATE))=6 THEN DO;
 MONTH=SUBSTR(END_DATE,5,2);
END;
ELSE DO;
 MONTH="99";
END;
RUN;
~~~
##### Python
~~~python
df_aaa['CODE'] = df_aaa['END_DATE'].apply(lambda x: '1' if ( '01' <= str(x)[4:6] <= '05') else
						    '2' if ( '06' <= str(x)[4:6] <= '11') else
						    '3' if (  str(x)[4:6] == '12') else
						    np.NAN)
						    
df_aaa['MONTH'] = df_aaa['END_DATE'].apply(lambda x: str(x)[4:6] if len(str(x).strip()) == 6 else
						     '99')
~~~
Example 3)
##### SAS
~~~sas
DATA AAA;
SET AAA;

IF AMOUNT = 0 OR AMOUNT=. THEN AMT_CODE = "0";
IF AMOUNT > 0 THEN AMT_CODE = "1";
RUN;

IF (REGION="1") OR (REGION="2") THEN REG_GROUP = "1"; 
ELSE IF (REGION="3") OR (REGION="4") THEN REG_GROUP = "2";
ELSE REG_GROUP="3";
RUN;
~~~
##### Python
~~~python
df_aaa['AMT_CODE'] = df_aaa['AMOUNT'].apply(lambda x: '0' if ( x == 0 or np.isnan(x)) else
						      '1' if ( x > 0 ) else
						      np.NAN)

df_aaa['REG_GROUP'] = df_aaa['REGION'].apply(lambda x: '1' if x in ['1','2'] else
                                                       '2' if x in ['3','4'] else
						       '3')
~~~




#### Extra 1: How to Log SAS Macros
~~~sas
FILENAME MPRINT 'C:\DATA\LOG01.SAS';
OPTIONS MPRINT MFILE MLOGIC SYMBOLGEN;
RUN;
~~~

#### Extra 2: Rewriting a SAS Macro Function in Python

~~~sas
%MACRO FIND_FML(SEQ);
	DATA _NULL_;
		LENGTH SEQ 5. FORMULA $ 500;
		SET formula_rule(WHERE=(SEQ=&SEQ)) end=last;
		CALL SYMPUT("FML_"||LEFT(_N_), COMPRESS(FORMULA)||';');
		IF LAST THEN CALL SYMPUT("ls_FML", _N_);
	RUN;

	%LET FML_LIST=;
	%LET FML_QUOTE=;

	%DO m = 1 %TO &ls_FML;
		%LET FML_LIST=&FML_LIST &&FML_&m ;	
	%END; 
%MEND FIND_FML;

%DO I = 1 %TO &fml_cnt;
  %LET FML_LIST=;
  %FIND_FML(&&seq&i);
  DATA AAA;
  SET AAA;
  IF &&ls_fml&i THEN DO;
     &FML_LIST;
  END;
%END;  
	
~~~

~~~python
from pandas.core.computation.ops import UndefinedVariableError

def find_fml_df(seq):
    rule_tmp = df_formula_rule.loc[df_formula_rule['SEQ']==seq]    
    return rule_tmp

fml_df = find_fml_df(seq)
for index, row in fml_df.iterrows():
   try:
      df_aaa.eval(row['formula'],inplace=True)
   except UndefinedVariableError as err:
      print(err)
      df_aaa[row['left_val']] = np.nan
   except TypeError as err:
      print(err)
      df_aaa[row['left_val']] = np.nan
   except Exception as err:
      print(err)
~~~
   
#### Extra 3: SAS WORK KILL
~~~sas
PROC DATASETS LIBRARY=WORK MEMTYPE=DATA KILL; QUIT; RUN;
~~~


#### PYTHON: Extracting DataFrame Columns Matching a Pattern
~~~python
pattern = re.compile("A*[0-9]")
t1_columns = ",".join('T1.'+ a_column for a_column in list(filter(lambda x: pattern.match(x), df_aaa.columns)))

pattern = re.compile("A*[0-9]")
columns = list(filter(lambda x: pattern.match(x), df_aaa.columns))
~~~

#### PYTHON: Extracting Rows Where Specific Columns Sum to 0
~~~python
pattern = re.compile("A*[0-9]")
columns = list(filter(lambda x: pattern.match(x), df_aaa.columns))

df_bbb = df_aaa.loc[df_aaa[columns].sum(axis=1) == 0]

~~~


#### PYTHON: Building a Separate DataFrame of Rows That Fail a Formula Check (A1=A2+A3)
~~~python
import operator
ops = {
    "+": operator.add,
    "-": operator.sub,
    "*": operator.mul,
    "/": operator.truediv,
    "<": operator.lt,
    ">": operator.gt,
    "=": operator.eq,
    "=<": operator.le,
    "<=": operator.le,
    "=>": operator.ge,
    ">=": operator.ge
}

column_list = ['NO', 'ORG', 'HAP', 'CHA']
invalid_df = pd.DataFrame(columns=column_list)
#A1=A2+A3
#left = A1, right=A2+A3
for idx, row in df_aaa.iterrows():
    for idx2, items in df_formula_rule.iterrows():
        op_func = ops[items['OPERATOR']]
        left = eval(items['org'].replace("A", "row.A"))
        right = eval(items['hap'].replace("A", "row.A"))

	if op_func(left, right):
            continue
        else:
            invalid_df.loc[len(invalid_df.index)] = [row['NO'], left, right, left-right]

invalid_df.sort_values(by=['NO'], inplace=True)
~~~

#### PYTHON: Evaluating With Postfix Notation
~~~python
"""**************************************************************************************************
 0. Function: implements arithmetic across dataframe columns using a postfix-notation algorithm
   - Infix notation: (A+B)*(C+D)
   - Postfix notation: AB+CD+*
   - Converting infix to postfix using a stack
     1) an operand is output directly, without pushing it onto the stack
	 2) if the stack is empty, push the operator
	 3) if the stack isn't empty, compare the operator's precedence with the one on top of the stack;
	    if the one on the stack has equal or higher precedence, pop and output it, then push the current operator
	 4) if the current operator has higher precedence than step 3's comparison, push the current operator
	 5) once the expression ends, pop everything remaining on the stack and output it
   rule  = ['A3', '+', 'A4', '+', 'A5', '+', 'A6'] 
   rule2 = ['A3', 'A4', '+', 'A5', '+', 'A6', '+'] 
**************************************************************************************************"""
def get_sum(df, rule) :
	p = re.compile('\W')
	rule = rule.replace(" ", "")
	match_all = p.finditer(rule)
	idx = 0
	tokens = []
	stack = []
	rule2 = []
	for match in match_all :
		logger.info(match)
		if idx != match.start() :
			tokens.append(rule[idx:match.start()])
		tokens.append(match.group())
		idx = match.end()
	if idx != len(rule) :
		tokens.append(rule[idx:])
	
	for token in tokens :
		if token == '+' or token == '-' or token == '*' or token == '/' or token == '(' or token == ')':
			if len(stack) == 0 :
				stack.append(token)
			else :
				if token == ')' :
					j = len(stack)-1
					while True :
						if stack[j] == '(' or j < 0 :
							stack.pop(j)	
							break
						rule2.append(stack[j])
						stack.pop(j)	
						j = j-1
				if token == '+' or token == '-' :
					if stack[len(stack)-1] != '(' :
						rule2.append(stack[len(stack)-1])
						stack.pop(len(stack)-1)
						stack.append(token)
					else :
						stack.append(token)
				elif token == '*' or token == '/' :
					if stack[len(stack)-1] != '+' and stack[len(stack)-1] != '-' and stack[len(stack)-1] != '(' :
						rule2.append(stack[len(stack)-1])
						stack.pop(len(stack)-1)
						stack.append(token)
				elif token == '(' :
					stack.append(token)
		else :
			rule2.append(token)
	for i in range(len(stack), 0, -1) :
		if stack[i-1] != '(' :
			rule2.append(stack[i-1])
	
	# process the rule now that it's been converted to postfix notation
	# if rule2 has a single operand, that value is the target value directly
	# scan rule2 from the front for an operator, process the two operands before it, remove the operator and both operands, and append the result back into rule2
	# target : result
	# rule2 = ['A3', 'A4', '+', 'A5', '+', 'A6', '+'] 
	# operation: df['result'] = df['A3']+df['A4']
    # rule2 = ['result','A5','+','A6','+']
	# operation: df['result'] = df['result']+df['A5']
    # rule2 = ['result','A6','+']
	# operation: df['result'] = df['result']+df['A6']
	col1 = 'result'
	col2 = ''
	col3 = ''
	idx = 0
	if len(rule2) == 1 :
		if rule2[0] == '0' :
			df[col1] = 0
		else :
			df[col1] = df[rule2[0]]
	else :
	    k = 0
		result = 0
	    while True :
			st = rule2[k] 
			if st == '+' or st == '-' or st == '*' or st == '/' :
			    col1 = 'result'
				col2 = rule2[k-2]
				col3 = rule2[k-1]
				rule2.pop(k)
				rule2.pop(k-1)
				rule2[k-2] = col1
				if col3 != '0' :
					if st == '+' :
						df[col1] = df[col2] + df[col3]
					elif st == '-' :
						df[col1] = df[col2] - df[col3]
					elif st == '*' :
						df[col1] = df[col2] * df[col3]
					elif st == '/' :
						df[col1] = df[col2] / df[col3]
				k = 0
			else :
			    k = k+1
			if len(rule2) == 1 :
			    break;
"""**************************************************************************************************
 3. Call get_sum to compute each formula-check ratio's column operation, producing a 'result' column
    (the right-hand side of the formula), then compare it against the target column ('org_column',
    the left-hand side) to pull out rows that don't satisfy the condition, and generate an error file.
    E.g. formula => A_column = B_column + C_column 
**************************************************************************************************"""
for index, row in formula_df.iterrows() :
    get_sum(df_aaa, row['formula_hap']) 
	if row['operater2'] == '<>' :
	    tmp_df = df_aaa[(df_aaa[row['org_column']] != df_aaa['result'])]
	elif row['operater2'] == '=<' :
	    tmp_df = df_aaa[(df_aaa[row['org_column']] <= df_aaa['result'])]
	elif row['operater2'] == '=>' :
	    tmp_df = df_aaa[(df_aaa[row['org_column']] >= df_aaa['result'])]
	elif row['operater2'] == '<' :
	    tmp_df = df_aaa[(df_aaa[row['org_column']] < df_aaa['result'])]
	elif row['operater2'] == '>' :
	    tmp_df = df_aaa[(df_aaa[row['org_column']] > df_aaa['result'])]
	if len(tmp_df) > 0 :
		tmp2_df = tmp_df[['no', row['org_column'], 'result']]
		tmp2_df.rename(columns={row['org_column']:'org'}, inplace=True)
		tmp2_df.rename(columns={'result':'hap'}, inplace=True)
		tmp2_df['cha'] = tmp2_df['org'] - tmp2_df['hap']
		tmp2_df['fml'] = row['fml']
		tmp2_df['column_no'] = row['org_column_no']
		invalid_df = pd.concat([invalid_df, tmp_df])
		invalid_df = invalid_df.drop(['result'], axis=1)
		invalid_df = invalid_df.drop_duplicates()
		invalid_sum_df = pd.concat([invalid_sum_df, tmp2_df])
~~~

#### Business-Analysis Data Preprocessing and Compilation Procedure

1. Load the data and confirm parameters
2. Validate the raw data for errors (take the absolute value and sum where a condition applies)
3. Transform account mappings (map dataset A -> dataset B)
4. Define new columns based on per-column conditions
5. Validate formulas (the check for whether A1 = A2 + A3 holds)
