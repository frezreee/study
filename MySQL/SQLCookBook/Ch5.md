# 메타 데이터 쿼리

<br>

## 5.1 스키마의 테이블 목록 보기
- INFORMATION_SCHEMA.TABLES를 쿼리한다.

```sql
select table_name
from information_schema.tables
where table_schema = 'SMEAGOL'
```

<br>

## 5.2 테이블의 열 나열하기
- SMEAGOL 스키마의 EMP라는 테이블에 열, 해당 데이터 유형 및 숫자 위치를 나열한다.

```sql
select column_name, data_type, ordinal_position
from information_schema.columns
where table_schema = 'SMEAGOL' and table_name = 'EMP'
```

<br>

## 5.3 테이블의 인덱싱된 열 나열하기
- SMEAGOL 스키마에서 EMP 테이블에 대한 인덱스를 나열한다.

```sql
show index from emp
```

<br>

## 5.4 테이블의 제약조건 나열하기
- EMP 테이블에 대한 제약조건 및 해당 제약조건이 있는 열을 찾으려고 한다.

```sql
select a.table_name,
       a.constraint_name,
       b.column_name,
       a.constraint_type
from information_schema.table_constraints a,
     information_schema.key_column_usage b
where a.table_name = 'EMP'
  and a.table_schema = 'SMEAGOL'
  and a.table_name = b.table_name
  and a.table_schema = b.table_schema
  and a.constraint_name = b.constraint_name
```

<br>

## 5.6 SQL로 SQL 생성하기
- 유지 관리 작업을 자동화하고자 동적 SQL 문을 생성하려고 한다.
  - 테이블의 행 수를 계산한다.
  - 테이블에 정의된 외래 키 제약조건을 비활성화한다.
  - 테이블의 데이터에서 삽입 스크립트를 생성한다.
 

```sql
-- 모든 테이블에서 모든 행의 수를 세는 SQL 생성
select 'select count(*) from '||table_name||';' cnts
from user_tables;
```
```sql
-- 모든 테이블의 외래 키를 비활성화하기
select 'alter table '||table_name|| ' disable constraint '||constraint_name||';' cons
from user_constraints
where constraint_type = 'R';
```
```sql
-- EMP 테이블의 일부 열에 삽입하는 스크립트 생성하기
select 'insert into emp(empno, ename, hiredate) '||char(10)||' values ('||empno||', '||''''||ename||''', to_date('||''''||hiredate||''') );' inserts
from emp
where deptno = 10;
```

<br>

## 5.7 Oracle에서 데이터 딕셔너리 뷰 확인하기
- DICTIONARY라는 뷰를 쿼리하여 데이터 딕셔너리 뷰 및 용도를 나열한다.

```sql
select table_name, comments
from dictionary
order by table_name;
```
```sql
select column_name, comments
from dict_columns
where table_name = 'ALL_TAB_COLUMNS';
```
