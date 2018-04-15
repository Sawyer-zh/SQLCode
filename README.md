# SQL语句训练

## 1、查询入职时间最晚的员工

[题目链接](https://www.nowcoder.com/practice/218ae58dfdcd4af195fff264e062138f?tpId=82&tqId=29753&tPage=1&rp=&ru=/ta/sql&qru=/ta/sql/question-ranking)

### 方法1

先查出最晚的时间是什么，然后再去查入职时间和它相等的那些员工：

```sql
select * from employees where hire_date = (select max(hire_date) from employees)
```

### 方法2

对hire_date字段排序降序，此时最晚的时间排在第一个，再用LIMIT取出：

```mysql
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 0,1
```

但是这种方法有局限，因为最晚入职的人可能有多个人，而这种方法只能取出一个人。

改进：

```mysql
select * from employees where hire_date = (SELECT hire_date FROM employees ORDER BY hire_date DESC LIMIT 0,1)
```

也就是先把那个时间查出来，然后再查满足条件的所有人。

## 2、查找入职员工时间排名倒数第三的员工所有信息

[题目链接](https://www.nowcoder.com/practice/ec1ca44c62c14ceb990c3c40def1ec6c?tpId=82&tqId=29754&rp=0&ru=/ta/sql&qru=/ta/sql/question-ranking)

通过order by 和 limit语句找出排名倒数第三的员工。

```mysql
select * from employees where hire_date = (select hire_date from employees order by hire_date desc limit 2, 1)
```

## 3、查找所有已经分配部门的员工的last_name和first_name

[题目链接](https://www.nowcoder.com/practice/6d35b1cd593545ab985a68cd86f28671?tpId=82&tqId=29756&rp=0&ru=/ta/sql&qru=/ta/sql/question-ranking)

```mysql
select last_name, first_name, dept_no from employees inner join dept_emp on employees.emp_no = dept_emp.emp_no
```

## 4、查找所有员工入职时候的薪水情况

[题目链接](https://www.nowcoder.com/practice/23142e7a23e4480781a3b978b5e0f33a?tpId=82&tqId=29758&rp=0&ru=/ta/sql&qru=/ta/sql/question-ranking)

```mysql
select employees.emp_no, salaries.salary from employees inner join salaries 
on employees.emp_no = salaries.emp_no 
and employees.hire_date = salaries.from_date 
order by employees.emp_no desc
```

## 5、查找薪水涨幅超过15次的员工号emp_no以及其对应的涨幅次数t

[题目](https://www.nowcoder.com/practice/6d4a4cff1d58495182f536c548fee1ae?tpId=82&tqId=29759&tPage=1&rp=&ru=/ta/sql&qru=/ta/sql/question-ranking)

```sql
select emp_no, count(emp_no) as t from salaries group by emp_no having t > 15
```

注意，`count(emp_no)`不要写成了`sum(emp_no)`，因为`sum(emp_no)`是把emp_no的值加起来，而不是统计有多少个emp_no。

注意，记得聚集函数和group by要一起用。因为分组后，每一组只能显示一条记录，所以需要和聚集函数配合使用。

## 6、找出所有员工当前薪水salary情况

找出所有员工当前(to_date='9999-01-01')具体的薪水salary情况，对于相同的薪水只显示一次,并按照逆序显示：

```mysql
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

方法一：

```mysql
select distinct salary from salaries where to_date = "9999-01-01" order by salary desc
```

方法二：

```mysql
select salary from salaries where to_date = '9999-01-01' group by salary order by salary desc
```

一般使用`group by`代替`distinct`，因为`group by`的性能更高。

## 7、获取所有部门当前manager的当前薪水情况

[题目](https://www.nowcoder.com/practice/4c8b4a10ca5b44189e411107e1d8bec1?tpId=82&tqId=29761&rp=0&ru=/ta/sql&qru=/ta/sql/question-ranking)

获取所有部门当前manager的当前薪水情况，给出dept_no, emp_no以及salary，当前表示to_date='9999-01-01'：

```mysql
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

```mysql
select dept_manager.dept_no, dept_manager.emp_no, salaries.salary from salaries join dept_manager on dept_manager.emp_no = salaries.emp_no where dept_manager.to_date = "9999-01-01" and salaries.to_date = "9999-01-01"
```

注意，dept_manager.to_date和salaries.to_date都需要，因为一个manage可能对应了多个to_date

## 8、获取所有非manager的员工emp_no

[题目](https://www.nowcoder.com/practice/32c53d06443346f4a2f2ca733c19660c?tpId=82&tqId=29762&rp=0&ru=/ta/sql&qru=/ta/sql/question-ranking)

```mysql
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

```mysql
select employees.emp_no from employees left join dept_manager on employees.emp_no = dept_manager.emp_no where dept_no is null
```

尽量使用连接查询而不用子查询。

## 9、获取所有员工当前的manager

[题目](https://www.nowcoder.com/practice/e50d92b8673a440ebdf3a517b5b37d62?tpId=82&tqId=29763&rp=0&ru=%2Fta%2Fsql&qru=%2Fta%2Fsql%2Fquestion-ranking&tPage=1)

获取所有员工当前的manager，如果当前的manager是自己的话结果不显示，当前表示to_date='9999-01-01'。
结果第一列给出当前员工的emp_no,第二列给出其manager对应的manager_no。

```mysql
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `dept_manager` (
`dept_no` char(4) NOT NULL,
`emp_no` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
```

```mysql
select dept_emp.emp_no, dept_manager.emp_no as manager_no 
from dept_emp join dept_manager 
on dept_emp.dept_no = dept_manager.dept_no 
where dept_emp.emp_no <> dept_manager.emp_no 
and dept_emp.to_date = '9999-01-01' 
and dept_manager.to_date = '9999-01-01'
```

## 10、获取所有部门中当前员工薪水最高的相关信息

获取所有部门中当前员工薪水最高的相关信息，给出dept_no, emp_no以及其对应的salary

```mysql
CREATE TABLE `dept_emp` (
`emp_no` int(11) NOT NULL,
`dept_no` char(4) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`dept_no`));
CREATE TABLE `salaries` (
`emp_no` int(11) NOT NULL,
`salary` int(11) NOT NULL,
`from_date` date NOT NULL,
`to_date` date NOT NULL,
PRIMARY KEY (`emp_no`,`from_date`));
```

```mysql
select d.dept_no, s.emp_no, max(s.salary) as salary
from dept_emp as d inner join salaries as s
on d.emp_no = s.emp_no
where d.to_date = '9999-01-01'
and s.to_date = '9999-01-01'
group by d.dept_no
```

## 11、查找employees表所有emp_no为奇数

[题目](https://www.nowcoder.com/practice/a32669eb1d1740e785f105fa22741d5c?tpId=82&tqId=29767&rp=0&ru=/ta/sql&qru=/ta/sql/question-ranking)

查找employees表所有emp_no为奇数，且last_name不为Mary的员工信息，并按照hire_date逆序排列：

```mysql
CREATE TABLE `employees` (
`emp_no` int(11) NOT NULL,
`birth_date` date NOT NULL,
`first_name` varchar(14) NOT NULL,
`last_name` varchar(16) NOT NULL,
`gender` char(1) NOT NULL,
`hire_date` date NOT NULL,
PRIMARY KEY (`emp_no`));
```

```mysql
select emp_no, birth_date, first_name, last_name, gender, hire_date 
from employees as e 
where e.emp_no % 2 != 0
and last_name <> "Mary"
order by hire_date desc
```

**在where中可以使用取模运算符**。



















































