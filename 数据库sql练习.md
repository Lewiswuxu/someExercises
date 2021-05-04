# 数据表介绍

--1.学生表
 Student(SId,Sname,Sage,Ssex)
 --SId 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别

--2.课程表
 Course(CId,Cname,TId)
 --CId 课程编号,Cname 课程名称,TId 教师编号

--3.教师表
 Teacher(TId,Tname)
 --TId 教师编号,Tname 教师姓名

--4.成绩表
 SC(SId,CId,score)
 --SId 学生编号,CId 课程编号,score 分数

## 学生表 Student

```sql
create table Student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-12-20' , '男');
insert into Student values('04' , '李云' , '1990-12-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-01-01' , '女');
insert into Student values('07' , '郑竹' , '1989-01-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2012-06-06' , '女');
insert into Student values('12' , '赵六' , '2013-06-13' , '女');
insert into Student values('13' , '孙七' , '2014-06-01' , '女');
```

## 科目表 Course

```sql
create table Course(CId varchar(10),Cname nvarchar(10),TId varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
```

## 教师表 Teacher

```sql
create table Teacher(TId varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
```

## 成绩表 SC

```sql
create table SC(SId varchar(10),CId varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

# 练习题目

## #1、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )

```sql
SELECT student.SId,student.Sname,table1.num,table1.total
from student left join
(select SId,count(*) as num,sum(score) as total from sc group by SId)table1
on student.SId = table1.SId;
```



## #2、查有成绩的学生信息

```sql
SELECT * from student where SId IN(SELECT SId from sc);
SELECT * from student WHERE EXISTS(SELECT sc.SId from sc where student.SId = sc.SId);
```



## #3、查询「李」姓老师的数量

```sql
SELECT count(*) from teacher where Tname like "李%";
```

## #4、查询学过「张三」老师授课的同学的信息

```sql
SELECT student.*
from student join sc join teacher join course
on student.SId = sc.SId and sc.CId = course.CId and course.TId = teacher.TId
where teacher.Tname = "张三";
select student.* from student,teacher,course,sc
where 
    student.sid = sc.sid 
    and course.cid=sc.cid 
    and course.tid = teacher.tid 
    and tname = '张三';
```



## #5、查询没有学全所有课程的同学的信息

```sql
SELECT student.* from student where SId not in
(select SId
from sc
group by sc.SId
HAVING count(*) = (SELECT count(*) from course)
);
```



## #6、查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

```sql
SELECT student.* from student where student.SId in(
select sc.SId from sc where sc.CId in
(select Cid from sc where sc.SId = "01"));
```



## #7、查询和" 01 "号的同学学习的课程完全相同的其他同学的信息

```sql
SELECT * from student where SId in(
SELECT sc.SId from sc where sid <> '01'
group by SId HAVING GROUP_CONCAT(CId order by CId ASC) =
(SELECT GROUP_CONCAT(CId ORDER BY CId ASC) from sc where sc.SId = '01'));
SELECT * FROM student WHERE SId in
(
select sid from sc where sid<>'01' group by sid -- 取出不是01学生的cid 并排序 和 取出01学生的cid并排序 进行比较
having group_concat(cid ORDER BY cid) = (select group_concat(cid ORDER BY cid) from SC where sid = '01')
);
```



## #8、查询没学过"张三"老师讲授的任一门课程的学生姓名

```sql
SELECT * from student where SId NOT IN
(SELECT DISTINCT SId from sc where CId IN
(SELECT CId from course where TId IN
(SELECT TId from teacher where Tname = '张三')));
```



## #9、查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```sql
SELECT student.SId,student.Sname,table1.avgscore
from student,
(SELECT SId,AVG(score) as avgscore from sc where sc.score < 60 GROUP BY SId HAVING count(*) >= 2)table1
where student.SId = table1.SId;
```



## #10、检索" 01 "课程分数小于 60，按分数降序排列的学生信息

```sql
SELECT student.*,sc.score
from student, sc 
where student.SId = sc.SId
and sc.CId='01'
and sc.score < 60
order by sc.score;
```



## #11、按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

```sql
SELECT student.*,another.score,(SELECT AVG(score) as avgscore from sc where sc.SId = another.SId)
from student,sc as another
where student.SId = another.SId
ORDER BY avgscore DESC;

