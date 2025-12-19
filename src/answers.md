# Ответы на задания семинара 5

## Часть 1: Библиотечная система

Поскольку в README.md было сказано "Если каких-то значений не будет хватать в БД, то добавьте их сами."
в файл library/init.sql было добавлено несколько значений (без них пункты 2 и 5 текущего задания выдавали пустые таблички).

### Вопрос 1.

Найдите фамилии читателей, проживающих в Москве.

```sql
SELECT DISTINCT LastName
FROM Reader
WHERE Address ILIKE 'Москва%';
```

### Вопрос 2

Найдите книги (author, title), которые брал Иван Иванов.

```sql
SELECT DISTINCT b.Author, b.Title
FROM Borrowing br
JOIN Reader r ON r.ID = br.ID
JOIN Book b ON b.ISBN = br.ISBN
WHERE r.LastName = 'Иванов'
  AND r.FirstName = 'Иван';
```

### Вопрос 3

Найдите книги (ISBN) из категории "Горы", которые не относятся к категории "Путешествия" (игнорируя подкатегории).

```sql
SELECT DISTINCT bc1.ISBN
FROM BookCategory bc1
WHERE bc1.CategoryName = 'Горы'
  AND NOT EXISTS (
    SELECT 1
    FROM BookCategory bc2
    WHERE bc2.ISBN = bc1.ISBN
      AND bc2.CategoryName = 'Путешествия'
  );
```

### Вопрос 4

Найдите читателей (LastName, FirstName), которые вернули копию книги.

```sql
SELECT DISTINCT r.LastName, r.FirstName
FROM Borrowing br
JOIN Reader r ON r.ID = br.ID
WHERE br.ReturnDate IS NOT NULL;

```

### Вопрос 5

Найдите читателей (LastName, FirstName), которые брали хотя бы одну книгу, которую также брал Иван Иванов (не включая
самого Ивана Иванова).

```sql
SELECT DISTINCT r.LastName, r.FirstName
FROM Borrowing br
JOIN Reader r ON r.ID = br.ID
WHERE br.ISBN IN (
    SELECT br2.ISBN
    FROM Borrowing br2
    JOIN Reader r2 ON r2.ID = br2.ID
    WHERE r2.LastName = 'Иванов'
      AND r2.FirstName = 'Иван'
)
AND NOT (r.LastName = 'Иванов' AND r.FirstName = 'Иван');

```

## Часть 2: Поезда

### Вопрос 1

Найдите все прямые рейсы из Москвы в Тверь.

```sql
SELECT
  c.TrainNr,
  c.FromStation,
  c.ToStation,
  c.Departure,
  c.Arrival
FROM Connection c
JOIN Station s_from ON s_from.Name = c.FromStation
JOIN Station s_to   ON s_to.Name   = c.ToStation
WHERE s_from.CityName = 'Москва'
  AND s_to.CityName   = 'Тверь'
ORDER BY c.Departure;
```

### Вопрос 2

Найдите все многосегментные маршруты, имеющие точно однодневный трансфер из Москвы в Санкт-Петербург (первое отправление
и прибытие в конечную точку должны быть в одну и ту же дату). Используйте функцию `DAY()` для атрибутов `Departure` и
`Arrival`.

```sql
SELECT
  c1.FromStation  AS start_station,
  c1.ToStation    AS transfer_station,
  c2.ToStation    AS end_station,
  c1.TrainNr      AS train_1,
  c1.Departure    AS departure_1,
  c1.Arrival      AS arrival_1,
  c2.TrainNr      AS train_2,
  c2.Departure    AS departure_2,
  c2.Arrival      AS arrival_2
FROM Connection c1
JOIN Connection c2
  ON c1.ToStation = c2.FromStation
 AND c1.TrainNr <> c2.TrainNr
 AND c1.Arrival <= c2.Departure
JOIN Station s_start ON s_start.Name = c1.FromStation
JOIN Station s_end   ON s_end.Name   = c2.ToStation
WHERE s_start.CityName = 'Москва'
  AND s_end.CityName   = 'Санкт-Петербург'
  AND DATE(c1.Departure) = DATE(c2.Arrival)
  AND EXTRACT(DAY FROM c1.Departure) = EXTRACT(DAY FROM c2.Arrival)
ORDER BY c1.Departure;
```
