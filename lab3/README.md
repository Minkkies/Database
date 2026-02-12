## วิธีใช้ jupyter ในการ query
### run รอบเดียวเพื่อติดตั้งแพ็กเกจ
```sql
%pip install jupysql psycopg2-binary 
```
### code ที่ต้องมีเพื่อให้ใช้งาน jupyter ได้
-  Load SQL extension

    ```sql
    %load_ext sql
    %config SqlMagic.displaylimit = 300 #กำหนด row ที่จะแสดง
    ```

- เชื่อมต่อ postgresql
    ```sql
    %sql postgresql://postgres:12345678@localhost:5432/postgres 
    ```

- ดูว่าใช้งานได้ไหม
    ```sql
    %sql SELECT version() #ลองเช็ค version 
    ```

### ทดลอง run
```sql
%sql SET search_path TO classicmodels, public
```

```sql
%%sql
select * from orders  #ถ้าถูกจะได้ผลลัพธ์
```