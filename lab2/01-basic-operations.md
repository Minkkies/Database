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

## วัตถุประสงค์

ในส่วนนี้เราจะทดสอบ Basic Operations ทั้ง 7 แบบ:
1. **Select** - เลือกแถวตามเงื่อนไข
2. **Project** - เลือกเฉพาะคอลัมน์
3. **Union** - รวมผลลัพธ์
4. **Intersection** - ข้อมูลที่ซ้ำกัน
5. **Set Difference** - ข้อมูลที่ต่างกัน
6. **Rename** - เปลี่ยนชื่อ
7. **Cartesian Product** - จับคู่ทุกแถว

---

## การตั้งค่าเริ่มต้น

ก่อนเริ่มการทดสอบ ให้ตั้งค่า search path ไปยัง schema `demo`:

```sql
CREATE SCHEMA IF NOT EXISTS demo;
SET search_path TO demo;
```

---

## 1. Select – เลือกแถว (tuple) ตามเงื่อนไข

Select (σ) ใช้เลือกแถวที่ตรงตามเงื่อนไขที่กำหนด

### เลือกทั้งหมด

```sql
SET search_path TO demo;
SELECT * FROM student;
```

**ผลลัพธ์**: จะได้ข้อมูลทั้งหมดของนักเรียน

![Select ทั้งหมด](img/image.png)

### เลือกตามเงื่อนไข

```sql
SET search_path TO demo;
SELECT * FROM student WHERE dept = 'CS';
```

**ผลลัพธ์**: จะได้เฉพาะนักเรียนแผนก CS

![Select with WHERE](img/image-1.png)

---

## 2. Project – เลือกเฉพาะคอลัมน์ที่ต้องการ

Project (∏) ใช้เลือกเฉพาะคอลัมน์ที่ต้องการจากตาราง

### เลือกคอลัมน์

```sql
SET search_path TO demo;
SELECT sname, dept FROM student;
```

**ผลลัพธ์**: จะได้เฉพาะคอลัมน์ sname และ dept

![Project](img/image-2.png)

### เลือกคอลัมน์พร้อม Select

```sql
SET search_path TO demo;
SELECT sname FROM student WHERE dept = 'CS';
```

**ผลลัพธ์**: จะได้เฉพาะชื่อนักเรียนแผนก CS

![Project with Select](img/image-3.png)

---

## 3. Union – รวมผลลัพธ์จาก 2 ตาราง (ไม่ซ้ำ)

Union (∪) ใช้รวมผลลัพธ์จากสองคำสั่ง SELECT โดยจะตัดข้อมูลที่ซ้ำออก

```sql
SET search_path TO demo;
SELECT sname FROM student WHERE dept = 'CS'
UNION
SELECT sname FROM student WHERE dept = 'Math';
```

**ผลลัพธ์**: รวมชื่อนักเรียนจากแผนก CS และ Math โดยไม่ซ้ำ

![Union](img/image-4.png)

**หมายเหตุ**: ถ้าต้องการเก็บข้อมูลซ้ำไว้ ให้ใช้ `UNION ALL` แทน

---

## 4. Intersection – แสดงข้อมูลที่อยู่เหมือนกันทั้งสองตาราง

Intersection (∩) แสดงเฉพาะข้อมูลที่ปรากฏในทั้งสองผลลัพธ์

```sql
SET search_path TO demo;
SELECT sid FROM student WHERE dept = 'CS'       -- output {1,2}
INTERSECT
SELECT sid FROM enroll WHERE cid = 'C1';        -- output {1,2,3}
```

**ผลลัพธ์**: แสดงเฉพาะ sid ที่อยู่ทั้งในแผนก CS และลงทะเบียนวิชา C1

![Intersection](img/image-5.png)

**การใช้งาน**: เหมาะสำหรับหาข้อมูลที่ตรงตามหลายเงื่อนไขพร้อมกัน

---

## 5. Set Difference – แสดงข้อมูลที่อยู่ในตารางแรก แต่ไม่อยู่ในตารางที่สอง

Set Difference (−) แสดงข้อมูลจากตารางแรกที่ไม่ปรากฏในตารางที่สอง ใน SQL ใช้คำสั่ง `EXCEPT`

```sql
SET search_path TO demo;
SELECT sid FROM enroll WHERE cid = 'C1'
EXCEPT
SELECT sid FROM student WHERE dept = 'CS';
```

**ผลลัพธ์**: แสดง sid ที่ลงทะเบียน C1 แต่ไม่ได้อยู่ในแผนก CS

![Set Difference](img/image-6.png)

**การใช้งาน**: เหมาะสำหรับหาข้อมูลที่ตรงเงื่อนไขแรก แต่ไม่ตรงเงื่อนไขที่สอง

---

## 6. Rename – เปลี่ยนชื่อตารางหรือคอลัมน์

Rename (ρ) ใช้เปลี่ยนชื่อตารางหรือคอลัมน์เพื่อให้อ่านง่ายขึ้นหรือหลีกเลี่ยงชื่อซ้ำ

### Rename ตารางโดยใช้ AS

```sql
SET search_path TO demo;
SELECT sid, sname FROM student AS S;
```

**ผลลัพธ์**: ตาราง student จะถูกเรียกว่า S ภายในคำสั่งนี้

### Rename คอลัมน์

```sql
SET search_path TO demo;
SELECT sid, sname AS name, dept FROM student;
```

**ผลลัพธ์**: คอลัมน์ sname จะแสดงเป็น "name" แทน

![Rename column](img/image-7.png)

**การใช้งาน**: 
- ทำให้ผลลัพธ์อ่านง่ายขึ้น
- จำเป็นเมื่อ JOIN ตารางเดียวกันกับตัวเอง (SELF JOIN)

---

## 7. Cartesian Product – จับคู่ทุกแถวของตารางหนึ่งกับทุกแถวของอีกตาราง

Cartesian Product (×) สร้างคู่จากทุกแถวของทั้งสองตาราง ผลลัพธ์จะมี n × m แถว

```sql
SET search_path TO demo;
SELECT * FROM course AS c1
CROSS JOIN course AS c2;
```

**ผลลัพธ์**: จะได้ทุกคู่ของแถวจากตาราง course

![Cartesian Product](img/image-9.png)

**หมายเหตุ**: 
- ถ้า course มี 5 แถว ผลลัพธ์จะมี 5 × 5 = 25 แถว
- ระวังการใช้งานกับตารางใหญ่ เพราะผลลัพธ์จะมีจำนวนมากมาก
- มักใช้ร่วมกับ WHERE เพื่อกรองผลลัพธ์

---

## สรุป

| Operation | SQL Keyword | ลักษณะผลลัพธ์ |
|-----------|-------------|---------------|
| Select | WHERE | กรองแถว |
| Project | SELECT columns | เลือกคอลัมน์ |
| Union | UNION | รวมไม่ซ้ำ |
| Intersection | INTERSECT | เหมือนกันทั้งสอง |
| Set Difference | EXCEPT | อยู่แรกไม่อยู่สอง |
| Rename | AS | เปลี่ยนชื่อ |
| Cartesian Product | CROSS JOIN | ทุกคู่ที่เป็นไปได้ |

---

**ไฟล์ต่อไป**: [02-sql-dml-advanced.md](02-sql-dml-advanced.md) - SQL DML ขั้นสูง
