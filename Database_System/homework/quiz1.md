# 数据库系统课堂测验（2024年4月24日）

学号：

姓名：

1. 用SQL查询姓王的学生，并按照所在院系和学号的字典序排序。（10分）

   ```sql
   SQL
   select * from Student where Sname like '王%' order by Sdept,Sno;
   ```

2. 用关系代数和SQL查询选修了'1002'号课程的学生的学号和姓名。（20分）

   ```sql
   RA
   Π Sno,Sname (σ Cno = '1001'(Student ⨝ SC))
   SQL
   select Sno,Sname from (Student natural join SC) where Cno = '1002';
   ```

3. 用关系代数和SQL查询选修了'1002'号课程但没选修'2003'号课程的学生的学号和姓名。（20分）

   ```sql
   RA
   Π Sno,Sname(σ Cno = '1002'(Student ⨝ SC)) - Π Sno Sname(σ Cno = '2003'(Student ⨝ SC))
   SQL
   select Sno,Sname from (Student natural join SC) where Cno = '1002' except (
   select Sno,Sname from (student natural join SC) where Cno = '2003');
   ```

4. 用关系代数和SQL查询所有学生选课的数量，列出学号、姓名和选课数。（20分）

   ```sql
   RA
   γ Sno,Sname;Count(SC.Cno)->Cnocount (Student ⨝ SC)
   SQL
   select Sno,Sname,Count(SC.Cno) from (Student join SC on Sno,Sname) group by Sno,Sname;
   ```

5. 用SQL查询选修了2门以上（含2门）课程且平均分不低于80的学生的学号、姓名。（10分）

   ```sql
   SQL
   select Student.Sno,Student.Sname,Count(SC.Cno),AVG(SC.Grade) from (Student join SC on Sno,Sname) group by Student.Sno,Student.Sname having AVG(SC.Grade) >= 80 and Count(SC.Cno) >= 2;
   ```

6. `SELECT * FROM Student AS S1 NATURAL JOIN Student AS S2`的结果是什么？（10分）

   ```
   Student原表，因为是自然内连接
   ```

7. `SELECT Sno FROM Student WHERE NOT EXISTS (SELECT * FROM SC WHERE SC.Sno = Student.Sno)`的结果是什么？（10分）

   ```
   查询没选课的学生
   ```
   