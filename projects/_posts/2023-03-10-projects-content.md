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

#### SAS 예시 ~~~

오라클 연동 시, 선행작업으로 오라클 클라이언트 설치와 tnsnames.ora 파일 설정이 필요하다

~~~sas
FILENAME PGM_PATH "&SAS_PGM_PTH./";
%INCLUDE PGM_PATH(common.sas);
LIBNAME ORADB ORACLE USER="test" PW="123456" PATH="TESTDB" SCHEMA="TEST" ORACLE_73=NO UPDATE_ISOLATION_LEVEL=READCOMMITTED DBINDEX=YES DBMAX_TEST=32767;

%LET DATA_DIR = "C:\DATA";
libname saslib "&DATA_DIR.\saslib";
~~~

#### 1.1 File upload From CSV, EXCEL, ORACLE 

#### SAS 예시 ~~~

가나다라마바사

~~~sas
# from CSV 

#### 1. File upload From CSV, EXCEL, ORACLE, 기생성된 데이터셋   

#### SAS 예시 ~~~

가나다라마바사

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

EXCEL

~~~sas
PROC IMPORT OUT=WORK.DATA
DATAFILE="C:\excel\file_name.xlsx"
DBMS=XLSX REPLACE;
RANGE="SHEET1$A1:B5";
GETNAMES=YES;
RUN;
~~~

ORACLE

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

기생성된 데이터셋 참조

~~~sas
DATA NEWAA(KEEP=NAME CODE DROP=CNT RENAME=(NO=NEW_NO));
SET AAA;
WHERE
~~~

#### Python 예시 ~~~

가나다라마바사

~~~python
def print_hi(name):
    # Use a breakpoint in the code line below to debug your script.
    print(f'Hi, {name}')  # Press Ctrl+F8 to toggle the breakpoint.


# Press the green button in the gutter to run the script.
if __name__ == '__main__':
    print_hi('PyCharm')
~~~    

#### 1.2 File download

#### SAS 예시 ~~~

dat

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

CSV
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

EXCEL
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

#### 2. DROP 컬럼

#### SAS 예시 ~~~

DATA, SET, OUT 블럭에서 사용가능

~~~sas
DATA aaa(DROP=AMOUNT NO);
RUN;

DATA aaa;
DROP AMOUNT;
RUN;
~~~

#### 3. SORT 정렬

#### SAS 예시 ~~~

PROC SORT 구문 이용

~~~sas
PROC SORT DATA=aaa; BY NO;
RUN;
~~~

#### 4. 데이터 전치(행 -> 열)

#### SAS 예시 ~~~

~~~sas
PROC TRANSPOSE DATA=aaa
OUT = aaa_TR(RENAME=(COL1=AMOUNT)) NAME=PRODUCT_CODE;
BY NO;
RUN;
~~~
[ASIS]
NO CODE1 CODE2
1 1000 2000
[TOBE]
NO AMOUNT PRODUCT_CODE
1 1000 CODE1
1 2000 CODE2


#### 5. 데이터 전치(열 -> 행)

#### SAS 예시 ~~~

~~~sas
PROC TRANSPOSE DATA=aaa
NAME = PRODUCT_CODE
LABEL = PRODUCT_NAME LET OUT = aaa_TR;
ID PRODUCT_CODE;
BY NO AMOUNT;
RUN;
~~~
[ASIS]
NO AMOUNT PRODUCT_CODE
1 1000 CODE1
1 2000 CODE2
[TOBE]
NO CODE1 CODE2
1 1000 2000


#### 6. 숫자형 변수 결측값 처리

#### SAS 예시 ~~~

~~~sas
DATA aaa;
SET aaa;
  ARRAY VALUE{*} _numeric_;
  do i=1 to dim(VALUE);
    IF VALUE(i) EQ . THEN VALUE(i)=0;
  end;
RUN;  
~~~

#### 7. SQL Query 로 데이터 작업

#### SAS 예시 ~~~
SAS는 PROC SQL구문 안에서 ANSI SQL 문법 이용하여 데이터셋 생성/삭제/변경 가능.

~~~sas
PROC SQL;
CREATE TABLE B AS
SELECT * FROM AAA
WHERE NO = '1'
QUIT;
RUN;

PROC SQL;
DELETE FROM AAA
WHERE NO = '1'
QUIT;
RUN;

~~~

#### 8. 데이터셋 세로로 결합

#### SAS 예시 ~~~
SAS는 PROC APPEND 이용

~~~sas
PROC APPEND BASE=NEW_DATA DATA=AAA FORCE; RUN;

CREATE TABLE B AS
SELECT * FROM AAA
WHERE NO = '1'
QUIT;
RUN;

PROC SQL;
DELETE FROM AAA
WHERE NO = '1'
QUIT;
RUN;

~~~

#### 기타1 SAS 매크로 로깅 방법
~~~sas
FILENAME MPRINT 'C:\DATA\LOG01.SAS';
OPTIONS MPRINT MFILE MLOGIC SYMBOLGEN;
RUN;
~~~

#### 기타2 SAS 매크로 함수 생성
~~~sas
	%MACRO FIND_FML(SEQ);
		DATA _NULL_;
			LENGTH KEY_SEQ 5. FORMULA $ 500;
			SET formula_rule(WHERE=(SEQ=&SEQ)) end=last;
			CALL SYMPUT("FML_"||LEFT(_N_), tranwrd(COMPRESS(FORMULA), 'ABS(','ABS(_')||';');
			IF LAST THEN CALL SYMPUT("ls_FML", _N_);
		RUN;

		%LET FML_LIST=;
		%LET FML_QUOTE=;

		%DO m = 1 %TO &ls_FML;
			%LET FML_LIST=&FML_LIST &&FML_&m ;	
		%END; 
	%MEND FIND_FML;
~~~
#### 기타3 SAS WORK KILL

PROC DATASETS LIBRARY=WORK MEMTYPE=DATA KILL; QUIT; RUN;


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
