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

<br>

## 3.5 다른 테이블 행과 일치하지 않는 행 검색하기
- 사원이 없는 부서를 찾고자 한다.
- null에 대한 외부조인 (outer join) 및 필터를 사용한다.
```sql
select d.*
from dept d left outer join emp e
on (d.deptno = e.deptno)
where e.deptno is null
```

<br>

## 3.6 다른 조인을 방해하지 않고 쿼리에 조인 추가하기
- 모든 사원명(ENAME), 근무 부서의 위치(LOC) 및 보너스 받은 날짜(RECEIVED)를 반환한다.
- 사원별 보너스 지급 날짜를 추가하려 EMP_BONUS 테이블과 조인하면 모든 사원이 보너스를 받는 것은 아니므로 적은 행을 반환한다.
```sql
select e.ename, d.loc, eb.received
from emp e, dept d, emp_bouns eb
where e.deptno = d.deptno and e.empno = eb.empno
```

<br>
- 외부 조인을 사용하여 원래 쿼리의 데이터 손실 없이 추가 정보를 얻는다.

```sql
select e.ename, d.loc, eb.received
from emp e join dept d on (e.deptno = d.deptno)
left join emp_bouns eb on (e.empno = eb.empno)
order by 2
```

```sql
select e.ename, d.loc, (select eb.received from emp_bouns eb where eb.empno = e.empno) as received
from emp e, dept d
where e.deptno = d.deptno
order by 2
```

<br>

## 3.7 두 테이블에 같은 데이터가 있는지 확인하기 
```sql
create view V
as
select * from emp where deptno != 10
union all
select * from emp where ename = 'WARD';

select * from V
```

<br>

- 상관 서브쿼리 및 UNION ALL을 사용하여 뷰 V가 아닌 EMP 테이블의 행과 EMP 테이블이 아닌 뷰 V의 행을 찾는다.
```sql
select *
from (
       select e.empno, e.ename, e.job, e.mgr, e.hiredate, e.sal, e.comm, e.deptno, count(*) as cnt
       from emp e
       group by empno, ename, job, mgr, hiredate, sal, comm, deptno ) e
where not exists (
                   select null
                   from (
                           select v.empno, v.ename, v.job, v.mgr, v.hiredate, v.sal, v.comm, v.deptno, count(*) as cnt
                           from v
                           group by empno, ename, job, mgr, hiredate, sal, comm, deptno ) v
                   where v.empno = e.empno
                     and v.ename = e.ename
                     and v.job = e.job
                     and coalesce(v.mgr, 0) = coalesce(e.mgr, 0)
                     and v.hiredate = e.hiredate
                     and v.sal = e.sal
                     and v.deptno = e.deptno
                     and v.cnt = e.cnt
                     and coalesce(v.comm, 0) = coalesce(e.comm, 0) )
union all

select *
from (
       select v.empno, v.ename, v.job, v.mgr, v.hiredate, v.sal, v.comm, v.deptno, count(*) as cnt
       from v
       group by empno, ename, job, mgr, hiredate, sal, comm, deptno ) v
where not exists (
                    select null
                    from (
                            select e.empno, e.ename, e.job, e.mgr, e.hiredate, e.sal, e.comm, e.deptno, count(*) as cnt
                            from emp e
                            group by empno, ename, job, mgr, hiredate, sal, comm, deptno ) e
where v.empno = e.empno
  and v.ename = e.ename
  and v.job = e.job
  and coalesce(v.mgr, 0) = coalesce(e.mgr, 0)
  and v.hiredate = e.hiredate
  and v.sal = e.sal
  and v.deptno = e.deptno
  and v.cnt = e.cnt
  and coalesce(v.comm, 0) = coalesce(e.comm, 0) )
```

<br>

## 3.8 데카르트 곱 식별 및 방지하기
- 부서 위치와 함께 부서 10의 각 사원명을 반환한다.
- 데카르트 곱을 피하려면 일반적으로 **n-1 규칙**을 적용한다.
- n은 FROM 절의 테이블 수를 나타내고, n-1은 데카르트 곱을 피하는 데 필요한 최소 조인 수이다.

```sql
select e.ename, d.loc
from emp e, dept d
where e.deptno = 10 and d.deptno = e.deptno
```

