# 레코드 검색

<br>

## 1.1 테이블의 모든 행과 열 검색하기
```sql
select *
from emp
```

<br>

## 1.2 테이블에서 행의 하위 집합 검색하기
- WHERE 절을 사용하면 관심 있는 행만 검색 가능하다.
```sql
select *
from emp
where deptno = 10
```

<br>

## 1.3 여러 조건을 충족하는 행 찾기
- WHERE 절은 다음과 같은 행을 찾는다.
  - DEPTNO가 10이다.
  - COMM이 NULL이 아니다.
  - DEPTNO가 20인 사원 중 급여가 2,000 달러 이하이다.
```sql
select *
from emp
where deptno = 10
	or comm is not null
    or sal <= 2000 and deptno = 20
```

<br>

## 1.4 테이블에서 열의 하위 집합 검색하기
```sql
select ename, deptno, sal
from emp
```

<br> 

## 1.5 열에 의미 있는 이름 지정하기
- 쿼리 결과의 이름을 변경하려면 '원래 이름 AS 새로운 이름' 형식을 사용한다.
```sql
select sal as salary, comm as commission
from emp
```

<br>

## 1.6 WHERE 절에서 별칭이 지정된 열 참조하기
- WHERE 절에서 별칭 이름을 참조하려다 실패한다.
```sql
select sal as salary, comm as comission
from emp
where salary < 5000

-- Error Code: 1054. Unknown column 'salary' in 'where clause'	0.000 sec
```

- 쿼리를 인라인 뷰로 감싸서 별칭이 지정된 열을 참조할 수 있다.
```sql
select *
	from (
            select sal as salary, comm as commission
            from emp ) x
where salary < 5000 
```

<br>

## 1.7 열 값 이어 붙이기
```sql
select concat(ename, ' WORKS AS A ', job) as msg
from emp
where deptno = 10
```

<br>

## 1.8 SELECT 문에서 조건식 사용하기
```sql
select ename, sal,
             case when sal <= 2000 then 'UNDERPAID'
                  when sal >= 4000 then 'OVERPAID'
                  else 'OK'
             end as status
from emp
```

<br>

## 1.9 반환되는 행 수 제한하기 
- DB에서 제공하는 내장 함수를 LIMIT을 사용하여 반환되는 행 수 제어
```sql
select *
from emp limit 5
```

<br>

## 1.10 테이블에서 n개의 무작위 레코드 반환하기
- 내장된 RAND 함수를 LIMIT 및 ORDER BY와 함께 사용
```sql
select ename, job
from emp
order by rand() limit 5
```

<br>

## 1.11 null 값 찾기
- 값이 null인지 확인하려면 IS NULL을 사용한다.
```sql
select *
from emp
where comm is null
```

<br>

## 1.12 null을 실젯값으로 변환하기
- COALESCE 함수 사용하여 null을 실젯값으로 대체한다.
  - COALESCE 함수 : 하나 이상의 값을 인수로 사용한다. 목록에서 첫 번째 null이 아닌 값을 반환한다.
```sql
select coalesce(comm, 0)
from emp
```

<br>

## 1.13 패턴 검색하기
- 부서 10과 20의 사원들 중 이름에  'I'가 있거나 직급명이 'ER'로 끝나는 사원만 반환한다.
```sql
select ename, job
from emp
where deptno in (10, 20)
and (ename LIKE '%I%' or job LIKE '%ER')
```
