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

#### 1. File upload From CSV, EXCEL, ORACLE 

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

# from EXCEL
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
NAME = PRODUCT_CODE
LABEL = PRODUCT_NAME LET OUT = aaa_TR;
ID PRODUCT_CODE;
BY NO AMOUNT;
RUN;
~~~

#### 5. 데이터 전치(열 -> 행)

#### SAS 예시 ~~~

~~~sas
#TODO
~~~


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


DATA aaa;
PROC SORT; BY NO;
RUN;
~~~

## Python 예시3 liquid, html,markdown

가나다라마바사

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
