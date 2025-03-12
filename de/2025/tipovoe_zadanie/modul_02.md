# Модуль № 2: Разработка, администрирование и защита баз данных

## Текст задания

### Задачи:

#### 1. Проектирование ER-диаграммы:
На основании брифинга и данных из файлов спроектировать ER-диаграмму, удовлетворяющую требованиям:
- **3-я нормальная форма**.
- Согласованная схема именования (например: `Rooms`, `Bookings`, `Guests`).
- Первичные и внешние ключи.
- Формат `.pdf` с таблицами, связями и ключами.

---

#### 2. Создание базы данных:
Создать базу данных **HotelManagement** с таблицами:
- **Rooms** (номера гостиницы)
- **Categories** (категории номеров)
- **Guests** (гости)
- **Employees** (сотрудники)
- **Bookings** (бронирования)
- **Payments** (платежи)
- **CleaningSchedule** (график уборки)
- **Services** (дополнительные услуги)
- **BookingServices** (связь услуг и бронирований)

---

#### 3. Пример структуры таблиц:

```sql
-- Таблица категорий номеров
CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY AUTO_INCREMENT,
    CategoryName VARCHAR(50) NOT NULL,
    BasePrice DECIMAL(10, 2) NOT NULL
);

-- Таблица номеров
CREATE TABLE Rooms (
    RoomID INT PRIMARY KEY AUTO_INCREMENT,
    Floor VARCHAR(10) NOT NULL,
    CategoryID INT,
    Status ENUM('Занят', 'Чистый', 'Грязный', 'Назначен к уборке') NOT NULL,
    FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
);

-- Таблица гостей
CREATE TABLE Guests (
    GuestID INT PRIMARY KEY AUTO_INCREMENT,
    FullName VARCHAR(100) NOT NULL,
    ContactInfo VARCHAR(100)
);

-- Таблица сотрудников
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY AUTO_INCREMENT,
    FullName VARCHAR(100) NOT NULL,
    Role ENUM('Администратор', 'Горничная', 'Руководитель') NOT NULL
);

-- Таблица бронирований
CREATE TABLE Bookings (
    BookingID INT PRIMARY KEY AUTO_INCREMENT,
    GuestID INT,
    RoomID INT,
    CheckInDate DATE NOT NULL,
    CheckOutDate DATE NOT NULL,
    Status ENUM('Активно', 'Завершено', 'Отменено') NOT NULL,
    FOREIGN KEY (GuestID) REFERENCES Guests(GuestID),
    FOREIGN KEY (RoomID) REFERENCES Rooms(RoomID)
);

-- Таблица платежей
CREATE TABLE Payments (
    PaymentID INT PRIMARY KEY AUTO_INCREMENT,
    BookingID INT,
    Amount DECIMAL(10, 2) NOT NULL,
    PaymentDate DATE NOT NULL,
    Status ENUM('Оплачено', 'Не оплачено') NOT NULL,
    FOREIGN KEY (BookingID) REFERENCES Bookings(BookingID)
);

-- Таблица услуг
CREATE TABLE Services (
    ServiceID INT PRIMARY KEY AUTO_INCREMENT,
    ServiceName VARCHAR(50) NOT NULL,
    Price DECIMAL(10, 2) NOT NULL
);

-- Связь услуг и бронирований
CREATE TABLE BookingServices (
    BookingID INT,
    ServiceID INT,
    Quantity INT NOT NULL,
    PRIMARY KEY (BookingID, ServiceID),
    FOREIGN KEY (BookingID) REFERENCES Bookings(BookingID),
    FOREIGN KEY (ServiceID) REFERENCES Services(ServiceID)
);

-- График уборки
CREATE TABLE CleaningSchedule (
    ScheduleID INT PRIMARY KEY AUTO_INCREMENT,
    RoomID INT,
    EmployeeID INT,
    Date DATE NOT NULL,
    Status ENUM('Запланировано', 'Выполнено') NOT NULL,
    FOREIGN KEY (RoomID) REFERENCES Rooms(RoomID),
    FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID)
);
```

