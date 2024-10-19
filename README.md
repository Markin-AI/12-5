# Домашнее задание к занятию "`Индексы`" - `Маркин Алексей`

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

---

### Решение 1

```sql
select sum(data_length) , sum(index_length), TRUNCATE (sum(index_length)*100.0/sum(data_length),1) as '%'
from information_schema.tables
where table_schema='sakila' and data_length is not null;
```

![Задание 1](https://github.com/Markin-AI/12-5/blob/main/img/1-1.png)

---

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

---

### Решение2

![Задание 2](https://github.com/Markin-AI/12-5/blob/main/img/2-1.png)

Узкие места:

 - Отсутствует JOIN.  
 - Функция **DATE()** на **p.payment_date**, может препятствовать использованию индексов.  
 - Отсутствие индексов необходимых индексов.

Оптимизация:

```sql
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name),
       SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE DATE(p.payment_date) = '2005-07-30';
```

![Задание 2](https://github.com/Markin-AI/12-5/blob/main/img/2-2.png)

```sql
SELECT DISTINCT CONCAT(c.last_name, ' ', c.first_name),
       SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p
JOIN rental r ON p.payment_date = r.rental_date
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE p.payment_date >= '2005-07-30 00:00:00' AND p.payment_date < '2005-07-31 00:00:00';
```

![Задание 2](https://github.com/Markin-AI/12-5/blob/main/img/2-3.png)

Сверху последний запрос, снизу первый.

![Задание 2](https://github.com/Markin-AI/12-5/blob/main/img/2-4.png)


---