# Задание демонстрационного экзамена (2024 год)  
**Код специальности:** 09.02.07 Информационные системы и программирование  
**Квалификация:** Программист, Flvbybcnhfnjh ,fp lfyys[  
**Вид аттестации:** Государственная итоговая аттестация  
**Уровень ДЭ:** Профильный  
**Шифр варианта:** В3_КОД 09.02.07-02-2024-ПУ  

---

## Модуль 1. Разработка модулей ПО для компьютерных систем  
**Описание предметной области:**  
См. [Прил_1_В3_КОД 09.02.07-2-2024-ПУ](data/В3_КОД%2009.02.07-2-2024-ПУ.docx).  

**Техническое задание:**  
См. [Прил_2_В3_КОД 09.02.07-2-2024-ПУ](data/Приложение%20к%20В3_КОД%2009.02.07-2-2024-ПУ.rar).  

**Ресурсы:**  
См. [Ресурсы](data/В3_Ресурсы.rar).

### Задачи:
1. **Анализ и спецификация:**  
   - Составить краткую спецификацию модуля для учета заявок на ремонт автомобилей.  
   - Определить входные/выходные данные.  
   - Разработать блок-схему алгоритма (по ГОСТ 19.701 или 24.301).  

2. **Реализация:**  
   - Создать интерфейс в среде разработки (C#, Java, Python или 1C).  
   - Использовать динамические списки/массивы для хранения данных.  
   - Пример алгоритма расчета среднего времени ремонта:  
     ```python
     def calculate_avg_repair_time(orders):
         total_time = sum(order.end_date - order.start_date for order in orders)
         return total_time / len(orders)
     ```

3. **Обработка исключений:**  
   - Проверка корректности дат (например, `end_date > start_date`).  
   - Уведомления об ошибках с пиктограммами (ошибка, предупреждение).  

4. **Тестирование:**  
   - Пример теста для функции расчета:  
     ```python
     def test_calculate_avg_repair_time():
         orders = [Order(start_date=1, end_date=3), Order(start_date=2, end_date=5)]
         assert calculate_avg_repair_time(orders) == 2.5
     ```

---

## Модуль 2. Разработка и защита баз данных  
**Задачи:**  
1. **Проектирование ER-диаграммы:**  
   - Сущности: `Clients`, `Cars`, `RepairOrders`, `Services`.  
   - Связи: Клиент может иметь много автомобилей, заказ — много услуг.  
   - Пример структуры таблиц:  
     ```sql
     CREATE TABLE Clients (
         ClientID INT PRIMARY KEY,
         FullName VARCHAR(100),
         Phone VARCHAR(20)
     );

     CREATE TABLE RepairOrders (
         OrderID INT PRIMARY KEY,
         ClientID INT,
         CarID INT,
         StartDate DATE,
         EndDate DATE,
         FOREIGN KEY (ClientID) REFERENCES Clients(ClientID)
     );
     ```

2. **Работа с данными:**  
   - Импорт данных из файлов заказчика (например, `repairs_import.csv`).  
   - Запрос для отчета:  
     ```sql
     SELECT Clients.FullName, COUNT(RepairOrders.OrderID) AS TotalRepairs
     FROM Clients
     JOIN RepairOrders ON Clients.ClientID = RepairOrders.ClientID
     GROUP BY Clients.FullName;
     ```

3. **Безопасность:**  
   - Роли: `Admin`, `Manager`, `Technician`.  
   - Права:  
     ```sql
     GRANT SELECT, INSERT ON RepairOrders TO Manager;
     GRANT ALL PRIVILEGES ON Clients TO Admin;
     ```

---

## Модуль 3. Сопровождение ПО  
**Задачи:**  
1. **Документация:**  
   - Составить [Руководство системного программиста](https://www.swrit.ru/doc/espd/19.503-79.pdf) по стандарту ЕСПД.  

2. **Модификация ПО:**  
   - Добавить роль **Менеджер** с функциями:  
     - Создание/редактирование заявок.  
     - Просмотр статистики.  
   - Пример SQL для новой роли:  
     ```sql
     CREATE ROLE Manager;
     GRANT EXECUTE ON CreateRepairOrder TO Manager;
     ```

3. **Анализ качества кода:**  
   - Критерии:  
     - Обработка ошибок: 100% критических сценариев.  
     - Тесты: минимум 1 тест на функцию.  
     - Комментарии: только для неочевидных участков.  

---

**Важно:**  
- Код должен соответствовать [стандартам именования](https://its.1c.ru/db/v8std#browse:13:-1:31).  
- Все скриншоты отладки и тестирования сохранить в папку [screenshots](screenshots/).  
