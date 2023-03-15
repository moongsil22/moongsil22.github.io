---
layout: post
title: How to Convert SAS to Python
description: >
  Howdy! This is an example blog post that shows several types of HTML content supported in this theme.
sitemap: false
hide_last_modified: true
---

Cum sociis natoque penatibus et magnis <a href="#">dis parturient montes</a>, nascetur ridiculus mus. *Aenean eu leo quam.* Pellentesque ornare sem lacinia quam venenatis vestibulum. Sed posuere consectetur est at lobortis. Cras mattis consectetur purus sit amet fermentum.

> Curabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.

Etiam porta **sem malesuada magna** mollis euismod. Cras mattis consectetur purus sit amet fermentum. Aenean lacinia bibendum nulla sed consectetur.

## Inline HTML elements

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `**To bold text**`.
- *To italicize text*, use `*To italicize text*`.
- Abbreviations, like HTML should be defined like this `*[HTML]: HyperText Markup Language`.
- Citations, like <cite>&mdash; Mark otto</cite>, should use `<cite>`.
- ~~Deleted~~ text should use `~~deleted~~` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Heading 2
Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

### Heading 3
Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor.

#### Heading 4
Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor.

##### Heading 5
Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor.

###### Heading 6
Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor.

## Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

~~~js
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those
// arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
~~~

## Code2

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

~~~js
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those
// arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
~~~

#### 0. 오라클 연동, 라이브러리 참조, 변수 선언

#### SAS 

오라클 연동 시, 선행작업으로 오라클 클라이언트 설치와 tnsnames.ora 파일 설정이 필요하다

~~~sas
FILENAME PGM_PATH "&SAS_PGM_PTH./";
%INCLUDE PGM_PATH(common.sas);
LIBNAME ORADB ORACLE USER="test" PW="123456" PATH="TESTDB" SCHEMA="TEST" ORACLE_73=NO UPDATE_ISOLATION_LEVEL=READCOMMITTED DBINDEX=YES DBMAX_TEST=32767;

%LET DATA_DIR = "C:\DATA";
libname saslib "&DATA_DIR.\saslib";
~~~

#### Python

cx_Oracle 설치 필요. -> pip install cx_Oracle
oracleTest.py 만든 다음사용. -> import oracleTest 

oracleTest.py
~~~python
import cx_Oracle as co
import pandas as pd
from _datetime import datetime


def query_OracleSQL(query):

    start_tm = datetime.now()

    #   DB Connecion
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


#### 1. Dataset Load From CSV, EXCEL, SQL query , Exist Dataset

##### 1.1 CSV LOAD

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
  NO = "고유번호"
  AMOUNT = "매출액"
  PRODUCT_CODE1 = "상품1"
  PRODUCT_CODE2 = "상품2";
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

T_001 = pd.read_csv(r"C:\Users\BOK\Desktop\단계별데이터추출\STEP1.T_001.csv", sep=',',
names = ['no', 'name', 'age'],
header=None,
dtype = {"no":str,
         "name":str,
	 "age":str
	 },
low_memory=False)

T_001.columns = map(lambda x: x.upper(), T_027P100.columns)
~~~



##### 1.2 EXCEL LOAD

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

##### 1.3 ORACLE/QUERY TABLE LOAD

###### SAS
PROC SQL에서 SAS라이브러리/WORK/ORACLE DATA SET 로드하여 사용 가능. 테이블 생성 이외에 dml 처리 가능
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
pandasql에서 dataframe 로드하여 사용 가능. 테이블 생성 이외에 dml 처리 가능
~~~python
import pandasql as ps
query = f"select no, name from df1"
df2 = ps.sqldf(query)
~~~

##### 1.4 기생성된 데이터셋 참조하여 새로운 데이터셋 만들기

###### SAS
~~~sas
DATA NEWAA(KEEP=NAME CODE DROP=CNT RENAME=(NO=NEW_NO));
SET AAA(where=(time='2019'));
~~~
###### Python
~~~python

