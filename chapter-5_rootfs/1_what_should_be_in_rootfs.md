# Bên trong Root Filesystem chứa gì ?

## Mục lục

- [Bên trong Root Filesystem chứa gì ?](#bên-trong-root-filesystem-chứa-gì-)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Từ khoá](#từ-khoá)
	- [Cây thư mục `root`](#cây-thư-mục-root)
		- [Chuẩn bị rootfs trên máy host](#chuẩn-bị-rootfs-trên-máy-host)
		- [Quyền hạn truy cập](#quyền-hạn-truy-cập)
		- [Chương trình cho root filesystem](#chương-trình-cho-root-filesystem)
			- [Chương trình `init`](#chương-trình-init)
			- [Shell](#shell)
			- [Utilities (tạm dịch: chương trình tiện ích)](#utilities-tạm-dịch-chương-trình-tiện-ích)
		- [BusyBox giải cứu!](#busybox-giải-cứu)
			- [Build BusyBox](#build-busybox)
		- [Thư viện cho root filesystem](#thư-viện-cho-root-filesystem)
		- [Device nodes](#device-nodes)
		- [`proc` và `sysfs` filesystems](#proc-và-sysfs-filesystems)

## Yêu cầu kiến thức

## Yêu cầu kỹ thuật

## Từ khoá

## Cây thư mục `root`

Cây thư mục gốc (/) trong hệ điều hành Linux được tổ chức theo một tiêu chuẩn gọi là Filesystem Hierarchy Standard (FHS). Một số thư mục quan trọng bao gồm:
- /bin: chứa các chương trình thực thi cơ bản cho toàn bộ người dùng.
- /sbin: chứa các chương trình quản trị hệ thống.
- /lib, /lib64: chứa các thư viện, ví dụ, thư viện chuẩn C.
- /usr: chứa phần mềm, thư viện và chương trình quản trị hệ thống, tương ứng trong /usr/bin, /usr/lib, /usr/sbin.
- /etc: chứa các file cấu hình hệ thống.
- /dev: chứa các file thiết bị (device nodes).
- /proc: filesystem ảo hiển thị thông tin kernel.
- /sys: filesystem ảo chứa thông tin thiết bị hệ thống.
- /tmp: nơi lưu file tạm thời.
- /var: chứa dữ liệu thay đổi như log, spool,…

### Chuẩn bị rootfs trên máy host

```sh
cd $HOME
mkdir rootfs
cd rootfs
mkdir bin dev etc home lib proc sbin sys tmp usr var
mkdir usr/bin usr/lib usr/sbin
mkdir -p var/log
```

Để xem cây thư mục, sử dụng lệnh `tree`:

```sh
tree -d
```

### Quyền hạn truy cập

Trong root filesystem, các file và thư mục thường thuộc về người dùng root. Điều này đảm bảo rằng chỉ những tiến trình có quyền cao nhất (ví dụ: init, systemd hoặc các script hệ thống) mới có thể sửa đổi các file quan trọng như /bin, /sbin, hoặc /etc.
Nếu các quyền sở hữu bị sai, hệ thống sẽ gặp lỗi trong quá trình khởi động (ví dụ: không tìm thấy init, không thể mount filesystem, lỗi cấu hình,…).

Giải thích về crw: TODO

Áp dụng quyền root lên các thư mục trong rootfs:

```sh
cd $HOME/rootfs
$ sudo chown -R root:root *
```

> Lưu ý: Trong quá trình build BusyBox sắp tới sẽ xảy ra lỗi nếu quyền hạn là root. Sau khi build BusyBox xong hãy quay lại đây và chạy lệnh này.

### Chương trình cho root filesystem

Bây giờ là lúc thêm các chương trình cần thiết vào trong root filesystem. Các chương trình chính gồm:

- Chương trình `init`
- Shell
- Utilities

#### Chương trình `init`

`init` là chương trình đầu tiên được chạy, nên không thể thiếu trong root filesystem. Ta sẽ sử dụng `init`do BusyBox cung cấp

#### Shell

Shell là một chương trình cho phép người dùng giao tiếp với hệ điều hành thông qua dòng lệnh. Nó đóng vai trò như một trình thông dịch giữa người dùng và kernel. Trong root filesystem, shell là thành phần thiết yếu để:

Một số shell phổ biến:
- sh: shell cơ bản, tiêu chuẩn POSIX (thường là link tới dash, bash hoặc busybox sh)
- bash: GNU Bourne Again Shell, mạnh và phổ biến nhất
- zsh: shell nâng cao, hỗ trợ nhiều tính năng tương tác

#### Utilities (tạm dịch: chương trình tiện ích)

Chương trình tiện ích là các chương trình bạn hay sử dụng trong shell:

- cd
- ls
- echo
- ...

Để shell trở nên hữu ích, bạn cần có các tiện ích dòng lệnh mà Unix dựa vào. Ngay cả với một root filesystem cơ bản, bạn cũng cần khoảng 50 tiện ích, điều này dẫn đến hai vấn đề:

- Tìm source code của từng tiện ích và biên dịch chéo chúng khá là mệt
- Đống chương trình build ra khá là nặng: vài chục MB, là cả vấn đề trong một môi trường chỉ có vài MB cho bạn.

Để giải quyết vấn đề này, BusyBox đã ra đời.

### BusyBox giải cứu!

BusyBox là một bộ sưu tập các tiện ích dòng lệnh Unix phổ biến, được gói gọn trong một tập tin thực thi duy nhất. Nó được thiết kế dành riêng cho các hệ thống nhúng (embedded systems) với mục tiêu nhỏ gọn, nhanh và dễ cấu hình.

Ví dụ, bạn muốn sử dụng lệnh `cd`, `ls`, bạn sẽ làm thế này:
```sh
busybox echo hello
# Tương đương với echo hello

busybox ls -l
# Tương đương với ls -l
```

Tất nhiên nếu bạn gõ:
```sh
echo hello
# Vẫn sẽ chạy như thường, vì chương trình "echo" được liên kết tới chương trình busybox

ls -l
# Tương tụ
```

Nói tóm lại, BusyBox gom tất cả các tiện ích vào một chương trình. khi bạn chạy một tiện ích bất kỳ, thực chất nó sẽ gọi BusyBox để chạy cho bạn.

#### Build BusyBox

Clone source code về, sử dụng cấu hình `defconfig`:

```sh
cd $HOME
git clone git://busybox.net/busybox.git
cd busybox
make defconfig
```

Cấu hình thư mục cài đặt bằng `menuconfig`:
```sh
make menuconfig
```
Bạn tìm tới:TODO

Dùng lệnh `make` để build:

- Cho `arm` 32-bit:
```sh
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- -j4
```

- Cho `aarch64` 64-bit:
```sh
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- -j4
```

Cuối cùng là cài đặt nó vào thư mục `rootfs` đã chuẩn bị trước đó:
```sh
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- install
# Cho arm64: make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- install
```

Bây giờ bạn vào thư mục `rootfs` để kiểm tra nha:

```sh
cd $HOME/rootfs
tree
```

### Thư viện cho root filesystem

Chương trình liên kết với thư viện. Như đã học ở chương 2 Toolchain (TODO), có hai cách liên kết với thư viện:

- Liên kết tĩnh (static link): Thư viện đi kèm chương trình luôn, chạy được ngay
- Liên kết động (dynamic link): Chương trình liên kết với thư viện nằm trong `/lib` hoặc `/lib64`, `/usr/lib`, `/usr/lib64`

Thư viện tới từ toolchain (TODO). Bạn cần copy thư viện từ toolchain đem qua `rootfs`. Đường dẫn chứa thư viện được gọi là `sysroot`, và bạn có thể in `sysroot` thông qua lệnh sau:

```sh
aarch64-none-linux-gnu-gcc -print-sysroot
TODO
```

> Lưu ý: Nếu bạn dùng cho arm 32bit thì dùng toolchain tương ứng (arm-none-linux-gnueabihf-gcc) nha

Để giảm bớt việc gõ tay, mình sẽ export nó vào biến `SYSROOT`:
```sh
export SYSROOT=$(aarch64-none-linux-gnu-gcc -print-sysroot)
```

TODO:

### Device nodes

Phần lớn thiết bị trong hệ thống Linux được biểu diễn bởi một `device node`, và theo như phương châm của Unix: mọi thứ đều là file. Một `device node` có thể là `block device` hoặc `character device`.

- `Block device`: đại diện cho bộ nhớ, ví dụ như thẻ SD, ổ cứng,... Bạn hay dùng lệnh `lsblk` để liệt kê tất cả các `block device`.
- `Character device`: các device khác không phải `block device`.

> Lưu ý: Kiến thức về device nodes sẽ được học kỹ ở chương TODO

Các `device node` nằm trong thư mục `/dev`. Ví dụ, bạn có thể tìm cổng Serial, với đại diện là device node `/dev/ttyS0`

`Device node` được tạo ra bởi câu lệnh:

```sh
mknod <tên> <kiểu> <major> <minor>
```

trong đó: TODO

Trong root filesystem đơn giản nhất, bạn cần hai device node là `console` và `null`. Bạn có thể tạo với lệnh sau:

```sh
cd $HOME/rootfs
sudo mknod -m 666 dev/null c 1 3
sudo mknod -m 600 dev/console c 5 1

```
Đoạn "-m 666" và "-m 600" để cấu hình quyền hạn. TODO

Chạy `ls` để xem các file vừa tạo:
```sh
ls -l dev
total 0
crw------- 1 root root 5, 1 Mar 22 20:01 console
crw-rw-rw- 1 root root 1, 3 Mar 22 20:01 null
```

### `proc` và `sysfs` filesystems

- Giải thích về proc và sysfs: TODO

- Cú pháp để mount các filesystem này: TODO

