# Basic Operations (การดำเนินการพื้นฐานใน SQL)

## ตารางเปรียบเทียบ พีชคณิตเชิงสัมพันธ์ vs SQL

| พีชคณิตเชิงสัมพันธ์   | สัญลักษณ์ | ความหมาย                                            | ตัวอย่างใน SQL                                   |
| --------------------- | --------- | --------------------------------------------------- | ------------------------------------------------ |
| **Select**            | σ         | เลือกแถว (tuple) ตามเงื่อนไข                        | `SELECT * FROM student WHERE age > 20;`          |
| **Project**           | ∏         | เลือกเฉพาะคอลัมน์ที่ต้องการ                         | `SELECT name, age FROM student;`                 |
| **Union**             | ∪         | รวมผลลัพธ์จาก 2 ตาราง (ไม่ซ้ำ)                      | `SELECT col FROM A UNION SELECT col FROM B;`     |
| **Set Difference**    | −         | แสดงข้อมูลที่อยู่ในตารางแรก แต่ไม่อยู่ในตารางที่สอง | `SELECT col FROM A EXCEPT SELECT col FROM B;`    |
| **Cartesian Product** | ×         | จับคู่ทุกแถวของตารางหนึ่งกับทุกแถวของอีกตาราง       | `SELECT * FROM A CROSS JOIN B;`                  |
| **Rename**            | ρ         | เปลี่ยนชื่อตารางหรือคอลัมน์                         | `SELECT name AS student_name FROM student;`      |
| **Intersection**      | ∩         | แสดงข้อมูลที่อยู่เหมือนกันทั้งสองตาราง              | `SELECT col FROM A INTERSECT SELECT col FROM B;` |
| **Join**              | ⨝         | เชื่อมตารางตั้งแต่ 2 ตารางขึ้นไปตามเงื่อนไข        | `SELECT * FROM student s JOIN dept d ON s.majorid = d.did;` |

---

## ใน lab นี้เราทำอะไร
- **ส่วนที่ 1**: ทดสอบ Basic Operations (Select, Project, Union, Intersection, Set Difference, Rename, Cartesian Product)
- **ส่วนที่ 2**: ลงลึก SQL DML และการจัดการข้อมูลขั้นสูง (WHERE, JOIN, Aggregate Functions, Subquery, Transaction)

---

## ส่วนที่ 1: Basic Operations (โดย อ.อภิรักษ์)

### 1. Select – เลือกแถว (tuple) ตามเงื่อนไข

**เลือกทั้งหมด:**
```sql
SET search_path TO demo;
SELECT * FROM student;
```
ผลลัพธ์: จะได้ข้อมูลทั้งหมดของนักเรียน

![Select ทั้งหมด](img/image.png)

**เลือกตามเงื่อนไข:**
```sql
SET search_path TO demo;
SELECT * FROM student WHERE dept = 'CS';
```
ผลลัพธ์: จะได้เฉพาะนักเรียนแผนก CS

![Select with WHERE](img/image-1.png)

---

### 2. Project – เลือกเฉพาะคอลัมน์ที่ต้องการ

**เลือกคอลัมน์:**
```sql
SET search_path TO demo;
SELECT sname, dept FROM student;
```
ผลลัพธ์: จะได้เฉพาะคอลัมน์ sname และ dept

![Project](img/image-2.png)

**เลือกคอลัมน์พร้อม Select:**
```sql
SET search_path TO demo;
SELECT sname FROM student WHERE dept = 'CS';
```
ผลลัพธ์: จะได้เฉพาะชื่อนักเรียนแผนก CS

![Project with Select](img/image-3.png)

---

### 3. Union – รวมผลลัพธ์จาก 2 ตาราง (ไม่ซ้ำ)

Union ใช้รวมผลลัพธ์จากสองคำสั่ง SELECT โดยจะตัดข้อมูลที่ซ้ำออก

```sql
SET search_path TO demo;
SELECT sname FROM student WHERE dept = 'CS'
UNION
SELECT sname FROM student WHERE dept = 'Math';
```
ผลลัพธ์: รวมชื่อนักเรียนจากแผนก CS และ Math โดยไม่ซ้ำ

![Union](img/image-4.png)

---

### 4. Intersection – แสดงข้อมูลที่อยู่เหมือนกันทั้งสองตาราง