<br>

## 3.9 집계를 사용할 때 조인 수행하기
- 부서 10에 해당하는 사원의 급여 합계와 보너스 합계를 찾는다.
- 유형 1 보너스 : 급여의 10% / 유형 2 보너스 : 급여의 20% / 유형 3 보너스 : 급여의 30%
```sql
select e.empno, e.ename, e.sal, e.deptno, e.sal * case when eb.type = 1 then .1
                                                       when eb.type = 2 then .2
                                                       else .3
                                                  end as bonus
from emp e, emp_bonus eb
where e.empno = eb.empno and e.deptno = 10
```

<br>
- DISTINCT 급여의 합계만 수행한다.

```sql
select deptno, sum(distinct sal) as total_sal, sum(bonus) as total_bonus
from (
       select e.empno, e.ename, e.sal, e.deptno, e.sal * case when eb.type = 1 then .1
                                                              when eb.type = 2 then .2
                                                              else .3
                                                         end as bonus
       from emp e, emp_bonus eb
       where e.empno = eb.empno and e.deptno = 10 ) x
group by deptno
```

<br>
- 합산하는 열에 중복되는 값이 있을 때 필요한 대체 해법

```sql
select d.deptno, d.total_sal, sum(e.sal * case when eb.type = 1 then .1
                                               when eb.type = 2 then .2
                                               else .3
                                          end as bonus
from emp e, emp_bonus eb, (
                            select deptno, sum(sal) as total_sal
                            from emp
                            where deptno = 10
                            group by deptno ) d
where e.deptno = d.deptno and e.empno = eb.empno
group by d.deptno, d.total_sal
```

<br>

## 3.10 집계 시 외부 조인 수행하기
- EMP_BONUS 테이블과 부서 10의 모든 급여 합계와, 부서 10의 모든 사원에 대한 모든 보너스 합계를 찾는다.
- 3.9와 유사하지만, 부서 10의 모든 사원이 포함되도록 **EMP_BONUS**에 외부 조인하는 것이 차이점이다.

```sql
select deptno, sum(distinct sal) as total_sal, sum(bonus) as total_bonus
from (
       select e.empno, e.ename, e.sal, e.deptno, e.sal * case when eb.type is null then 0
                                                              when eb.type = 1 then .1
                                                              when eb.type = 2 then .2
                                                              else .3 end as bonus
       from emp e left outer join emp_bonus eb on (e.empno = eb.empno)
       where e.deptno = 10 )
group by deptno
```

<br>

- 부서 10의 모든 급여 합계가 먼저 계산된 다음 EMP 테이블에 조인하고, EMP_BONUS 테이블에 조인된다. (외부 조인을 피함)

```sql
select d.deptno, d.total_sal, sum(e.sal * case when eb.type = 1 then .1
                                               when eb.type = 2 then .2
                                               else .3 end) as total_bonus
from emp e, emp_bonus eb, (
                            select deptno, sum(sal) as total_sal
                            from emp
                            where deptno = 10
                            group by deptno ) d
where e.deptno = d.deptno and e.empno = eb.empno
group by d.deptno, d.total_sal
```

<br>

## 3.11 여러 테이블에서 누락된 데이터 변환하기
- DEPT의 모든 DEPTNO 및 DNAME을 각 부서의 (사원이 있는 경우) 모든 사원명과 함께 반환하는 쿼리를 찾아본다.

```sql
select d.deptno, d.dname, e.ename
from dept d right outer join emp e on (d.deptno = e.deptno)
union
select d.deptno, d.dname, e.ename
from dept d left outer join emp e on (d.deptno = e.deptno)
```

<br>

## 3.12 연산 및 비교에서 null 사용하기
- 커미션(COMM)이 사원 워드(WARD)의 커미션보다 적은 모든 사원을 EMP 테이블에서 찾으려고 한다. 이때 커미션이 null인 사원도 포함해야 한다.
- COALESCE와 같은 함수를 사용하여 null 값을 표준 평가에서 사용할 수 있는 실젯값으로 변환한다.

```sql
select ename, comm
from emp
where coalesce(comm, 0) < ( select comm
                            from emp
                            where ename = 'WARD' )
```