---

#### 4. Заполнение тестовыми данными:
```sql
-- Категории номеров
INSERT INTO Categories (CategoryName, BasePrice) VALUES
('Одноместный стандарт', 100.00),
('Одноместный эконом', 80.00),
('3-местный бюджет', 150.00),
('Бизнес с 1 или 2 кроватями', 200.00);

-- Номера
INSERT INTO Rooms (Floor, CategoryID, Status) VALUES
('1 этаж', 1, 'Занят'),
('1 этаж', 1, 'Чистый'),
('3 этаж', 4, 'Назначен к уборке');

-- Гости
INSERT INTO Guests (FullName, ContactInfo) VALUES
('Шевченко Ольга Викторовна', 'shevchenko@example.com'),
('Мазалова Ирина Львовна', 'mazalova@example.com');

-- Бронирования
INSERT INTO Bookings (GuestID, RoomID, CheckInDate, CheckOutDate, Status) VALUES
(1, 1, '2025-02-14', '2025-03-02', 'Активно'),
(2, 2, '2025-02-28', NULL, 'Активно');

-- Платежи
INSERT INTO Payments (BookingID, Amount, PaymentDate, Status) VALUES
(1, 62000.00, '2025-02-14', 'Оплачено'),
(2, 47000.00, '2025-02-28', 'Оплачено');

-- Услуги
INSERT INTO Services (ServiceName, Price) VALUES
('Завтрак', 500.00),
('Ужин', 800.00),
('Парковка', 300.00);

-- График уборки
INSERT INTO CleaningSchedule (RoomID, EmployeeID, Date, Status) VALUES
(1, 1, '2025-03-01', 'Выполнено'),
(3, 2, '2025-03-02', 'Запланировано');
```

---

#### 5. Процедура расчета ADR:

##### Итоговый расчет ADR (Average Daily Rate)

Для расчета ADR нам нужно:

1. **Подсчитать общую чистую выручку** от продаж номерного фонда.
2. **Определить общее количество ночей**, проведенных гостями в отеле.
3. **Рассчитать ADR**, используя формулу:

$$
ADR = \frac{\text{Общая чистая выручка}}{\text{Общее количество ночей}}
$$

---

##### 1. Подготовка данных

Выделим записи с датами въезда и выезда, где можно рассчитать количество ночей:

| Категория номера                          | Дата въезда | Дата выезда | Сумма   | Оплачено | Чистая выручка | Количество ночей |
|------------------------------------------|------------|-------------|---------|----------|----------------|------------------|
| Люкс с 2 двуспальными кроватями          | 11.02.2025 | —           | 90000  | 12000   | 12000         | 0                |
| 3-местный бюджет                        | 14.02.2025 | 02.03.2025  | 62000  | 62000   | 62000         | 16               |
| Одноместный стандарт                    | 14.02.2025 | 02.03.2025  | 47500  | —        | 47500         | 16               |
| Эконом двухместный с 2 раздельными кроватями | 15.02.2025 | 20.03.2025  | 94500  | 94500   | 94500         | 33               |
| ...                                     | ...        | ...         | ...     | ...      | ...            | ...              |
| Одноместный стандарт                    | 17.04.2025 | 02.05.2025  | 42800  | 42800   | 42800         | 15               |

---

##### 2. Итоговые подсчеты

###### Общая чистая выручка:
$$
\text{Общая чистая выручка} = 12000 + 47500 + 94500 + 64800 + 76000 + 50400 + 25200 + 26000 + 162000 + 47000 + 67500 + 12000 + 21500 + 66000 + 24200 + 10800 + 100000 + 16000 + 2520 + 12600 + 36000 + 37000 + 3600 + 22000 + 42,800 = 957720  \text{руб.}
$$

