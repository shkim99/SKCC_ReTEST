# Part 2

## Problem 1.

```
SELECT  a.id,
        b.type,
        a.status,
        a.amount,
        (a.amount - avg(b.amount)) as difference
FROM account a, account b
WHERE a.type = b.type
AND a.status = 'Active'
GROUP BY a. id, b.type, a.status, a.amount;
```
#### [실행결과]   
![Result](/img/P2-1.PNG)
```
mkdir problem1
cd problem1
vi solution.sql
hive --database problem1 -f solution.sql
```
#### [실행결과]   
![Result](/img/P2-3.PNG)
![Result](/img/P2-2.PNG)

## Problem 2.

```
create database problem2;

create external table solution
(id INT, fname STRING, lname STRING, address STRING, city STRING, state STRING,
zip STRING, birthday STRING, hireday STRING)
row format SERDE 'parquet.hive.serde.ParquetHiveSerDe'
STORED AS
INPUTFORMAT "parquet.hive.DeprecatedParquetInputFormat"
OUTPUTFORMAT "parquet.hive.DeprecatedParquetOutputFormat"
LOCATION '/user/training/problem2/data/employee';
```
#### [실행결과]   
![Result](/img/P2-4.PNG)
![Result](/img/P2-5.PNG)

## Problem 3.
```
create table solution as
select b.id as id , b.fname as fname, b.lname as lname, regexp_replace(b.hphone,'[ )(-]','') as hphone
from account a join customer b
on a.custid=b.id
where a.amount < 0
;
```
#### [실행결과]   
![Result](/img/P2-6.PNG)
![Result](/img/P2-7.PNG)


## Problem 4.
```
create database problem4;
create external table employee1
(
cust_id int,
lname string,
fname string,
address string,
city string,
state string,
zip_cd string
)
row format delimited
fields terminated by '\t'
location '/user/training/problem4/data/employee1/.'
```
#### [실행결과]   
![Result](/img/P2-8.PNG)
```
create external table employee2
(
cust_id int,
number int,
lname string,
fname string,
address string,
city string,
state string,
zip_cd string
)
row format delimited
fields terminated by ','
location '/user/training/problem4/data/employee2/.'
```
#### [실행결과]   
![Result](/img/P2-9.PNG)
```
create table solution as
select res.cust_id, res.fname, res.lname, res.address, res.city, res.state, res.zip_cd
from
(
select cust_id, fname, lname, address, city, state, substr(zip_cd,0,5) zip_cd
from employee1
where state = 'CA'

union all

select cust_id, initcap(fname) fname, initcap(lname) lname, address, city, state, zip_cd
from employee2
where state = 'CA'
) res
;
```
#### [실행결과]   
![Result](/img/P2-10.PNG)

```
insert OVERWRITE DIRECTORY '/user/training/problem4/solution/'
row format delimited
FIELDS TERMINATED BY '\t'
SELECT * FROM solution
;
```
#### [실행결과]   
![Result](/img/P2-11.PNG)
![Result](/img/P2-12.PNG)


## Problem 5.

```
select concat_ws(',', cus.fname, cus.lname, cus.zip)
from customer cus
where city='Palo Alto'
and state = 'CA'

union all

select concat_ws(',', emp.fname, emp.lname, emp.zip)
from employee emp
where city='Palo Alto'
and state = 'CA'
;
```
#### [실행결과]   
![Result](/img/P2-13.PNG)

```
mkdir problem5
cd problem5
vi solution.sql
hive --database problem5 -f solution.sql
```
#### [실행결과]   
![Result](/img/P2-14.PNG)
![Result](/img/P2-15.PNG)

## Problem 6.
```
create table solution as
select id, fname, lname, address, city, state, zip,
      substr(birthday,7,4) as birthyear
from employee
;
```
#### [실행결과]   
![Result](/img/P2-16.PNG)
![Result](/img/P2-17.PNG)

## Problem 7.
```
select concat_ws(',',res.lname,res.fname)
from
(
select lname,fname
from employee
where city = 'Seattle'
order by lname, fname
) res
;
```
#### [실행결과]   
![Result](/img/P2-18.PNG)

```
mkdir problem7
cd problem7
vi solution.sql
hive --database problem7 -f solution.sql
```
#### [실행결과]   
![Result](/img/P2-19.PNG)
![Result](/img/P2-20.PNG)


## Problem 8.

```
sqoop export                              \
--connect jdbc:mysql://localhost/problem8 \
--username cloudera \
--password cloudera \
--table solution    \
--fields-terminated-by '\t' \
--export-dir /user/training/problem8/data/customer/.
```
#### [실행결과]   
![Result](/img/P2-21.PNG)

```
mysql -u cloudera -p problem8

mysql> select * from solution;
```
#### [실행결과]   
![Result](/img/P2-22.PNG)

## Problem 9.

```
use problem9 ;
create table solution as
select concat('A',id) id, fname, lname, address, city, state, zip from customer
;
```
#### [실행결과]   
![Result](/img/P2-23.PNG)


## Problem 10.

```
create view solution as
select c.id id, c.fname fname, c.lname lname, c.state state, c.zip zip, b.charge charge,
to_date(b.tstamp)  as billdata
from billing b, customer c
where b.id = c.id;
select * from solution;
```
#### [실행결과]   
![Result](/img/P2-24.PNG)


## Problem 11.
#### A.
```
select o.prod_id, p.name, count(*) cnt from order_details o, products p
where o.prod_id = p.prod_id
and p.brand = 'Dualcore'
group by o.prod_id, p.name
order by cnt desc
limit 3;
```
#### [실행결과]   
![Result](/img/P2-25.PNG)

```
mkdir problem11
cd problem11
vi solution.sql
hive --database default -f solution.sql
```
#### [실행결과]   
![Result](/img/P2-26.PNG)
![Result](/img/P2-27.PNG)

#### B.
```
select to_date(o.order_date) as dt,
      sum(p.price) as revenue,
      sum(p.price-p.cost) as profit
from orders o, order_details d, products p
where o.order_id = d.order_id
and d.prod_id = p.prod_id
and p.brand = 'Dualcore'
group by to_date(o.order_date);
```
```
vi solution2.sql
hive --database default -f solution2.sql
```
#### [실행결과]   
![Result](/img/P2-28.PNG)
![Result](/img/P2-29.PNG)
![Result](/img/P2-30.PNG)

#### C.
```
select o.order_id order_id, sum(p.price) revenue
from orders o, order_details d, products p
where o.order_id = d.order_id
and d.prod_id = p.prod_id
group by o.order_id
order by revenue desc
limit 10;
```
#### [실행결과]   
![Result](/img/P2-31.PNG)
![Result](/img/P2-32.PNG)
![Result](/img/P2-33.PNG)

## End Of Document
