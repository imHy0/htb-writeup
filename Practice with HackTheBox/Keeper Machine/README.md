# Keeper Machine - Hack The Box

- Target IP Address: `10.10.11.227`

- Kiểm tra kết nối: `ping -c 2 10.10.11.227`

![Ping](../img/keeper/image.png)

## Enumeration

- sử dụng `nmap` để quét các cổng dịch vụ đang chạy: `nmap -sC -sV 10.10.11.227 -oA nmap.out`

![Nmap Scan](../img/keeper/image-1.png)

    --> Target mở port `22/open`, `80/open`

- Truy cập với địa chỉ `http://10.10.11.227`, ta nhận được link mới `tickets.keeper.htb`

![Web port 80](../img/keeper/image-2.png)

- Tuy nhiên trang web này không thể truy cập được, ta sẽ giải quyết nó bằng cách thêm vào `/etc/hosts`

![/etc/hosts](../img/keeper/image-3.png)

- Truy cập thành công, ta sẽ thấy trang `login`

![login page](../img/keeper/image-4.png)

- Tuy nhiên ta không có tài khoản để đăng nhập, và cũng không thấy chức năng đăng ký tài khoản, ta sẽ đi tìm tài khoản đăng nhập.

## Initial Access

- Phát hiện trang web sử dụng `RT 4.4.4+dfsg-2ubuntu1 (Debian) Copyright 1996-2019 Best Practical Solutions, LLC`, tìm kiếm một hồi trên google, ta phát hiện có tài khoản mặc định `root:password`

![find username and password](../img/keeper/image-5.png)

- Đăng nhập thành công, tìm một hồi các chức năng, ta phát hiện người dùng `lnorgaard` và email `lnorgaard@keeper.htb`

![file user](../img/keeper/image-6.png)

- Xem chi tiết thì ta thu thập thêm được thông tin về người dùng này như, `Language: Danish`, và đặc biệt là mật khẩu của tài khoản `Welcome2023!`

![find password](../img/keeper/image-7.png)

- Thực hiện `ssh` đối với tài khoản này ta thấy file `user.txt` --> Tìm thấy `User Flag`

![find user flag](../img/keeper/image-8.png)

## Privilege Escalation

- Kiểm tra quyền `sudo` --> không thể sử dụng

![sudo](../img/keeper/image-9.png)

- Ngoài `user.txt`, ta còn thấy có tệp zip `RT30000.zip`, giải nén ta được 2 file mới

![file zip](../img/keeper/image-10.png)

- Kiểm tra tệp:

![file type](../img/keeper/image-11.png)

- Do không thể dùng quyền `sudo`, nên ta sẽ tải 2 file này về máy của mình để chuẩn bị khai thác: `scp lnorgaard@keeper.htb:/home/lnorgaard/RT30000.zip ./`

![export](../img/keeper/image-12.png)

- Phát hiện sử dụng phiên bản `Keepass 2.0`, tìm kiếm thì phát hiện phiên bản này có lỗ hổng `CVE-2023-32784`

![CVE-2023-32784](../img/keeper/image-13.png)

- Tìm kiếm cách khai thác: https://github.com/dawnl3ss/CVE-2023-32784

![exploit](../img/keeper/image-14.png)

- Tiến hành khai thác: `python3 poc.py <PathToDmp>`

![dump](../img/keeper/image-15.png)

- Dựa vào Language ta đã tìm được là Danish, ta tìm thử thì nhận được chuỗi `rødgrød med fløde`

![keepass](../img/keeper/image-16.png)

- Truy cập trang web của keepass, mở file `passcode.kdbx` với pass vừa tìm được

![get content passcode](../img/keeper/image-17.png)

- Ta thấy có sử dụng `Putty`, ta sẽ dùng putty để lấy private key: `puttygen key.ppk -O private-openssh -o id_rsa`

![id_rsa](../img/keeper/image-18.png)

- Leo quyền thành công và lấy `Root Flag`: `ssh root@keeper.htb -i id_rsa`

![root flag](../img/keeper/image-19.png)