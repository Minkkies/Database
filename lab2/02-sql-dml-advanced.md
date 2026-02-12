# SQL DML ขั้นสูง (Advanced SQL Data Manipulation Language)

## ภาพรวม

ในส่วนนี้จะเจาะลึก SQL DML (Data Manipulation Language) สำหรับการจัดการข้อมูลขั้นสูง รวมถึง:
- SELECT ขั้นสูงพร้อมฟังก์ชันต่างๆ
- WHERE และการกรองข้อมูล
- Aggregate Functions และการจัดกลุ่ม
- JOIN หลายรูปแบบ
- Subquery และ Transaction

---

## เอกสารอ้างอิง

- [เอกสารจารย์ - COS3103 SQL DML](https://luckkrit.github.io/cos3103/slides/2_68/sql_dml/)
- [ดาวน์โหลด PostgreSQL](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
- [Classic Models Database Schema](https://luckkrit.github.io/cos3103/sql/postgresql-classicmodels.sql)

---

## การตั้งค่าฐานข้อมูล

### ติดตั้ง PostgreSQL

1. ดาวน์โหลดและติดตั้ง PostgreSQL
2. (Optional) เปิดใช้งาน PostGIS Extension:
   ```sql
   CREATE EXTENSION postgis;
   ```

### โหลด Classic Models Database

```sql
-- ดาวน์โหลดไฟล์ SQL จาก URL ข้างต้น แล้ว execute
SET search_path TO public, classicmodels;
```

### โครงสร้างฐานข้อมูล Classic Models

| Table | Purpose |
|-------|---------|
| **customers** | ข้อมูลลูกค้า |
| **products** | ข้อมูลสินค้า/รุ่นรถ |
| **productLines** | หมวดหมู่สินค้า |
| **orders** | คำสั่งซื้อ |
| **orderDetails** | รายละเอียดแต่ละรายการในคำสั่งซื้อ |
| **payments** | ข้อมูลการชำระเงิน |
| **employees** | ข้อมูลพนักงาน |
| **offices** | สาขา/สำนักงาน |

---

## 2.1 SELECT ขั้นสูง

### 2.1.1 DISTINCT – ตัดค่าที่ซ้ำกันออก

DISTINCT ใช้เลือกเฉพาะค่าที่ไม่ซ้ำกัน (unique)

```sql
SELECT DISTINCT jobtitle FROM classicmodels.employees;
```

**ผลลัพธ์**: แสดงตำแหน่งงานที่ไม่ซ้ำกัน

**การใช้งาน**: เหมาะสำหรับหาค่าที่ไม่ซ้ำกัน เช่น รายชื่อเมือง, ประเทศ หรือประเภทสินค้า

---

### 2.1.2 คำนวนและ LIMIT

```sql
SELECT checkNumber, paymentDate, amount + 10000 AS new_amount
FROM classicmodels.payments
LIMIT 5;  -- เลือกเพียง 5 แถว
```

**ผลลัพธ์**: แสดงหมายเลขเช็ค วันที่ชำระ และจำนวนเงินบวกเพิ่มเติม 10000

**หมายเหตุ**: 
- สามารถทำการคำนวณใน SELECT ได้โดยตรง (+, -, *, /)
- LIMIT จำกัดจำนวนแถวที่แสดง มักใช้ร่วมกับ ORDER BY

---

### 2.1.3 ฟังก์ชันสตริง

PostgreSQL มีฟังก์ชันสตริงมากมายสำหรับจัดการข้อความ

#### UPPER – เปลี่ยนเป็นตัวพิมพ์ใหญ่

```sql
SELECT customername, UPPER(customername) AS upper_name
FROM classicmodels.customers;
```

**ผลลัพธ์**: แสดงชื่อลูกค้าและชื่อที่เป็นตัวพิมพ์ใหญ่

#### LOWER – เปลี่ยนเป็นตัวพิมพ์เล็ก

```sql
SELECT customername, LOWER(customername) AS lower_name
FROM classicmodels.customers;
```

#### REVERSE – กลับตัวอักษร

```sql
SELECT customername, REVERSE(customername)
FROM classicmodels.customers;
```

#### CONCAT – ต่อข้อความ

```sql
SELECT CONCAT(firstname, ' ', lastname) AS fullname
FROM employees;
```

**การใช้งาน**: รวมชื่อ-นามสกุล, สร้างรหัสอ้างอิง, หรือสร้างข้อความแสดงผล

#### RPAD – เติมทางขวา

```sql
SELECT customername, RPAD(customername, 30, '-') AS padded
FROM classicmodels.customers;
```

**ผลลัพธ์**: ชื่อลูกค้าจะมีความยาว 30 อักษร เติมด้วย '-' ทางขวา

#### LPAD – เติมทางซ้าย

```sql
SELECT customername, LPAD(customername, 30, '*') AS lpadded
FROM classicmodels.customers;
```

**การใช้งาน**: จัดรูปแบบข้อความให้มีความยาวเท่ากัน เหมาะสำหรับรายงาน

---

### 2.1.4 Boolean Expression

```sql
SELECT creditLimit, creditLimit > 100000 AS is_high_credit
FROM classicmodels.customers
LIMIT 10;
```

**ผลลัพธ์**: แสดงวงเงินและค่า TRUE/FALSE บ่งชี้ว่ามากกว่า 100000 หรือไม่

**การใช้งาน**: สร้างเงื่อนไขแบบ flag เพื่อจำแนกข้อมูล

---

## 2.2 CAST – แปลงชนิดข้อมูล

CAST ใช้เปลี่ยนชนิดข้อมูล (data type) จากชนิดหนึ่งเป็นอีกชนิดหนึ่ง

### รูปแบบที่ 1: ใช้ CAST()

```sql
SELECT CAST(officeCode AS INTEGER) AS office_code_int
FROM employees;
```

### รูปแบบที่ 2: ใช้ ::

```sql
SELECT officeCode::INTEGER FROM employees;
```

**ผลลัพธ์**: officeCode จะถูกแปลงเป็นตัวเลขจำนวนเต็ม

**การใช้งาน**: 
- แปลง TEXT เป็น INTEGER/NUMERIC สำหรับการคำนวณ
- แปลง VARCHAR เป็น DATE/TIMESTAMP เพื่อเปรียบเทียบวันที่
- แปลงชนิดข้อมูลให้ตรงกับที่ฟังก์ชันต้องการ

---

## 2.3 WHERE และตัวดำเนินการ

### 2.3.1 Alias และ Multiple Conditions

```sql
SELECT E.firstName AS fname, E.officeCode AS ocode
FROM employees AS E
WHERE (E.officeCode)::INTEGER > 3 
  AND (E.officeCode)::INTEGER <= 5;
```

**ผลลัพธ์**: แสดงชื่อพนักงานและรหัสสำนักที่อยู่ในช่วง 3-5

![WHERE with Alias](img/image-10.png)

**การใช้งาน**: 
- Alias (AS) ทำให้โค้ดสั้นและอ่านง่าย
- AND/OR สำหรับรวมหลายเงื่อนไข

---

### 2.3.2 ตัวดำเนินการเปรียบเทียบ

| Operator | ความหมาย |
|----------|----------|
| `=` | เท่ากับ |
| `<>` หรือ `!=` | ไม่เท่ากับ |
| `<` | น้อยกว่า |
| `>` | มากกว่า |
| `<=` | น้อยกว่าหรือเท่ากับ |
| `>=` | มากกว่าหรือเท่ากับ |

**ตัวอย่าง**:
```sql
SELECT * FROM products WHERE buyPrice >= 50;
SELECT * FROM orders WHERE status <> 'Shipped';
```

---

### 2.3.3 LIKE – ค้นหาแบบแพทเทิร์น

LIKE ใช้ค้นหาข้อความตามรูปแบบ โดยใช้ wildcard characters

#### % – แทนหลายอักษร (0 ตัวขึ้นไป)

```sql
-- ขึ้นต้นด้วย '1'
SELECT * FROM employees WHERE officeCode LIKE '1%';

-- ลงท้ายด้วย 'A'
SELECT * FROM customers WHERE city LIKE '%A';

-- มี 'A' อยู่ในคอลัมน์
SELECT * FROM employees WHERE lastName LIKE '%A%';
```

#### _ – แทนอักษรเดียวพอดี

```sql
SELECT firstName, lastName
FROM employees
WHERE birthDate LIKE '195_-__-__';
```

**ผลลัพธ์**: พนักงานที่เกิดปี 195X (1950-1959)

#### NOT LIKE และ ILIKE

```sql
SELECT employeeNumber, lastName, firstName
FROM employees
WHERE lastName NOT ILIKE 'B%';  -- ILIKE = case-insensitive LIKE
```

**ผลลัพธ์**: แสดงพนักงานที่นามสกุลไม่ขึ้นต้นด้วย B (ไม่สำคัญตัวใหญ่-เล็ก)

**หมายเหตุ**: 
- `LIKE` สำคัญตัวพิมพ์ใหญ่-เล็ก
- `ILIKE` ไม่สำคัญตัวพิมพ์ใหญ่-เล็ก (PostgreSQL)

---

### 2.3.4 IS NULL / IS NOT NULL

```sql
-- หาลูกค้าที่ไม่มีข้อมูลจังหวัด
SELECT customername, state
FROM customers
WHERE state IS NULL;

-- หาลูกค้าที่มีข้อมูลจังหวัด
SELECT customername, state
FROM customers
WHERE state IS NOT NULL;
```

**หมายเหตุ**: ห้ามใช้ `= NULL` หรือ `<> NULL` เด็ดขาด ต้องใช้ `IS NULL` หรือ `IS NOT NULL` เท่านั้น

---

### 2.3.5 BETWEEN – ช่วงค่า

```sql
SELECT *
FROM classicmodels.orderdetails
WHERE priceEach BETWEEN 150 AND 200;
```

**ผลลัพธ์**: แสดงรายการสั่งซื้อที่ราคาอยู่ระหว่าง 150-200 (รวม 150 และ 200)

**เทียบเท่า**:
```sql
WHERE priceEach >= 150 AND priceEach <= 200
```

---

### 2.3.6 IN / NOT IN

```sql
-- หาลูกค้าที่สัมพันธ์กับพนักงานหมายเลขดังกล่าว
SELECT contactLastName, salesRepEmployeeNumber
FROM classicmodels.customers
WHERE salesRepEmployeeNumber IN (1370, 1501, 1504);

-- หาสินค้าที่ไม่ใช่รหัสดังกล่าว
SELECT *
FROM products
WHERE productCode NOT IN ('S10_1678', 'S10_1949');
```

**การใช้งาน**: สะดวกกว่าการใช้ OR หลายครั้ง

---

## 2.4 Aggregate Functions – ฟังก์ชันการรวม

Aggregate Functions ใช้รวมและประมวลผลข้อมูลหลายแถวให้เป็นค่าเดียว

```sql
SELECT
    COUNT(*) AS total_employees,
    COUNT(reportsTo) AS has_manager,
    SUM(reportsTo) AS sum_managers,
    AVG(reportsTo) AS avg_manager_id,
    MAX(officeCode) AS max_office,
    MIN(officeCode) AS min_office
FROM employees;
```

### ฟังก์ชันที่สำคัญ

| Function | ความหมาย |
|----------|----------|
| `COUNT(*)` | นับทั้งหมด (รวม NULL) |
| `COUNT(col)` | นับที่ไม่ใช่ NULL |
| `SUM(col)` | รวมค่า |
| `AVG(col)` | ค่าเฉลี่ย |
| `MAX(col)` | ค่าสูงสุด |
| `MIN(col)` | ค่าต่ำสุด |

**หมายเหตุ**: Aggregate Functions จะข้าม NULL โดยอัตโนมัติ (ยกเว้น COUNT(*))

---

## 2.5 GROUP BY และ HAVING

### โครงสร้างคำสั่ง SQL แบบเต็ม

```sql
SELECT      <attribute list>
FROM        <table list>
[JOIN ON    <condition>]
[WHERE      <condition>]
[GROUP BY   <grouping attribute(s)>]
[HAVING     <group condition>]
[ORDER BY   <attribute list>];
```

---

### ORDER BY – จัดเรียงผลลัพธ์

เรียงลำดับผลลัพธ์ (ค่าเริ่มต้น ASC น้อย→มาก) หรือใช้ `DESC` เพื่อกลับลำดับ

```sql
SELECT contactLastname, contactFirstname
FROM classicmodels.customers
-- ORDER BY RANDOM()  -- หากต้องการสุ่ม
ORDER BY contactLastname;
```

**ผลลัพธ์**: จะเห็นว่าเรียงลำดับตามนามสกุล

![ORDER BY](img/image010.png)

**ตัวเลือก**:
- `ASC` - น้อย → มาก (default)
- `DESC` - มาก → น้อย
- `RANDOM()` - สุ่มลำดับ

---

### GROUP BY – จัดกลุ่มข้อมูล

```sql
SELECT status, COUNT(*) AS total_orders
FROM classicmodels.orders
GROUP BY status;
```

**ผลลัพธ์**: จะเห็นว่ามีการจับกลุ่ม (GROUP BY) และนับจำนวน (COUNT)

![GROUP BY](img/image-10.png)

**การใช้งาน**:
- จัดกลุ่มข้อมูลตามคอลัมน์ที่กำหนด
- ใช้ร่วมกับ Aggregate Functions
- คอลัมน์ที่เลือกใน SELECT ต้องอยู่ใน GROUP BY หรือเป็น Aggregate Function

---

### HAVING – กรองหลัง GROUP BY

HAVING ใช้กรองผลลัพธ์ที่จัดกลุ่มแล้ว (ใช้กับ Aggregate Functions)

#### WHERE vs HAVING

| คำสั่ง | ทำงานเมื่อไร | ใช้กับ | GROUP BY |
|--------|--------------|--------|----------|
| `WHERE` | กรองข้อมูลแต่ละแถว | คอลัมน์ปกติ | ก่อน GROUP BY |
| `HAVING` | กรองข้อมูลที่จัดกลุ่มแล้ว | Aggregate Functions | หลัง GROUP BY |

#### ตัวอย่าง

```sql
-- หาลูกค้าที่สั่งซื้อมากกว่า 5 ครั้ง แล้วเรียงจำนวนมากไปน้อย
SELECT customerNumber, COUNT(*) AS total_orders
FROM classicmodels.orders
GROUP BY customerNumber
HAVING COUNT(*) > 5
ORDER BY total_orders DESC;
```

**ผลลัพธ์**:

![HAVING](img/image-20.png)

---

## 2.6 ORDER BY และ LIMIT (เพิ่มเติม)

ORDER BY ใช้เรียงลำดับและสามารถทำงานร่วมกับ LIMIT เพื่อตัดจำนวนผลลัพธ์

```sql
SELECT orderNumber, orderDate, totalAmount
FROM orders
ORDER BY orderDate DESC
LIMIT 10 OFFSET 0;  -- OFFSET ข้ามแถวแรก 0 แถว
```

**ผลลัพธ์**: แสดง 10 อเดอร์ล่าสุด

```sql
SELECT * FROM products
ORDER BY quantityInStock ASC
LIMIT 5;
```

**ผลลัพธ์**: แสดง 5 สินค้าที่เหลือในสต็อกน้อยที่สุด

**การใช้งาน OFFSET**:
- `LIMIT 10` - เอา 10 แถวแรก
- `LIMIT 10 OFFSET 20` - ข้าม 20 แถว แล้วเอา 10 แถว (แถวที่ 21-30)
- เหมาะสำหรับทำ Pagination

---

## 2.7 JOIN – เชื่อมตาราง

JOIN ใช้เชื่อมข้อมูลจากหลายตาราง

### ประเภท JOIN

| JOIN Type | ความหมาย | แสดงจากตารางซ้าย | แสดงจากตารางขวา | เงื่อนไข |
|-----------|----------|------------------|------------------|----------|
| **JOIN** / **INNER JOIN** | แสดงเฉพาะที่ match | ✔️ เฉพาะที่ match | ✔️ เฉพาะที่ match | ต้องตรงตามเงื่อนไข |
| **LEFT JOIN** | แสดงทั้งหมดจากซ้าย | ✔️ ทั้งหมด | ✔️ ที่ match (ไม่ตรง=NULL) | LEFT OUTER JOIN ... ON ... |
| **RIGHT JOIN** | แสดงทั้งหมดจากขวา | ✔️ ที่ match (ไม่ตรง=NULL) | ✔️ ทั้งหมด | RIGHT OUTER JOIN ... ON ... |
| **CROSS JOIN** | Cartesian Product | ✔️ | ✔️ | ทุกคู่ที่เป็นไปได้ |
| **SELF JOIN** | Join ตารางกับตัวเอง | ✔️ | ✔️ | ใช้ alias แยกตาราง |
| **NATURAL JOIN** | Join อัตโนมัติ | ✔️ | ✔️ | ตามชื่อคอลัมน์เหมือนกัน |

**หมายเหตุ**:
- `JOIN` เฉยๆ = `INNER JOIN`
- ใช้ `LEFT/RIGHT JOIN` เมื่ออยากเก็บฝั่งที่ไม่มีคู่
- `NATURAL JOIN` สะดวกแต่เสี่ยงจับคู่คอลัมน์ผิดโดยไม่ตั้งใจ

---

### ON vs USING

- **ON**: กำหนดเงื่อนไขใดก็ได้ ไม่จำเป็นต้องมีชื่อคอลัมน์ตรงกัน
- **USING(col)**: ใช้เมื่อคอลัมน์ชื่อเดียวกัน ผลลัพธ์จะแสดงคอลัมน์นั้นเพียงชุดเดียว

---

### ตัวอย่าง INNER JOIN

```sql
SELECT c.customerName, o.orderNumber, o.orderDate
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber;
```

**ผลลัพธ์**: แสดงชื่อลูกค้าและหมายเลขอเดอร์สำหรับลูกค้าที่มีอเดอร์

---

### ตัวอย่าง LEFT JOIN

```sql
SELECT c.customerName, COUNT(o.orderNumber) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName;
```

**ผลลัพธ์**: แสดงลูกค้าทั้งหมด รวมทั้งที่ไม่มีอเดอร์ (จำนวน 0)

**การใช้งาน**: หาลูกค้าที่ยังไม่เคยสั่งซื้อ

---

### ตัวอย่าง CROSS JOIN

```sql
SELECT * FROM customers CROSS JOIN products LIMIT 5;
```

**ผลลัพธ์**: แสดงคู่ของลูกค้า-สินค้า (Cartesian Product)

**หมายเหตุ**: ระวังการใช้กับตารางใหญ่

---

### ตัวอย่าง USING Clause

```sql
SELECT c.customerName, o.orderNumber
FROM customers c
JOIN orders o USING (customerNumber);
```

**การใช้งาน**: โค้ดสั้นลงเมื่อชื่อคอลัมน์เหมือนกัน

---

### NATURAL JOIN vs INNER JOIN

- หากไม่มีคอลัมน์ชื่อเหมือนกัน:
  - `INNER JOIN` → Error (ต้องระบุเงื่อนไข)
  - `NATURAL JOIN` → Cartesian Product (อันตราย!)

**แนะนำ**: หลีกเลี่ยง NATURAL JOIN ในโปรเจกต์จริง ใช้ JOIN ... ON หรือ USING แทน

---

### COUNT() กับ JOIN

```sql
SELECT COUNT(*)
FROM classicmodels.orders;
```

**การใช้งาน**: นับจำนวนแถวหลัง JOIN

---

## 2.8 ลำดับการทำงานของ SQL

```
FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

### ตัวอย่างครบทุกขั้นตอน

```sql
SELECT department, COUNT(*) AS emp_count, AVG(salary) AS avg_sal
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) >= 3
ORDER BY avg_sal DESC
LIMIT 5;
```

### ขั้นตอนการทำงาน

1. **FROM** - เลือกตารางจาก employees
2. **WHERE** - กรองเฉพาะพนักงานที่เงินเดือน > 50000
3. **GROUP BY** - จัดกลุ่มตามแผนก
4. **HAVING** - กรองเฉพาะแผนกที่มีพนักงาน >= 3 คน
5. **SELECT** - เลือกแผนก นับจำนวน และค่าเฉลี่ยเงินเดือน
6. **ORDER BY** - เรียงตามเงินเดือนเฉลี่ยมากไปน้อย
7. **LIMIT** - แสดง 5 แผนกแรก

**การจำ**: "**F**rom **W**here **G**roup **H**aving **S**elect **O**rder **L**imit"

---

## 2.9 Subquery – คำสั่ง SELECT ซ้อน

Subquery (Inner Query) ใช้ผลลัพธ์จากคำสั่ง SELECT หนึ่งเป็นเงื่อนไขของอีกคำสั่ง

### Subquery in WHERE with IN

```sql
SELECT customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber FROM orders
    WHERE YEAR(orderDate) = 2023
);
```

**ผลลัพธ์**: แสดงลูกค้าที่มีอเดอร์ในปี 2023

**การทำงาน**:
1. Subquery รันก่อน → ได้รายการ customerNumber
2. Query หลักใช้ผลลัพธ์จาก Subquery

---

### Subquery with EXISTS

```sql
SELECT customerName
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customerNumber = c.customerNumber
      AND YEAR(o.orderDate) = 2023
);
```

**ผลลัพธ์**: เช่นเดียวกัน แต่ใช้ EXISTS ซึ่งมักมีประสิทธิภาพดีกว่า

**ความแตกต่าง**:
- `IN` - เช็คว่าค่าอยู่ในลิสต์หรือไม่
- `EXISTS` - เช็คว่ามีแถวที่ตรงเงื่อนไขหรือไม่ (หยุดทันทีที่เจอแถวแรก)

---

### Scalar Subquery

```sql
SELECT customerName, (
    SELECT COUNT(*)
    FROM orders
    WHERE orders.customerNumber = customers.customerNumber
) AS order_count
FROM customers;
```

**ผลลัพธ์**: แสดงชื่อลูกค้าพร้อมจำนวนอเดอร์ของแต่ละคน

**หมายเหตุ**: Scalar Subquery ต้องคืนค่าเพียงค่าเดียว (1 แถว 1 คอลัมน์)

---

## 2.10 COALESCE – จัดการ NULL

COALESCE แสดงค่าแรกที่ไม่ใช่ NULL

```sql
SELECT customerName, COALESCE(state, 'N/A') AS state_or_na
FROM customers;
```

**ผลลัพธ์**: แสดงจังหวัดของลูกค้า หากเป็น NULL จะแสดง 'N/A' แทน

**รูปแบบการใช้**:
```sql
COALESCE(value1, value2, value3, ..., default_value)
```

**การใช้งาน**:
- แทนค่า NULL ด้วยค่า default
- รวมข้อมูลจากหลายคอลัมน์ (เลือกคอลัมน์แรกที่ไม่เป็น NULL)

---

## 2.11 Set Operations: UNION, INTERSECT, EXCEPT

### UNION – รวมและตัดซ้ำ

```sql
(SELECT city FROM customers WHERE country = 'USA')
UNION
(SELECT city FROM customers WHERE country = 'France');
```

**ผลลัพธ์**: เมืองทั้งหมดจาก USA และ France (ไม่ซ้ำ)

**หมายเหตุ**: ใช้ `UNION ALL` ถ้าต้องการเก็บข้อมูลซ้ำ

---

### INTERSECT – เฉพาะที่อยู่ในทั้งสอง

```sql
(SELECT city FROM customers WHERE country = 'USA')
INTERSECT
(SELECT city FROM orders WHERE orderDate > '2023-01-01');
```

**ผลลัพธ์**: เมืองที่อยู่ในทั้งสอง query

---

### EXCEPT – อยู่ในแรกแต่ไม่ในที่สอง

```sql
(SELECT city FROM customers WHERE country = 'USA')
EXCEPT
(SELECT city FROM customers WHERE state IS NULL);
```

**ผลลัพธ์**: เมืองจาก USA ที่มี state ระบุ

**หมายเหตุ**:
- ทุก query ต้องมีจำนวนคอลัมน์และชนิดข้อมูลเท่ากัน
- ใช้ ORDER BY ได้เฉพาะคำสั่งสุดท้าย

---

## 2.12 Transaction – ธุรกรรมข้อมูล

Transaction ใช้ควบคุมการเปลี่ยนแปลงข้อมูลให้ปลอดภัย เป็นการรวมหลายคำสั่งให้สำเร็จหรือล้มเหลวพร้อมกัน

```sql
BEGIN;

