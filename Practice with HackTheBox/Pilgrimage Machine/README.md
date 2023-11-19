# [Pilgrimage Machine - Hack The Box](https://app.hackthebox.com/machines/Pilgrimage)

- Target IP Address: `10.10.11.219`

- Kiểm tra kết nối: `ping -c 2 10.10.11.219`

![Ping](../img/pilgrimage/image.png)

## Enumeration

- sử dụng `nmap` để quét các cổng dịch vụ đang chạy: `nmap -sC -sV 10.10.11.219 -oA nmap.out`

![Nmap Scan](../img/pilgrimage/image-1.png)

    --> Target mở port `22/open`, `80/open`

- Truy cập với địa chỉ `http://10.10.11.219`, sẽ redirect sang trang `pilgrimage.htb`, tuy nhiên trang web này không thể truy cập được, ta sẽ giải quyết nó bằng cách thêm vào `/etc/hosts`

![/etc/hosts](../img/pilgrimage/image-2.png)

- Truy cập thành công, ta sẽ thấy trang web

![web page](../img/pilgrimage/image-3.png)


## Initial Access

- Ta chưa thấy thông tin gì để khai thác, sử dụng `dirsearch` ta phát hiện có thư mục `.git`

![.git](../img/pilgrimage/image-4.png)

- Sử dụng [GitHack](../img/pilgrimage/) để tải source code từ `.git` về

![download .git](../img/pilgrimage/image-5.png)

- Ta phát hiện trong file `index.php` có sử dụng `magick binary` để xử lý hình ảnh: `exec("/var/www/pilgrimage.htb/magick convert /var/www/pilgrimage.htb/tmp/" . $upload->getName() . $mime . " -resize 50% /var/www/pilgrimage.htb/shrunk/" . $newname . $mime);`

![magick binary](../img/pilgrimage/image-6.png)

- Phát hiện kiểu và phiên bản của `magick`

![magick version](../img/pilgrimage/image-7.png)

- Tìm kiếm thì ta phát hiện có lỗi `Arbitrary File Read`

    - https://www.exploit-db.com/exploits/51261

    - https://github.com/voidz0r/CVE-2022-44268

![magick vuln](../img/pilgrimage/image-8.png)

- Khai thác:

    ![cargo run "/etc/pasword](../img/pilgrimage/image-9.png)

    - Tải `image.png` lên web để shrink và download file về `output.png`

    ![shrink](../img/pilgrimage/image-10.png)

    - Sử dụng `exiftool output.png` ta sẽ nhận được đoạn mã hex

    ![hex](../img/pilgrimage/image-11.png)

    - Sử dụng [CyberChef](../img/pilgrimage/https://gchq.github.io/CyberChef/) để giải mã thì đây chính là nội dung của file `/etc/passwd`

    ![cyberchef](../img/pilgrimage/image-12.png)

- Ngoài ra trong `index.php`, ta thấy web sử dụng database `sqlite3` và được lưu trữ tại `/var/db/pilgrimage`.

![database](../img/pilgrimage/image-13.png)

- Khai thác tương tự như trên ta sẽ đọc được nội dung file này

![file sqlite](../img/pilgrimage/image-14.png)

- Download, sử dụng `sqlite` để mở file và ta sẽ đọc được thông tin về người dùng `emily:abigchonkyboi123`

![find user](../img/pilgrimage/image-15.png)

- Thực hiện `ssh` đối với tài khoản này ta thấy file `user.txt` --> Tìm thấy `User Flag`

![user flag](../img/pilgrimage/image-16.png)

## Privilege Escalation

- Kiểm tra quyền `sudo` --> không thể sử dụng

![sudo](../img/pilgrimage/image-17.png)

- Sử dụng `ps -aux` để giám sát các quy trình linux, ta tìm thấy một quy trình đáng ngờ `/bin/bash /usr/sbin/malwarescan.sh`

![malware](../img/pilgrimage/image-18.png)

- Đọc nội dung của file trên, ta thấy các tệp mới được tạo bằng `binwalk`, lưu trữ lại `/var/www/pilgrimage.htb/shrunk/` và tự động xóa các tệp phù hợp với tiêu chí xác định trong `blacklist`

![malware file content](../img/pilgrimage/image-19.png)

- Phát hiện phiên bản của `binwalk`

![Alt text](../img/pilgrimage/image-20.png)

- Tìm kiếm thì phát hiện phiên bản này có lỗi `RCE` có thể thực thi lỗi từ xa: https://www.exploit-db.com/exploits/51249

![Alt text](../img/pilgrimage/image-21.png)

- Sử dụng code trong link trên lưu thành file `rce.py` để tiến hành khai thác ta sẽ được file `binwalk_exploit.png`, ở đây ta lựa chọn port `8000`

![Alt text](../img/pilgrimage/image-22.png)

- Tạo server để `emily` có thể tải file độc hại vừa tạo

![Alt text](../img/pilgrimage/image-23.png)

- Tải file và up lên đường dẫn binwalk tạo ở trên

![Alt text](../img/pilgrimage/image-24.png)

- Lắng nghe ở cổng `8000`, ta thấy đã có connect, và đã khai thác `RCE` thành công --> lấy `Root Flag`

![Alt text](../img/pilgrimage/image-25.png)