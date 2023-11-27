# 삽입, 갱신, 삭제

<br>

- 데이터베이스에 새로운 레코드 삽입
- 기존 레코드 갱신
- 필요 없는 레코드 삭제

<br>

## 4.1 새로운 레코드 삽입하기
- DEPT 테이블에 새 레코드를 삽입한다. DEPTNO 값 50, DNAME 'PROGRAMMING', LOC 'BALTIMORE'

```sql
insert into dept (deptno, dname, loc) values (50, 'PROGRAMMING', 'BALTIMORE')
```

```sql
insert into dept (deptno, dname, loc) values (1, 'A', 'B'), (2, 'B', 'C')
```

```sql
insert into dept values (50, 'PROGRAMMING', 'BALTIMORE')
```

<br>

## 4.2 기본값 삽입하기
- 값을 지정하지 않고 기본값 행을 삽입한다.

```sql
create table D (id integer default 0)
```

<br>

- 테이블의 모든 열에 값을 삽입하지 않을 때는 수행해야 하는 열 이름을 명시적으로 지정한다.

```sql
insert into D (id) values (default)
```

<br>

## 4.3 null로 기본값 오버라이딩하기
- 기본값을 사용하는 열에 값을 삽입할 때, 열을 null로 설정하여 기본값을 오버라이딩한다.

```sql
create table D (id integer default 0, foo VARCHAR(10))
```

<br>

- 값 목록에서 명시적으로 null을 지정한다.

```sql
insert into d (id, foo) values (null, 'Brighten')
```

<br>

## 4.4 한 테이블에서 다른 테이블로 행 복사하기
- DEPT 테이블에서 DEPT_EAST 테이블로 행을 복사한다.
- INSERT 문에 원하는 행을 반환하는 쿼리와 함께 이어 쓴다.

```sql
insert into dept_east (deptno, dname, loc)
select deptno, dname, loc
from dept
where loc in ('NEW YORK', 'BOSTON')
```

<br>

## 4.5 테이블 정의 복사하기
- DEPT 테이블의 사본을 생성하고 DEPT_2 라고 부르려고 한다.
- 행은 복사하지 않고 테이블의 열 구조만 복사한다.

```sql
create table dept_2
as
select *
from dept
where 1 = 0
```

<br>

- CTAS (Create Table As Select) 문을 사용하는 경우
  - WHERE 절에 거짓 조건 지정 X : 쿼리의 모든 행으로 생성 중인 새 테이블을 채운다.
  - WHERE 절에 거짓 조건 지정 O : 행이 반환되지 않고 쿼리의 SELECT 절에 있는 열에 기반을 두는 빈 테이블이다.
 
<br>

## 4.6 특정 열에 대한 삽입 차단하기
- 프로그램이 EMP 테이블에 값을 삽입하도록 허용하되 EMPNO, ENAME 및 JOB 열에만 삽입하도록 한다.

```sql
create view new_emps as
select empno, ename, job
from emp
```
- 테이블에 표시할 열만 노출되는 뷰를 만든다.
- 모든 삽입 내용이 해당 뷰를 통과하도록 한다.

<br>

```sql
insert into new_emps (empno, ename, job) values (1, 'Jonathan', 'Editor')
```
```sql
insert into emp (empno, ename, job) values (1, 'Jonathan', 'Editor')
```
- 단순 뷰에 삽입하면 DB 서버는 삽입 내용을 기본 테이블로 변환한다.

<br>

## 4.8 테이블에서 레코드 수정하기
- 부서 20에 속한 모든 사원의 급여를 10% 인상한다.

```sql
update emp set sal = sal * 1.10
where deptno = 20
```

<br>

- 대량의 업데이트를 준비할 땐 SET 절에 들어갈 식이 포함된 SELECT문을 실행한다.
```sql
select deptno,
       ename,
       sal        as orig_sal,
       sal * .10  as amt_to_add,
       sal * 1.10 as new_sal
from emp
where deptno = 20
order by 1, 5
```

<br>

## 4.9 일치하는 행이 있을 때 업데이트하기
- EMP_BONUS 테이블에 사원 정보가 있다면, 해당 사원의 급여를 20% 인상한다.

```sql
update emp
   set sal = sal * 1.20
where empno in (select from emp_bonus)
```
```sql
update emp
   set sal = sal * 1.20
where exists (select null
              from emp_bonus
              where emp.empno = emp_bonus.empno)
```
<br>

## 4.10 다른 테이블 값으로 업데이트하기
- 특정 사원의 새로운 급여가 저장된 NEW_SAL 테이블이 있다.
- EMP.SAL을 NEW_SAL.SAL로 업데이트한 뒤, EMP.COMM을 NEW_SAL.SAL의 50%로 업데이트한다.

