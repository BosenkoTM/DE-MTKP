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
- **Формула:**  
$$
ADR = Общая выручка / Количество проданных ночей
$$

---

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

