
# SQL Practice: Northwind Database

## 1. О проекте

**Цель:** продемонстрировать практические навыки работы с SQL на примере учебной базы данных Northwind.

В проекте представлены запросы для:
- фильтрации и поиска данных;
- объединения результатов нескольких выборок;
- добавления, изменения и удаления записей;
- соединения нескольких таблиц;
- группировки и агрегирования данных;
- расчёта показателей на основе данных заказов.

**Учебная база данных:** Northwind  
**Диалект SQL:** Microsoft SQL Server (T-SQL).

Все запросы являются учебными примерами. Изменяющие данные запросы не предназначены для выполнения в рабочей базе без предварительной проверки.

## 2. Фильтрация и выборка данных

### 2.1. Поиск поставщиков по городу и стране

**Задача:** получить поставщиков, город которых начинается с буквы S, исключив поставщиков из стран, название которых также начинается с S.

```sql
SELECT *
FROM Suppliers
WHERE City LIKE 'S%'
AND NOT Country LIKE 'S%';
```

### 2.2. Объединение результатов выборок

**Задача:** получить список городов клиентов и поставщиков, исключив записи из Германии, Франции и Англии.

```sql
SELECT City FROM Customers
WHERE Country NOT IN ('Germany', 'France', 'England')
UNION
SELECT City FROM Suppliers
WHERE Country NOT IN ('Germany', 'France', 'England')
ORDER BY City;
```

`UNION` объединяет результаты двух выборок и удаляет дубликаты.

### 2.3. Поиск по шаблону

**Задача:** найти сотрудников, у которых поле Notes содержит указанный шаблон.

```sql
SELECT * FROM Employees
WHERE Notes LIKE '%a_BA%'
ORDER BY LastName, FirstName;
```

`LIKE` используется для поиска по шаблону. Символ `%` обозначает любое количество символов, а `_` — ровно один символ.

## 3. Добавление, изменение и удаление данных

### 3.1. Добавление нового клиента

```sql id="zjd8ks"
INSERT INTO Customers (CustomerName, ContactName, Address, City, PostalCode, Country)
VALUES('Arcobaleno', 'Mark Glushko','Red Str. 5','Krasnodar', '350063', 'Russia');
```

Запрос демонстрирует добавление записи с помощью `INSERT INTO`.

### 3.2. Изменение описания категории

```sql id="w5nm0q"
UPDATE Categories
SET Description = 'hot chocolate, cocoa, wine'
WHERE CategoryID = 1;
```

Запрос демонстрирует изменение данных с помощью `UPDATE` и условия `WHERE`.

### 3.3. Удаление категории

```sql id="p3hz9v"
DELETE FROM Categories WHERE CategoryName='Seafood';
```

Запрос демонстрирует удаление записи с помощью `DELETE`.

**Важно:** перед выполнением `UPDATE` и `DELETE` необходимо проверить условие с помощью `SELECT` и убедиться, какие записи будут изменены или удалены. Удаление также может быть ограничено связями с другими таблицами.

## 4. Соединение таблиц (JOIN)

### 4.1. Получение контактов клиентов по категории товаров — старый синтаксис

```sql
SELECT DISTINCT Customers.ContactName
FROM Customers, Orders, OrderDetails, Products, Categories
WHERE Customers.CustomerID = Orders.CustomerID
AND Orders.OrderID = OrderDetails.OrderID
AND OrderDetails.ProductID = Products.ProductID
AND Products.CategoryID = Categories.CategoryID
AND Categories.CategoryID = 8;
```

Запрос демонстрирует соединение пяти таблиц через условия `WHERE` и использование `DISTINCT` для исключения повторов.

### 4.2. Та же задача с использованием INNER JOIN

```sql
SELECT DISTINCT Customers.ContactName
FROM (((Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID)
INNER JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID)
INNER JOIN Products ON OrderDetails.ProductID = Products.ProductID)
INNER JOIN Categories ON Products.CategoryID = Categories.CategoryID
WHERE Categories.CategoryID = 8;
```

Запрос демонстрирует явное соединение нескольких связанных таблиц с помощью `INNER JOIN`.

### 4.3. Подсчёт позиций заказов по клиентам — старый синтаксис

