# Local File Inclusion & File Upload Vuln Write-ups

## Index

[Local-File-Inclusion](#local-file-inclusion)

[Local-File-Inclusion-Double-encoding](#local-file-inclusion-double-encoding)

[File-upload-Double-extensions](#file-upload-double-extensions)

[File-upload-MIME-type](#file-upload-mime-type)

[File-upload-Null-byte](#file-upload-null-byte)

[File-upload-ZIP](#file-upload-zip)

[Level 24 - WebSec](#level-24---websec)


## Local-File-Inclusion

![image](https://user-images.githubusercontent.com/80137840/222891263-21c635f2-be28-4402-8d10-3e81271615a0.png)

Đề yêu cầu ta truy cập được vào trang admin để lấy password.

Truy cập vào bài lab, ta thấy có trang admin nhưng không truy cập được

![image](https://user-images.githubusercontent.com/80137840/222892316-99eec821-11e7-4547-aed7-0f9a43df6302.png)

Truy cập vào 1 số chức năng ta thấy tên chức năng được truyền vào param trên URL

![image](https://user-images.githubusercontent.com/80137840/222892257-cf80394d-0120-4c4a-9897-c0ea957e6890.png)

Sử dụng path traversal để duyệt lên thư mục cha thì ta thấy có thư mục **admin**

![image](https://user-images.githubusercontent.com/80137840/222892397-0d5dbe35-9a60-4144-86b8-2c2a7213d683.png)

Đi tới `../admin` thì thấy 1 file index.php, truy cập tiếp thì nhận được password của admin được khai báo trong source code

![image](https://user-images.githubusercontent.com/80137840/222892851-4992d98f-cc82-4f59-9523-04daf8799fca.png)

Submit password `OpbNJ60xYpvAQU8`

![image](https://user-images.githubusercontent.com/80137840/222893008-b72c6eff-6940-4e60-92e7-853b31ab9568.png)


## Local-File-Inclusion-Double-encoding

![image](https://user-images.githubusercontent.com/80137840/222894810-bbb5a389-acf0-4ce2-9b31-dbc76c5ab34b.png)

Bài này ta cần tìm password trong source files của website. Ngoài ra tên đề bài là `LFI - Double encoding` nên ta sẽ cần phải encode payload 2 lần trước khi send tới server.

Do cần tìm flag trong source code nên ta sẽ sử dụng PHP filters. Và ta có thể sử dụng payload `php://filter/convert.base64-encode/resource=home` để convert source nhận được thành base64. Tiến hành encode URL 2 lần rồi gán cho param `page` và send tới server:

Payload sau khi encode 2 lần: `php%253A%252F%252Ffilter%252Fconvert%252Ebase64-encode%252Fresource%253Dhome`

![image](https://user-images.githubusercontent.com/80137840/222895487-f618933a-817a-4e31-b4ae-d5318e40063e.png)

Nhận được 1 đoạn base64, decode ta nhận được source code:

```
<?php include("conf.inc.php"); ?>
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>J. Smith - Home</title>
  </head>
  <body>
    <?= $conf['global_style'] ?>
    <nav>
      <a href="index.php?page=home" class="active">Home</a>
      <a href="index.php?page=cv">CV</a>
      <a href="index.php?page=contact">Contact</a>
    </nav>
    <div id="main">
      <?= $conf['home'] ?>
    </div>
  </body>
</html>

```

Ta thấy rằng có 1 file `conf.inc.php` được include, tiến hành đọc file này bằng cách thay payload ở trên thành:

`php%253A%252F%252Ffilter%252Fconvert%252Ebase64-encode%252Fresource%253Dconf`

![image](https://user-images.githubusercontent.com/80137840/222895635-7c65264f-95d4-4378-a4fd-055d7c70a40b.png)

Decode base64 lần nữa nhận được flag:

![image](https://user-images.githubusercontent.com/80137840/222895659-cfbad1d1-eef1-40d2-8f39-927fe794ecda.png)

Submit flag: `Th1sIsTh3Fl4g!`

![image](https://user-images.githubusercontent.com/80137840/222895729-f3fd9070-ce72-4623-bb13-569483c8955e.png)


## File-upload-Double-extensions

![image](https://user-images.githubusercontent.com/80137840/222895754-eeda87f8-401f-4df2-85d9-c693fe2ceaf0.png)

Bài yêu cầu upload file .php để lấy password trong file .passwd ở thư mục root của trang web

Tạo 1 file php webshell như sau:

`<?php system($_GET[command]); ?>`

![image](https://user-images.githubusercontent.com/80137840/222900615-62cc363d-3a2b-40c9-a35c-02cfa6682976.png)

Nhận được response `Wrong file extension !`, tức là ta phải upload file có extension hợp lệ. Double extensions file thành `.php.jpg` thì upload thành công

![image](https://user-images.githubusercontent.com/80137840/222900641-6fde1529-e8a8-4e9c-9772-14160feb5c64.png)

Và file được upload ở đường dẫn: `./galerie/upload/be9b9221db0c706156915743327be19f/shell.php.jpg` 

Truy cập vào đường dẫn này với param `command=ls+-la` để liệt kê tất cả các file trong thư mục hiện tại

![image](https://user-images.githubusercontent.com/80137840/222900847-4444f60f-628c-44a6-9ed9-3b2b2e400faa.png)

Sử dụng path traversal để list các file trong mục cha, ta tìm thấy file .passwd trong path `../../../`. Tiến hành cat file này lấy flag

![image](https://user-images.githubusercontent.com/80137840/222900947-7f76d0a5-def0-4de5-bc88-f455199952e2.png)

Submit flag: `Gg9LRz-hWSxqqUKd77-_q-6G8`

![image](https://user-images.githubusercontent.com/80137840/222897348-5ad7b851-155d-49a1-8f3b-f50c9ff17651.png)


## File-upload-MIME-type

![image](https://user-images.githubusercontent.com/80137840/222898408-e84efc46-613a-4a57-867f-7358bd813eb3.png)

Bài này cũng yêu cầu upload file .php để lấy password trong file .passwd

Upload PHP webshell: `<?php system($_GET[command]); ?>`

![image](https://user-images.githubusercontent.com/80137840/222901030-2a4751d2-ed70-43e9-94cc-2bf0b536f352.png)

Nhận được response `Wrong file type !`, tức là ta phải upload file có Content-Type hợp lệ. Sửa Content-Type file thành `image/jpeg` thì upload thành công

![image](https://user-images.githubusercontent.com/80137840/222901046-a9f921f5-c186-409c-916a-7754d784fccd.png)

Đến đây ta cần tìm xem file được upload ở đường dẫn nào. Show response in browser và thấy file được upload tại `/web-serveur/ch21/galerie/upload/e1d637faecd349ab0abeecbc2c947a95//shell.php`

Thêm param `command` để tìm file .passwd như bài trước

![image](https://user-images.githubusercontent.com/80137840/222901335-19fa9127-37bf-4488-bbdf-46d088aa5af8.png)

Submit flag: `a7n4nizpgQgnPERy89uanf6T4`

![image](https://user-images.githubusercontent.com/80137840/222901357-9c6db268-6600-46e0-b96d-8c9bf02fb8d7.png)


## File-upload-Null-byte

![image](https://user-images.githubusercontent.com/80137840/222901392-be001754-5908-4e45-9e73-4e2303666f15.png)

Bài này cũng yêu cầu upload file .php

Upload PHP webshell: `<?php system($_GET[command]); ?>`

![image](https://user-images.githubusercontent.com/80137840/222902520-0fd7d910-05f7-4bf1-b839-85f8440628bd.png)

Nhận được response `Wrong file type !`. Sửa Content-Type file thành `image/jpeg` thì lại báo lỗi `Wrong extension !`

![image](https://user-images.githubusercontent.com/80137840/222902692-5e6d2cf1-bd07-469e-99bc-109598b720d6.png)

Double extensions file thành `.php.jpg` thì tiếp tục báo lỗi `Wrong file name !`, tức là tên file không hợp lệ. Vậy ta sẽ thêm byte null `%00` vào trước `.jpg` để bypass invalid filename nhưng extension vẫn hợp lệ.
> Khi server xử lý file `.php%00.jpg`, nó sẽ xử lý như một file ảnh jpeg nhưng nó sẽ bỏ qua phần sau của tên file (%00.jpg) và chỉ xem phần trước (.php), vì byte null thường được dùng làm ký tự kết thúc chuỗi (null-terminated string) trong các ngôn ngữ lập trình. Do đó server sẽ thực thi mã trong file này như một file PHP.

![image](https://user-images.githubusercontent.com/80137840/222903314-705bbb18-b477-438d-aa0d-ddc2c5e03125.png)

Đến đây ta truy cập vào đường dẫn file đã upload thì nhận được flag

![image](https://user-images.githubusercontent.com/80137840/222903454-2feefeee-660b-4b25-a9f0-0128ab4c9fd2.png)

Submit flag: `YPNchi2NmTwygr2dgCCF`

![image](https://user-images.githubusercontent.com/80137840/222903529-e4f89499-c3ca-4927-b4cb-07a096181958.png)


## File-upload-ZIP

![image](https://user-images.githubusercontent.com/80137840/222903716-95fbe9ae-fb86-4b8b-8cfa-ccee613afb9a.png)

Mục tiêu bài này là đọc được file index.php.

Chức năng upload file chỉ cho phép upload file zip. Thử nén file php webshell thành .zip rồi upload thì file được upload ở đường dẫn `/web-serveur/ch51/tmp/upload/64034d90c7a0f3.19399146/` và được giải nén

![image](https://user-images.githubusercontent.com/80137840/222906384-9cae33a6-9147-401c-9660-8c0f129323ef.png)

Nhưng khi truy cập vào file webshell thì không truy cập được

![image](https://user-images.githubusercontent.com/80137840/222906436-a8e0c4a9-f7d6-4991-87f2-cd1636cde2a0.png)

Thử nén zip và upload 1 số file khác như txt, jpg, png, docx thì truy cập được tới các file upload này.

Vậy thì để đọc được file index.php, ta sẽ tạo 1 symlink từ file txt tham chiếu tới file index.php sau đó nén zip lại và upload lên server.

> Cú pháp tạo symlink trong Linux: `ln -s /path/to/target /path/to/symlink`
>
> Cú pháp tạo symlink trong Windows: `mklink /path/to/symlink /path/to/target`

Ở trên ta thấy file upload được lưu ở `/web-serveur/ch51/tmp/upload/[random]/` nên khả năng file index.php được lưu ở `/web-serveur/ch51/`. Các bước tạo symlink và nén zip như sau:

```
$touch index.php
$mkdir -r ./test1/test2/test3/
$cd test1/test2/test3/
$ln -s ../../../index.php test.txt
$zip -y test.zip test.txt
```

Upload file test.zip và truy cập tới file test.txt được giải nén trên server thì nhận được flag

![image](https://user-images.githubusercontent.com/80137840/222907524-f54d0ed0-1729-4133-a8e7-a7572b0cb3bc.png)

Submit flag: `N3v3r_7rU5T_u5Er_1npU7`

![image](https://user-images.githubusercontent.com/80137840/222907551-e1e12586-ee21-4614-8c4b-68c1cc65c050.png)


## Level 24 - WebSec

Truy cập challenge, ta thấy có ô input nhập filename cùng source code

![image](https://user-images.githubusercontent.com/80137840/222907758-0c9b0faa-c784-4416-b9df-0071a5817c29.png)

```PHP
<?php
ini_set('display_errors', 'on');
ini_set('error_reporting', E_ALL);

session_start();

include 'clean_up.php';

/* periodic cleanup */
foreach (glob("./uploads/*") as $file) {
    if (is_file($file)) {
        unlink($file);
    } else {
        if (time() - filemtime($file) >= 60 * 60 * 24 * 7) {
            Delete($file);
        }
    }
}

$upload_dir = sprintf("./uploads/%s/", session_id());
@mkdir($upload_dir, 0755, true);

/* sandboxing ! */
chdir($upload_dir);
ini_set('open_basedir', '.');

$p = "list";
$data = "";
$filename = "";

if (isset($_GET['p']) && isset($_GET['filename']) ) {
    $filename = $_GET['filename'];
    if ($_GET['p'] === "edit") {
        $p = "edit";
        if (isset($_POST['data'])) {
            $data = $_POST['data'];
            if (strpos($data, '<?')  === false && stripos($data, 'script')  === false) {  # no interpretable code please.
                file_put_contents($_GET['filename'], $data);
                die ('<meta http-equiv="refresh" content="0; url=.">');
            }
        } elseif (file_exists($_GET['filename'])){
            $data = file_get_contents($_GET['filename']);
        }
    }
}
?>

<!DOCTYPE html>
<html>
<head>
        <title>#WebSec Level 24</title>
        <link rel="stylesheet" href="https://websec.fr/static/bootstrap.min.css" />
    <!-- Credits to blotus and bui for finding an unintended solution -->
</head>
    <body>
        <div class="container">
            <div class="row">
                <h1>Level TwentyFour<small> - Cloud-based plain-text storage</small></h1>
            </div>

            <div class="row">
                <p class="lead">
                Please enjoy our super-convenient cloud-based text storage facility,<br>
                    its sync-up scaled workflow connects with agile API worldwide on a global scale,<br>
                    to achieve synergy with outside the box diving devops.
                <br><br>
                    Another fine level by our very own <mark>nurfed</mark>, that you can check the source <a href="source.php">here</a>.
                </p>
            </div>

            <br>

            <div class="row">

<?php if ($p === "list") {
    echo "<div class='panel panel-default'>";
    echo "<div class='panel-heading'><h3 class='panel-title''>Index of <mark>$upload_dir</mark>:</h3></div>";
    echo "<div class='panel-body'><ul>";
    foreach (array_diff(scandir('.'), ['..', '.']) as $fn)
        echo "<li><a href='?p=edit&filename=$fn'>$fn</a></li>";
    echo "</ul></div></div>";
    ?>
    <form method="GET" class="form-inline">
        <div class="form-group">
            <input type="hidden" name="p" value="edit">
            <label for="filename">Create a new file:</label>
            <input class='form-control' type="text" name="filename" placeholder="file name">
        </div>
        <div class="form-group">
            <button type="submit" class='btn btn-default col-md-2 form-control' value="Submit">Create</button>
        </div>
    </form>

<?php } elseif ($p === "edit") {
    echo "<div class='panel panel-default'>";
    echo "<div class='panel-heading'><h3 class='panel-title''>";
    echo "File <mark>$filename</mark>'s content: <a type='button' class='close' href='.'><span>&times;</span></a></h3>";
    echo "</div>";
    echo "<div class='panel-body'>";
    echo "<form action='?p=edit&filename=$filename' method='post'>";
    echo "<input type='hidden' name='filename' value='$filename'>";
    echo "<textarea class='form-control' name='data' rows='6'>" . htmlentities($data) . "</textarea><br>";
    echo "<a type='button' class='btn btn-default' href='.'>Cancel</a> ";
    echo "<button type='submit' class='btn btn-default' value='Submit'>Save changes</button>";
    echo "</form>";
    echo "</div></div>";
} ?>
</div>
```

Trang web yêu cầu nhập filename để Create và sau đó nhập nội dung file rồi Save changes để send filename và data lên server. Tuy nhiên phía server kiểm tra rằng nếu không tìm thấy vị trí các chuỗi `<?` và `script` không phân biệt hoa thường, thì mới put data vào filename. Để bypass phép kiểm tra trên server này ta có thể encode base64 data gửi đi và khi truy cập vào file thì data mới được decode Để làm điều này ta sẽ sử dụng PHP filter `php://filter/convert.base64-decode/resource=filename` cho param filename với data file được encode base64. Khi đó, data khi put_contents vẫn ở dạng base64 nhưng khi truy cập tới file, nó sẽ được decode và thực thi mã PHP trên server.

Tuy nhiên, khi send data `<?php system("ls -la"); ?>` được encode base64 và truy cập tới đường dẫn file này thì response không nhận được nội dung gì

![image](https://user-images.githubusercontent.com/80137840/222916834-e43fd9bd-ff90-41a7-a5a5-2789724efe91.png)

![image](https://user-images.githubusercontent.com/80137840/222916846-59ba1cf3-9a8c-4156-be49-bab46c0f3426.png)

Tương tự với các hàm shell_exec(), exec(), có thể server không cho phép sử dụng những hàm thực thi lệnh trên hệ thống này. Vậy ta sẽ sử dụng hàm scandir() để liệt kê tất cả các file và thư mục có trong một đường dẫn cụ thể. Hàm này trả về một array các file và thư mục được liệt kê nên ta cần sử dụng hàm print_r() để hiển thị cấu trúc của array.

Scan tất cả các file và thư mục ở đường dẫn hiện tại: `<?php print_r(scandir('.'));  ?>
`
![image](https://user-images.githubusercontent.com/80137840/222917064-67aefa15-7627-46e5-80f4-9f1523727f23.png)

![image](https://user-images.githubusercontent.com/80137840/222917205-d8370d56-f6e7-4b58-8b3b-99b092ca43f2.png)

![image](https://user-images.githubusercontent.com/80137840/222917189-ed9372ac-41e3-4a21-950a-3db0b660fc65.png)

Scan các thư mục cha thì ở path `../../` thấy có file `flag.php`

![image](https://user-images.githubusercontent.com/80137840/222917539-459ad9f7-0986-4415-b665-ef209133770e.png)

Tiến hành đọc nội dung file này sử dụng hàm file_get_contents():

`<?php echo file_get_contents('../../flag.php'); ?>`

Và nhận được flag

![image](https://user-images.githubusercontent.com/80137840/222917382-64b1dcad-ef24-4810-9c84-91fd556cd957.png)

Submit flag: `WEBSEC{no_javascript_no_php_I_guess_you_used_COBOL_to_get_a_RCE_right?}`

![image](https://user-images.githubusercontent.com/80137840/222917631-d32b3c11-7da6-4642-ae9a-a3f6dc0cd699.png)

Done!!!