Intersection แสดงเฉพาะข้อมูลที่ปรากฏในทั้งสองผลลัพธ์

```sql
SET search_path TO demo;
SELECT sid FROM student WHERE dept = 'CS'       -- output {1,2}
INTERSECT
SELECT sid FROM enroll WHERE cid = 'C1';        -- output {1,2,3}
```
ผลลัพธ์: แสดงเฉพาะ sid ที่อยู่ทั้งในแผนก CS และลงทะเบียนวิชา C1

![Intersection](img/image-5.png)

---

### 5. Set Difference – แสดงข้อมูลที่อยู่ในตารางแรก แต่ไม่อยู่ในตารางที่สอง

Set Difference (EXCEPT) แสดงข้อมูลจากตารางแรกที่ไม่ปรากฏในตารางที่สอง

```sql
SET search_path TO demo;
SELECT sid FROM enroll WHERE cid = 'C1'
EXCEPT
SELECT sid FROM student WHERE dept = 'CS';
```
ผลลัพธ์: แสดง sid ที่ลงทะเบียน C1 แต่ไม่ได้อยู่ในแผนก CS

![Set Difference](img/image-6.png)

---

### 6. Rename – เปลี่ยนชื่อตารางหรือคอลัมน์

**Rename ตารางโดยใช้ AS:**
```sql
SET search_path TO demo;
SELECT sid, sname FROM student AS S;
```
ผลลัพธ์ (ก่อน Rename):

![Rename before](img/image-7.png)

ผลลัพธ์ (หลัง Rename):

![Rename after](img/image-8.png)

**Rename คอลัมน์:**
```sql
SET search_path TO demo;
SELECT sid, sname AS name, dept FROM student;
```
ผลลัพธ์: คอลัมน์ sname จะแสดงเป็น "name" แทน

![Rename column](img/image-7.png)

---

### 7. Cartesian Product – จับคู่ทุกแถวของตารางหนึ่งกับทุกแถวของอีกตาราง

Cartesian Product (CROSS JOIN) สร้างคู่จากทุกแถวของทั้งสองตาราง ผลลัพธ์จะมี n × m แถว

```sql
SET search_path TO demo;
SELECT * FROM course AS c1
CROSS JOIN course AS c2;
```
ผลลัพธ์: จะได้ทุกคู่ของแถวจากตาราง course

![Cartesian Product](img/image-9.png)

หมายเหตุ: ถ้า course มี 5 แถว ผลลัพธ์จะมี 5 × 5 = 25 แถว

---

## ส่วนที่ 2: SQL DML ขั้นสูง (โดย อ.กฤษ)

