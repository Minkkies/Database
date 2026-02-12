# Referential Integrity Constraints (ข้อจำกัดความถูกต้องเชิงอ้างอิง)

Referential Integrity Constraints คือข้อกำหนดในฐานข้อมูลเชิงสัมพันธ์ที่ควบคุมความถูกต้องของความสัมพันธ์ระหว่างตาราง โดยอาศัย Primary Key (PK) และ Foreign Key (FK) เพื่อป้องกัน “ค่าอ้างอิงลอย” (orphan records)

- Primary Key (PK): คีย์หลักของตารางแม่ (Parent) ที่มีความเป็นเอกลักษณ์และไม่เป็น NULL
- Foreign Key (FK): คอลัมน์ในตารางลูก (Child) ที่อ้างอิงไปยัง PK ของตารางแม่

ระบบจะกำหนดพฤติกรรมเมื่อมีการ:
- DELETE แถวในตารางแม่ (Parent)
- UPDATE ค่า PK ในตารางแม่ (ส่งผลต่อ FK ในตารางลูก)

พฤติกรรมมาตรฐานของ FK ที่ใช้บ่อย:
- NO ACTION (ค่าเริ่มต้นใน PostgreSQL): ปฏิเสธคำสั่งถ้ายังมีแถวลูกอ้างอิงอยู่
- SET NULL: ตั้งค่า FK ในตารางลูกเป็น NULL (คอลัมน์ FK ต้องรับค่า NULL ได้)
- CASCADE: ลบหรืออัปเดตข้อมูลในตารางลูกตามตารางแม่
- SET DEFAULT: ตั้งค่า FK เป็นค่า DEFAULT (ถ้ามี) — ใน lab นี้ไม่ได้สาธิต

หมายเหตุ (PostgreSQL):
- NO ACTION และ RESTRICT คล้ายกันในผลลัพธ์ (ต่างกันที่เวลาเช็ก) แต่ทั้งคู่จะ “ปฏิเสธ” ถ้ายังมีการอ้างอิงอยู่
- ควรมีดัชนี (index) บนคอลัมน์ FK เพื่อประสิทธิภาพที่ดีขึ้นเมื่อ JOIN/DELETE/UPDATE

---

## โครงเรื่องของ lab นี้
สร้างตารางที่มีความสัมพันธ์กัน แล้วลองกำหนด Constraint 3 แบบ
1) NO ACTION 2) SET NULL 3) CASCADE  
พร้อมลอง DELETE/UPDATE ฝั่งแม่และลูก เพื่อดูผลลัพธ์ที่ต่างกัน

โครงสร้างตาราง:
- dept = ตารางแม่ (Parent) มี PK = did
- student = ตารางลูก (Child) มี FK = majorid อ้างอิง dept(did)

ภาพรวมที่ใช้ใน lab (รูปตัวอย่างคงไว้ตามเดิมในโฟลเดอร์ img/)

---

## เตรียมสคีมาและข้อมูลเริ่มต้น (ครั้งแรก)
ใช้ PostgreSQL

```sql
-- แนะนำให้รีเซ็ตก่อนเริ่มแต่ละส่วน
CREATE SCHEMA IF NOT EXISTS demo;
SET search_path TO demo;

-- ลบตารางหากมีอยู่เดิม (รวมถึงข้อจำกัดที่พึ่งพา)
DROP TABLE IF EXISTS student CASCADE;
DROP TABLE IF EXISTS dept CASCADE;

-- ตารางแม่
CREATE TABLE dept (
    did   INTEGER PRIMARY KEY,
    dname VARCHAR(30) NOT NULL
);

INSERT INTO dept (did, dname) VALUES
(10, 'compsci'),
(20, 'math'),
(30, 'drama');

-- ตารางลูก (เริ่มต้นแบบทั่วไป)
CREATE TABLE student (
    sid      INTEGER PRIMARY KEY,
    sname    VARCHAR(30) NOT NULL,
    gradyear INTEGER,           -- อนุญาตให้เป็น NULL ในส่วนแรก
    majorid  INTEGER NOT NULL,

    CONSTRAINT chk_gradyear_gt_2000 CHECK (gradyear > 2000),

    /* Constraint = ข้อจำกัด/กฎเกณฑ์ */
    CONSTRAINT fk_student_dept 
        FOREIGN KEY (majorid)
        REFERENCES dept(did)
);

INSERT INTO student (sid, sname, gradyear, majorid) VALUES
(1, 'joe', 2004, 10),
(2, 'amy', 2004, 20),
(3, 'sue', 2005, 20),
(4, 'bob', 2005, 20),
(5, 'kim', 2003, 30),  -- 2003 > 2000 ผ่าน CHECK
(6, 'amy', 2001, 20),  -- 2001 > 2000 ผ่าน CHECK
(7, 'art', 2004, 30),
(8, 'pat', 2004, 20),
(9, 'lee', 2004, 10);
```

