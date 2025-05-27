# Chương trình `init`

## Mục lục

- [Chương trình `init`](#chương-trình-init)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Từ khoá](#từ-khoá)
	- [Chương trình `init`](#chương-trình-init-1)
	- [Khởi chạy tiến trình `daemon`](#khởi-chạy-tiến-trình-daemon)

## Yêu cầu kiến thức

## Yêu cầu kỹ thuật

## Từ khoá

## Chương trình `init`

Lần trước bạn đã chạy `sh` khi khởi động kernel bằng tham số `rdinit=/bin/sh`. Đây là trường hợp đơn giản nhất, để minh hoạ cho bạn thấy `initramfs` có chạy hay không. Trong thực tế phức tạp hơn nhiều. Hệ thống Unix sẽ chạy một chương trình tên là `init`. Có rất nhiều chương trình `init` khác nhau, trong bài này bạn sẽ được tìm hiểu về `init` của BusyBox.

Chương trình `init` bắt đầu bằng việc đọc file cấu hình, nằm trong `/etc/inittab`. Đây là ví dụ của một cấu hình:

```
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/ash
```

Giải thích:
- Dòng đầu tiên chạy shell script tên "rcS"
- askfirst: sẽ in ra màn hình dòng **Please press Enter to activate this console**. Khi bạn nhấn Enter thì shell sẽ chạy.
- dấu "-" phía trước `/bin/ash` nghĩa là: shell sẽ đăng nhập và source các biến môi trường trong `/etc/profile` và `$HOME/.profile`.

Nếu trong `rootfs` không có `/etc/inittab`, BusyBox sẽ sử dụng cấu hình mặc định (phức tạp hơn tí).

Shell script `/etc/init.d/rcS` là nơi để đặt tất cả những lệnh khởi động, cấu hình cần thiết lúc khởi động hệ thống. Chẳng hạn, ở bài trước bạn đã biết cần phải mount `proc` và `sys`:

```
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
```

Và phải cấp quyền thực thi cho nó:

```sh
cd $HOME/rootfs
chmod +x etc/init.d/rcS
```

Bây giờ, bạn có thể thử chạy chương trình `init` trên QEMU bằng cách thay đổi dòng `-append` thành:

```
-append "console=ttyAMA0 rdinit=/sbin/init"
```

Đối với Raspberry Pi 4, bạn thay đổi lệnh `setenv` như sau:

```sh
setenv bootargs "8250.nr_uarts=1 console=ttyS0,115200 rdinit=/sbin/init"
```

## Khởi chạy tiến trình `daemon`

Thường thì bạn muốn chạy một số tiến trình nền khi khởi động. Những tiến trình đó gọi là `daemon`. Lấy ví dụ, `syslogd` - là daemon quản lý việc ghi nhật ký (log). Mục đích của `syslogd` là thu thập tất cả các log của các chương trình khác.

Chạy một daemon khá là đơn giản bằng cách thêm một dòng trong `etc/inittab`:

```
::respawn:/sbin/syslogd -n
```

`respawn` kiến cho `syslogd` khi bị đóng (bị kill) thì vẫn sẽ tự khởi động lại. `-n` khiến cho `syslogd` không chạy nền (quan trọng). Các log được viết vào file `/var/log`


Bạn có thể chạy `klogd` bằng cách tương tự. `klogd` gửi log của kernel tới `syslogd`.