UPDATE products
SET quantityInStock = quantityInStock - 1
WHERE productCode = 'S10_1678';

UPDATE orders
SET status = 'Processing'
WHERE orderNumber = 12345;

COMMIT;  -- บันทึกการเปลี่ยนแปลงทั้งหมด
-- หรือ ROLLBACK; -- ยกเลิกการเปลี่ยนแปลงทั้งหมด
```

### คำสั่งหลัก

| คำสั่ง | ความหมาย |
|--------|----------|
| `BEGIN;` | เริ่ม transaction |
| `COMMIT;` | บันทึกการเปลี่ยนแปลงถาวร |
| `ROLLBACK;` | ยกเลิกการเปลี่ยนแปลงทั้งหมด |

### ACID Properties

Transaction ต้องมีคุณสมบัติ ACID:
- **A**tomicity - ทำสำเร็จทั้งหมดหรือไม่ทำเลย
- **C**onsistency - ข้อมูลต้องถูกต้องตามกฎทั้งก่อนและหลัง
- **I**solation - Transaction แต่ละอันแยกกัน
- **D**urability - เมื่อ COMMIT แล้วข้อมูลคงอยู่ถาวร

### การใช้งาน

```sql
BEGIN;
-- ทำหลายคำสั่ง
-- หากมีข้อผิดพลาด
ROLLBACK;  -- ยกเลิกทั้งหมด