SELECT * from sc,
(SELECT sc.SId,AVG(score) as avgscore from sc GROUP BY SId)table1
where sc.SId = table1.SId
ORDER BY table1.avgscore desc;
```



## #12、查询各科成绩最高分、最低分和平均分：

#以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
#及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
#要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```sql
SELECT 
CId,
MAX(score)as 最高分,
MIN(score)as 最低分,
AVG(score)as 平均分,
COUNT(*)as 选修人数,
SUM(case when score>=60 then 1 else 0 end)/count(*) as 及格率,
SUM(case when score>=70 and score<80 then 1 else 0 end)/count(*) as 中等率,
SUM(case when score>=80 and score<90 then 1 else 0 end)/count(*) as 优良率,
SUM(case when score>=90 then 1 else 0 end)/count(*) as 优秀率
from sc
GROUP BY CId
ORDER BY count(*) desc,CId asc;
select 
sc.CId ,
max(sc.score)as 最高分,
min(sc.score)as 最低分,
AVG(sc.score)as 平均分,
count(*)as 选修人数,
sum(case when sc.score>=60 then 1 else 0 end )/count(*)as 及格率,
sum(case when sc.score>=70 and sc.score<80 then 1 else 0 end )/count(*)as 中等率,
sum(case when sc.score>=80 and sc.score<90 then 1 else 0 end )/count(*)as 优良率,
sum(case when sc.score>=90 then 1 else 0 end )/count(*)as 优秀率 
from sc
GROUP BY sc.CId
ORDER BY count(*)DESC, sc.CId ASC;
```



## #13、按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺

```sql
select a.CId,a.SId,a.score,count(b.score)+1 as rank
from sc as a
LEFT JOIN sc as b
on a.cid = b.cid and a.score<b.score
GROUP BY a.CId,a.SId,a.score
ORDER BY a.cid asc,rank asc;
```



## #14、统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比

```sql
SELECT course.CId,course.Cname,
sum(case when score >= 85 then 1 else 0 end)/count(*) as "[100-85]",
sum(case when score >= 70 and score < 85 then 1 else 0 end)/count(*) as "[85-70]",
sum(case when score >= 60 and score < 70 then 1 else 0 end)/count(*) as "[70-60]",
sum(case when score <60 then 1 else 0 end)/count(*) as "[60-0]"
from sc,course
where sc.CId = course.CId
group by sc.CId;

select course.cname, course.cid,
sum(case when sc.score<=100 and sc.score>85 then 1 else 0 end)/count(*)  as "[100-85]",
sum(case when sc.score<=85 and sc.score>70 then 1 else 0 end)/count(*)  as "[85-70]",
sum(case when sc.score<=70 and sc.score>60 then 1 else 0 end)/count(*)  as "[70-60]",
sum(case when sc.score<=60 and sc.score>0 then 1 else 0 end)/count(*)  as "[60-0]"
from sc left join course
on sc.cid = course.cid
group by sc.cid;
```



## #15、查询各科成绩前三名的记录

```sql
select a.CId,a.SId,a.score,count(b.score)+1 as rank
from sc as a
LEFT JOIN sc as b
on a.cid = b.cid and a.score<b.score
GROUP BY a.CId,a.SId,a.score
HAVING count(b.score)<3
ORDER BY a.cid asc,rank asc;

select * from sc
where (
select count(*) from sc as a 
where sc.cid = a.cid and sc.score<a.score 
)< 3
order by cid asc, sc.score desc;
```



## #16、查询每门课程被选修的学生数

```sql
SELECT CId,count(*)
from sc
group by CId;
```

