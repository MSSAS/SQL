# Решения SQL задач

В этом файле собраны решения различных SQL задач с условиями и примерами запросов.

---

## 1. Сотрудники проекта "Video Database"
**Условие:**  
Найдите всех сотрудников, занятых на проекте "Video Database". Выведите номер сотрудника, имя, фамилию, дату приёма на работу и код должности. Отсортируйте результат по фамилиям, а при совпадении — по коду должности.

**Решение:**
```sql
SELECT E.EMP_NO, E.FIRST_NAME, E.LAST_NAME, E.HIRE_DATE, E.JOB_CODE
FROM EMPLOYEE E
JOIN EMPLOYEE_PROJECT EP ON EP.EMP_NO = E.EMP_NO
JOIN PROJECT P ON P.PROJ_ID = EP.PROJ_ID
WHERE PROJ_NAME = 'Video Database'
ORDER BY LAST_NAME, JOB_CODE;
````

---

---

## 3. Сотрудники, принятые в 1992 году

**Условие:**
Выведите полное имя и дату приёма на работу всех сотрудников, принятых в 1992 году. Отсортируйте по дате приёма.

**Решение:**

```sql
SELECT LAST_NAME || ', ' || FIRST_NAME AS FULL_NAME, HIRE_DATE
FROM EMPLOYEE
WHERE HIRE_DATE >= '1992-01-01 00:00:00'
  AND HIRE_DATE < '1993-01-01 00:00:00';
```

---

## 4. Сотрудники с максимальной зарплатой в подразделении

**Условие:**
Найдите сотрудников, имеющих максимальную зарплату в своём подразделении, с помощью оконной функции. Отсортируйте по убыванию зарплаты.

**Решение:**

```sql
WITH TAB AS (
    SELECT D.DEPARTMENT,
           E.EMP_NO,
           E.FIRST_NAME,
           E.LAST_NAME,
           E.SALARY,
           DENSE_RANK() OVER(PARTITION BY D.DEPARTMENT ORDER BY E.SALARY DESC) AS SALARY_RANK
    FROM DEPARTMENT D
    JOIN EMPLOYEE E ON D.DEPT_NO = E.DEPT_NO
)
SELECT DEPARTMENT, EMP_NO, FIRST_NAME, LAST_NAME, SALARY
FROM TAB
WHERE SALARY_RANK = 1
ORDER BY SALARY DESC;
```

---

## 5. Среднее время активности клиента

**Условие:**
Вычислите среднее время между первой и последней арендой клиента, округлите до целого.

**Решение:**

```sql
WITH t AS (
    SELECT c.customer_id,
           MAX(r.rental_date) AS last_rental,
           MIN(r.rental_date) AS first_rental
    FROM customer c
    JOIN rental r ON r.customer_id = c.customer_id
    GROUP BY c.customer_id
)
SELECT ROUND(AVG(DATEDIFF(last_rental, first_rental))) AS avg_life_time
FROM t;
```

---

## 6. Рейтинг сотрудников по зарплате

**Условие:**
Составьте рейтинг сотрудников так, чтобы на первом месте был сотрудник с максимальной зарплатой. При одинаковом рейтинге сортируйте по имени.

**Решение:**

```sql
SELECT FULL_NAME,
       SALARY,
       RANK() OVER(ORDER BY SALARY DESC) AS RANK
FROM EMPLOYEE
ORDER BY RANK ASC, FULL_NAME;
```

---

## 7. Последний клиент, арендовавший диск с inventory\_id = 30

**Условие:**
Найдите клиента, который последним арендовал диск с указанным inventory\_id.

**Решение:**

```sql
WITH tab AS (
    SELECT c.first_name, c.last_name, c.email, a.address,
           ROW_NUMBER() OVER(ORDER BY rental_date DESC) AS NUM_DATE
    FROM inventory i
    JOIN rental r ON r.inventory_id = i.inventory_id
    JOIN customer c ON c.customer_id = r.customer_id
    JOIN address a ON a.address_id = c.address_id
    WHERE i.inventory_id = 30
)
SELECT first_name, last_name, email, address
FROM tab
WHERE num_date = 1;
```

---

## 8. Фильмы, отсутствующие в прокате

**Условие:**
Найдите фильмы, которых нет в таблице inventory.

**Решение:**

```sql
SELECT DISTINCT f.title AS film_title
FROM film f
LEFT JOIN inventory i ON i.film_id = f.film_id
WHERE inventory_id IS NULL;
```

---

## 9. Языки без фильмов

**Условие:**
Выведите языки, на которых нет доступных фильмов.

**Решение:**

```sql
SELECT name AS language
FROM language l
LEFT JOIN film f ON f.language_id = l.language_id
WHERE f.language_id IS NULL
ORDER BY name;
```

---

## 10. Фильмы в наличии, но не выданные в прокат

**Условие:**
Найдите фильмы, которые есть в inventory, но не выдавались в прокат.

**Решение:**

```sql
SELECT title
FROM film f
JOIN inventory i ON i.film_id = f.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
WHERE r.rental_id IS NULL;
```

---

## 11. Разделение email на имя и домен

**Условие:**
Выведите email, часть до «@» и часть после «@» для клиентов.

**Решение:**

```sql
SELECT email,
       SUBSTRING_INDEX(email, '@', 1) AS address,
       SUBSTRING_INDEX(email, '@', -1) AS domain
FROM customer
GROUP BY customer_id
ORDER BY email;
```

---

## 12. Средняя ставка аренды по категориям

**Условие:**
Выведите фильмы с их категорией, ставкой аренды и средней ставкой аренды по категории.

**Решение:**

```sql
SELECT f.title,
       c.name AS category,
       rental_rate,
       AVG(rental_rate) OVER(PARTITION BY c.name) AS category_avg_rental_rate
FROM film f
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON c.category_id = fc.category_id;
```

---

## 13. Платежи за август 2005 с накопительной суммой

**Условие:**
Выведите платежи за август 2005 с нарастающим итогом по сумме.

**Решение:**

```sql
SELECT payment_id,
       payment_date,
       amount,
       SUM(amount) OVER (ORDER BY payment_date, payment_id) AS rolling_sum
FROM payment
WHERE payment_date >= '2005-08-01'
  AND payment_date < '2005-09-01';
```

---

## 14. Количество фильмов в каждой категории

**Условие:**
Посчитайте количество фильмов в каждой категории и отсортируйте по названию категории.

**Решение:**

```sql
SELECT c.name AS category,
       COUNT(fc.film_id) AS film_count
FROM film_category fc
LEFT JOIN category c ON fc.category_id = c.category_id
GROUP BY fc.category_id;
```

---

## 15. Первый и последний платеж клиента

**Условие:**
Для каждого клиента выведите дату первого и последнего платежа, а также сумму всех платежей. Отсортируйте по убыванию суммы, при равенстве — по customer\_id.

**Решение:**

```sql
SELECT customer_id,
       DATE(MIN(payment_date)) AS first_payment_date,
       DATE(MAX(payment_date)) AS last_payment_date,
       SUM(amount) AS total_paid
FROM payment
GROUP BY customer_id
ORDER BY total_paid DESC, customer_id ASC;
```

---

```

---
