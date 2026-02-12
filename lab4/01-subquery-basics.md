# Subquery Basics (Subquery พื้นฐาน)

## 1. โครงสร้างและลำดับการทำงานของ SQL

### โครงสร้างคำสั่ง SELECT แบบครบ

```sql
SELECT      -- เลือกคอลัมน์
FROM        -- ระบุตาราง
JOIN        -- เชื่อมตาราง
WHERE       -- กรองแถว
GROUP BY    -- จัดกลุ่ม
HAVING      -- กรองกลุ่ม
ORDER BY    -- เรียงผลลัพธ์
LIMIT       -- จำกัดจำนวน
```

### ⚡ ลำดับการประมวลผลจริง

SQL จะประมวลผลตามลำดับนี้:

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

**การจำ**: SQL ทำงานจาก "สร้างชุดข้อมูล" ก่อน แล้วค่อย "คัด/จัด/แสดงผล"

---

## 2. Subquery คืออะไร?

### ความหมาย

**Subquery** คือ query ซ้อนใน query อีกที  
อีกชื่อหนึ่งคือ **Inner Query** หรือ **Nested Query**

### เมื่อไร

ใช้ Subquery เมื่อ:
- ยังไม่รู้ค่าที่ต้องใช้เปรียบเทียบ
- ต้องเอาผลลัพธ์จาก query หนึ่งไปใช้เป็นเงื่อนไขของอีก query

### ตัวอย่างคลาสสิก

> **โจทย์**: หาคนที่คะแนนดีกว่านักเรียนรหัส V002 ทั้งที่ไม่รู้คะแนนของ V002
>
> **ทำไมต้อง Subquery?** เพราะต้องรู้คะแนน V002 ก่อน (subquery) แล้วค่อยใช้มาเปรียบเทียบ

```sql
SELECT * FROM student
WHERE score > (SELECT score FROM student WHERE id = 'V002');
```

### Subquery อยู่ที่ไหนได้บ้าง?

| ตำแหน่ง | ตัวอย่าง | ใช้เมื่อ |
|--------|---------|---------|
| **SELECT clause** | `SELECT (SELECT COUNT(*) ...) AS cnt` | คืนค่าเดี่ยวเป็นคอลัมน์ |
| **FROM clause** | `FROM (SELECT ... WHERE x) AS tbl` | สร้าง derived table |
| **WHERE clause** | `WHERE x = (SELECT y ...)` | กรองแถวด้วยเงื่อนไข |
| **HAVING clause** | `HAVING COUNT(*) > (SELECT ...)` | กรองกลุ่มหลัง GROUP BY |
| **JOIN clause** | `JOIN (SELECT ...) t1 ON ...` | เชื่อมตารางที่มาจาก subquery |

---

## 3. ประเภท Subquery

### แบ่งตาม "จำนวนแถว / จำนวนคอลัมน์"

![Subquery Types](./img/diagram.png)

---

## 3.1 Single-Row Subquery

### ความหมาย

- คืนค่า **1 แถว** และ **1 คอลัมน์**
- ใช้ได้กับตัวดำเนินการเปรียบเทียบทั่วไป: `=`, `>`, `<`, `>=`, `<=`, `<>`

### ตัวอย่าง

```sql
WHERE salary > (SELECT AVG(salary) FROM employees)
```

### ตัวอย่างที่ 1: Single-Row ใน WHERE

หา employee เดียวกับ "Diane Murphy"

```sql
SELECT
    firstname || ' ' || lastname AS fullname,
    employeenumber,
    email
FROM
    classicmodels.employees
WHERE
    employeenumber = (
        SELECT employeenumber
        FROM classicmodels.employees
        WHERE firstname || ' ' || lastname ILIKE 'diane murphy'
    );
```

**ผลลัพธ์**:

| fullname | employeenumber | email |
|----------|----------------|-------|
| Diane Murphy | 1086 | dmurphy@... |

![Result 1](./img/result_1.png)

### ตัวอย่างที่ 2: Single-Row ใน HAVING

หา order ที่มีปริมาณเฉลี่ยมากกว่า ปริมาณเฉลี่ยโดยรวม

```sql
SELECT o.orderNumber, AVG(o.quantityOrdered) AS avg_qty
FROM classicmodels.orderdetails o
GROUP BY o.orderNumber
HAVING AVG(o.quantityOrdered) > 
    (
        SELECT AVG(d.quantityOrdered) 
        FROM classicmodels.orderdetails AS d
    );
```

**ผลลัพธ์**:

![Result 4](img/result_4.png)

---

## 3.2 Scalar Subquery (1 Row, 1 Column)

### ความหมาย

- **1 แถว 1 คอลัมน์** เหมือน Single-Row Subquery
- เรียกว่า "Scalar" เพราะคืนค่าเดี่ยวเหมือน scalar value

### ตัวอย่าง

```sql
SELECT 
    customerName,
    (SELECT COUNT(*) FROM orders WHERE orders.customerNumber = customers.customerNumber) AS order_count
FROM customers;
```

**คำอธิบาย**:
- Subquery ในวงเล็บคืนค่าเดี่ยว (จำนวนอเดอร์) สำหรับแต่ละลูกค้า
- ใช้ **correlated subquery** (อ้างอิงค่าจากแถวของ outer query)

---

## 3.3 Multi-Column Subquery (1 Row, Multiple Columns)

### ความหมาย

- คืนค่า **1 แถว แต่หลายคอลัมน์**
- เช่น `(dept, salary)` หรือ `(firstname, lastname)`

### ตัวอย่าง

หา customer ที่มี sales rep และสาขา เดียวกับ employee รหัส 1370

