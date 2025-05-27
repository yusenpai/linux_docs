# Hướng dẫn build và cài đặt U-Boot

## Mục lục
- [Hướng dẫn build và cài đặt U-Boot](#hướng-dẫn-build-và-cài-đặt-u-boot)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Từ khoá](#từ-khoá)
	- [Giới thiệu về U-Boot](#giới-thiệu-về-u-boot)
	- [Build U-Boot](#build-u-boot)
		- [Cài đặt cross compiler](#cài-đặt-cross-compiler)
		- [Tải U-Boot từ source:](#tải-u-boot-từ-source)
		- [Build U-Boot cho Raspberry Pi 4](#build-u-boot-cho-raspberry-pi-4)
		- [Build U-Boot cho Orange Pi Plus 2](#build-u-boot-cho-orange-pi-plus-2)
	- [Cài đặt U-Boot](#cài-đặt-u-boot)
		- [Cài đặt U-Boot cho Rasberry Pi 4](#cài-đặt-u-boot-cho-rasberry-pi-4)
		- [Cài đặt U-Boot cho Orange Pi 2 Plus](#cài-đặt-u-boot-cho-orange-pi-2-plus)
	- [Sử dụng U-Boot](#sử-dụng-u-boot)
		- [Biến môi trường](#biến-môi-trường)
		- [Định dạng file ảnh U-Boot](#định-dạng-file-ảnh-u-boot)
		- [Nạp file ảnh từ thẻ nhớ](#nạp-file-ảnh-từ-thẻ-nhớ)
	- [Khởi động Linux](#khởi-động-linux)
		- [Tự động hoá với U-Boot scripts](#tự-động-hoá-với-u-boot-scripts)

## Yêu cầu kiến thức

- Biết cách sử dụng `make` để build U-Boot
- Biết cách sử dụng các lệnh shell cơ bản
- Biết cách sử dụng lệnh `dd`

## Yêu cầu kỹ thuật

- Cross compiler aarch64-none-linux-gnu cho ARM64 và arm-none-linux-gnueabihf cho ARM
- Thẻ SD tối thiểu 8G đã được định dạng. Tham khảo cách định dạng tại (TODO: thêm đường dẫn)
- Board Raspberry Pi 4 hoặc Orange Pi Plus 2

## Từ khoá

- Kernel Image: Dịch tạm là "file ảnh kernel".
- File ảnh kernel: Nói tới file được tạo ra sau khi build kernel, thường có tên là `Image` hoặc `zImage`.
- scripts: gồm nhiều câu lệnh. Ở đây nói về nhiều câu lệnh U-Boot nhằm thực thi một tác vụ nào đó.

## Giới thiệu về U-Boot

U-Boot, tên đầy đủ là **Das U-Boot**, ban đầu là một open source bootloader bo mạch PowerPC nhúng. Sau đó nó được port sang ARM và sau này là các kiến trúc khác như MIPS và SH.

## Build U-Boot

### Cài đặt cross compiler

Để build U-Boot, các bạn sử dụng cross-toolchain phù hợp. Hướng dẫn cài đặt cross-toolchain nằm ở chương 2 (TODO: add link)

Bài này sử dụng hai cross-toolchain:
- aarch64-none-linux-gnu: Cho Raspberry Pi 4 64 bit
- arm-none-linux-gnueabihf: Cho Orange Pi 2+ 32 bit

### Tải U-Boot từ source:

```
git clone https://github.com/u-boot/u-boot.git
cd u-boot
# git checkout v2024.01 (chạy lệnh này nếu bạn muốn đổi phiên bản khác)
```

Có hơn 1000 cấu hình cho các board phổ biến nằm trong thư mục `configs/`. Bạn có thể dựa trên tên file cấu hình để đoán xem nó dành cho board nào. Để cho chắc, bạn có thể xem file README trong thư mục `board/`, hoặc tìm thông tin trên web hoặc docs của U-Boot.

Ví dụ cho board Raspberry Pi.

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

Theo tài liệu xem tại [đây](https://docs.u-boot.org/en/latest/board/broadcom/raspberrypi.html), thì board Raspberry Pi 4 64 bit có thể sử dụng rpi_4_defconfig hoặc rpi_arm64_defconfig

### Build U-Boot cho Raspberry Pi 4
Build U-Boot với `aarch64-none-linux-gnu` toolchain và cấu hình `rpi_4_defconfig`:

```
make distclean
make CROSS_COMPILE=aarch64-none-linux-gnu- rpi_4_defconfig
make CROSS_COMPILE=aarch64-none-linux-gnu- -j(nproc)
```

Kết quả của quá trình build là các file sau:

- `u-boot`:  U-Boot ở định dạng đối tượng ELF, phù hợp để sử dụng với debugger
- `u-boot.map`: Bảng ký hiệu (symbol table)
- `u-boot.bin`: U-Boot ở định dạng nhị phân, phù hợp để chạy trực tiếp trên board
- `u-boot.srec`: U-Boot ở định dạng Motorola S-record (SRECORD hoặc SRE), phù hợp để truyền qua kết nối Serial

```
parallels@ubuntu-linux-22-04-desktop:~/u-boot$ ls -l u-boot*
-rwxrwxr-x 1 parallels parallels 6749872 May 11 16:12 u-boot
-rwxrwxr-x 1 parallels parallels  702056 May 11 16:12 u-boot-nodtb.bin
-rwxrwxr-x 1 parallels parallels  702056 May 11 16:12 u-boot.bin
-rw-rw-r-- 1 parallels parallels   14021 May 11 16:12 u-boot.cfg
-rw-rw-r-- 1 parallels parallels    1315 May 11 16:12 u-boot.lds
-rw-rw-r-- 1 parallels parallels  986555 May 11 16:12 u-boot.map
-rwxrwxr-x 1 parallels parallels 2017752 May 11 16:12 u-boot.srec
-rw-rw-r-- 1 parallels parallels  273922 May 11 16:12 u-boot.sym
```

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
- bcm2711-rpi-4-b.dtb: Device tree binary dành cho Raspberry Pi 4
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
U-Boot 2025.04-01080-g8c98b57d72d5 (May 11 2025 - 16:12:20 +0700)

DRAM:  948 MiB (total 1.9 GiB)
RPI 4 Model B (0xb03115)
Core:  215 devices, 17 uclasses, devicetree: board
MMC:   mmcnr@7e300000: 1, mmc@7e340000: 0
Loading Environment from FAT... Unable to read "uboot.env" from mmc0:1... 
In:    serial,usbkbd
Out:   serial,vidconsole
Err:   serial,vidconsole
Net:   eth0: ethernet@7d580000
PCIe BRCM: link up, 5.0 Gbps x1 (SSC)
starting USB...
Bus xhci_pci: Register 5000420 NbrPorts 5
Starting the controller
USB XHCI 1.00
scanning bus xhci_pci for devices... 2 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found
Hit any key to stop autoboot:  0 
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

Số trong U-Boot được hiểu là số Hexa. Ví dụ dòng lệnh sau:
```
U-Boot> fatload mmc 0:1 2000000 Image
```

Lệnh này sẽ tìm và nạp file có tên `Image` ở trong phân vùng 1 của thẻ SD (mmc 0:1) vào RAM từ địa chỉ `0x02000000`.

### Biến môi trường

U-Boot sử dụng biến môi trường rất nhiều để lưu trữ và truyền thông tin giữa các lệnh với nhau. Biến môi trường là các cặp `name=value` (giống với biến môi trường trong shell), được lưu trữ trong một vùng bộ nhớ.

Bạn có thể tạo biến môi trường với lệnh `setenv`:
```
setenv bootargs "root=/dev/mmcblk0p2"
```
Lệnh này tạo một biến tên là `bootargs`, có giá trị là `root=/dev/mmcblk0p2`.

Dùng lệnh `printenv` để in tất cả các biến môi trường:
```
U-Boot> printenv
arch=arm
baudrate=115200
board=rpi
board_name=4 Model B
board_rev=0x11
board_rev_scheme=1
board_revision=0xB03115
boot_targets=mmc usb pxe dhcp
bootcmd=bootflow scan
bootdelay=2
cpu=armv8
dfu_alt_info=u-boot.bin fat 0 1;uboot.env fat 0 1; config.txt fat 0 1;Image fat 0 1
dhcpuboot=usb start; dhcp u-boot.uimg; bootm
ethaddr=d8:3a:dd:9c:20:1b
fdt_addr=2eff2000
fdt_addr_r=0x02600000
fdt_high=ffffffffffffffff
fdtcontroladdr=3af37dd0
fdtfile=broadcom/bcm2711-rpi-4-b.dtb
initrd_high=ffffffffffffffff
kernel_addr_r=0x00080000
loadaddr=0x1000000
preboot=pci enum; usb start;
pxefile_addr_r=0x02500000
ramdisk_addr_r=0x02700000
scriptaddr=0x02400000
serial#=1000000036d699aa
soc=bcm283x
stderr=serial,vidconsole
stdin=serial,usbkbd
stdout=serial,vidconsole
usb_ignorelist=0x1050:*,
usbethaddr=d8:3a:dd:9c:20:1b
vendor=raspberrypi

Environment size: 827/16380 bytes
```

### Định dạng file ảnh U-Boot

U-Boot không sử dụng filesystem. Thay vì thế, nó gắn một đoạn header dài 64 byte vào file, chứa thông tin về file đó. Ta chuẩn bị các file cho U-Boot bằng `mkimage`, là một tool bên trong gói `u-boot-tools`. Cài gói bằng `apt`:

```
sudo apt install u-boot-tools
```

Sau đó bạn có thể sử dụng `mkimage`:
```
parallels@ubuntu-linux-22-04-desktop:~/linux$ mkimage
Error: Missing output filename
Usage: mkimage -l image
          -l ==> list image header information
       mkimage [-x] -A arch -O os -T type -C comp -a addr -e ep -n name -d data_file[:data_file...] image
          -A ==> set architecture to 'arch'
          -O ==> set operating system to 'os'
          -T ==> set image type to 'type'
          -C ==> set compression type 'comp'
          -a ==> set load address to 'addr' (hex)
          -e ==> set entry point to 'ep' (hex)
          -n ==> set image name to 'name'
          -d ==> use image data from 'datafile'
          -x ==> set XIP (execute in place)
       mkimage [-D dtc_options] [-f fit-image.its|-f auto|-F] [-b <dtb> [-b <dtb>]] [-E] [-B size] [-i <ramdisk.cpio.gz>] fit-image
           <dtb> file is used with -f auto, it may occur multiple times.
          -D => set all options for device tree compiler
          -f => input filename for FIT source
          -i => input filename for ramdisk file
          -E => place data outside of the FIT structure
          -B => align size in hex for FIT structure and header
Signing / verified boot options: [-k keydir] [-K dtb] [ -c <comment>] [-p addr] [-r] [-N engine]
          -k => set directory containing private keys
          -K => write public keys to this .dtb file
          -G => use this signing key (in lieu of -k)
          -c => add comment in signature node
          -F => re-sign existing FIT image
          -p => place external data at a static position
          -r => mark keys used as 'required' in dtb
          -N => openssl engine to use for signing
       mkimage -V ==> print version information and exit
Use '-T list' to see a list of available image types
```

Ví dụ, để chuẩn bị file ảnh kernel cho vi xử lý ARM 64 bit, bạn có thể dùng lệnh sau:

```
$ mkimage -A arm64 -O linux -T kernel -C gzip -a 0x00080000 -e 0x00080000 -n 'Linux' -d Image uImage
```

Với ví dụ này, kiến trúc là `arm64`, hệ điều hành là `linux`, kiểu file ảnh là `kernel`. Thêm vào đó, kiểu nén là `gzip`, địa chỉ nạp (load address) và địa chỉ chạy (entry point) là `0x00080000`. File ảnh có tên là `Linux`, file ban đầu là `Image`, file được tạo ra là `uImage`.

### Nạp file ảnh từ thẻ nhớ

Thường thì bạn sẽ nạp file ảnh kernel từ thẻ nhớ hoặc qua mạng LAN (TODO). Thẻ nhớ được điều khiển bởi `mmc` có trong U-Boot. Cách nạp file ảnh vào bộ nhớ RAM như sau:

```
U-Boot> mmc rescan
U-Boot> fatload mmc 0:1 2000000 uImage
23802432 bytes read in 1017 ms (22.3 MiB/s)
U-Boot> iminfo 2000000

## Checking Image at 02000000 ...
   Legacy image found
   Image Name:   Linux
   Image Type:   AArch64 Linux Kernel Image (gzip compressed)
   Data Size:    23802368 Bytes = 22.7 MiB
   Load Address: 00200000
   Entry Point:  00200000
   Verifying Checksum ... OK
U-Boot> 
```

`mmc rescan` dùng để phát hiện thẻ SD mới được gắn. `fatload` dùng để đọc một file từ phân vùng định dạng FAT trên thẻ nhớ. Cách dùng `fatload` như sau:

```
fatload <interface> <dev:part> <addr> <filename> [bytes [pos]]
```

`<interface>` trong trường hợp này là `mmc`. `<dev:part>` là số của thiết bị `mmc`, đếm từ 0; và số của phần vùng trên thiết bị. Vì thế, `<0:1>` nghĩa là phân vùng 1 của thiết bị đầu tiên, mà `mmc 0` là thẻ SD (với `mmc 1` là eMMC trên board).

Địa chỉ bộ nhớ, 0x02000000 là địa chỉ trong RAM mà file ảnh kernel sẽ được nạp vào.

> Lưu ý: Khi chạy kernel, kernel sẽ được giải nén và ghi vào vùng nhớ có địa chỉ 0x00080000 - là địa chỉ trước đó đã sử dụng trong lệnh `mkimage`. Ta phải chọn địa chỉ nạp kernel sao cho không bị ghi đè, chẳng hạn địa chỉ 0x02000000

## Khởi động Linux

Bạn chạy kernel và khởi động Linux với lệnh `bootm`:

```
bootm [địa chỉ của kernel] [địa chỉ của ramdisk] [địa chỉ của
dtb]
```

Ví dụ, kernel được nạp vào `0x02000000`, device tree được nạp vào `0x06000000`, cú pháp sẽ là:

```
bootm 2000000 - 6000000
```

Địa chỉ của Ramdisk bị bỏ trống (thay bằng dấu `-`) vì Ramdisk là tuỳ chọn.

### Tự động hoá với U-Boot scripts

Gõ một đoạn lệnh dài chỉ để khởi động Linux là điều không thể chấp nhận. Do đó, một biến môi trường đặc biệt có tên là `bootcmd` được dùng. `bootcmd` chứa một scripts, được chạy ngay khi U-Boot khởi động. Ta có thể viết những dòng lệnh nạp kernel, device tree, `bootm` vào biến `bootcmd` để tự động chạy Linux:

```
setenv bootcmd fatload mmc 0:1 2000000 uImage\; fatload mmc 0:1 6000000 bcm2711-rpi-4-b.dtb\; bootm 0x2000000 - 0x6000000
```

Mỗi dòng lệnh cách nhau bởi dấu `;`, mỗi dấu `;` buộc phải có dấu `\` đằng trước.

Bạn có thể lưu các biến môi trường vào chính thẻ nhớ SD. Khi U-Boot khởi động, nó sẽ đọc biến `bootcmd` rồi thực thi những lệnh trên để tự động khởi động Linux.