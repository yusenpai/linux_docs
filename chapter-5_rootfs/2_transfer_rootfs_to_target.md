# Chuyển rootfs sang board

## Mục lục
- [Chuyển rootfs sang board](#chuyển-rootfs-sang-board)
	- [Mục lục](#mục-lục)
	- [Yêu cầu kiến thức](#yêu-cầu-kiến-thức)
	- [Yêu cầu kỹ thuật](#yêu-cầu-kỹ-thuật)
	- [Từ khoá](#từ-khoá)
	- [Chuyển rootfs sang board](#chuyển-rootfs-sang-board-1)
	- [Tạo ra `initramfs`](#tạo-ra-initramfs)
	- [Boot `initramfs`](#boot-initramfs)
		- [Bắt đầu với QEMU](#bắt-đầu-với-qemu)
		- [Boot trên Raspberry Pi 4](#boot-trên-raspberry-pi-4)
		- [Mount `proc`](#mount-proc)
		- [Boot trên Orange Pi 2 Plus](#boot-trên-orange-pi-2-plus)
	- [Build `initramfs` vào trong kernel image](#build-initramfs-vào-trong-kernel-image)


## Yêu cầu kiến thức

## Yêu cầu kỹ thuật

## Từ khoá

## Chuyển rootfs sang board

Bây giờ bạn đã có một root filesystem hoàn chỉnh nằm trong `$HOME/rootfs`. Công việc còn lại chỉ là đem nó lên board và chạy thôi. Nhưng làm sao? Có ba phương án để thực hiện:

- `initramfs`: TODO

- Disk image (ảnh đĩa): TODO

- Network filesystem (NFS): TODO

Ở bài này mình sẽ dùng `initramfs` để làm ví dụ.

## Tạo ra `initramfs`

`initramfs`, viết tắt của **initial RAM filesystem**, được nén ở định dạng `cpio`. `cpio` là một định dạng cũ, khá giống với `RAR` hay `ZIP` những dễ để giải nén và ít code trong kernel hơn. Bạn tìm tới thư mục `$HOME/rootfs` rồi nén nó thành `initramfs`:

```sh
cd $HOME/rootfs
find . | cpio -H newc -ov --owner root:root > ../initramfs.cpio
cd ..
gzip initramfs.cpio
mkimage -A arm -O linux -T ramdisk -d initramfs.cpio.gz uRamdisk
# Nếu bạn build cho arm64, chạy cái này:
# mkimage -A arm64 -O linux -T ramdisk -d initramfs.cpio.gz uRamdisk
```

## Boot `initramfs`

### Bắt đầu với QEMU

QEMU có option là -initrd để bạn truyền initramfs vào bộ nhớ. Cho là bạn đã build một cái kernel 32bit (ở chương 4), tên là `zImage` với toolchain `arm-none-linux-gnueabihf` và device tree cho board Versatile PB. Cùng với `initramfs` vừa tạo, chứa BusyBox, bạn có thể chạy lệnh sau:

```sh
qemu-system-arm -m 256M -nographic -M versatilepb -kernel zImage \
				-append "console=ttyAMA0 rdinit=/bin/sh" \
				-dtb versatile-pb.dtb \
				-initrd initramfs.cpio.gz
```

### Boot trên Raspberry Pi 4

Với Raspberry Pi 4, bạn cần thẻ nhớ SD đã chuẩn bị sẵn có `u-boot` và `kernel` ở *chương 3, bootloader* và *chương 4, kernel*. Copy file `uRamdisk` vào phân vùng boot:

```sh
cd $HOME
sudo mount /dev/sdX1 bootfs
sudo cp uRamdisk bootfs
sudo umount /dev/sdX1
```

Cắm thẻ SD rồi khởi động board. Trong giao diện U-Boot, bạn chạy `fatload` để nạp `initramfs`:

```sh
setenv bootargs "8250.nr_uarts=1 console=ttyS0,115200 rdinit=/bin/sh"
fatload mmc 0:1 2000000 Image
fatload mmc 0:1 4000000 uRamdisk
fatload mmc 0:1 6000000 bcm2711-rpi-4-b.dtb
booti 0x2000000 0x4000000 0x6000000

```

Dòng `rdinit=/bin/sh` cho kernel biết rằng, bạn muốn chạy `/bin/sh` là chương trình `init`. Khi kernel khởi động xong thì bạn có thể gõ vài dòng lệnh `sh` ngay.

### Mount `proc`

Như đã giải thích ở phần [trước](1_what_should_be_in_rootfs.md), bạn cần mount `proc` để "nhìn" vào kernel khi nó hoạt động. Chẳng hạn bạn muốn liệt kê các tiến trình đang chạy với lệnh `ps`:

```
ps -a
```

...sẽ chẳng có gì xảy ra cả. Bây giờ bạn mount `proc` với lệnh

```sh
mount -t proc proc /proc
```

thì lệnh `ps` mới chạy được.

### Boot trên Orange Pi 2 Plus

Bạn làm tương tự với Raspberry Pi 4:

- Sử dụng thẻ SD đã chuẩn bị từ các chương trước
- Copy initramfs vào phân vùng boot
- Gõ vài lệnh fatload rồi boot kernel

## Build `initramfs` vào trong kernel image

Bạn có thể nhúng `initramfs` vào thẳng kernel: TODO

