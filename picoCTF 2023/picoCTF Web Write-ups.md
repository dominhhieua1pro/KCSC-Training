# picoCTF Web

## Index
- [findme](#findme)
- [MatchTheRegex](#matchtheregex)
- [SOAP](#soap)
- [More SQLi](#more-sqli)
- [Java Code Analysis!?!](#java-code-analysis)


## findme

![image](https://user-images.githubusercontent.com/80137840/228012694-a34b20b5-35dd-48ab-9897-96cf3f15628f.png)

Truy cập trang web và login với account `test:test!`

![image](https://user-images.githubusercontent.com/80137840/227990147-4c531e9a-fcf6-46ae-93a8-21efab62a3ee.png)

Kiểm tra HTTP History thấy có 2 request redirect `GET /next-page/id=...`

![image](https://user-images.githubusercontent.com/80137840/227991555-1ca19fc1-f139-4967-bf3a-467b8b9bfea4.png)

Decode base64 2 giá trị id này rồi ghép lại thành flag

![image](https://user-images.githubusercontent.com/80137840/227992271-f7bf0579-e7ff-4c0d-9285-375d35bea39a.png)

![image](https://user-images.githubusercontent.com/80137840/227992335-395b306d-5d1a-425f-a751-78836de4a0de.png)

Flag: `picoCTF{proxies_all_the_way_25bbae9a}`


## MatchTheRegex

![image](https://user-images.githubusercontent.com/80137840/228012842-baaf0d1b-9ed9-4210-adcf-90893b2f5144.png)

Truy cập trang web 

![image](https://user-images.githubusercontent.com/80137840/227992966-821505d5-0fe8-4691-8481-723047c38b09.png)

Kiểm tra source code thấy có đoạn javascript xử lý input được nhập vào

![image](https://user-images.githubusercontent.com/80137840/227993701-b9ae25c1-beb5-4cbf-996b-cc1d9ae5fc22.png)

Trong đó có 1 dòng comment đoạn regex gợi ý là `^p.....F!?`, regex này chỉ định các chuỗi các ký tự có độ dài là 7 hoặc 8 và bắt đầu bằng ký tự "p", kế tiếp là 5 ký tự bất kỳ, và kết thúc bằng ký tự "F" hoặc "F!" do ký tự "?" được sử dụng để chỉ rằng ký tự "!" có thể có hoặc không xuất hiện ở cuối chuỗi. Nhập input `picoCTF` để nhận flag

![image](https://user-images.githubusercontent.com/80137840/227996085-2ff98cef-1312-40aa-8e05-52b1b1cc159f.png)

Flag: `picoCTF{succ3ssfully_matchtheregex_8ad436ed}`


## SOAP

![image](https://user-images.githubusercontent.com/80137840/228011532-2c30a8ac-07f5-4606-a11f-a638e6deea05.png)

Truy cập trang web

![image](https://user-images.githubusercontent.com/80137840/228010535-44a1d891-08f6-4d56-bca3-6037f9aabbd4.png)

Click Details của 1 trong 3 entries và kiểm tra request

![image](https://user-images.githubusercontent.com/80137840/228010893-b3b458cc-97cb-481e-b11b-6e2f97903bbf.png)

Thay ID thành abcxyz thì chuỗi này được reflected trong error response

![image](https://user-images.githubusercontent.com/80137840/228012115-d7c88578-6325-42a8-ba6c-c472aeabca9e.png)

Chức năng này bị lỗi XXE cơ bản, inject payload XXE cơ bản để đọc flag tại **/etc/passwd**

`<!DOCTYPE root [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>`

Và tham chiếu entity `test` trong thẻ ID

![image](https://user-images.githubusercontent.com/80137840/228012514-53dcea6a-7bff-47fa-8ebb-91c772919845.png)

Flag: `picoctf:x:1001:picoCTF{XML_3xtern@l_3nt1t1ty_4dbeb2ed}`


## More SQLi

![image](https://user-images.githubusercontent.com/80137840/228013475-ab9d30d3-e272-4f3f-8ee4-8478dfeb6fa4.png)

Theo hint của author, bài này sử dụng SQL Lite

Truy cập trang web

![image](https://user-images.githubusercontent.com/80137840/228014165-bf647296-76bd-494c-9737-ca76baf8817a.png)

Nhập thử username, password thì trang hiển thị lỗi với cả câu SQL query

![image](https://user-images.githubusercontent.com/80137840/228014752-28341668-0853-47df-89b5-1db7005a982e.png)

SQLi login đơn giản với password `' or 1=1-- -` và username bất kỳ

Trang web truy cập sau khi login 

![image](https://user-images.githubusercontent.com/80137840/228015346-858ca954-3ae1-456a-b8c3-10b354488226.png)

Sử dụng truy vấn order by ta tìm được 3 cột

Trích xuất các table và column được tạo trong csdl SQL lite với payload

`' UNION SELECT 111,sql,333 FROM sqlite_master WHERE type='table'-- -`

![image](https://user-images.githubusercontent.com/80137840/228016981-3e39cb50-9d74-46f2-bb00-e70341a12882.png)

Ta thấy có column `flag` trong table `more_table`. Trích xuất flag với payload

`' UNION SELECT 111,flag,333 FROM more_table-- -`

![image](https://user-images.githubusercontent.com/80137840/228017331-c4c65f44-a380-47b9-bd6a-94d158ae092b.png)

Flag: `picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_c8ee9477}`


## Java Code Analysis!?!

![image](https://user-images.githubusercontent.com/80137840/228018843-959ed4a9-1c69-4197-b70a-aef1fce73dd1.png)

Truy cập trang web và login account `user:user`

![image](https://user-images.githubusercontent.com/80137840/228018734-fa5da714-6742-47e8-9dcb-36ced22e0bcd.png)

Trang web sau khi login

![image](https://user-images.githubusercontent.com/80137840/228019053-6ef385be-e655-420b-90dc-48173a8690cd.png)

Chall yêu cầu read book **Flag**, nhưng book này cần quyền Admin nên ta cần đạt quyền Admin để read được book.

Kiểm tra HTTP history, ta thấy trang web sử dụng JWT để xác thực người dùng và id book chứa flag = 5

![image](https://user-images.githubusercontent.com/80137840/228022266-4c6eb171-b573-4aa9-92e7-a26086c2ab74.png)

Trong phần Payload có 1 claim **role** với giá trị Free và JWT sử dụng thuật toán HMAC-SHA256, ta thay đổi role thành **Admin**

![image](https://user-images.githubusercontent.com/80137840/228023012-6ce93d67-3ff5-4d25-bdc7-b2b3a21a823c.png)

Bây giờ ta cần tìm secret key để sign lại JWT. Sử dụng công cụ hashcat để brute-force secret key với wordlist `jwt.secrets.list` như sau:

`hashcat -a 0 -m 16500 jwt_token /path/to/jwt.secrets.list`

Với các options:
- -a 0: set attack mode Straight
- -m 16500: chỉ định hash type JWT (JSON Web Token)
- jwt_token: giá trị jwt trong header `Authorization: Bearer `
- /path/to/jwt.secrets.list: đường dẫn tới wordlist chứa danh sách các secret keys

![image](https://user-images.githubusercontent.com/80137840/228024574-1bd4138d-0bb6-4a44-8137-7c7a1ae97b29.png)

Nhận được secret key là 1234. Tiến hành generate New Symmetric Key với base64 của 1234 cho tham số **k** 

![image](https://user-images.githubusercontent.com/80137840/228025099-f701f3b1-3b7b-451c-b0f2-1bc914e9f648.png)

Sau đó sign lại JWT với Symmetric Key vừa generate

![image](https://user-images.githubusercontent.com/80137840/228025643-72498bee-cbe4-4cd9-9f15-3f47d7f195ea.png)

Trong source code, path để read book pdf tại `/base/books/pdf/{id}` với id cho book **Flag** = 5

![image](https://user-images.githubusercontent.com/80137840/228027646-1c489225-7287-4846-be76-0ed2e1855e4d.png)

![image](https://user-images.githubusercontent.com/80137840/228025954-374d68cb-ab63-4a96-8a12-96ddca5a6478.png)

Truy cập tới `/base/books/pdf/5` thì nhận được status 403 Access denied

![image](https://user-images.githubusercontent.com/80137840/228027774-d508048b-9861-4e83-b537-a68e4048054a.png)

Tức là chưa nhận được quyền Admin. Đọc qua hint thì author có nhắc đến cả 2 tham số role và userId => ta cần phải thay đổi cả 2 tham số này thì mới nhận được quyền Admin. Lúc trước ta đã thay đổi role thành Admin, vậy ta sẽ thử thay userId=2 thì truy cập thành công book **Flag**

![image](https://user-images.githubusercontent.com/80137840/228028747-531cbb18-81e2-41bc-927c-66851966cee5.png)

Và nhận được flag

![image](https://user-images.githubusercontent.com/80137840/228028864-650b4236-9664-4373-b417-ec6d0d9abd62.png)

Flag: `picoCTF{w34k_jwt_n0t_g00d_7745dc02}`