-- หากสำเร็จทั้งหมด
COMMIT;  -- บันทึก
```

**ตัวอย่างการใช้งาน**:
- การโอนเงินระหว่างบัญชี
- การสร้างออเดอร์พร้อมลดสต็อก
- การอัปเดตข้อมูลหลายตารางพร้อมกัน

---

## สรุป

### ลำดับการเรียนรู้แนะนำ

1. ✅ **SELECT, WHERE, ORDER BY, LIMIT** - พื้นฐาน
2. ✅ **Aggregate Functions, GROUP BY, HAVING** - การรวมข้อมูล
3. ✅ **JOIN** - เชื่อมตาราง
4. ✅ **Subquery** - SELECT ซ้อน
5. ✅ **Transaction** - ควบคุมการเปลี่ยนแปลง

### Tips สำคัญ

1. **ลำดับการทำงาน**: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
2. **WHERE vs HAVING**: WHERE กรองแถว, HAVING กรองกลุ่ม
3. **JOIN**: ใช้ ON เมื่อกำหนดเงื่อนไข, ใช้ USING เมื่อชื่อเหมือนกัน
4. **NULL**: ใช้ IS NULL / IS NOT NULL, ห้ามใช้ = NULL
5. **Aggregate Functions**: ต้องใช้ร่วมกับ GROUP BY ถ้าเลือกคอลัมน์อื่นด้วย

---

## แนะนำเพิ่มเติม

1. **ฝึกทำบ่อยๆ**: ลองเขียน query เอง อย่าแค่อ่าน
2. **ทดสอบทีละส่วน**: เขียนทีละขั้นตอน แล้วรันดูผล
3. **ใช้ EXPLAIN**: ดูแผนการทำงานของ query
4. **สร้าง Index**: เพิ่มประสิทธิภาพสำหรับคอลัมน์ที่ใช้บ่อย
5. **Transaction**: ใช้เมื่อต้องการความปลอดภัยของข้อมูล

---

**ไฟล์ก่อนหน้า**: [01-basic-operations.md](01-basic-operations.md) - Basic Operations (พีชคณิตเชิงสัมพันธ์)