---

## ส่วนที่ 1: NO ACTION
แนวคิด: ถ้ามีแถวลูกอ้างอิงอยู่ จะ “ไม่อนุญาต” ให้ลบ/อัปเดตแถวแม่ที่เกี่ยวข้อง และไม่อนุญาตให้ลูกอัปเดต FK เป็นค่าที่ไม่มีในแม่  
ใช้เมื่อ: ต้องการป้องกันการลบข้อมูลสำคัญโดยไม่ได้ตั้งใจ

เพิ่ม NO ACTION ให้ FK

```sql
SET search_path TO demo;

/* ALTER TABLE ใช้เมื่อต้องการเปลี่ยนโครงสร้างตารางหลังสร้างไปแล้ว */
ALTER TABLE student
    DROP CONSTRAINT fk_student_dept;

ALTER TABLE student
    ADD CONSTRAINT fk_student_dept
    FOREIGN KEY (majorid)
    REFERENCES dept(did)
    ON DELETE NO ACTION
    ON UPDATE NO ACTION;
```

### ทดสอบที่ตาราง Student
1) ลบแถวลูก: ลบ sid = 5,7 ได้ (ไม่ติด FK ไปแม่)
```sql
DELETE FROM student WHERE sid IN (5,7);
SELECT * FROM student;
```
**ผลลัพธ์ (ลบในลูกสำเร็จ)**

![no action – ลบลูก](img/no_action1.png)

2) อัปเดต FK ไปค่าที่ไม่มีในแม่ (sid=1 → majorid=40): ล้มเหลว
```sql
UPDATE student SET majorid = 40 WHERE sid = 1;
-- ERROR: Key (majorid)=(40) is not present in table "dept".
```

3) อัปเดต FK ไปค่าที่มีในแม่ (sid=1 → majorid=20): สำเร็จ
```sql
UPDATE student SET majorid = 20 WHERE sid = 1;
SELECT * FROM student;
```
![no action – อัปเดตลูกสำเร็จ](img/no_action2.png)

### ทดสอบที่ตาราง Dept
1) ลบ did = 10: ล้มเหลว (ยังมีลูกอ้างอิงอยู่)
```sql
DELETE FROM dept WHERE did = 10;
/*
ERROR: update or delete on table "dept" violates foreign key constraint "fk_student_dept" on table "student"
Key (did)=(10) is still referenced from table "student".
*/
```
วิธีแก้:
- ลบ/ตัดความสัมพันธ์ฝั่งลูกที่อ้าง did=10 ให้หมดก่อน แล้วค่อยลบแม่
- หรือเปลี่ยน constraint เป็น CASCADE/SET NULL ตามความต้องการทางธุรกิจ

2) อัปเดต did 30 → 31: สำเร็จ (เพราะไม่มีลูกอ้าง did=30 แล้วจากที่ลบ sid 5,7)
```sql
UPDATE dept SET did = 31 WHERE did = 30;
SELECT * FROM dept;
```
![no action – อัปเดตแม่สำเร็จ](img/no_action3.png)

3) อัปเดต did 20 → 21: ล้มเหลว (ยังมีลูกอ้างอิงอยู่)
```sql
UPDATE dept SET did = 21 WHERE did = 20; -- ล้มเหลว
```

4) ลองลบ did 31 และ 20
```sql
-- DELETE FROM dept WHERE did = 31;  -- ลบได้ (ไม่มีลูกอ้าง)
-- DELETE FROM dept WHERE did = 21;  -- ไม่มีแถว (จากข้อ 3 อัปเดตไม่สำเร็จ)
-- DELETE FROM dept WHERE did = 20;  -- ลบไม่ได้ (มีลูกอ้าง)
SELECT * FROM student;
```
สรุป: ถ้า did นั้นไม่ถูกอ้างอิง ลบ/อัปเดตได้ตามปกติ  
- student ไม่เปลี่ยน (เมื่อไม่ยุ่งกับ FK)

![no action – สถานะ student](img/no_action2.png)

- dept did 31 ถูกลบออกได้เมื่อไม่ถูกอ้าง

![no action – ลบ did 31](img/no_action4.png)

---

