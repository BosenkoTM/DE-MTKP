# Модуль № 1: Разработка, администрирование и защита баз данных

## Инструменты реализации и программное обеспечение

Для выполнения задания на платформе «Альт Образование» 10 используется следующее программное обеспечение:

1. **Операционная система:**  
   - Альт Образование 10 (версия 10.4) [alt-education-10.4-x86_64.iso](https://download.basealt.ru/pub/distributions/ALTLinux/p10/images/education/x86_64/alt-education-10.4-x86_64.iso).

2. **СУБД:**  
   - MySQL Server 8.0 или выше.  
     Установить MySQL можно через пакетный менеджер APT или YUM, в зависимости от конфигурации системы.  

3. **Инструменты для управления базами данных:**  
   - MySQL Workbench: графический интерфейс для работы с MySQL.  
     Скачать можно здесь: [MySQL Workbench](https://dev.mysql.com/downloads/workbench/).
   - Командная строка MySQL: доступна после установки MySQL Server.

4. **Текстовый редактор для написания скриптов:**  
   - VS Code с плагином SQLTools для подключения к MySQL.  
     Скачать VS Code: [Visual Studio Code](https://code.visualstudio.com/).  
     Плагин SQLTools: [SQLTools for VS Code](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools).

5. **Шифрование паролей:**  
   - Использование функций шифрования MySQL (`AES_ENCRYPT` и `AES_DECRYPT`).  

---

## ПО

* [Проверка connect to DB.](/Help/CrudTest)

* [Подготовленный образ ALT Linux 10 Образование v 0.1 от17.05.22](https://disk.yandex.ru/d/UAM1dnC0P1kpKw)

---

## Текст задания

### Общие требования:
Выберите СУБД (MySQL) и среду для управления инфраструктурой. Установите ядро выбранной СУБД и среду для управления SQL (на виртуальную машину с ОС «Альт Образование» 10).

- **Имя сервера:** При установке задайте имя сервера в формате `Server_номер вашего рабочего места`, например, `Server_10`.
- **Режим аутентификации:** Включите режим смешанной аутентификации.
- **Пользователь sa:** Включите или создайте пользователя `root` с паролем в формате `De_номер вашего рабочего места`, например, `De_10`.

---

### Задачи:

#### 1. Автоматизация создания пользователей и баз данных:
Напишите скрипт, который автоматически:
- Создаст 10 пользователей (`user1`, `user2`, ..., `user10`) с паролями длиной 5 символов (буквы и цифры), сформированными случайным образом.
- Создаст 10 баз данных (`BD1`, `BD2`, ..., `BD10`).
- Настраивает права доступа так, чтобы пользователь `user1` имел доступ только к базе данных `BD1`, `user2` — только к `BD2` и т.д.

---

#### 2. Создание базы данных для хранения пользователей:
- Создайте базу данных `BD` и таблицу `Users` для хранения информации о пользователях и их паролях.
- Заполните таблицу `Users` данными о созданных пользователях и их паролях.
- **Важно:** Пароли должны храниться в зашифрованном виде для обеспечения безопасности доступа к серверу.

---

#### 3. Шифрование и дешифрование паролей:
- Напишите скрипт, который зашифрует все пароли в таблице `Users` с использованием функции `AES_ENCRYPT`.
- Создайте дополнительный скрипт, который позволит отобразить данные из таблицы `Users` с расшифрованными паролями с использованием функции `AES_DECRYPT`.

---

#### 4. Резервное копирование базы данных:
- Напишите скрипт для резервного копирования базы данных `BD`. Необходимо предоставить как сам скрипт, так и файл бэкапа.

---

#### 5. Восстановление базы данных:
- Напишите скрипт для восстановления базы данных `BD` из созданного ранее резервного копирования.

---

### Пример структуры решения:

1. **Скрипт создания пользователей и баз данных:**
   ```sql
   -- Создание пользователей и баз данных
   DELIMITER $$

   CREATE PROCEDURE CreateUserAndDB()
   BEGIN
       DECLARE i INT DEFAULT 1;
       WHILE i <= 10 DO
           SET @username = CONCAT('user', i);
           SET @password = SUBSTRING(MD5(RAND()) FROM 1 FOR 5); -- Генерация случайного пароля
           SET @dbname = CONCAT('BD', i);

           -- Создание базы данных
           SET @create_db = CONCAT('CREATE DATABASE IF NOT EXISTS ', @dbname);
           PREPARE stmt FROM @create_db;
           EXECUTE stmt;
           DEALLOCATE PREPARE stmt;

           -- Создание пользователя
           SET @create_user = CONCAT('CREATE USER ''', @username, '''@''%'' IDENTIFIED BY ''', @password, '''');
           PREPARE stmt FROM @create_user;
           EXECUTE stmt;
           DEALLOCATE PREPARE stmt;

           -- Назначение прав
           SET @grant_privileges = CONCAT('GRANT ALL PRIVILEGES ON ', @dbname, '.* TO ''', @username, '''@''%''');
           PREPARE stmt FROM @grant_privileges;
           EXECUTE stmt;
           DEALLOCATE PREPARE stmt;

           SET i = i + 1;
       END WHILE;
   END$$

   DELIMITER ;

   CALL CreateUserAndDB();
   ```

2. **Создание базы данных `BD` и таблицы `Users`:**
   ```sql
   CREATE DATABASE BD;
   USE BD;

   CREATE TABLE Users (
       UserID INT AUTO_INCREMENT PRIMARY KEY,
       Username VARCHAR(50),
       PasswordHash VARBINARY(256)
   );

   -- Заполнение таблицы Users
   INSERT INTO Users (Username, PasswordHash)
   SELECT CONCAT('user', number), AES_ENCRYPT(SUBSTRING(MD5(RAND()) FROM 1 FOR 5), 'encryption_key')
   FROM (SELECT @row := @row + 1 AS number FROM (SELECT @row := 0) r, information_schema.columns LIMIT 10) numbers;
   ```

3. **Шифрование и дешифрование паролей:**
   ```sql
   -- Отображение расшифрованных паролей
   SELECT 
       UserID,
       Username,
       AES_DECRYPT(PasswordHash, 'encryption_key') AS DecryptedPassword
   FROM Users;
   ```

4. **Резервное копирование базы данных:**
   ```bash
   # Создание бэкапа через командную строку
   mysqldump -u root -p BD > /path/to/backup/BD_backup.sql
   ```

5. **Восстановление базы данных:**
   ```bash
   # Восстановление базы данных из бэкапа
   mysql -u root -p BD < /path/to/backup/BD_backup.sql
   ```

---

### Ожидаемый результат:
- Все скрипты выполнены и протестированы.
- Бэкап базы данных `BD` создан и предоставлен.
- Процедура восстановления успешно проверена.

Прикрепить электронный отчет №1, в котором отобразить:

1. Работоспособность ОС «Альт Образование» 10 в виртуальной машине Virtual Box. 

2. Скрипты задач 1-2. Бэкап базы данных `BD`. Процедура восстановления.

Продемонстрировать работу скриптов задачи 3-5 преподавателю.
