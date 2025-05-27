# Giới thiệu về Linux Kernel

## Mục lục

- [Giới thiệu về Linux Kernel](#giới-thiệu-về-linux-kernel)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Từ khoá](#từ-khoá)
	- [Linux Kernel là gì?](#linux-kernel-là-gì)
	- [Chọn kernel](#chọn-kernel)
	- [Build kernel](#build-kernel)
		- [Tải kernel](#tải-kernel)
		- [Biên dịch kernel](#biên-dịch-kernel)
	- [Biên dịch device tree](#biên-dịch-device-tree)
	- [Biên dịch module](#biên-dịch-module)
	- [Dọn dẹp source code](#dọn-dẹp-source-code)

## Yêu cầu kiến thức

## Yêu cầu kỹ thuật

## Từ khoá

## Linux Kernel là gì?

Linux Kernel là lõi của hệ điều hành Linux, đóng vai trò là tầng trung gian giữa phần cứng và phần mềm. Kernel chịu trách nhiệm quản lý tài nguyên hệ thống như CPU, bộ nhớ, thiết bị ngoại vi, hệ thống tập tin và cung cấp các dịch vụ thiết yếu cho các tiến trình người dùng.
Có nhiều biến thể của kernel tùy theo kiến trúc phần cứng (x86, ARM, RISC-V, …), nhưng chúng đều tuân theo cùng một nguyên tắc thiết kế chính: mã nguồn mở, mô-đun hóa và khả năng mở rộng cao.

## Chọn kernel

Để bắt đầu làm việc với kernel, bạn cần chọn phiên bản phù hợp với thiết bị và mục đích phát triển của mình.
Ví dụ:

- LTS (Long Term Support): ổn định, hỗ trợ lâu dài, phù hợp cho sản phẩm.
- Mainline: phiên bản mới nhất, có các tính năng mới, nhưng không ổn định bằng LTS.

Bạn có thể tìm danh sách kernel tại: https://www.kernel.org

## Build kernel

### Tải kernel

```sh
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.12.30.tar.xz
tar xf linux-6.12.30.tar.xz
mv linux-6.12.30 linux
```
Một số thư mục trong kernel:
- arch/: chứa mã nguồn theo kiến trúc CPU.
- drivers/: mã nguồn driver thiết bị.
- fs/: các hệ thống tập tin (ext4, FAT, …).
- kernel/: các hàm chính của kernel như quản lý tiến trình.
- include/: tệp header dùng chung.
- scripts/: công cụ hỗ trợ biên dịch.
- tools/: các công cụ người dùng hỗ trợ phát triển kernel.

KBuild: là hệ thống build tích hợp trong Linux kernel, sử dụng Makefile ở nhiều cấp thư mục để tự động hóa việc biên dịch kernel, module và device tree.
Nó hỗ trợ build cho nhiều kiến trúc và nền tảng nhờ biến ARCH, CROSS_COMPILE.

### Biên dịch kernel

Đối tượng cần build:
- zImage hoặc Image: nhân Linux được nén.
- uImage: format dành cho U-Boot.
- vmlinux: file ELF chưa nén (dùng để debug).

```sh
make -j 4 ARCH=arm CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- zImage
```

Giải thích:
- make -j4: biên dịch song song với 4 luồng.
- ARCH=arm: biên dịch cho kiến trúc ARM.
- CROSS_COMPILE=...: trình biên dịch chéo.
- zImage: tên file đối tượng

## Biên dịch device tree

Device tree là tập tin nhị phân chứa thông tin phần cứng được kernel sử dụng để khởi tạo hệ thống, đặc biệt quan trọng với kiến trúc ARM.

```sh
make ARCH=arm dtbs
```

## Biên dịch module

Module là các thành phần mở rộng cho kernel (driver, hệ thống tập tin, …), có thể nạp hoặc gỡ động mà không cần khởi động lại hệ thống.

```sh
make -j4 ARCH=arm CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- modules
```

## Dọn dẹp source code

```
make clean
make distclean
```

Giải thích:

- make clean: xóa các file được tạo ra trong quá trình build (object, binary), nhưng vẫn giữ cấu hình .config.
- make distclean: xóa toàn bộ file build và cấu hình để trả về trạng thái ban đầu như lúc mới giải nén.