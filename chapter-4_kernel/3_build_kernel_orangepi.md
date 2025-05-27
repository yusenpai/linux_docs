# Build kernel 32-bit cho Orange Pi Plus 2

## Mục lục

- [Build kernel 32-bit cho Orange Pi Plus 2](#build-kernel-32-bit-cho-orange-pi-plus-2)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Từ khoá](#từ-khoá)
	- [Build kernel 32-bit cho Raspberry Pi 4](#build-kernel-32-bit-cho-raspberry-pi-4)
	- [Boot kernel cho Raspberry Pi 4](#boot-kernel-cho-raspberry-pi-4)


## Yêu cầu kiến thức

## Yêu cầu kỹ thuật

## Từ khoá

## Build kernel 32-bit cho Raspberry Pi 4

- Cần có: TODO
  - Fork của kernel dành cho Raspberry Pi 4: 
  - Cross compiler dành cho arm64. Tải về ở https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads

- Gói cần thiết:

```sh
sudo apt install subversion libssl-dev
```

- Kernel 4.19.y:

```sh
git clone --depth=1 -b rpi-4.19.y https://github.com/raspberrypi/linux.git
```

- Build kernel

```sh
cd linux
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- bmc2711_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- -j8

```

Sau khi build xong sẽ có các file: TODO

Sao chép các file ấy vào thư mục boot:

```sh
mkdir ../boot
cp arch/arm64/boot/Image ../boot/kernel8.img
cp arch/arm64/boot/dts/overlays/*.dtbo ../boot/overlays/
cp arch/arm64/boot/dts/broadcom/*.dtb ../boot/
cat << EOF > ../boot/config.txt
enable_uart=1
arm_64bit=1
EOF

cat << EOF > ../boot/cmdline.txt
console=serial0,115200 console=tty1 root=/dev/mmcblk0p2
rootwait
EOF
```

- Giải thích các bước ở trên: TODO

## Boot kernel cho Raspberry Pi 4

- Mô tả từng bước thực hiện: TODO

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