```sql
update emp e, new_sal ns
   set e.sal = ns.sal,
       e.comm = ns.sal / 2
where e.deptno = ns.deptno
```

<br>

## 4.11 레코드 병합하기
- EMP_COMMISSION 테이블을 다음과 같이 수정하려고 한다.
  - EMP_COMMISSION의 사원이 EMP 테이블에 있을 때, 해당 사원의 커미션(COMM)을 1,000으로 업데이트
  - COMM을 1,000으로 업데이트할 가능성이 있는 모든 사원에 대해, SAL이 2,000 미만이면 해당 사원을 삭제
  - 그렇지 않으면 EMP 테이블의 EMPNO, ENAME, DEPTNO 값을 EMP_COMMISSION 테이블에 삽입
 
```sql
merge into emp_commission ec
using (select * from emp) emp on (ec.empno = emp.empno)
when matched then
     update set ec.comm = 1000
     delete where (sal < 2000)
when not matched then
     insert (ec.empno, ec.ename, ec.deptno, ec.comm)
     values (emp.empno, emp.ename, emp.deptno, emp.comm)
```

<br>

## 4.12 테이블에서 모든 레코드 삭제하기
- EMP에서 모든 레코드를 삭제하려고 한다.

```sql
delete from emp
```

<br>

## 4.13 특정 레코드 삭제하기
- 삭제할 행을 지정하는 WHERE절과 함께 DELETE 명령을 사용한다.

```sql
delete from emp where deptno = 10
```

<br>

## 4.14 단일 레코드 삭제하기
- 삭제하려는 레코드 하나만 지정할 수 있도록 한다.

```sql
delete from emp where empno = 7782
```

<br>

## 4.15 참조 무결성 위반 삭제하기
- 레코드가 다른 테이블에 존재하지 않는 레코드를 참조할 때, 테이블에서 해당 레코드를 삭제하려고 한다.
- 일부 사원이 현재 존재하지 않는 부서에 할당되었을 때, 해당 사원을 삭제하려고 한다.

```sql
delete from emp
where not exists (
                   select * from dept
                   where dept.deptno = emp.deptno )
```
- 상관 서브쿼리를 사용하여 지정된 EMP 레코드와 DEPTNO가 일치하는 레코드가 DEPT에 있는지 검증한다.
- 이러한 레코드가 있으면 EMP 레코드가 유지되며, 그렇지 않으면 삭제된다.

<br>

```sql
delete from emp
where deptno not in (select deptno from dept)
```
- 서브쿼리를 사용하여 유효한 부서 번호 목록을 검색한다.
- 각 EMP 레코드의 DEPTNO를 해당 목록과 비교한다.
- 목록에 없는 DEPTNO를 쓰는 EMP 레코드가 발견되면 해당 EMP 레코드가 삭제된다.

<br>

## 4.16 중복 레코드 삭제하기
```sql
create table dupes (id integer, name varchar(10))

insert into dupes values (1, 'NAPOLEON')
insert into dupes values (2, 'DYNAMITE')
insert into dupes values (3, 'DYNAMITE')
insert into dupes values (4, 'SHE SELLS')
insert into dupes values (5, 'SEA SHELLS')
insert into dupes values (6, 'SEA SHELLS')
insert into dupes values (7, 'SEA SHELLS')
```

<br>

- 'SEA SHELLS'와 같은 각 중복 이름 그룹에 대해 하나의 ID만 유지하고 나머지는 삭제하려고 한다.
```sql
delete from dupes
where id not in ( select min(id)
                  from (select id, name from dupes) tmp
                  group by name )
```
- 중복되는 값을 기준으로 그룹화한 다음, 집계 함수를 사용하여 보존할 킷값 하나만 선택한다.

<br>

## 4.17 다른 테이블에서 참조된 레코드 삭제하기
- DEPT_ACCIDENT 테이블은 제조업체에서 발생하는 각 사고에 대해 하나의 행을 포함한다.
- 각 행은 사고가 발생한 부서와 사고 유형을 기록한다.
```sql
create table dept_accidents
(deptno        integer,
 accident_name varchar(20))

insert into dept_accidents values (10, 'BROKEN FOOT')
insert into dept_accidents values (10, 'FLESH WOUND')
insert into dept_accidents values (20, 'FIRE')
insert into dept_accidents values (20, 'FIRE')
insert into dept_accidents values (20, 'FLOOD')
insert into dept_accidents values (30, 'BRUISED GLUTE')
```

<br>

- 사고가 세 번 이상 발생한 부서에서 근무하는 사원 기록을 EMP 테이블에서 삭제한다.

```sql
delete from emp
where deptno in ( select deptno
                  from dept_accidents
                  group by deptno
                  having count(*) >= 3 )
```
