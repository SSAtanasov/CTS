# Примери за основни обекти и операции в СУДБ

---

## Изглед (View)

Изгледът е логическа таблица, която представя данни от една или повече таблици. Той не съдържа самите данни, а динамично генерира резултати въз основа на заявка.

```sql
-- Създаване на изглед за активни студенти
CREATE VIEW active_students AS
SELECT name, faculty_number, grade
FROM students
WHERE status = 'active';

-- Използване – изглежда като обикновена таблица
SELECT * FROM active_students;
```

---

## Индекс (Index)

Индексът може да се асоциира със съдържанието на книга. Ускорява търсенето в базата данни, като създава структури за бърз достъп до данните. Използва се при големи таблици.

```sql
-- Без индекс – търси ред по ред (пълно сканиране)
SELECT * FROM students WHERE faculty_number = '12345';

-- Създаване на индекс
CREATE INDEX idx_faculty_number ON students(faculty_number);

-- Същата заявка вече използва индекса – многократно по-бърза
SELECT * FROM students WHERE faculty_number = '12345';
```

---

## Ограничения (Constraints)

Задават правила за данните, които не могат да бъдат нарушавани.

```sql
CREATE TABLE students (
    id        INT PRIMARY KEY,               -- първичен ключ: уникален, не NULL
    name      VARCHAR(100) NOT NULL,         -- задължително поле
    age       INT CHECK (age >= 18),         -- проверка на стойност
    dept_id   INT REFERENCES departments(id) -- външен ключ
);

-- Опит за нарушение – ще върне грешка
INSERT INTO students VALUES (1, 'Иван', 15, 1); -- грешка: age < 18
```

---

## Тригер (Trigger)

Процедура, която се изпълнява автоматично при INSERT, UPDATE или DELETE.

```sql
-- Тригер, който записва кой е изтрил запис и кога
CREATE TRIGGER log_delete
AFTER DELETE ON students
FOR EACH ROW
INSERT INTO audit_log(action, student_id, deleted_at)
VALUES ('DELETE', OLD.id, NOW());
```

---

## Курсор (Cursor)

Позволява обработка на редовете от дадена заявка по един ред наведнъж.

```sql
DECLARE cur CURSOR FOR
    SELECT order_id, total FROM orders WHERE status = 'pending';

OPEN cur;
FETCH cur INTO @oid, @total;

WHILE @@FETCH_STATUS = 0 DO
    -- обработка на всяка поръчка поотделно
    CALL process_order(@oid, @total);
    FETCH cur INTO @oid, @total;
END WHILE;

CLOSE cur;
```

---

## Съхранена процедура (Stored Procedure)

SQL код, съхранен в базата данни, компилиран еднократно, изпълняван многократно. Приема входни параметри.

```sql
CREATE PROCEDURE enroll_student(
    IN p_student_id INT,
    IN p_course_id  INT
)
BEGIN
    INSERT INTO enrollments(student_id, course_id, enrolled_at)
    VALUES (p_student_id, p_course_id, NOW());
END;

-- Извикване
CALL enroll_student(42, 7);
```

---

## Функция (Function)

Подпрограма, създадена от една или повече SQL заявки, която връща стойност.

```sql
CREATE FUNCTION get_average_grade(p_student_id INT)
RETURNS DECIMAL(4,2)
BEGIN
    DECLARE avg_grade DECIMAL(4,2);
    SELECT AVG(grade) INTO avg_grade
    FROM grades WHERE student_id = p_student_id;
    RETURN avg_grade;
END;

-- Използване в заявка
SELECT name, get_average_grade(id) AS average FROM students;
```

---

## Транзакция (Transaction)

Последователност от заявки, изпълнявани атомарно – или всичко, или нищо.

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE id = 1; -- тегли от сметка А
UPDATE accounts SET balance = balance + 500 WHERE id = 2; -- внася в сметка Б

-- Ако нещо се обърка:
ROLLBACK;  -- и двете операции се отменят

-- Ако всичко е наред:
COMMIT;    -- и двете операции се записват трайно
```