**อ้างอิง:**
- [เอกสารจารย์](https://luckkrit.github.io/cos3103/slides/2_68/sql_dml/)
- [ดาวน์โหลด PostgreSQL](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)

---

### 2.1 SELECT ขั้นสูง

#### 2.1.1 DISTINCT – ตัดค่าที่ซ้ำกันออก

DISTINCT ใช้เลือกเฉพาะค่าที่ไม่ซ้ำกัน (unique)

```sql
SELECT DISTINCT jobtitle FROM classicmodels.employees;
```
ผลลัพธ์: แสดงตำแหน่งงานที่ไม่ซ้ำกัน

#### 2.1.2 คำนวนและ LIMIT

```sql
SELECT checkNumber, paymentDate, amount + 10000 AS new_amount
FROM classicmodels.payments
LIMIT 5;  -- เลือกเพียง 5 แถว
```
ผลลัพธ์: แสดงหมายเลขเช็ค วันที่ชำระ และจำนวนเงินบวกเพิ่มเติม 10000

#### 2.1.3 ฟังก์ชันสตริง

**UPPER – เปลี่ยนเป็นตัวพิมพ์ใหญ่:**
```sql
SELECT customername, UPPER(customername) AS upper_name
FROM classicmodels.customers;
```
ผลลัพธ์: แสดงชื่อลูกค้าและชื่อที่เป็นตัวพิมพ์ใหญ่

**LOWER – เปลี่ยนเป็นตัวพิมพ์เล็ก:**
```sql
SELECT customername, LOWER(customername) AS lower_name
FROM classicmodels.customers;
```

**REVERSE – กลับตัวอักษร:**
```sql
SELECT customername, REVERSE(customername)
FROM classicmodels.customers;
```

**CONCAT – ต่อข้อความ:**
```sql
SELECT CONCAT(firstname, ' ', lastname) AS fullname
FROM employees;
```

**RPAD – เติมทางขวา:**
```sql
SELECT customername, RPAD(customername, 30, '-') AS padded
FROM classicmodels.customers;
```
ผลลัพธ์: ชื่อลูกค้าจะมีความยาว 30 อักษร เติมด้วย '-' ทางขวา

**LPAD – เติมทางซ้าย:**
```sql
SELECT customername, LPAD(customername, 30, '*') AS lpadded
FROM classicmodels.customers;
```

#### 2.1.4 Boolean Expression

```sql
SELECT creditLimit, creditLimit > 100000 AS is_high_credit
FROM classicmodels.customers
LIMIT 10;
```
ผลลัพธ์: แสดงวงเงินและค่า TRUE/FALSE บ่งชี้ว่ามากกว่า 100000 หรือไม่

---

### 2.2 CAST – แปลงชนิดข้อมูล

CAST ใช้เปลี่ยนชนิดข้อมูล (data type) จากชนิดหนึ่งเป็นอีกชนิดหนึ่ง

```sql
SELECT CAST(officeCode AS INTEGER) AS office_code_int
FROM employees;
```

**หรือใช้สัญลักษณ์ ::**
```sql
SELECT officeCode::INTEGER FROM employees;
```

ผลลัพธ์: officeCode จะถูกแปลงเป็นตัวเลขจำนวนเต็ม

---

### 2.3 WHERE และตัวดำเนินการ

#### 2.3.1 Alias และ Multiple Conditions

```sql
SELECT E.firstName AS fname, E.officeCode AS ocode
FROM employees AS E
WHERE (E.officeCode)::INTEGER > 3 
  AND (E.officeCode)::INTEGER <= 5;
```
ผลลัพธ์: แสดงชื่อพนักงานและรหัสสำนักที่อยู่ในช่วง 3-5
![WHERE with Alias](img/image-10.png)

#### 2.3.2 ตัวดำเนินการเปรียบเทียบ

- `=` เท่ากับ
- `<>` ไม่เท่ากับ
- `<` น้อยกว่า
- `>` มากกว่า
- `<=` น้อยกว่าหรือเท่ากับ
- `>=` มากกว่าหรือเท่ากับ

#### 2.3.3 LIKE – ค้นหาแบบแพทเทิร์น

**% – แทนหลายอักษร (case-insensitive):**
```sql
-- ขึ้นต้นด้วย '1'
SELECT * FROM employees WHERE officeCode LIKE '1%';

-- ลงท้ายด้วย 'A'
SELECT * FROM customers WHERE city LIKE '%A';

-- มี 'A' อยู่ในคอลัมน์
SELECT * FROM employees WHERE lastName LIKE '%A%';
```

**_ – แทนอักษรเดียว:**
```sql
SELECT firstName, lastName
FROM employees
WHERE birthDate LIKE '195_-__-__';
```
ผลลัพธ์: พนักงานที่เกิดปี 195X

**NOT LIKE:**
```sql
SELECT employeeNumber, lastName, firstName
FROM employees
WHERE lastName NOT ILIKE 'B%';  -- ILIKE = case-insensitive LIKE
```
ผลลัพธ์: แสดงพนักงานที่นามสกุลไม่ขึ้นต้นด้วย B (ไม่สำคัญตัวใหญ่-เล็ก)

#### 2.3.4 IS NULL / IS NOT NULL

```sql
SELECT customername, state
FROM customers
WHERE state IS NULL;
```
ผลลัพธ์: แสดงลูกค้าที่ไม่มีข้อมูลจังหวัด

```sql
SELECT customername, state
FROM customers
WHERE state IS NOT NULL;
```
ผลลัพธ์: แสดงลูกค้าที่มีข้อมูลจังหวัด

#### 2.3.5 BETWEEN – ช่วงค่า

```sql
SELECT *
FROM classicmodels.orderdetails
WHERE priceEach BETWEEN 150 AND 200;
```
ผลลัพธ์: แสดงรายการสั่งซื้อที่ราคาอยู่ระหว่าง 150-200

#### 2.3.6 IN / NOT IN

```sql
SELECT contactLastName, salesRepEmployeeNumber
FROM classicmodels.customers
WHERE salesRepEmployeeNumber IN (1370, 1501, 1504);
```
ผลลัพธ์: แสดงลูกค้าที่สัมพันธ์กับพนักงานหมายเลข 1370, 1501 หรือ 1504

```sql
SELECT *
FROM products
WHERE productCode NOT IN ('S10_1678', 'S10_1949');
```
ผลลัพธ์: แสดงสินค้าที่ไม่ใช่รหัสดังกล่าว

---

### 2.4 Aggregate Functions – ฟังก์ชันการรวม

Aggregate Functions ใช้รวมและประมวลผลข้อมูลหลายแถว

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

**ฟังก์ชันที่สำคัญ:**
- `COUNT(*)` นับทั้งหมด
- `COUNT(col)` นับที่ไม่ใช่ NULL
- `SUM(col)` รวมค่า
- `AVG(col)` เฉลี่ย
- `MAX(col)` ค่าสูงสุด
- `MIN(col)` ค่าต่ำสุด

---

### 2.5 GROUP BY และ HAVING

คำสั่งสำหรับจัดกลุ่ม (GROUP BY), กรองกลุ่ม (HAVING) และเรียงลำดับ (ORDER BY) ในขั้นตอนหลังการเลือกข้อมูล

```sql
SELECT      <attribute list>
FROM        <table list>
[JOIN ON    <condition>]
[WHERE      <condition>]
[GROUP BY   <grouping attribute(s)>]
[HAVING     <group condition>]
[ORDER BY   <attribute list>];
```

#### ORDER BY – จัดเรียงผลลัพธ์
- เรียงลำดับผลลัพธ์ (ค่าเริ่มต้น ASC น้อย-มาก) หรือใช้ `DESC`/`RANDOM()` เพื่อสลับทิศทางหรือสุ่ม

```sql
SELECT contactLastname, contactFirstname
FROM classicmodels.customers
-- ORDER BY RANDOM()  -- หากต้องการสุ่ม
ORDER BY contactLastname;
```
**result**
- จะเห็นว่าเรียงลำดับตามนามสกุล

![alt text](img/image010.png)


#### GROUP BY – จัดกลุ่มข้อมูล
```sql
SELECT status, COUNT(*) AS total_orders
FROM classicmodels.orders
GROUP BY status;
```
**result**
- จะเห็นว่ามีการจับกลุ่ม(GROUP BY) และนำจำนวน(COUNT)

![alt text](img/image-10.png)

#### HAVING – กรองหลัง GROUP BY
ใช้เมื่อกรองผลลัพธ์ที่จัดกลุ่มแล้ว โดยเฉพาะเมื่อเกี่ยวข้องกับ aggregate functions เช่น `SUM()`, `COUNT()`, `AVG()`

คำสั่ง | ทำงานเมื่อไร | GROUP BY |
---|---|---|
`WHERE` | กรองข้อมูลแต่ละแถว (ทำงานก่อนจัดกลุ่ม) | ก่อน `GROUP BY`
`HAVING` | กรองข้อมูลที่จัดกลุ่มแล้ว | หลัง `GROUP BY`

**ตัวอย่าง**
```sql
-- หาลูกค้าที่สั่งซื้อมากกว่า 5 ครั้ง แล้วเรียงจำนวนมากไปน้อย
SELECT customerNumber, COUNT(*) AS total_orders
FROM classicmodels.orders
GROUP BY customerNumber
HAVING COUNT(*) > 5
ORDER BY total_orders DESC;
```
**Result**

![alt text](img/image-20.png)

---

### 2.6 ORDER BY และ LIMIT (เพิ่มเติม)

ORDER BY ใช้เรียงลำดับและสามารถทำงานร่วมกับ LIMIT เพื่อตัดจำนวนผลลัพธ์

```sql
SELECT orderNumber, orderDate, totalAmount
FROM orders
ORDER BY orderDate DESC
LIMIT 10 OFFSET 0;  -- OFFSET ข้ามแถวแรก 0 แถว
```
ผลลัพธ์: แสดง 10 อเดอร์ล่าสุด

```sql
SELECT * FROM products
ORDER BY quantityInStock ASC
LIMIT 5;
```
ผลลัพธ์: แสดง 5 สินค้าที่เหลือในสต็อกน้อยที่สุด

---

### 2.7 JOIN – เชื่อมตาราง

JOIN ใช้เชื่อมข้อมูลจากหลายตาราง

| JOIN             | ความหมาย                                         | แสดงข้อมูลจากตารางซ้าย              | แสดงข้อมูลจากตารางขวา               | เงื่อนไขการจับคู่                       |
| ---------------- | ------------------------------------------------ | ----------------------------------- | ----------------------------------- | --------------------------------------- |
| **JOIN**         | ค่าเริ่มต้นของ JOIN (เหมือน INNER JOIN)          | ✔️ เฉพาะที่ match                   | ✔️ เฉพาะที่ match                   | ต้องตรงตามเงื่อนไข                      |
| **INNER JOIN**   | แสดงเฉพาะแถวที่มีข้อมูลตรงกันทั้งสองตาราง        | ✔️                                  | ✔️                                  | ใช้ ```ON``` หรือ ```USING``` ระบุเงื่อนไข                     |
| **LEFT JOIN**    | แสดงทุกแถวจากตารางซ้าย และเฉพาะที่ตรงจากตารางขวา | ✔️ ทั้งหมด                          | ✔️ เฉพาะที่ match (ไม่ตรงเป็น NULL) | ใช้ ```Left  outer    JOIN  orders o  ON  A.id = B.id```                                  |
| **RIGHT JOIN**   | แสดงทุกแถวจากตารางขวา และเฉพาะที่ตรงจากตารางซ้าย | ✔️ เฉพาะที่ match (ไม่ตรงเป็น NULL) | ✔️ ทั้งหมด                          | ใช้ ```RIGHT  outer    JOIN  orders o  ON  A.id = B.id```                                  |
| **SELF JOIN**    | Join ตารางเดียวกันกับตัวเอง                      | ✔️                                  | ✔️                                  | ใช้ ```alias ```แยกตาราง                      |
| **NATURAL JOIN** | Join โดยอัตโนมัติจากชื่อคอลัมน์ที่เหมือนกัน      | ✔️                                  | ✔️                                  | ใช้ ```natural join``` |

หมายเหตุ
- `JOIN` เฉยๆ = `INNER JOIN`
- ใช้ `LEFT/RIGHT JOIN` เมื่ออยากเก็บฝั่งที่ไม่มีคู่
- `NATURAL JOIN` สะดวกแต่เสี่ยงจับคู่คอลัมน์ผิดโดยไม่ตั้งใจ

**ON vs USING**
- ใช้ `ON` เมื่อกำหนดเงื่อนไขใดก็ได้ ไม่จำเป็นต้องมีชื่อคอลัมน์ตรงกัน
- ใช้ `USING(col)` เมื่อคอลัมน์ชื่อเดียวกัน และต้องการผลลัพธ์คอลัมน์นั้นเพียงชุดเดียว

**ตัวอย่างหลัก**
```sql
SELECT c.customerName, o.orderNumber, o.orderDate
FROM customers c
JOIN orders o ON c.customerNumber = o.customerNumber;
```
ผลลัพธ์: แสดงชื่อลูกค้าและหมายเลขอเดอร์สำหรับลูกค้าที่มีอเดอร์

**LEFT JOIN:**
```sql
SELECT c.customerName, COUNT(o.orderNumber) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName;
```
ผลลัพธ์: แสดงลูกค้าทั้งหมด รวมทั้งที่ไม่มีอเดอร์ (จำนวน 0)

**CROSS JOIN:**
```sql
SELECT * FROM customers CROSS JOIN products LIMIT 5;
```
ผลลัพธ์: แสดงคู่ของลูกค้า-สินค้า (Cartesian Product)

**USING Clause (ทำให้โค้ดสั้นลง):**
```sql
SELECT c.customerName, o.orderNumber
FROM customers c
JOIN orders o USING (customerNumber);
```

**NATURAL JOIN vs INNER JOIN**
- หากไม่มีคอลัมน์ชื่อเหมือนกัน: `INNER JOIN` จะไม่ join และแจ้ง error, ส่วน `NATURAL JOIN` จะกลายเป็น Cartesian product ได้

**COUNT() สั้นๆ**
ฟังก์ชัน `COUNT()` ใช้นับจำนวนแถว เช่น
```sql
SELECT COUNT(*)
FROM classicmodels.orders;
```

---

### 2.8 ลำดับการทำงานของ SQL

```
FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

**ตัวอย่าง:**
```sql
SELECT department, COUNT(*) AS emp_count, AVG(salary) AS avg_sal
FROM employees
WHERE salary > 50000
GROUP BY department
HAVING COUNT(*) >= 3
ORDER BY avg_sal DESC
LIMIT 5;
```

1. **FROM** เลือกตารางจาก employees
2. **WHERE** กรองเฉพาะพนักงานที่เงินเดือน > 50000
3. **GROUP BY** จัดกลุ่มตามแผนก
4. **HAVING** กรองเฉพาะแผนกที่มีพนักงาน >= 3 คน
5. **SELECT** เลือกแผนก นับจำนวน และค่าเฉลี่ยเงินเดือน
6. **ORDER BY** เรียงตามเงินเดือนเฉลี่ยมากไปน้อย
7. **LIMIT** แสดง 5 แผนกแรก

---

### 2.9 Subquery – คำสั่ง SELECT ซ้อน

Subquery (Inner Query) ใช้ผลลัพธ์จากคำสั่ง SELECT หนึ่ง เป็นเงื่อนไขของอีกคำสั่ง

**Subquery in WHERE with IN:**
```sql
SELECT customerName
FROM customers
WHERE customerNumber IN (
    SELECT customerNumber FROM orders
    WHERE YEAR(orderDate) = 2023
);
```
ผลลัพธ์: แสดงลูกค้าที่มีอเดอร์ในปี 2023

**Subquery with EXISTS:**
```sql
SELECT customerName
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customerNumber = c.customerNumber
      AND YEAR(o.orderDate) = 2023
);
```
ผลลัพธ์: เช่นเดียวกัน แต่ใช้ EXISTS ซึ่งมักมีประสิทธิภาพดีกว่า

**Scalar Subquery:**
```sql
SELECT customerName, (
    SELECT COUNT(*)
    FROM orders
    WHERE orders.customerNumber = customers.customerNumber
) AS order_count
FROM customers;
```
ผลลัพธ์: แสดงชื่อลูกค้าพร้อมจำนวนอเดอร์ของแต่ละคน

---

### 2.10 COALESCE – จัดการ NULL

COALESCE แสดงค่าแรกที่ไม่ใช่ NULL

```sql
SELECT customerName, COALESCE(state, 'N/A') AS state_or_na
FROM customers;
```
ผลลัพธ์: แสดงจังหวัดของลูกค้า หากเป็น NULL จะแสดง 'N/A' แทน

---

### 2.11 UNION, INTERSECT, EXCEPT

**UNION – รวมและตัดซ้ำ:**
```sql
(SELECT city FROM customers WHERE country = 'USA')
UNION
(SELECT city FROM customers WHERE country = 'France');
```
ผลลัพธ์: เมืองทั้งหมดจาก USA และ France (ไม่ซ้ำ)

**INTERSECT – เฉพาะที่อยู่ในทั้งสอง:**
```sql
(SELECT city FROM customers WHERE country = 'USA')
INTERSECT
(SELECT city FROM orders WHERE orderDate > '2023-01-01');
```

**EXCEPT – อยู่ในแรกแต่ไม่ในที่สอง:**
```sql
(SELECT city FROM customers WHERE country = 'USA')
EXCEPT
(SELECT city FROM customers WHERE state IS NULL);
```

---

### 2.12 Transaction – ธุรกรรมข้อมูล

Transaction ใช้ควบคุมการเปลี่ยนแปลงข้อมูลให้ปลอดภัย

```sql
BEGIN;
UPDATE products
SET quantityInStock = quantityInStock - 1
WHERE productCode = 'S10_1678';

UPDATE orders
SET status = 'Processing'
WHERE orderNumber = 12345;

COMMIT;  -- บันทึกการเปลี่ยนแปลง
-- หรือ ROLLBACK; -- ยกเลิกการเปลี่ยนแปลง
```

**คำสั่งหลัก:**
- `BEGIN;` เริ่ม transaction
- `COMMIT;` บันทึกการเปลี่ยนแปลง
- `ROLLBACK;` ยกเลิกการเปลี่ยนแปลง
---