no_list = ['1','3','7']

df2 = df1.copy() #원본데이터 변경 방지를 위해 copy() 사용
df2 = df2.loc[(~df2['no'].isin(no_list)) & (df2['name'] !='KANG')]

~~~

#### 1.2 File download

##### SAS

1) dat
~~~sas
DATA _NULL_;
SET saslib.AA;
FILE OUT_PATH(&OUT) MOD;
IF FIRST.NO THEN DO;
  PUT @20 "&TITLE";
  PUT /"&ls_t1_line";
  PUT @2 "데이터파일" @40 "처리건수";
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
  if _n_ = 1 then /*첫째줄 컬럼명 */
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
   PUT "종류" '09'X "사용여부" '09'X "건수" ;
   PUT "&ls_t1_line"/;
  end;
  PUT NO USE_YN CNT
run;  
~~~

##### Python
~~~python
df.to_csv('./aa.csv')
df.to_excel('./aa.xls')
~~~


#### 2. DROP 컬럼

##### SAS
DATA, SET, OUT 블럭에서 사용가능
~~~sas
DATA aaa(DROP=AMOUNT NO);
RUN;

DATA aaa;
DROP AMOUNT;
RUN;
~~~

#### 3. RENAME 컬럼

##### SAS
DATA, SET, OUT 블럭에서 사용가능
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


#### 3. SORT 정렬

##### SAS
~~~sas
PROC SORT DATA=aaa; BY NO;
RUN;
~~~
##### Python
~~~python
df.sort_values(by=['no','name'], inplace=True)
~~~

#### 4. 데이터 전치(행 -> 열)
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

#### 5. 데이터 전치(열 -> 행)
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
~~~python
df_tr = pd.pivot_table(data=df, index=['NO'], columns='PRODUCT_CODE', values='AMOUNT')
df_tr.reset_index(inplace=True)
~~~

#### 6. 숫자형 변수 결측값 0으로 처리

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

#### 7. 삭제

##### 행 삭제

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

##### 열 삭제

###### SAS
~~~sas
DATA AA(DROP=NO NAME);
RUN;
~~~

###### Python
~~~python
df.drop(['NO', 'NAME'], axis=1, inplace=True)
~~~

#### 8. 데이터셋 세로로 결합

##### SAS
~~~sas
PROC APPEND BASE=NEW_DATA DATA=AAA FORCE; RUN;
~~~

##### Python
~~~python
#1.append
df_new = df_new.append(df_aaa)
#2.concat
df_new = pd.concat([df_aaa, df_bbb], axis=0)
~~~

#### 8. 데이터셋 가로로 결합(left merge)

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

#### 10. 중복제거
하기방법 이외에 쿼리에서 distinct 처리
##### SAS
~~~sas
PROC SORT DATA=aa NODUPKEY;BY NO;
RUN;
~~~
##### Python
~~~python
df.drup_duplicates(subset=['NO','NAME'], inplace=True, keep='first')
~~~

#### 10. 조건에 따른 새로운 컬럼 속성 추가
하기방법 이외에 쿼리에서 CASE WHEN 처리
예시1)
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
예시2)
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
예시3)
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





#### 기타1 SAS 매크로 로깅 방법
~~~sas
FILENAME MPRINT 'C:\DATA\LOG01.SAS';
OPTIONS MPRINT MFILE MLOGIC SYMBOLGEN;
RUN;
~~~

#### 기타2 SAS 매크로 함수 생성 to Python

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
   
#### 기타3 SAS WORK KILL
~~~sas
PROC DATASETS LIBRARY=WORK MEMTYPE=DATA KILL; QUIT; RUN;
~~~


#### PYTHON 데이터프레임 컬럼중에 특정패턴 가지는 컬럼 추출
~~~python
pattern = re.compile("A*[0-9]")
t1_columns = ",".join('T1.'+ a_column for a_column in list(filter(lambda x: pattern.match(x), df_aaa.columns)))

