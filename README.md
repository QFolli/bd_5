1 задание
Используя курсор, посчитать средние значения всех числовых столбцов для результатов, возвращенных запросом №1 своего варианта индивидуального задания к лабораторной работе №3.
DECLARE
  @totalAmount decimal(12, 2),
  @totalAmountSum AS decimal(12, 2),
  @rowsCount AS int;
SET @totalAmountSum = 0;

DECLARE @totalAmounts TABLE (
  TotalAmount decimal(12, 2)
);

INSERT INTO @totalAmounts (TotalAmount)
  SELECT
    o.TotalAmount
  FROM
    [Order] AS o
  WHERE
    o.OrderDate BETWEEN '2013-12-1' AND '2014-2-1'

DECLARE avgNumbers CURSOR FOR
SELECT * FROM @totalAmounts;

OPEN avgNumbers;
FETCH NEXT FROM avgNumbers INTO @totalAmount;
WHILE @@FETCH_STATUS = 0 BEGIN
  SET @totalAmountSum = @totalAmountSum + @totalAmount;
  FETCH NEXT FROM avgNumbers INTO @totalAmount;
END
CLOSE avgNumbers;
DEALLOCATE avgNumbers;

SELECT @rowsCount = COUNT(*) FROM @totalAmounts;
PRINT N'Средняя стоимость: ';
PRINT @totalAmountSum / @rowsCount;
![image](https://github.com/QFolli/bd_5/assets/88559269/9e46481c-663c-40d5-ae6f-afc179338b08)
![image](https://github.com/QFolli/bd_5/assets/88559269/13bdf9a4-1046-49fc-935e-7fd3ab69095e)

2 задание
В базе данных, созданной в лабораторной работе №2, выбрать 2 таблицы, связанные между собой отношением один-ко-многим, и для них написать хранимые процедуры, реализующие добавление, обновление и удаление записей.

CREATE TABLE Bank
(
    Id INT PRIMARY KEY,
    [Name] VARCHAR(255) NOT NULL UNIQUE CHECK (LEN([Name]) BETWEEN 2 AND 255)
);

CREATE TABLE ATM
(
    Number INT PRIMARY KEY,
    [Address] VARCHAR(255) NOT NULL CHECK (LEN(Address) BETWEEN 2 AND 255),
    RemainingCurrency MONEY NOT NULL,
    BankId INT NOT NULL REFERENCES Bank(Id)
);
CREATE PROCEDURE AddBank
  @id AS INT,
  @name AS VARCHAR(255)
AS
BEGIN
  INSERT INTO Bank (Id, [Name])
    VALUES (@id, @name)
END
EXEC AddBank 1, N'Exim'
CREATE PROCEDURE UpdateBank
  @id AS INT,
  @name AS VARCHAR(255)
AS
BEGIN
  UPDATE Bank
    SET [Name] = @name
  WHERE Id = @id
END
EXEC UpdateBank 1, N'EximUpdated'
CREATE PROCEDURE DeleteBank
  @id AS INT
AS
BEGIN
  DELETE FROM Bank
    WHERE Id = @id
END
EXEC DeleteBank 1
CREATE PROCEDURE AddATM
  @address AS VARCHAR(255),
  @remainingCurrency AS MONEY,
  @bankId AS INT
AS
BEGIN
  INSERT INTO ATM ([Address], RemainingCurrency, BankId)
    VALUES (@address, @remainingCurrency, @bankId)
END
EXEC AddATM N'Tiraspol', 16.35, 1
CREATE PROCEDURE UpdateATMAddress
  @number AS INT,
  @address AS VARCHAR(255)
AS
BEGIN
  UPDATE ATM
    SET [Address] = @address
  WHERE Number = @number
END
EXEC UpdateATMAddress 1, N'Bender'
CREATE PROCEDURE UpdateATMRemainingCurrency
  @number AS INT,
  @remainingCurrency AS MONEY
AS
BEGIN
  UPDATE ATM
    SET RemainingCurrency = @remainingCurrency
  WHERE Number = @number
END
EXEC UpdateATMRemainingCurrency 1, 16.30
CREATE PROCEDURE DeleteATM
  @number AS INT
AS
BEGIN
  DELETE FROM ATM
    WHERE Number = @number
END
EXEC DeleteATM 1

3 задание
Написать скалярную функцию, принимающую в качестве параметра название продукта, и возвращающую его количество во всех заказах.

CREATE FUNCTION GetProductsCountByProductName
(
  @queryProductName AS VARCHAR(255)
)
RETURNS INT
AS
BEGIN
  DECLARE @productsCount AS INT;
  SELECT @productsCount =
    COUNT(*)
  FROM
    OrderItem
    JOIN Product ON Product.Id = OrderItem.ProductId
  WHERE
    Product.ProductName = @queryProductName
  RETURN @productsCount
END

SELECT dbo.GetProductsCountByProductName(N'Chang')
![image](https://github.com/QFolli/bd_5/assets/88559269/e2a93f82-66cd-4130-adde-0a88ea583986)

4 задание
Для базы данных, созданной в лабораторной работе №2, написать табличную функцию, которая будет возвращать следующую информацию:

Вариант №1: Сведения об операциях определенного клиента на определенную дату. Фамилия клиента и дата передаются в качестве параметров.
Создание и заполнение таблиц
CREATE FUNCTION GetClientOperations
(
  @clientLastName AS VARCHAR(20),
  @operationDate AS DATE
)
RETURNS TABLE
AS
RETURN
(
  SELECT
    Client.*,
    Operation.*
  FROM 
    Client
    JOIN CreditCard ON CreditCard.ClientPassportId = Client.PassportId
    JOIN Operation ON Operation.CreditCardNumber = CreditCard.Number
  WHERE
    Client.LastName = @clientLastName
    AND Operation.Date = @operationDate
)

SELECT * FROM dbo.GetClientOperations(N'Doe', '2022-03-22')

![image](https://github.com/QFolli/bd_5/assets/88559269/c1b4b8d5-6ba4-405d-83de-7b7caf6f07fa)


