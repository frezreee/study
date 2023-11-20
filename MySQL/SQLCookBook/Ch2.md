# 쿼리 결과 정렬

## 2.1 지정한 순서대로 쿼리 결과 반환하기
- 부서 10에 속한 사원명, 직책 및 급여를 급여 순서대로 표시
```sql
select ename, job, sal
from emp
where deptno = 10
order by sal asc
```
- 기본적으로 ORDER BY는 오름차순으로 정렬되므로 내림차순으로 정렬하고자 할 때는 DESC를 지정한다.
```sql
select ename, job, sal
from emp
where deptno = 10
order by sal desc
```
<br>

## 2.2 다중 필드로 정렬하기
- emp 테이블에서 DEPTNO 기준 오름차순으로 행 정렬 후, 급여 내림차순으로 정렬한다.
- ORDER BY 우선순위 : 왼쪽 -> 오른쪽
```sql
select empno, deptno, sal, ename, job
from emp
order by deptno, sal desc
```
<br>

## 2.3 부분 문자열로 정렬하기
- EMP 테이블에서 사원명과 직급을 반환하되 JOB 열의 마지막 두 문자를 기준으로 정렬한다.
```sql
select ename, job
from emp
order by substr(job, lenth(job)-1)
```

<br>

## 2.4 혼합 영숫자 데이터 정렬하기
- 혼합 영숫자 데이터가 있을 때 데이터의 숫자 또는 문자 부분을 기준으로 정렬한다
```sql
create view V
as
   select ename||' '||deptno as data
    from emp;

select * from V
```
- MySQL은 TRANSALTE 함수를 지원하지 않아 해법을 제공하지 않는다.
- Oracle, SQL Server, POstgreSQL 해법
```sql
/* DEPTNO로 정렬하기 */
select data
from V
order by replace(data,
         replace(
                  translate(data, '0123456789', '#########'), '#', ''). '')

/* ENAME으로 정렬하기 */
select data
from V
order by replace(
                  translate(data, '0123456789', '#########'), '#', '')
```

<br>

## 2.5 정렬할 때 null 처리하기
- null을 마지막에 정렬할지 아닐지 지정하는 방법
```sql
select ename, sal, comm -- null을 오름차순으로 정렬
from emp
order by 3

select ename, sal, comm -- null을 내림차순으로 정렬
from emp
order by 3 desc
```
- null이 아닌 값을 오름차순 또는 내림차순으로 정렬하고 모든 null 값을 마지막으로 정렬한다
```sql
/* NULL이 아닌 COMM을 우선 오름차순 정렬하고, 모든 NULL은 마지막에 나타냄 */
select ename, sal, comm
from (
       select ename, sal, comm,
              case when comm is null then 0 else 1 end as is_null
       from emp ) x
order by is_null desc, comm
```
```sql
/* NULL이 아닌 COMM을 우선 내림차순 정렬하고, 모든 NULL은 마지막에 나타냄 */
select ename, sal, comm
from (
       select ename, sal, comm,
              case when comm is null then 0 else 1 end as is_null
       from emp ) x
order by is_null desc, comm desc
```
```sql
/* NULL을 처음에 나타낸 후, NULL이 아닌 COMM은 오름차순 정렬 */
select ename, sal, comm
from (
       select ename, sal, comm,
              case when comm is null then 0 else 1 end as is_null
       from emp ) x
order by is_null, comm
```
```sql
/* NULL을 처음에 나타낸 후, NULL이 아닌 COMM은 내림차순 정렬 */
select ename, sal, comm
from (
       select ename, sal, comm,
              case when comm is null then 0 else 1 end as is_null
       from emp ) x
order by is_null, comm desc
```

<br> 

## 2.6 데이터 종속 키 기준으로 정렬하기
- JOB이 'SALESMAN'이면 COMM 기준으로 정렬하고, 그렇지 않으면 SAL 기준으로 정렬한다.
```sql
select ename, sal, job, comm
from emp
order by case when job = 'SALESMAN' then comm else sal end
```
