# 部门工资前三高的员工

```mysql
SELECT
    d.Name AS 'Department', e1.Name AS 'Employee', e1.Salary
FROM
    Employee e1
        JOIN
    Department d ON e1.DepartmentId = d.Id
WHERE
    3 > (SELECT
            COUNT(DISTINCT e2.Salary)
        FROM
            Employee e2
        WHERE
            e2.Salary > e1.Salary
                AND e1.DepartmentId = e2.DepartmentId
        )
;

```

------



写个的sql查询语句，如有一张表示英语口语练习每个学员的学时的表a，字段有studentid(学号) name(可重复) grade(年级) hours（学时），找出那些学时高于他们同一年级的平均学时的学生。

```sql
select 
	id,name 
from 
	table  a 
		left  join  
	(SELECT 
     	grade,AVG(hours) 
     as 
     	hours 
     FROM table GROUP BY grade) 
as 
	b on a.grade=b.grade 
where   
	a.hours>b.hours AND a.grade=b.grade
```