```sql
SELECT 
    a.customernumber, 
    a.customername, 
    a.salesrepemployeenumber 
FROM 
    (SELECT e.employeenumber, o.officecode, c.customername, c.customernumber,
            c.salesrepemployeenumber 
     FROM classicmodels.employees e 
     LEFT JOIN classicmodels.offices o ON e.officecode = o.officecode 
     LEFT JOIN classicmodels.customers c ON c.salesrepemployeenumber = e.employeenumber) AS a
WHERE (a.salesrepemployeenumber, a.officecode) = 
    (SELECT e.employeenumber, o.officecode 
     FROM classicmodels.employees e 
     LEFT JOIN classicmodels.offices o ON e.officecode = o.officecode 
     WHERE e.employeenumber = 1370);
```

**Subquery ภายใน**:
```sql
-- สำหรับ employee 1370
SELECT e.employeenumber, o.officecode 
FROM classicmodels.employees e 
LEFT JOIN classicmodels.offices o ON e.officecode = o.officecode 
WHERE e.employeenumber = 1370;
```

**ผลลัพธ์**: `(1370, 4)` - 2 values

![Result 5](img/result_5.png)

**Output สุดท้าย**:

![Result 6](img/result_6.png)

---

## 3.4 Multi-Row Subquery

### ความหมาย

- คืนค่า **หลายแถว** (อาจ 1 คอลัมน์ หรือหลายคอลัมน์)
- ใช้ได้กับ: `IN`, `NOT IN`, `ANY`, `ALL`, `EXISTS`

### ตัวอย่าง 1: Multi-Row ใน FROM

หา orderdetails ที่มี quantityOrdered น้อยกว่า 10

```sql
SELECT quantityordered, ordernumber 
FROM (
    SELECT quantityordered, ordernumber 
    FROM classicmodels.orderdetails 
    WHERE quantityordered < 10
) AS small_orders;
```

**ผลลัพธ์**: หลายแถว

![Result small](img/result_3.png)

### ตัวอย่าง 2: Multi-Row ใน IN

หา order ที่ซื้อสินค้า 'S10_1678' แล้วแสดงรายการสินค้าทั้งหมดในใบ order นั้น

```sql
SELECT productCode, ordernumber
FROM classicmodels.orderdetails
WHERE ordernumber IN 
    (
        SELECT ordernumber 
        FROM classicmodels.orderdetails 
        WHERE productCode = 'S10_1678'
    );
```

**Subquery ภายใน** (หาหมายเลข order ที่มี S10_1678):
```sql
SELECT ordernumber 
FROM classicmodels.orderdetails 
WHERE productCode = 'S10_1678';
```

**ผลลัพธ์**: หลายค่า (หลาย order numbers)

![Result 8](img/result_8.png)

**Output สุดท้าย** (รายการสินค้าทั้งหมดในทุก order ดังกล่าว):

กรอบสีแดง = order ที่มี S10_1678  
ที่ไม่มีกรอบ = สินค้าอื่นในใบ order เดียวกัน

![Result 7](img/result_7.png)

---

## 4. Correlated Subquery

### ความหมาย

Subquery ที่อ้างอิงค่าจาก **outer query** (query ชั้นนอก)

ทำให้ subquery รันซ้ำสำหรับแต่ละแถวของ outer query

### ตัวอย่าง

หาการชำระเงินที่มียอดมากกว่า ค่าเฉลี่ยการชำระเงินของลูกค้าคนนั้น

```sql
SELECT p.customerNumber, p.checkNumber, p.amount
FROM classicmodels.payments p
WHERE p.amount > (
    SELECT AVG(p2.amount)
    FROM classicmodels.payments p2
    WHERE p2.customerNumber = p.customerNumber  -- ← อ้างอิงจาก p
);
```

**ความหมาย**:
- สำหรับแต่ละแถวใน p
- คำนวณค่าเฉลี่ยการชำระเงินของลูกค้านั้น
- เปรียบเทียบ p.amount กับค่าเฉลี่ยนั้น

---

## สรุป Subquery Types

| Type | Rows | Columns | ใช้ที่ | Operators |
|------|------|---------|-------|-----------|
| **Single-Row** | 1 | 1 | WHERE, HAVING, SELECT | `=`, `>`, `<`, etc |
| **Scalar** | 1 | 1 | SELECT (เป็นคอลัมน์) | `=`, `>`, `<`, etc |
| **Multi-Column** | 1+ | 2+ | WHERE, FROM | `IN`, `=` |
| **Multi-Row** | 2+ | 1+ | WHERE, FROM | `IN`, `ANY`, `ALL`, `EXISTS` |
| **Correlated** | 1 per outer row | 1+ | WHERE, HAVING, SELECT | varies |

---

## 📝 แนะนำเพิ่มเติม

### ข้อดีของ Subquery
✅ ตรรมชาติและอ่านง่าย  
✅ เหมาะกับการเปรียบเทียบค่า  
✅ สามารถใช้ซ้ำได้ในหลายที่  

### ข้อเสีย
❌ บางครั้งมีประสิทธิภาพต่ำกว่า JOIN  
❌ ถ้า subquery ซับซ้อนจะอ่านยากขึ้น  

### Tips
- ทดสอบ subquery แยกก่อน (รันแค่ subquery ดูผลลัพธ์)
- ใช้ alias เพื่อให้ชัดเจน
- หลีกเลี่ยงการใช้ `NOT IN` กับ NULL (ใช้ `NOT EXISTS` แทน)

---

**ไฟล์ต่อไป**: [02-subquery-advanced.md](02-subquery-advanced.md) - Subquery Advanced (IN, EXISTS, ANY, ALL)
