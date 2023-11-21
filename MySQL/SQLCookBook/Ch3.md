# 다중 테이블 작업

<br>

## 3.1 행 집합을 다른 행 위에 추가하기
- EMP 테이블에 있는 부서 10의 사원명 및 부서 번호와 함께, DEPT 테이블에 있는 각 부서명 및 부서 번호를 표시한다.
```sql
select ename as ename_and_dname, deptno
from emp
where deptno = 10
union all
select '------', null
from t1
union all
select dname, deptno
from dept
```

<br>

- UNION ALL : 중복 항목 포함 / UNION : 중복 항목 필터링
- UNION = UNION ALL + DISTINCT
```sql
select distinct deptno
from (
        select deptno
        from emp
        union all
        select deptno
        from dept )
```

<br>

## 3.2 연관된 여러 행 결합하기
- 부서 10의 모든 사원명과 각 사원의 부서 위치를 함께 표시한다.
```sql
select e.ename, d.loc
from emp e, dept d
where e.deptno = d.deptno and e.deptno = 10
```

<br>

- 동등 조인 (equi-join) : 조인 조건이 동등 조건에 기반을 두는 조인
- FROM 절에 나열된 테이블에서 데카르트 곱 (cartesian product) 을 우선으로 생성
```sql
select e.ename, d.loc,
       e.deptno as emp_deptno,
       d.deptno as dept_deptno
from emp e, dept d
where e.deptno = 10
```

<br>

- 대체 해법
```sql
select e.ename, d.loc
from emp e inner join dept d on (e.deptno = d.deptno)
where e.deptno = 10
```

<br>

## 3.3 두 테이블의 공통 행 찾기
```sql
create view V
as
select ename, job, sal
from emp
where job = 'CLERK';

select * from V
```

<br>

- INTERSECT 사용하지 않고 두 테이블의 교차점(공통 행)을 반환한다.
```sql
select e.empno, e.ename, e.job, e.sal, e.deptno
from emp e, V
where e.ename = v.ename
  and e.job = v.job
  and e.sal = v.sal
```
```sql
select e.empno e.ename, e.job, e.sal, e.deptno
from emp e join V
on (e.ename = v.ename
and e.job = v.job
and e.sal = v.sal)
```

<br>

## 3.4 한 테이블에서 다른 테이블에 존재하지 않는 값 검색하기
```sql
select deptno
from dept
where deptno not in (select deptno from emp)
```

<br>

- NOT IN을 사용할 때는 null에 유의하고, 방지하기 위해 **NOT EXISTS**와 함께 서브쿼리를 사용한다.
- 외부쿼리의 행이 서브쿼리에서 참조되므로 **상관 서브쿼리** 라는 용어를 사용한다.
```sql
select d.deptno
from dept d
where not exists (
                   select 1
                   from emp e
                   where d.deptno = e.deptno )
```
