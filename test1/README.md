# oracle作业一

##### 姓名：陈奕妍       										学号：201810414101 										班级：软工一班 

## 实验目的



### 1、由pic1和pic2对比可以发现，pic1比pic2更优。

##### 1）查询1执行计划如下图：

其中Cost=3，Rows=20，Predicate Information中有两次索引搜索access，无全表搜素filter。

![pic1](.\pic1.png)



```sql
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
from hr.departments d,hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT','Sales')
GROUP BY d.department_name;
```

##### 2）查询2执行计划如下图：

其中Cost=6，Rows=106，Predicate Information中有两次索引搜索access，一次全表搜素filter。

![pic2](.\pic2.png)

```sql
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
FROM hr.departments d,hr.employees e
WHERE d.department_id = e.department_id
GROUP BY d.department_name
HAVING d.department_name in ('IT','Sales');
```

总的来说，全表搜索越少越好，最好采取索引搜索的方式。查询一，先过滤后汇总，查询二则相反，先汇总后过滤，参与汇总和计算数据量比较大。



### 2、执行上面两个比较复杂的返回相同查询结果数据集的SQL语句，通过分析SQL语句各自的执行计划，判断哪个SQL语句是最优的。最后将你认为最优的SQL语句通过优化指导工具进行优化指导，看看该工具有没有给出优化建议

通过工具可以看到，仍然有可以优化的部分，如下图：

![pic3](.\pic3.png)

优化建议：

```
1- Index Finding (see explain plans section below)
--------------------------------------------------
  通过创建一个或多个索引可以改进此语句的执行计划。

  Recommendation (estimated benefit: 50.2%)
  -----------------------------------------
  - 考虑运行可以改进物理方案设计的访问指导或者创建推荐的索引。
    create index HR.IDX$$_026E0001 on HR.DEPARTMENTS("DEPARTMENT_NAME","DEPARTM
    ENT_ID");

  Rationale
  ---------
    创建推荐的索引可以显著地改进此语句的执行计划。但是, 使用典型的 SQL 工作量运行 "访问指导"
    可能比单个语句更可取。通过这种方法可以获得全面的索引建议案, 包括计算索引维护的开销和附加的空间消耗。

```

说明：这个建议是让我们在department表上创建一个基于"DEPARTMENT_NAME","DEPARTMENT_ID"字段的索引，这样可以加快查询DEPARTMENT_NAME的速度。

优化之后的效果，如下图：

![pic4](.\pic4.png)

