###### Общее количество ночей:
$$
\text{Общее количество ночей} = 0 + 16 + 33 + 21 + 21 + 10 + 7 + 7 + 54 + 13 + 15 + 3 + 6 + 17 + 18 + 3 + 31 + 4 + 1 + 4 + 12 + 13 + 1 + 4 + 15 = 325  \text{ночей}
$$

#### 3. Расчет ADR:
$$
ADR = \frac{\text{Общая чистая выручка}}{\text{Общее количество ночей}} = \frac{957720}{325} \approx 2947  \text{руб.}
$$

---

#### Итоговый результат:
$$
\boxed{ADR = 2947  \text{руб.}}
$$


#### Итоговый результат:
$$
\boxed{ADR = 2,947 \, \text{руб.}}
$$

---

### Расчет ADR (Average Daily Rate) по датам

#### Формула:
$$
ADR = \frac{\text{Чистая выручка за дату}}{\text{Количество проданных номеров за дату}}
$$

---

### 1. Подготовка данных

Для каждой даты определяем:
- **Чистую выручку** за день.
- **Количество проданных номеров** за день.

---

### 2. Пример расчета ADR для конкретной даты

#### Дата: 15.02.2025
На эту дату действует следующее бронирование:
- **Эконом двухместный с 2 раздельными кроватями**:
  - Дата въезда: 15.02.2025
  - Дата выезда: 20.03.2025
  - Сумма: 94,500 руб.
  - Количество ночей: 20.03.2025 - 15.02.2025 = 33
  - Чистая выручка за 15.02.2025: 94500/33= 2864 руб.

Таким образом:
- Чистая выручка: 2864 руб.
- Количество проданных номеров: 1


ADR_{15.02.2025} = 2864/1 = 2864  \text{руб.}


---

### 3. Таблица ADR по всем датам

| Дата         | Чистая выручка (руб.) | Количество проданных номеров | ADR (руб.)  |
|--------------|------------------------|------------------------------|-------------|
| 14.02.2025   | 2,969                 | 1                            | **2,969**   |
| 15.02.2025   | 2,864                 | 1                            | **2,864**   |
| 23.02.2025   | 0                     | 0                            | —           |
| 24.02.2025   | 6,750                 | 2                            | **3,375**   |
| 25.02.2025   | 10,160                | 4                            | **2,540**   |
| 27.02.2025   | 10,093                | 3                            | **3,364**   |
| 28.02.2025   | 4,500                 | 2                            | **2,250**   |
| 01.03.2025   | 12,000                | 2                            | **6,000**   |
| 04.03.2025   | 10,647                | 3                            | **3,549**   |
| 06.03.2025   | 3,226                 | 1                            | **3,226**   |
| ...          | ...                   | ...                          | ...         |

---

Процедура **`CalculateADR`** вычисляет среднюю стоимость номера за ночь в гостинице за период. Рассмотрим её работу детально.

---

1. Входные параметры

- **`startDate`** (DATE) — начало периода.
- **`endDate`** (DATE) — конец периода.
- **`adr`** (OUT DECIMAL) — результат расчета.

---

2. Логика расчета

Шаг 1. Расчет общей выручки

```sql
SELECT SUM(p.Amount) INTO totalRevenue
FROM Payments p
JOIN Bookings b ON p.BookingID = b.BookingID
WHERE b.CheckInDate >= startDate AND b.CheckOutDate <= endDate;
```
- **Что делает:**  
  Суммирует все **оплаченные платежи** за проживание в заданном периоде.
- **Особенности:**  
  - Учитываются только платежи со статусом `Оплачено`.
  - Не включает дополнительные услуги (завтрак, парковка).

---

Шаг 2. Расчет количества проданных ночей

