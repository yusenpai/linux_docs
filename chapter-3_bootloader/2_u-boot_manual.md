# Hướng dẫn build và cài đặt U-Boot

## Mục lục
- [Hướng dẫn build và cài đặt U-Boot](#hướng-dẫn-build-và-cài-đặt-u-boot)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Build U-Boot](#build-u-boot)
		- [Cài đặt cross compiler](#cài-đặt-cross-compiler)
		- [Tải U-Boot từ source:](#tải-u-boot-từ-source)
		- [Build U-Boot cho Raspberry Pi 4](#build-u-boot-cho-raspberry-pi-4)
		- [Build U-Boot cho Orange Pi Plus 2](#build-u-boot-cho-orange-pi-plus-2)
	- [Cài đặt U-Boot](#cài-đặt-u-boot)
		- [Cài đặt U-Boot cho Rasberry Pi 4](#cài-đặt-u-boot-cho-rasberry-pi-4)
		- [Cài đặt U-Boot cho Orange Pi 2 Plus](#cài-đặt-u-boot-cho-orange-pi-2-plus)
	- [Sử dụng U-Boot](#sử-dụng-u-boot)
		- [Tạo file Image](#tạo-file-image)
		- [Chạy kernel](#chạy-kernel)

## Yêu cầu kiến thức

- Biết cách sử dụng `make` để build U-Boot
- Biết cách sử dụng các lệnh shell cơ bản
- Biết cách sử dụng lệnh `dd`

## Yêu cầu kỹ thuật

- Cross compiler aarch64-none-linux-gnu cho ARM64 và arm-none-linux-gnueabihf cho ARM
- Thẻ SD tối thiểu 8G đã được định dạng. Tham khảo cách định dạng tại (TODO: thêm đường dẫn)
- Board Raspberry Pi 4 hoặc Orange Pi Plus 2

## Build U-Boot

### Cài đặt cross compiler

Để build U-Boot, các bạn sử dụng cross-toolchain phù hợp. Hướng dẫn cài đặt cross-compiler nằm ở chương 2 (TODO: add link)

### Tải U-Boot từ source:

```
git clone https://github.com/u-boot/u-boot.git
cd u-boot
```

Bên trong thư mục `configs` chứa rất nhiều file cấu hình cho các board phổ biến. Ví dụ cho board Raspberry Pi:

```
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

### Build U-Boot cho Raspberry Pi 4
Build U-Boot với `aarch64-none-linux-gnu` toolchain và cấu hình `rpi_4_defconfig`:

```
make distclean
make CROSS_COMPILE=aarch64-none-linux-gnu- rpi_4_defconfig
make CROSS_COMPILE=aarch64-none-linux-gnu- -j(nproc)
```

Sau khi build xong, file `u-boot.bin` xuất hiện. Ta sử dụng file này để cài đặt U-Boot.

### Build U-Boot cho Orange Pi Plus 2

Build U-Boot với `arm-none-linux-gnueabihf` toolchain và cấu hình `orangepi_plus2e_defconfig`:

```
make distclean
make CROSS_COMPILE=arm-none-linux-gnueabihf- orangepi_plus2e_defconfig
make CROSS_COMPILE=arm-none-linux-gnueabihf- -j(nproc)
```

Sau khi build xong, file `u-boot-sunxi-with-spl.bin` chứa cả SPL và U-Boot.

## Cài đặt U-Boot

### Cài đặt U-Boot cho Rasberry Pi 4

Ta sử dụng thẻ nhớ SD đã được ghi sẵn **Raspberry Pi OS 64 bit (bất kỳ phiên bản nào)**. Xem hướng dẫn ở chương (TODO: link)

Sử dụng lệnh `mount` để gắn phân vùng đầu tiên của thẻ nhớ SD (là phân vùng `bootfs`) vào thư mục `bootfs`:
```
mkdir $HOME/bootfs
cd $HOME/bootfs
sudo mount /dev/sdX1 $HOME/bootfs
```

Bên trong thư mục `bootfs` là toàn bộ dữ liệu chứa trong phân vùng đầu tiên của thẻ SD. Trong đó có một số file cần chú ý:

- start4.elf: Đoạn chương trình được Bootloader ROM code nạp vào **lõi GPU của Raspberry Pi 4**. Nó thực hiện khởi tạo hệ thống, đọc file cấu hình `config.txt`; nạp rồi khởi chạy kernel Linux.
- fixup4.data: Đoạn code đi cùng với `start4.elf`.
- bcm2711-rpi-4-b.dtb: Device tree
- config.txt: File cấu hình khi khởi động Raspberry Pi 4.

Ta sao chép `u-boot.bin` vào `$HOME/bootfs`:
```
cp $HOME/u-boot/u-boot.bin $HOME/bootfs
```

Sau đó, sửa nội dung file `config.txt` như sau:

```
enable_uart=1
arm_64bit=1
kernel=u-boot.bin
```

Dòng `kernel=u-boot.bin` sẽ khiến cho Raspberry Pi 4 nạp và chạy U-Boot thay vì kernel.

Gắn mạch chuyển USB - UART lên board Raspberry Pi 4. Mở cổng Serial ở baudrate 115200. Gắn thẻ nhớ lên board Pi rồi cắm nguồn. Lúc này U-Boot sẽ gửi message lên Serial.

### Cài đặt U-Boot cho Orange Pi 2 Plus

Chuẩn bị sẵn một thẻ SD card, đã được nạp sẵn image của hệ điều hành Orange Pi OS.

Đối với Orange Pi 2 Plus, quá trình khởi động có sự khác biệt. Chip H3 sẽ tìm tới địa chỉ 8KB hoặc 128KB trên thẻ SD rồi nạp chương trình bootloader ở đó. Do đó ta chỉ cần copy `u-boot-sunxi-with-spl.bin` vào vị trí tương ứng. Ví dụ, ta muốn ghi vào địa chỉ 128KB:

```
sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/dbX bs=1k seek=128
```

Làm tương tự với board Pi, gắn mạch USB - UART lên board Orange Pi, nhưng nơi gắn nằm ở bên cạnh jack nguồn DC. Gắn thẻ nhớ lên board, cắm nguồn. U-Boot sẽ khởi động và in message lên Serial.

## Sử dụng U-Boot

Khi vừa khởi động U-Boot, nó sẽ tự động bước vào chế độ tự động boot. Bấm một phím bất kỳ để dừng.

```
U-Boot 2021.01 (May 06 2025 - 22:16:12 +0700)

DRAM:  1.9 GiB
RPI 4 Model B (0xb03115)
MMC:   mmcnr@7e300000: 1, mmc@7e340000: 0
Loading Environment from FAT... ** No partition table - mmc 0 **
In:    serial
Out:   serial
Err:   serial
Net:   eth0: ethernet@7d580000
PCIe BRCM: link up, 5.0 Gbps x1 (SSC)
starting USB...
Bus xhci_pci: Register 5000420 NbrPorts 5
Starting the controller
USB XHCI 1.00
scanning bus xhci_pci for devices... 2 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found
Hit any key to stop autoboot:  2 ... 0 
U-Boot> 
```

U-Boot có giao diện dòng lệnh. Gõ `help` để hiển thị danh sách các lệnh.

```
U-Boot> help
?         - alias for 'help'
base      - print or set address offset
bdinfo    - print Board Info structure
blkcache  - block cache diagnostics and control
boot      - boot default, i.e., run 'bootcmd'
bootd     - boot default, i.e., run 'bootcmd'
bootefi   - Boots an EFI payload from memory
bootelf   - Boot from an ELF image in memory
booti     - boot Linux kernel 'Image' format from memory
bootm     - boot application image from memory
bootp     - boot image via network using BOOTP/TFTP protocol
bootvx    - Boot vxWorks from an ELF image
cmp       - memory compare
coninfo   - print console devices and information
cp        - memory copy
crc32     - checksum calculation
...
```

Sau khi U-Boot đã khởi động, ta có thể rút thẻ nhớ ra. Cắm lại thẻ vào máy tính, copy các file: kernel image, dtb và initramfs vào phân vùng đầu tiên của thẻ. Gắn lại thẻ nhớ vào board. Dùng các lệnh `fatload` để nạp ba file ấy vào RAM. Ví dụ với board Orange Pi Plus 2:

```
fatload mmc 0:1 82000000 uImage
fatload mmc 0:1 83000000 uInitrd
fatload mmc 0:1 84000000 sun8i-h3-orangepi-plus2e.dtb
```

Các lệnh trên nạp lần lượt kernel, initramfs, device tree vào địa chỉ 0x82000000, 0x83000000, 0x84000000 trên RAM. Sau đó dùng lệnh `bootm` để khởi động Linux:

```
bootm 82000000 83000000 84000000
```

### Tạo file Image
TODO:

### Chạy kernel