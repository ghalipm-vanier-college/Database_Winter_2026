
# Module: SQL Server Triggers (T-SQL)


---

## 1️⃣ What Is a Trigger?

A **trigger** is a special SQL Server object that **automatically executes** when a data-modification event occurs on a table or view.

Triggers are **event-driven** and **cannot be called directly**.

---

## 2️⃣ Types of Triggers in SQL Server

###  AFTER Triggers
- Execute **after** `INSERT`, `UPDATE`, or `DELETE`
- Commonly used for:
  - Validation
  - Auditing
  - Updating related tables

###  INSTEAD OF Triggers
- Execute **instead of** the triggering operation
- Most often used on **views**
- Allow custom logic before data is written to base tables

---

## 3️⃣ When Should Triggers Be Used?

Triggers are useful for:

- Enforcing **business rules**
- Preventing invalid data changes
- Maintaining **data consistency**
- Automatically updating related data
- Protecting sensitive tables

 Avoid heavy logic inside triggers to reduce performance impact.

---

## 4️⃣ Special Trigger Tables

SQL Server provides two **virtual tables** inside triggers:

| Table | Description |
|------|------------|
| `inserted` | New rows (INSERT / UPDATE) |
| `deleted` | Old rows (DELETE / UPDATE) |

 Always write triggers assuming **multiple rows**.

---

## 5️⃣ AFTER Trigger – Essential Example

### Example: Prevent Negative Inventory

```sql
CREATE TRIGGER trg_NoNegativeInventory
ON Product
AFTER UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM inserted
        WHERE P_QOH < 0
    )
    BEGIN
        ROLLBACK;
        PRINT 'Error: Inventory cannot be negative.';
    END
END;
````

---

## 6️⃣ AFTER Trigger – Business Logic Example

### Example: Automatically Update Invoice Total

```sql
CREATE TRIGGER trg_UpdateInvoiceTotal
ON Invoice
AFTER INSERT, UPDATE
AS
BEGIN
    UPDATE Invoice
    SET INV_TOTAL = INV_SUBTOTAL + INV_TAX
    WHERE INV_NUMBER IN (SELECT INV_NUMBER FROM inserted);
END;
```

---

## 7️⃣ INSTEAD OF Trigger – Why and When?

**INSTEAD OF triggers**:

* Replace the default `INSERT`, `UPDATE`, or `DELETE`
* Are commonly used on **views**
* Allow validation or transformation before data reaches base tables

---

## 8️⃣ INSTEAD OF Trigger – Essential Example (View Insert)

### Example: Insert Products via a View

```sql
CREATE VIEW vwProductSimple
AS
SELECT P_CODE, P_DESCRIPT, P_PRICE, V_CODE
FROM Product;
```

```sql
CREATE TRIGGER trg_InsertProductView
ON vwProductSimple
INSTEAD OF INSERT
AS
BEGIN
    INSERT INTO Product (P_CODE, P_DESCRIPT, P_PRICE, P_QOH, P_MIN, P_DISCOUNT, V_CODE)
    SELECT P_CODE, P_DESCRIPT, P_PRICE, 0, 0, 0, V_CODE
    FROM inserted;
END;
```

---

## 9️⃣ INSTEAD OF Trigger – Protection Example

### Example: Prevent Deleting Invoices

```sql
CREATE VIEW vwInvoice
AS
SELECT * FROM Invoice;
```

```sql
CREATE TRIGGER trg_NoInvoiceDelete
ON vwInvoice
INSTEAD OF DELETE
AS
BEGIN
    PRINT 'Invoice deletion is not allowed.';
END;
```



## 🔟 Key Differences (Quick View)

| Feature              | AFTER Trigger | INSTEAD OF Trigger |
| -------------------- | ------------- | ------------------ |
| Runs after operation | ✅             | ❌                  |
| Replaces operation   | ❌             | ✅                  |
| Common on tables     | ✅             | ❌                  |
| Common on views      | ❌             | ✅                  |


#  Practice Questions – SQL Server Triggers

## Product Vendor Tables
```sql
CREATE TABLE Vendor (
    V_CODE INT PRIMARY KEY,
    V_NAME VARCHAR(50),
    V_STATE CHAR(2),
    V_CONTACT VARCHAR(50)
);

