# Build kernel 64-bit cho Raspberry Pi 4

## Mục lục

- [Build kernel 64-bit cho Raspberry Pi 4](#build-kernel-64-bit-cho-raspberry-pi-4)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Từ khoá](#từ-khoá)
	- [Build kernel 64-bit cho Raspberry Pi 4](#build-kernel-64-bit-cho-raspberry-pi-4-1)
	- [Boot kernel cho Raspberry Pi 4](#boot-kernel-cho-raspberry-pi-4)


## Yêu cầu kiến thức

## Yêu cầu kỹ thuật

## Từ khoá

## Build kernel 64-bit cho Raspberry Pi 4

Cần có:
  - Linux Kernel 6.12.30 (tải từ kernel.org)
  - Trình biên dịch chéo dành cho kiến trúc AArch64: aarch64-none-linux-gnu, tải tại: https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads
  - Thẻ nhớ SD chuẩn bị sẵn, đã chia phân vùng FAT32

- Gói cần thiết:

```sh
sudo apt install subversion libssl-dev
```

- Build kernel: Sau khi giải nén, đổi tên thư mục thành `linux`:

```sh
cd linux
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- bmc2711_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- -j8
```

Sau khi build xong, bạn sẽ thu được các file quan trọng sau:

- arch/arm64/boot/Image: kernel đã build, dạng chưa nén.
- arch/arm64/boot/dts/broadcom/*.dtb: các file device tree cho các board Raspberry Pi.
- modules: (nếu cần) dùng lệnh make modules để build các module.

## Boot kernel cho Raspberry Pi 4

Bạn làm theo các bước sau để khởi chạy kernel:

1. Bạn sử dụng thẻ nhớ đã cài đặt U-Boot ở bài trước. Mount phân vùng đầu tiên vào `$HOME/bootfs`
2. Xoá hết các file không cần thiết, chỉ giữ lại các file (start4.elf, fixup4.data, bcm2711-rpi-4-b.dtb, config.txt, u-boot.bin)
3. Copy file kernel "Image" đã build vào `$HOME/bootfs`
4. Umount
5. Gỡ thẻ nhớ, gắn vào Raspberry Pi rồi khởi động.
6. Trong giao diện U-Boot, chạy các lệnh sau:
	```
	setenv bootargs "8250.nr_uarts=1 console=ttyS0,115200"
	fatload mmc 0:1 2000000 Image
	fatload mmc 0:1 6000000 bcm2711-rpi-4-b.dtb
	booti 0x2000000 - 0x6000000;
	```

- Kết quả:

```
[ 0.000000] Booting Linux on physical CPU 0x0000000000 [0x410fd083]
[ 0.000000] Linux version 4.19.127-v8+ (frank@franktop) (gcc version 10.2.1 20201103 (GNU Toolchain for the A-profile Architecture 102-2020.11 (arm-10.16))) #1 SMP PREEMPT Sat Feb 6 16:19:37 PST 2021
[ 0.000000] Machine model: Raspberry Pi 4 Model B Rev 1.1
[ 0.000000] efi: Getting EFI parameters from FDT:
[ 0.000000] efi: UEFI not found.
[ 0.000000] cma: Reserved 64 MiB at 0x0000000037400000
[ 0.000000] random: get_random_bytes called from start_kernel+0xb0/0x480 with crng_init=0
[ 0.000000] percpu: Embedded 24 pages/cpu s58840 r8192 d31272 u98304
[ 0.000000] Detected PIPT I-cache on CPU0
[…]
```