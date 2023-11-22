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