```sql id="g7k2vp"
SELECT Customers.ContactName, COUNT(OrderDetails.OrderID) AS Orders
FROM Customers, Orders, OrderDetails
WHERE Customers.CustomerID = Orders.CustomerID
AND Orders.OrderID = OrderDetails.OrderID
GROUP BY Customers.ContactName
```

Запрос демонстрирует соединение таблиц, группировку и подсчёт строк с помощью `COUNT`.

### 4.4. Та же задача с использованием INNER JOIN

```sql id="m4x8qd"
SELECT Customers.ContactName, COUNT(OrderDetails.OrderID) AS Orders
FROM (Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID)
INNER JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID
GROUP BY Customers.ContactName
```

`COUNT(OrderDetails.OrderID)` считает строки с позициями заказов, а не количество уникальных заказов.

## 5. Агрегатные функции и группировка

### 5.1. Топ-5 товаров по количеству продаж

```sql
SELECT TOP 5 Products.ProductName, Products.ProductID, SUM(OrderDetails.Quantity) AS Quantity
FROM Products
INNER JOIN OrderDetails ON Products.ProductID = OrderDetails.ProductID
GROUP BY Products.ProductName, Products.ProductID
ORDER BY SUM(OrderDetails.Quantity) DESC
```

Используются `INNER JOIN`, `SUM`, `GROUP BY`, `ORDER BY` и `TOP 5`.

### 5.2. Подсчёт заказов по перевозчикам

```sql
SELECT Shippers.ShipperName, COUNT(Orders.ShipperID) AS QuantityOrder
FROM Shippers
INNER JOIN Orders ON Orders.ShipperID = Shippers.ShipperID
GROUP BY Shippers.ShipperName
```

Запрос демонстрирует соединение таблиц `Shippers` и `Orders`, группировку и подсчёт связанных записей.

### 5.3. Среднее значение суммы позиций заказов по клиентам

```sql
SELECT Customers.ContactName, ROUND(AVG(OrderDetails.Quantity * Products.Price), 2) AS AverageSumma
FROM ((Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID)
INNER JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID)
INNER JOIN Products ON OrderDetails.ProductID = Products.ProductID
GROUP BY Customers.ContactName
HAVING ROUND(AVG(OrderDetails.Quantity * Products.Price), 2) >= 1000
```

Используются `INNER JOIN`, `AVG`, `ROUND`, `GROUP BY` и `HAVING`.

Запрос рассчитывает среднее значение произведения количества товара на цену для строк заказа, а не среднюю стоимость целого заказа.

## 6. Используемые SQL-конструкции и навыки

| SQL-конструкция | Назначение |
|---|---|
| SELECT | Получение данных |
| WHERE | Фильтрация записей |
| LIKE | Поиск по шаблону |
| NOT, IN | Исключение и проверка значений |
| UNION | Объединение выборок |
| ORDER BY | Сортировка |
| DISTINCT | Исключение повторов |
| INSERT INTO | Добавление записей |
| UPDATE | Изменение записей |
| DELETE | Удаление записей |
| INNER JOIN | Соединение таблиц |
| COUNT | Подсчёт записей |
| SUM | Суммирование |
| AVG | Вычисление среднего |
| GROUP BY | Группировка |
| HAVING | Фильтрация групп |
| ROUND | Округление |
| TOP | Ограничение количества результатов |

## 7. Применение SQL в тестировании

Полученные навыки можно применять для:
- проверки наличия и корректности записей;
- поиска данных по заданным условиям;
- проверки связей между таблицами;
- проверки результатов добавления, изменения и удаления данных;
- подсчёта записей и сравнения агрегированных значений;
- проверки данных, связанных с заказами и товарами.

## 8. Примечания

- Запросы написаны с использованием синтаксиса Microsoft SQL Server (T-SQL).
- Названия таблиц и полей сохранены в соответствии с исходными учебными запросами.
- В разных версиях Northwind названия некоторых таблиц и столбцов могут отличаться. Например, в некоторых вариантах таблица позиций заказа называется `Order Details`, а не `OrderDetails`.
- Изменяющие данные запросы следует выполнять только в учебной копии базы данных.
- Перед удалением или изменением записей рекомендуется предварительно проверить результат соответствующим `SELECT`-запросом.
- `TOP 5` — конструкция T-SQL для ограничения количества возвращаемых строк.
- `INNER JOIN` возвращает только записи, для которых найдены совпадения в обеих таблицах.

---

**Проект выполнен в учебных целях на базе Northwind.**