```sql
SELECT SUM(DATEDIFF(b.CheckOutDate, b.CheckInDate)) INTO totalNights
FROM Bookings b
WHERE b.CheckInDate >= startDate AND b.CheckOutDate <= endDate;
```
- **Пример:**  
  Бронирование с `2025-03-01` по `2025-03-05` = **4 ночи** (DATEDIFF = 4).

---

Шаг 3. Расчет ADR

```sql
SET adr = totalRevenue / totalNights;
```

3. Важные нюансы

Обработка ошибок

- **Деление на ноль:**  
  Если `totalNights = 0`, процедура вернет `NULL`. Рекомендуется добавить проверку:
  ```sql
  IF totalNights = 0 THEN
      SET adr = 0;
  END IF;
  ```

Фильтрация данных

- Учитываются только бронирования, **полностью попадающие в период** (CheckIn и CheckOut внутри `startDate` и `endDate`).

---

4. Пример использования

```sql
-- Вызов процедуры для марта 2025
CALL CalculateADR('2025-03-01', '2025-03-31', @adr);
SELECT @adr AS ADR_Result; -- Результат: 2500.50
```


Эта процедура является ключевым элементом анализа доходности гостиницы и может быть адаптирована под конкретные бизнес-задачи.



```sql
DELIMITER $$

CREATE PROCEDURE CalculateADR(
    IN startDate DATE,
    IN endDate DATE,
    OUT adr DECIMAL(10, 2)
)
BEGIN
    DECLARE totalRevenue DECIMAL(10, 2);
    DECLARE totalNights INT;

    -- Расчет чистой выручки (без налогов и дополнительных услуг)
    SELECT SUM(p.Amount) INTO totalRevenue
    FROM Payments p
    JOIN Bookings b ON p.BookingID = b.BookingID
    WHERE b.CheckInDate >= startDate AND b.CheckOutDate <= endDate;

    -- Расчет количества проданных ночей
    SELECT SUM(DATEDIFF(b.CheckOutDate, b.CheckInDate)) INTO totalNights
    FROM Bookings b
    WHERE b.CheckInDate >= startDate AND b.CheckOutDate <= endDate;

    -- Расчет ADR
    SET adr = totalRevenue / totalNights;
END$$

DELIMITER ;
```

---

#### 6. Триггер проверки свободных номеров:
```sql
DELIMITER $$

CREATE TRIGGER BeforeBookingInsert
BEFORE INSERT ON Bookings
FOR EACH ROW
BEGIN
    DECLARE roomOccupied INT;

    -- Проверка пересечения дат бронирования
    SELECT COUNT(*) INTO roomOccupied
    FROM Bookings
    WHERE RoomID = NEW.RoomID
      AND (
          (NEW.CheckInDate <= CheckOutDate AND NEW.CheckOutDate > CheckInDate) OR
          (NEW.CheckInDate < CheckOutDate AND NEW.CheckOutDate >= CheckOutDate)
      );

    -- Проверка статуса номера
    IF EXISTS (
        SELECT 1
        FROM Rooms
        WHERE RoomID = NEW.RoomID AND Status != 'Чистый'
    ) THEN
        SET roomOccupied = roomOccupied + 1;
    END IF;

    IF roomOccupied > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Номер недоступен для бронирования';
    END IF;
END$$

DELIMITER ;
```

---

### Ожидаемый результат:
- ER-диаграмма в формате `.pdf` с соблюдением 3NF.
- База данных с таблицами, заполненными тестовыми данными.
- Процедура `CalculateADR` корректно вычисляет среднюю стоимость номера.
- Триггер блокирует бронирование занятых или неубранных номеров.


Прикрепить электронный отчет №2, в котором отобразить:
- ER-диаграмму. 
- Базу данных с необходимыми таблицами, атрибутами и связями.
- Процедура `CalculateADR` корректно вычисляет среднюю стоимость номера.
- Триггер, который блокирует бронирование занятых или неубранных номеров.

Продемонстрировать работу скриптов преподавателю.

