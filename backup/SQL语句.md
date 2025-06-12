# 简介
将DB学习过程中遇到的不明白的SQL语句记录在此。

## Constraint
创建一个带有约束的、用户自定义的数据类型，目的是为了代码复用，不需要在多个表的同一类型添加重复的限制。
```sql
create domain WAMvalue float
check (value between 0.0 and 100.0);

CREATE DOMAIN WAMvalue AS double precision
CHECK (VALUE >= 0.0 AND VALUE <= 100.0);
``` 

## Data Modification
```sql
insert into Enrollments(student, course, mark)
vales(3312345, 5542, 75);
INSERT 0 1
-- 1代表修改了一个touple ，0代表没有为该touple生成内部的对象标识符

update Enrollments set mark = 77
where student = 3312345 AND course = 5542;

delete Enrollments
where student = 3312345;
``` 