INSERT INTO Vendor VALUES
(101, 'Tech Supply Co.', 'ON', 'Michael Lee'),
(102, 'OfficePro Inc.', 'BC', 'Samantha Chan');

CREATE TABLE Product (
    P_CODE VARCHAR(10) PRIMARY KEY,
    P_DESCRIPT VARCHAR(100),
    P_QOH INT,
    P_MIN INT,
    P_PRICE DECIMAL(10,2),
    P_DISCOUNT DECIMAL(4,2),
    V_CODE INT FOREIGN KEY REFERENCES Vendor(V_CODE)
);

INSERT INTO Product VALUES
('P100', 'Laptop', 20, 5, 1200.00, 0.10, 101),
('P200', 'Mouse', 8, 4, 25.00, 0.05, 101),
('P300', 'Desk Chair', 30, 10, 150.00, 0.00, 102);

CREATE TABLE Customer (
    CUS_CODE INT PRIMARY KEY,
    CUS_LNAME VARCHAR(30),
    CUS_FNAME VARCHAR(30),
    CUS_INITIAL CHAR(1),
    CUS_AREACODE CHAR(5),
    CUS_PHONE VARCHAR(15)
);

INSERT INTO Customer VALUES
(1001, 'Brown', 'Alice', 'A', '514', '555-1111'),
(1002, 'Smith', 'Bob', 'B', '416', '555-2222'),
(1003, 'Jones', 'Cathy', 'C', '418', '555-3333'),
(10011, 'Miller', 'David', 'D', '403', '555-4444'),
(10012, 'Taylor', 'Emma', 'E', '604', '555-5555');

CREATE TABLE Invoice (
    INV_NUMBER INT PRIMARY KEY,
    CUS_CODE INT FOREIGN KEY REFERENCES Customer(CUS_CODE),
    INV_DATE DATE,
    INV_SUBTOTAL DECIMAL(10,2),
    INV_TAX DECIMAL(10,2),
    INV_TOTAL DECIMAL(10,2)
);

INSERT INTO Invoice VALUES
(1009, 1001, '2025-01-10', 500.00, 65.00, 565.00),
(1010, 1002, '2025-02-12', 1200.00, 156.00, 1356.00);

CREATE TABLE Line (
    LINE_ID INT IDENTITY PRIMARY KEY,
    INV_NUMBER INT FOREIGN KEY REFERENCES Invoice(INV_NUMBER),
    P_CODE VARCHAR(10) FOREIGN KEY REFERENCES Product(P_CODE),
    LINE_UNITS INT,
    LINE_PRICE DECIMAL(10,2)
);

INSERT INTO Line VALUES
(1009, 'P100', 1, 1200.00);
```

## AFTER Trigger Questions

### Q1. Low Stock Warning 

Create an **AFTER UPDATE trigger** on the `Product` table that prints a message when `P_QOH` drops below `P_MIN`.

---

### Q2. Prevent Negative Quantity 

Create a trigger that **rolls back** any update that sets `P_QOH` to a negative value.

---

### Q3. Reduce Inventory on Sale 

Create an **AFTER INSERT trigger** on the `Line` table that decreases `P_QOH` in `Product` by `LINE_UNITS`.

---

### Q4. Protect Vendor Deletion 

Create a trigger that prevents deleting a vendor if products exist for that vendor.

---

## INSTEAD OF Trigger Questions

### Q5. Insert Through a View 

Create a view that shows only `P_CODE`, `P_DESCRIPT`, and `P_PRICE`.
Then create an **INSTEAD OF INSERT trigger** that allows inserting products through this view.

---

### Q6. Prevent Deletes Through a View 

Create a view on the `Invoice` table and an **INSTEAD OF DELETE trigger** that blocks all delete attempts and displays a message.

---

### Q7. Validate Data Before Insert

Using an **INSTEAD OF INSERT trigger** on a product view, prevent inserting products with a negative price.

---

##  Key Takeaways

* AFTER triggers enforce rules **after** data changes
* INSTEAD OF triggers **replace** default behavior
* Use `inserted` and `deleted` tables correctly
* Keep triggers **simple and efficient**

---

 **End of Trigger Module – SQL Server (T-SQL)**

````

