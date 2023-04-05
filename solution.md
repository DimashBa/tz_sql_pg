# Техзадание

## Имеются таблицы со следующей структурой
```sql
CREATE TABLE departments  (
	department_id serial PRIMARY KEY,
	departments_name VARCHAR ( 150 ) UNIQUE NOT NULL
);


CREATE TABLE employees (
	employee_id serial PRIMARY KEY,
	first_name VARCHAR ( 150 ),
	last_name VARCHAR ( 150 ),
	birth_date DATE
);


CREATE TABLE positions  (
	position_id serial PRIMARY KEY,
	position_title VARCHAR ( 150 ) UNIQUE NOT NULL
);


CREATE TABLE job_history  (
	job_id serial PRIMARY KEY,
	employee_id INTEGER references employees (employee_id),	
	department_id INTEGER references departments (department_id),	
	position_id INTEGER references positions (position_id),
	amount INTEGER,
	hire_date DATE
);
```
## Данные таблиц
```sql
insert into public.departments (departments_name) values('Снабжение');
insert into public.departments (departments_name) values('Разработка');
insert into public.departments (departments_name) values('Логистика');

insert into public.employees (first_name, last_name, birth_date) values('Давид', 'Манукян', '1980-02-01');
insert into public.employees (first_name, last_name, birth_date) values('Давид', 'Микеланджело', '1981-02-01');
insert into public.employees (first_name, last_name, birth_date) values('Богдан', 'Шинкарев', '1980-02-14');
insert into public.employees (first_name, last_name, birth_date) values('Николай', 'Рихтер', '1985-12-01');

insert into public.positions (position_title) values ('Менеджер');
insert into public.positions (position_title) values ('Дизайнер');

INSERT INTO public.job_history (employee_id, department_id, position_id, amount, hire_date) VALUES(1, 1, 1, 60000, '2020-01-01');
INSERT INTO public.job_history (employee_id, department_id, position_id, amount, hire_date) VALUES(2, 2, 2, 110000, '2019-06-01');
INSERT INTO public.job_history (employee_id, department_id, position_id, amount, hire_date) VALUES(3, 3, 1, 90000, '2005-05-01');
INSERT INTO public.job_history (employee_id, department_id, position_id, amount, hire_date) VALUES(4, 1, 1, 75000, '2021-08-01');
```

## Задачи и решения
### 1)	Сделать выборку всех работников с именем “Давид” из отдела “Снабжение” с полями ФИО, заработная плата, должность

### Решение:
```sql
select e.first_name, e.last_name, p.position_title, jh.amount from job_history jh
inner join employees e on jh.employee_id = e.employee_id  
inner join positions p on jh.position_id = p.position_id
where e.first_name  = 'Давид'
```

### Результат (фамилию и имя можно объединить):

|first_name|last_name|position_title|amount|
|----------|---------|--------------|------|
|Давид|Манукян|Менеджер|60000|
|Давид|Микеланджело|Дизайнер|110000|


### 2)	Посчитать среднюю заработную плату работников по отделам

### Решение:
```sql
select p.position_title, avg(jh.amount) from job_history jh
inner join positions p on jh.position_id = p.position_id
group by p.position_title;
```

### Результат:

|position_title|avg|
|--------------|---|
|Менеджер|75000|
|Дизайнер|110000|


### 3)	Сделать выборку по должностям, в результате которой отобразятся данные, больше ли средняя ЗП по должности, чем средняя ЗП по всем работникам

### Решение:
```sql
select p.position_title, avg(jh.amount),
	case
		when avg(jh.amount)>97000 then 'Да'
		else 'Нет'
	end 
  as moreThanAvg
from job_history jh
inner join positions p on jh.position_id = p.position_id
group by p.position_title;
```

### Результат:

|position_title|avg|morethanavg|
|--------------|---|-----------|
|Менеджер|75000|Нет|
|Дизайнер|110000|Да|


### 4)	Сделать представление, в котором собраны данные по должностям (Должность, в каких отделах встречается эта должность (в виде массива), список сотрудников, начавших работать в этом отделе не раньше 2021 года (Сгруппировать по отделам) (в формате JSON), средняя заработная плата по должности)

### Решение (частичное):
```sql
CREATE VIEW positions_data as
SELECT 
    p.position_title AS "Должность",
    jsonb_agg(json_build_array(d.departments_name))  as jt
FROM job_history jh
INNER JOIN positions p ON jh.position_id = p.position_id
INNER JOIN employees e ON jh.employee_id = e.employee_id
INNER JOIN departments d ON jh.department_id = d.department_id 
GROUP BY p.position_title;

-- использование --
select * from positions_data pd 
```

### Результат:

|Должность|jt|
|---------|--|
|Менеджер|[["Снабжение"], ["Логистика"], ["Снабжение"]]|
|Дизайнер|[["Разработка"]]|