## ส่วนที่ 2: SET NULL
แนวคิด: เมื่อแม่ถูกลบหรือเปลี่ยน PK → FK ในลูกจะถูกตั้งเป็น NULL อัตโนมัติ  
ใช้เมื่อ: ต้องการเก็บข้อมูลลูกไว้ แต่ยกเลิกการผูกกับแม่

รีเซ็ตตารางและสร้างใหม่ (majorid ต้องอนุญาตให้เป็น NULL)

```sql
SET search_path TO demo;

DROP TABLE IF EXISTS student CASCADE;
DROP TABLE IF EXISTS dept CASCADE;

CREATE TABLE dept (
    did   INTEGER PRIMARY KEY,
    dname VARCHAR(30) NOT NULL
);

INSERT INTO dept (did, dname) VALUES
(10, 'compsci'),
(20, 'math'),
(30, 'drama');

CREATE TABLE student (
    sid      INTEGER PRIMARY KEY,
    sname    VARCHAR(30) NOT NULL,
    gradyear INTEGER NOT NULL,
    majorid  INTEGER,  -- อนุญาต NULL

    CONSTRAINT chk_gradyear_gt_2000 CHECK (gradyear > 2000),

    CONSTRAINT fk_student_dept
        FOREIGN KEY (majorid)
        REFERENCES dept(did)
        ON DELETE SET NULL
        ON UPDATE SET NULL
);

INSERT INTO student (sid, sname, gradyear, majorid) VALUES
(1, 'joe', 2004, 10),
(2, 'amy', 2004, 20),
(3, 'sue', 2005, 20),
(4, 'bob', 2005, 20),
(5, 'kim', 2003, 30),
(6, 'amy', 2001, 20),
(7, 'art', 2004, 30),
(8, 'pat', 2004, 20),
(9, 'lee', 2004, 10);
```

### ทดสอบที่ตาราง Student
- ลบ sid = 5,7 → ลบได้เหมือน NO ACTION  
- อัปเดต sid=1 → majorid=40 → ล้มเหลวเหมือนเดิม (ค่า 40 ไม่มีในแม่)  
- อัปเดต sid=1 → majorid=20 → สำเร็จ

### ทดสอบที่ตาราง Dept
1) ลบ did = 10 → แถวลูกที่อ้าง 10 (sid 1 และ 9) จะถูกตั้งเป็น NULL
```sql
DELETE FROM dept WHERE did = 10;
SELECT * FROM dept;     -- ไม่มี did=10 แล้ว
SELECT * FROM student;  -- แถวที่เคย majorid=10 กลายเป็น NULL
```
ภาพตัวอย่างผลลัพธ์:

![set null – dept หลังลบ did=10](img/set_null1.png)

![set null – student majorid จาก 10 → NULL](img/set_null2.png)

2) อัปเดต did 30 → 31 → สำเร็จ (ลูกที่อ้าง 30 จะถูกตั้งเป็น NULL ตามพฤติกรรม ON UPDATE SET NULL)
```sql
UPDATE dept SET did = 31 WHERE did = 30;
SELECT * FROM dept;
```

3) อัปเดต did 20 → 21 → แถวลูกที่อ้าง 20 ทั้งหมดจะถูกตั้งเป็น NULL
```sql
UPDATE dept SET did = 21 WHERE did = 20;
SELECT * FROM student;
```
ภาพตัวอย่างผลลัพธ์:

![set null – dept ถูกอัปเดตเป็น 21 และ 31](img/set_null3.png)

![set null – student ที่เคยอ้าง 20 กลายเป็น NULL](img/set_null4.png)

4) ลองลบ did 31 และ 21 → ลบได้เพราะไม่มีลูกอ้างแล้ว (ลูกถูกตั้งเป็น NULL ไปก่อนหน้า)
```sql
DELETE FROM dept WHERE did = 31;
DELETE FROM dept WHERE did = 21;
SELECT * FROM dept;
SELECT * FROM student;
```
ภาพตัวอย่างผลลัพธ์:

![set null – dept ถูกลบหมด](img/set_null5.png)

สรุป SET NULL:
- ลบ/อัปเดตแม่ → ลูกยังคงอยู่ แต่ยกเลิกการผูกความสัมพันธ์ (FK = NULL)
- ต้องเปิดให้คอลัมน์ FK รับ NULL

---

## ส่วนที่ 3: CASCADE
แนวคิด: เมื่อแม่ถูกลบ/อัปเดต → ลูกจะถูกลบ/อัปเดตตามทันที  
ใช้เมื่อ: ข้อมูลลูกไม่มีความหมายหากไม่มีข้อมูลแม่

รีเซ็ตตารางและสร้างใหม่