pattern = re.compile("A*[0-9]")
columns = list(filter(lambda x: pattern.match(x), df_aaa.columns))
~~~

#### PYTHON 데이터프레임 특정 컬럼들의 합이 0인 row 추출
~~~python
pattern = re.compile("A*[0-9]")
columns = list(filter(lambda x: pattern.match(x), df_aaa.columns))

df_bbb = df_aaa.loc[df_aaa[columns].sum(axis=1) == 0]

~~~


#### PYTHON dataframe 특정 산식(A1=A2+A3)에 맞지 않는 오류건 row 추출하여 별도의 dataframe 생성 
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

#### PYTHON 후위표기법으로 계산하기
~~~python
"""**************************************************************************************************
 0. 함수 : Dataset의 컬럼간의 사칙연산을 후위표기법 알고리즘에 따라 구현
   - 중위표기법 : (A+B)*(C+D)
   - 후위표기법 : AB+CD+*
   - 중위표기법을 후위표기법으로 스택(STACK)을 이용
     1) 피연산자는 스택에 넣지않고 그냥 출력
	 2) 연산자는 스택이 비었으면 스택에 push
	 3) 연산자는 스택이 비어있지 않으면 스택에 있는 연산자와 우선순위를 비교해 스택에 있는 연산자의 우선순위가 
	    같거나 크다면 스택에 있는 연산자를 pop을 한 후 출력하고 현재 연산자를 push
	 4) 만약 3번에서 우선순위가 현재 연산자가 더 크면 현재 연산자를 push
	 5) 수식이 킅나면 스택이 빌때까지 pop한 후 출력
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
	
	# 후기표기법으로 변경된 rule을 연산처리
	# rule2의 피연산자가 1개이면 그값이 바로 target값
	# rule2의 앞부터 연산자를 찾아 그 앞 2개의 피연산자를 연산처리한다. 연산자와 피연산자2개를 삭제하고 연산결과를 rule2에 추가한다
	# target : result
	# rule2 = ['A3', 'A4', '+', 'A5', '+', 'A6', '+'] 
	# 연산: df['result'] = df['A3']+df['A4']
    # rule2 = ['result','A5','+','A6','+']
	# 연산: df['result'] = df['result']+df['A5']
    # rule2 = ['result','A6','+']
	# 연산: df['result'] = df['result']+df['A6']
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
 3. get_sum 함수를 호출하여 오류체크 비율식의 컬럼별 연산을 하여 'result' 컬럼(우측 식) 을 생성하여 비교대상컬럼('org_column')(좌측 식)과
    비교하여 조건에 맞지 않는 데이터를 추려내 오류파일 생성 EX) fml => A_column = B_column + C_column 
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
<pre>
<code>

{% highlight python %}
{% raw %}
{%-- if page.comments --%}
def print_hi(name):
    # Use a breakpoint in the code line below to debug your script.
    print(f'Hi, {name}')  # Press Ctrl+F8 to toggle the breakpoint.


# Press the green button in the gutter to run the script.
if __name__ == '__main__':
    print_hi('PyCharm')
~~~
{%-- endif --%}
{% endraw %}
{% endhighlight %}
</code>
</pre>

## Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

HyperText Markup Language (HTML)
: The language used to describe and define the content of a Web page

Cascading Style Sheets (CSS)
: Used to describe the appearance of Web content

JavaScript (JS)
: The programming language used to build advanced Web sites and applications

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

## Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![800x400](https://via.placeholder.com/800x400 "Large example image")

![400x200](https://via.placeholder.com/400x200 "Medium example image")

![200x200](https://via.placeholder.com/200x200 "Small example image")

## Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

| Name     | Upvotes   | Downvotes |
|:---------|:----------|:----------|
| Alice    |        10 |        11 |
| Bob      |         4 |         3 |
| Charlie  |         7 |         9 |
|==========|===========|===========|
|Totals    |        21 |        23 |

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.

*[HTML]: HyperText Markup Language
*[CSS]: Cascading Style Sheets
*[JS]: JavaScript
