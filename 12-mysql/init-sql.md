EMP

```sql
create table emp (
  empno bigint (4) not null,
  ename varchar (10),
  job varchar (9),
  mgr bigint (4),
  hiredate datetime,
  sal decimal (7, 2),
  comm decimal (7, 2),
  deptno int (2)
);
```

```sql
insert into emp values (7369, 'SMITH', 'CLERK', 7902,str_to_date('17-DEC-1980', '%e-%b-%Y'), 800, null, 20);
insert into emp values (7499, 'ALLEN', 'SALESMAN', 7698,str_to_date('20-FEB-1981', '%e-%b-%Y'), 1600, 300, 30);
insert into emp values (7521, 'WARD', 'SALESMAN', 7698,str_to_date('22-FEB-1981', '%e-%b-%Y'), 1250, 500, 30);
insert into emp values (7566, 'JONES', 'MANAGER', 7839,str_to_date('2-APR-1981', '%e-%b-%Y'), 2975, null, 20);
insert into emp values (7654, 'MARTIN', 'SALESMAN', 7698,str_to_date('28-SEP-1981', '%e-%b-%Y'), 1250, 1400, 30);
insert into emp values (7698, 'BLAKE', 'MANAGER', 7839,str_to_date('1-MAY-1981', '%e-%b-%Y'), 2850, null, 30);
insert into emp values (7782, 'CLARK', 'MANAGER', 7839,str_to_date('9-JUN-1981', '%e-%b-%Y'), 2450, null, 10);
insert into emp values (7788, 'SCOTT', 'ANALYST', 7566,str_to_date('09-DEC-1982', '%e-%b-%Y'), 3000, null, 20);
insert into emp values (7839, 'KING', 'PRESIDENT', null,str_to_date('17-NOV-1981', '%e-%b-%Y'), 5000, null, 10);
insert into emp values (7844, 'TURNER', 'SALESMAN', 7698,str_to_date('8-SEP-1981', '%e-%b-%Y'), 1500, 0, 30);
insert into emp values (7876, 'ADAMS', 'CLERK', 7788,str_to_date('12-JAN-1983', '%e-%b-%Y'), 1100, null, 20);
insert into emp values (7900, 'JAMES', 'CLERK', 7698,str_to_date('3-DEC-1981', '%e-%b-%Y'), 950, null, 30);
insert into emp values (7902, 'FORD', 'ANALYST', 7566,str_to_date('3-DEC-1981', '%e-%b-%Y'), 3000, null, 20);
insert into emp values (7934, 'MILLER', 'CLERK', 7782,str_to_date('23-JAN-1982', '%e-%b-%Y'), 1300, null, 10);
```

DEPT

```sql
create table dept
(deptno bigint(2),
dname varchar(14),
loc varchar(13)
);
```

```sql
insert into dept values (10, 'ACCOUNTING', 'NEW YORK');
insert into dept values (20, 'RESEARCH', 'DALLAS');
insert into dept values (30, 'SALES', 'CHICAGO');
insert into dept values (40, 'OPERATIONS', 'BOSTON ');
```