```sql
SET search_path TO demo;

DROP TABLE IF EXISTS student CASCADE;
DROP TABLE IF EXISTS dept CASCADE;

CREATE TABLE dept (
    did   INTEGER PRIMARY KEY,
    dname VARCHAR(30) NOT NULL
);

INSERT INTO dept (did, dname) VALUES
(10, 'compsci'),
(20, 'math'),
(30, 'drama');


CREATE TABLE student (
    sid      INTEGER PRIMARY KEY,
    sname    VARCHAR(30) NOT NULL,
    gradyear INTEGER NOT NULL,
    majorid  INTEGER NOT NULL,

    CONSTRAINT chk_gradyear_gt_2000 CHECK (gradyear > 2000),

    CONSTRAINT fk_student_dept
        FOREIGN KEY (majorid)
        REFERENCES dept(did)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);


INSERT INTO student (sid, sname, gradyear, majorid) VALUES
(1, 'joe', 2004, 10),
(2, 'amy', 2004, 20),
(3, 'sue', 2005, 20),
(4, 'bob', 2005, 20),
(5, 'kim', 2003, 30),
(6, 'amy', 2001, 20),
(7, 'art', 2004, 30),
(8, 'pat', 2004, 20),
(9, 'lee', 2004, 10);
```
### ทดสอบที่ตาราง Student
1) ลบ sid = 5,7
```sql
DELETE FROM student WHERE sid IN (5,7);
SELECT * FROM student;
```
**ผลลัพธ์**: Delete ได้เหมือนตอน no action

---

2) Try update sid = 1 with majorid = 40
```sql
UPDATE student SET majorid = 40 WHERE sid = 1;
-- ERROR: Key (majorid)=(40) is not present in table "dept".
```
**ผลลัพธ์**: update ไม่ได้เพราะเหตุผลเดิม

---

3) Try update sid = 1 with majorid = 20
```sql
UPDATE student SET majorid = 20 WHERE sid = 1;
SELECT * FROM student;
```
**ผลลัพธ์**: update ได้เหมือนตอน no action

---

### ทดสอบที่ตาราง Dept
1) Delete: Did = 10
```sql
DELETE FROM dept WHERE did = 10;
SELECT * FROM dept;
SELECT * FROM student;
```
**ผลลัพธ์**
- dept did 10 ถูกลบออกไป 
- student ที่มี majorid=10 ถูกลบออกไปด้วย (joe และ lee หายไป)

![cascade – student หลังลบแผนก 10](img/cascade1.png)

---

2) Update: Did 30 → 31
```sql
UPDATE dept SET did = 31 WHERE did = 30;
SELECT * FROM dept;
```
**ผลลัพธ์**: did ถูก update เป็น 31

---

3) Update: Did 20 → 21
```sql
UPDATE dept SET did = 21 WHERE did = 20;
SELECT * FROM dept;
SELECT * FROM student;
```
**ผลลัพธ์**
- dept อัปเดต did จาก 20 → 21
- student ที่มี majorid=20 จะถูกอัปเดตเป็น 21 ตาม dept

![cascade – student อัปเดตตาม](img/cascade2.png)

---

4) Try delete did 31, 21
```sql
DELETE FROM dept WHERE did = 31;
DELETE FROM dept WHERE did = 21;
SELECT * FROM dept;
SELECT * FROM student;
```
**ผลลัพธ์**
- dept ถูก delete 2 record จนหมด
- student ถูกลบออกไปตาม dept

![cascade – student ถูกลบตาม](img/cascade3.png)

สรุป CASCADE:
- ลบแม่ → ลบลูกที่อ้างอิงทันที
- อัปเดตแม่ → ลูกอัปเดต FK ตามทันที

---

## คำแนะนำเพิ่มเติม
- ตั้งชื่อข้อจำกัดให้สื่อความหมาย เช่น fk_student_dept เพื่อสะดวกเวลา DROP/ALTER
- ใช้ DROP TABLE ... CASCADE เมื่อมี FK/VIEW/OBJECT อื่นพึ่งพา
- สร้างดัชนีบนคอลัมน์ FK เพื่อช่วย JOIN และบำรุงรักษา FK
```sql
CREATE INDEX IF NOT EXISTS idx_student_majorid ON student(majorid);
```
- ทำงานในทรานแซกชันเมื่อทดสอบหลายขั้นตอน จะได้ ROLLBACK ได้หากผิดพลาด
```sql
BEGIN;
/* คำสั่งทดสอบจำนวนมาก */
ROLLBACK; -- หรือ COMMIT เมื่อผลลัพธ์ถูกต้อง
```
---