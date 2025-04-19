# Hướng dẫn build và cài đặt U-Boot

## Mục lục
- [Hướng dẫn build và cài đặt U-Boot](#hướng-dẫn-build-và-cài-đặt-u-boot)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Build U-boot](#build-u-boot)
		- [Cài đặt cross compiler](#cài-đặt-cross-compiler)
		- [Tải U-boot từ source:](#tải-u-boot-từ-source)
		- [Build U-boot cho Raspberry Pi 4](#build-u-boot-cho-raspberry-pi-4)
		- [Build U-boot cho Orange Pi Plus 2](#build-u-boot-cho-orange-pi-plus-2)
	- [Cài đặt U-boot](#cài-đặt-u-boot)
		- [Cài đặt U-boot cho Rasberry Pi 4](#cài-đặt-u-boot-cho-rasberry-pi-4)
		- [Cài đặt U-boot cho Orange Pi 2 Plus](#cài-đặt-u-boot-cho-orange-pi-2-plus)
	- [Sử dụng U-boot](#sử-dụng-u-boot)

## Yêu cầu kỹ thuật

- Cross compiler aarch64-none-linux-gnu cho ARM64 và arm-none-linux-gnueabihf cho ARM
- Thẻ SD tối thiểu 8G
- Board Raspberry Pi 4 hoặc Orange Pi Plus 2

## Build U-boot

### Cài đặt cross compiler

Tải cross compiler từ [ARM](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads). 

Sau khi tải về, giải nén rồi đổi tên thư mục. Sau đó dùng lệnh `export` để thêm vào biến môi trường `PATH`. Ví dụ: đổi tên thư mục thành aarch64-none-linux-gnu, lưu trong thư mục `HOME`:

```sh
export PATH=~/aarch64-none-linux-gnu/bin:$PATH
```

### Tải U-boot từ source:

```sh
git clone https://github.com/u-boot/u-boot.git
cd u-boot
```

Bên trong thư mục `configs` chứa rất nhiều file cấu hình cho các board phổ biến. Ví dụ cho board Raspberry Pi:

```sh
ls -1 configs | grep rpi
```

```
rpi_0_w_defconfig
rpi_2_defconfig
rpi_3_32b_defconfig
rpi_3_b_plus_defconfig
rpi_3_defconfig
rpi_4_32b_defconfig
rpi_4_defconfig
rpi_arm64_defconfig
rpi_defconfig
```

### Build U-boot cho Raspberry Pi 4

```sh
make distclean
make CROSS_COMPILE=aarch64-none-linux-gnu- rpi_4_defconfig
make CROSS_COMPILE=aarch64-none-linux-gnu- -j(nproc)
```

Sau khi build xong, file `u-boot.bin` xuất hiện. Ta sử dụng file này để cài đặt U-boot.

### Build U-boot cho Orange Pi Plus 2

```sh
make distclean
make CROSS_COMPILE=arm-none-linux-gnueabihf- orangepi_plus2e_defconfig
make CROSS_COMPILE=arm-none-linux-gnueabihf- -j(nproc)
```

Sau khi build xong, file `u-boot-sunxi-with-spl.bin` chứa cả SPL và U-boot.

## Cài đặt U-boot

### Cài đặt U-boot cho Rasberry Pi 4

Yêu cầu tối thiểu nhất để chạy U-Boot trên Raspberry Pi 4: trong phân vùng `boot` của thẻ nhớ có những file sau:

- bootcode.bin
- start4.elf
- fixup4.data
- bcm2711-rpi-4-b.dtb
- u-boot.bin
- config.txt

Ý nghĩa của các file `bootcode.bin`, `start4.elf`, `fixup4.data` tham khảo ở https://www.raspberrypi.com/documentation/computers/configuration.html#boot-folder-content.

`config.txt` chứa:

```
enable_uart=1
arm_64bit=1
kernel=u-boot.bin
```

Gắn mạch chuyển USB - UART lên board Raspberry Pi 4, TX của board này nối RX của board kia. Trên máy tính mở phần mềm xem Serial bất kỳ, baudrate ở 115200. Gắn thẻ nhớ lên board Pi rồi cắm nguồn. Lúc này U-boot sẽ gửi message lên Serial.

### Cài đặt U-boot cho Orange Pi 2 Plus

Chuẩn bị sẵn một thẻ SD card, đã được nạp sẵn image của hệ điều hành Orange Pi OS.

Ghi file `u-boot-sunxi-with-spl.bin` vào thẻ SD, bắt đầu tại địa chỉ 8K hoặc 128K. Chip H3 trên board sẽ tìm bootloader ở những địa chỉ này:

```sh
sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/dbX bs=1k seek=128
```

Với dbX là block device ứng với thẻ SD. Nếu chưa xác định được device nào là thẻ SD thì dùng lệnh `lsblk`, xem kích thước của các device được liệt kê xem có đúng với thẻ nhớ của mình không.

Làm tương tự với board Pi, gắn mạch USB - UART lên board Orange Pi, nhưng nơi gắn nằm ở bên cạnh jack nguồn DC. Gắn thẻ nhớ lên board, cắm nguồn. U-boot sẽ khởi động và in message lên Serial.

## Sử dụng U-boot

U-boot có giao diện lệnh giống với Linux. Ta có thể sử dụng lệnh `fatload` để nạp dữ liệu từ thẻ SD lên RAM.

Sau khi U-boot đã khởi động, ta có thể rút thẻ nhớ ra. Cắm lại thẻ vào máy tính, copy các file: kernel image, dtb và initramfs vào phân vùng đầu tiên của thẻ. Gắn lại thẻ nhớ vào board. Dùng các lệnh fatload để nạp ba file ấy vào RAM. Ví dụ với board Orange Pi Plus 2:

```sh
fatload mmc 0:1 82000000 uImage
fatload mmc 0:1 83000000 uInitrd
fatload mmc 0:1 84000000 sun8i-h3-orangepi-plus2e.dtb
```

Các lệnh trên nạp lần lượt kernel, initramfs, device tree vào địa chỉ 0x82000000, 0x83000000, 0x84000000 trên RAM. Sau đó dùng lệnh `bootm` để khởi động Linux:

```sh
bootm 82000000 83000000 84000000
```