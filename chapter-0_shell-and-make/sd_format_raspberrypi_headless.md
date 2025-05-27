# Hướng dẫn cấu hình headless cho Raspberry Pi

## Mục lục

- [Hướng dẫn cấu hình headless cho Raspberry Pi](#hướng-dẫn-cấu-hình-headless-cho-raspberry-pi)
	- [Mục lục](#mục-lục)
	- [Raspberry Pi Headless](#raspberry-pi-headless)
	- [Ghi ảnh đĩa Raspberry Pi OS](#ghi-ảnh-đĩa-raspberry-pi-os)
	- [Kết nối tới Raspberry Pi](#kết-nối-tới-raspberry-pi)


## Raspberry Pi Headless

Bạn vừa mua một board Raspberry Pi 4 về mới toanh. Nhưng lúc làm theo hướng dẫn sử dụng để bật nó lên thì... bạn quên mua cáp màn hình nên không có gì hiện lên cả...

Nhưng đừng lo, bạn có thể cấu hình Raspberry Pi Headless để sử dụng Pi mà không cần màn hình (hay bàn phím hay chuột gì cả)

## Ghi ảnh đĩa Raspberry Pi OS

Đầu tiên, bạn cần chuẩn bị những thứ sau:

- Thẻ nhớ microSD trống có dung lượng ít nhất 8GB và đầu đọc thẻ
- Phần mềm ghi ảnh đĩa của nhà Raspberry Pi, tên là Raspberry Pi Imager. Bạn tải ở đây: [Link](https://www.raspberrypi.com/software/)
- Board Raspberry Pi. Ở đây mình dùng Raspberry Pi 4B. Các board khác làm tương tự.

Mở phần mềm Raspberry Pi Imager lên, bạn sẽ thấy giao diện thế này:

<img src=assets/rpi_imager.png width=800>

Bạn bấm `Choose Device` rồi chọn board tương ứng.

<img src=assets/rpi_imager-choose-device.png width=800>

Tiếp theo bạn bấm `Choose OS`. Ở đây bạn sẽ chọn hệ điều hành. Loại phổ biến nhất là Raspberry Pi OS. Bạn có thể chọn bản 32bit hoặc 64bit. Bản Lite là bản nhẹ hơn (không có giao diện desktop). Mình chọn bản Raspberry Pi OS Lite (64bit) ở trong mục `Raspberry Pi OS (other)` > `Raspberry Pi OS Lite (64-bit)`.

Cuối cùng là chọn `Storage`. Chọn thẻ nhớ mà bạn muốn ghi vào.

<img src=assets/rpi_imager-choose-storage.png width=800>


Bạn bấm Next, rồi chọn `Edit Settings`. Có các cài đặt sau:

- Host Name: là tên sẽ hiển thị trên mạng LAN. Mình đặt tên là rpi4.local
- Set username and password: Tạo người dùng và mật khẩu. Bạn đặt theo ý bạn
- Configure Wireless LAN: Bạn nhập tên WiFi và mật khẩu. Khi khởi động, Pi sẽ kết nối tới WiFi này và bạn có thể truy cập tới nó.

<img src=assets/rpi_imager-setting-1.png width=800>

Tiếp theo, bạn chuyển qua tab `Services`, rồi tích vào ô `Enable SSH`. Chọn `Use password authentication`

Cuối cùng bấm `Save`, rồi bấm `Yes` và `Yes`. Bạn ngồi đợi tầm 10p là xong.

## Kết nối tới Raspberry Pi

Bạn cắm thẻ SD vào board rồi bật nguồn, đợi tầm 5p cho Raspberry Pi kết nối tới WiFi.

Ta sẽ dùng `ssh` để kết nối tới Raspberry Pi. `ssh` viết tắt là Secure Shell, là một giao thức mạng giúp kết nối tới một thiết bị khác một cách bảo mật, trong một kết nối kém bảo mật. Nói tóm lại, `ssh` cho phép bạn sử dụng `Terminal` của Raspberry Pi ở trên laptop của mình. Cú pháp kết nối `ssh` như sau:

```
ssh [username]@[ip address]
```

`username` là tên đăng nhập vào Raspberry Pi. `ip address` là địa chỉ IPv4 của Pi. Địa chỉ IPv4 của Pi thường được cấp phát động (do cục WiFi phát cho). Tới đây bạn có 2 cách:

- Cách 1: Bạn không cần biết địa chỉ IP của Pi. Khi kết nối mạng LAN, Pi sẽ hiện tên `host name` của mình lên mạng. Do đó bạn có thể kết nối với lệnh:

	```
	ssh yusenpai@rpi4.local
	```

- Cách 2: Nếu bạn không làm cách 1 được, thì chỉ còn cách tìm ra địa chỉ IP của Raspberry Pi. Bạn có thể dùng phần mềm bên thứ 3 để tìm. Mình dùng phần mềm `IP Scanner`. Phần mềm này sẽ quét và tìm kiếm thiết bị trên mạng LAN, từ đó tìm ra địa chỉ IP của Pi. Ví dụ địa chỉ IP mình kiếm được là `192.168.1.14`, mình có thể kết nối như sau:

	```
	ssh yusenpai@192.168.1.14
	```

Sau khi kết nối được, bạn nhập mật khẩu đăng nhập vào Pi. Thế là xong, bạn có thể sử dụng Terminal của Pi trên chính máy laptop.