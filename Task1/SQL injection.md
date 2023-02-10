# SQL injection 

## SQL injection (SQLi) là gì?

SQL injection (SQLi) là một lỗ hổng bảo mật web cho phép attacker can thiệp vào câu truy vấn mà ứng dụng thực hiện trong cơ sở dữ liệu. SQLi cho phép attacker xem những dữ liệu mà thông thường không thể truy xuất được như dữ liệu thuộc về người dùng khác, mật khẩu, thông tin thẻ tín dụng, thông tin người dùng cá nhân hoặc bất kỳ dữ liệu nhạy cảm nào mà ứng dụng có thể truy cập được. Trong nhiều trường hợp, attacker có thể sửa đổi hoặc xóa dữ liệu này, gây ra những thay đổi đối với nội dung hoặc hành vi của ứng dụng.

Trong một số trường hợp, attacker có thể leo thang một cuộc tấn công SQL injection để xâm phạm server hoặc cơ sở hạ tầng back-end khác hoặc thực hiện một cuộc tấn công từ chối dịch vụ (DoS).

![image](https://user-images.githubusercontent.com/80137840/217992678-894d3f32-0280-4182-b274-f27f8e469e0b.png)

## Nguyên nhân gây ra SQLi

- Dữ liệu đầu vào từ người dùng hoặc từ các nguồn khác không được kiểm tra hoặc kiểm tra không kỹ lưỡng.
- Ứng dụng sử dụng các câu lệnh SQL động, trong đó dữ liệu được kết nối với mã SQL gốc để tạo câu lệnh SQL hoàn chỉnh.

## Các dạng tấn công SQLi

### In-band SQLi (Classic SQLi)

Đây là dạng tấn công phổ biến nhất và cũng dễ để khai thác lỗ hổng SQL Injection nhất. Xảy ra khi attacker có thể tấn công và thu thập kết quả trực tiếp trên cùng một kênh liên lạc. 

In-Band SQLi chia làm 2 loại chính:

- Error-based SQLi

Là một kỹ thuật tấn công SQL Injection dựa vào thông báo lỗi được trả về từ Database Server có chứa thông tin về cấu trúc của cơ sở dữ liệu. Trong một số trường hợp, chỉ riêng việc Error-based SQLi là đủ để attacker liệt kê toàn bộ cơ sở dữ liệu.

- Union-based SQLi

Là một kỹ thuật SQLi tận dụng sức mạnh của toán tử UNION để kết hợp kết quả của hai hoặc nhiều câu lệnh SELECT thành một kết quả duy nhất, sau đó được trả về như một phần của HTTP response.

### Inferential SQL (Blind SQLi)

Không giống như In-band SQLi, Inferential SQL Injection (Blind SQLi) tốn nhiều thời gian hơn cho việc tấn công do không có bất kì dữ liệu nào được thực sự trả về thông qua ứng dụng web và attacker không thể xem kết quả trực tiếp như kiểu tấn công In-band. Thay vào đó, attacker sẽ cố gắng xây dựng lại cấu trúc cơ sở dữ liệu bằng việc gửi đi các payloads, dựa vào kết quả phản hồi của ứng dụng web và kết quả hành vi của database server.

Blind SQLi có 2 dạng tấn công chính:

- Boolean-based Blind SQLi

Là kĩ thuật tấn công SQL Injection dựa vào việc gửi các truy vấn tới cơ sở dữ liệu, buộc ứng dụng trả về các kết quả khác nhau phụ thuộc vào câu truy vấn là True hay False. Kiểu tấn công này thường chậm (đặc biệt với cơ sở dữ liệu có kích thước lớn) do attacker cần phải liệt kê từng dữ liệu, hoặc lần mò từng kí tự.

- Time-based Blind SQLi

Time-base Blind SQLi là kĩ thuật tấn công dựa vào việc gửi những câu truy vấn tới cơ sở dữ liệu và buộc cơ sở dữ liệu phải chờ một khoảng thời gian (thường tính bằng giây) trước khi phản hồi. Thời gian phản hồi (ngay lập tức hay delay theo khoảng thời gian được set) cho phép kẻ tấn công suy đoán kết quả truy vấn là True hay False. Kiểu tấn công này cũng tốn nhiều thời gian tương tự như Boolean-based SQLi.

### Out-of-band SQLi

Out-of-band SQLi không phải dạng tấn công phổ biến, chủ yếu là do nó phụ thuộc vào các tính năng được bật trên Database Server được sử dụng bởi ứng dụng Web. Kiểu tấn công này xảy ra khi attacker không thể tấn công và thu thập kết quả trực tiếp trên cùng một kênh (In-band SQLi), và đặc biệt là việc phản hồi từ server là không ổn định. Kiểu tấn công này phụ thuộc vào khả năng server thực hiện các DNS hoặc HTTP request để chuyển dữ liệu cho attacker.

Ví dụ như câu lệnh xp_dirtree trên Microsoft SQL Server có thể sử dụng để thực hiện DNS request tới một server khác do kẻ tấn công kiểm soát, hoặc Oracle Database’s UTL HTTP Package có thể sử dụng để gửi HTTP request từ SQL và PL/SQL tới server của kẻ tấn công.

## Payloads SQL injection Cheatsheet

[SQLi PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

[SQLi Payloads List](https://github.com/payloadbox/sql-injection-payload-list)

[SQLi Cheatsheet Portswigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)

[Advanced SQLi Cheatsheet](https://github.com/kleiton0x00/Advanced-SQL-Injection-Cheatsheet)

[SQL Injection Bypassing WAF](https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF)

## Phòng chống tấn công SQL injection

Hầu hết các trường hợp chèn SQL có thể được ngăn chặn bằng cách sử dụng truy vấn được tham số hóa (còn được gọi là prepared statements) thay vì nối chuỗi trong truy vấn.

Đoạn mã sau dễ bị tấn công bởi SQL injection vì input của người dùng được nối trực tiếp vào truy vấn:

```
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```

Đoạn mã này có thể được viết lại theo cách ngăn input của người dùng can thiệp vào cấu trúc truy vấn:

```
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

Truy vấn được tham số hóa có thể được sử dụng cho bất kỳ tình huống nào mà đầu vào không tin cậy xuất hiện dưới dạng dữ liệu trong truy vấn, bao gồm mệnh đề WHERE và giá trị trong câu lệnh INSERT hoặc UPDATE. Chúng không thể được sử dụng để xử lý đầu vào không tin cậy trong các phần khác của truy vấn, chẳng hạn như tên bảng hoặc cột hoặc mệnh đề ORDER BY.

Để một truy vấn được tham số hóa có hiệu quả trong việc ngăn chặn việc SQL injection, chuỗi được sử dụng trong truy vấn phải luôn là một hard-coded constant và không bao giờ được chứa bất kỳ dữ liệu có thể thay đổi nào từ bất kỳ nguồn gốc nào. Không nên cố gắng validate từng param xem dữ liệu có an toàn hay không và tiếp tục sử dụng nối chuỗi trong truy vấn cho các trường hợp được coi là an toàn. Tất cả đều rất dễ mắc lỗi về nguồn gốc có thể có của dữ liệu hoặc những thay đổi trong mã khác vi phạm các sự giả định về dữ liệu là độc hại.

Ngoài ra, có một số cách giúp giảm thiểu thiệt hại khi trang web bị tấn công SQL Injection đó là:
- Tránh sử dụng tài khoản quản trị như “root” hay “sa” để kết nối ứng dụng web với máy chủ cơ sở dữ liệu. Tốt nhất nên sử dụng tài khoản chỉ có quyền truy cập read - write đơn giản để vào từng cơ sở dữ liệu riêng biệt, trong trường hợp website bị tấn công SQL Injection, phạm vi thiệt hại vẫn nằm trong ranh giới của cơ sở dữ liệu đó.
- Mã hóa những dữ liệu nhạy cảm trong cơ sở dữ liệu bao gồm mật khẩu, câu hỏi và câu trả lời về bảo mật, dữ liệu tài chính, thông tin y tế và các thông tin khác có ích cho các tác nhân độc hại.
- Không lưu trữ dữ liệu nhạy cảm nếu không cần thiết, trừ khi thực sự cần. Và sau đó, xóa thông tin khi không còn sử dụng.
