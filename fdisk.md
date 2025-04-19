# Hướng dẫn sử dụng `fdisk` để phân vùng cho thẻ SD

- [Hướng dẫn sử dụng `fdisk` để phân vùng cho thẻ SD](#hướng-dẫn-sử-dụng-fdisk-để-phân-vùng-cho-thẻ-sd)
	- [`fdisk`](#fdisk)
	- [Sử dụng](#sử-dụng)
		- [Định dạng thẻ SD cho Linux](#định-dạng-thẻ-sd-cho-linux)
		- [Ghi dữ liệu vào phân vùng không được đánh dấu](#ghi-dữ-liệu-vào-phân-vùng-không-được-đánh-dấu)

## `fdisk`

`fdisk` là công cụ dòng lệnh trên Linux giúp ta quản lý phân vùng trên các thiết bị lưu trữ.

Một số khái niệm cần nắm:

- Phân vùng (partition): là một vùng nhớ trong một thiết bị lưu trữ dữ liệu. 
Trên một thiết bị có thể có nhiều phân vùng, các phân vùng không chồng lấn lên nhau.
Mỗi phân vùng được đánh dấu bởi điểm đầu và điểm cuối: `first sector` và `last sector`.
- Sector: là đơn vị dữ liệu nhỏ nhất có thể đọc, ghi, xoá. Thường thì 1 sector = 512 bytes.
- Bảng phân vùng: Là bảng chứa danh sách các phân vùng, bao gồm first sector, last sector, kích thước (last sector - first sector) và kiểu định dạng của phân vùng đó.
- Định dạng phân vùng (filesystem): Có nhiều định dạng cho phân vùng. Thường dùng `FAT32` cho phân vùng boot, `ext4` cho root filesytem. Ngoài ra còn có một số phân vùng khác: `NTFS` của Windows, `APFS` của Apple,...
- Sơ đồ phân vùng (partitioning schemes): Thường sử dụng Master Boot Record (MBR). Tham khảo thêm tại Google.

## Sử dụng

Đầu tiên ta tìm xem device nào tương ứng với thẻ nhớ SD. Dùng lệnh `lsblk` để liệt kê toàn bộ block device:

```sh
parallels@ubuntu-linux-22-04-desktop:~/u-boot$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
...
sda      8:0    0   192G  0 disk 
├─sda1   8:1    0     1G  0 part /boot/efi
└─sda2   8:2    0 111.4G  0 part /
sdb      8:16   1  29.1G  0 disk 
├─sdb1   8:17   1    50M  0 part /media/parallels/CE4D-3000
└─sdb2   8:18   1  29.1G  0 part /media/parallels/6e1d235a-11f0-49ff-a0d3-409d7261dfab
sr0     11:0    1  1024M  0 rom  

```

`sdb` là device đại diện cho thẻ nhớ (vì có kích thước gần 32G - tương ứng với thẻ nhớ). `sda` là ổ cứng chứa Linux.

Sau khi xác định được device của thẻ SD, dùng lệnh `sudo fdisk /dev/sdX`:

```sh
parallels@ubuntu-linux-22-04-desktop:~/u-boot$ sudo fdisk /dev/sdb
Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help): 
```

Nhập phím `m` để hiển thị danh sách các lệnh. Một số lệnh thường dùng:

- p: in bảng phân vùng của ổ địa đang chọn
- o: tạo bảng phân vùng DOS mới
- n: tạo phân vùng mới
- d: xoá phân vùng
- t: thay đổi định dạng phân vùng

### Định dạng thẻ SD cho Linux

Để xoá toàn bộ phân vùng của thẻ SD, quay lại terminal (bấm Ctrl + C), rồi dùng lệnh `sudo wipefs /dev/sdX`.

Sau khi xoá xong, dùng lệnh `sudo fdisk /dev/sdX` để quay lại fdisk. Thực hiện các bước sau:

- p: Kiểm tra bảng phân vùng. Vì đã xoá bằng phân vùng bằng `wipefs` nên sẽ không hiển thị bảng nào cả.
- o: Tạo bảng phân vùng mới
- n: Tạo phân vùng mới. `fdisk` sẽ hỏi phân vùng số mấy (1-4), loại nào (primary hay extend), sector bắt đầu và sector kết thúc.
	
	Ví dụ: tạo phân vùng 1, kích thước 50M, bắt đầu từ vị trí 20M trong thẻ nhớ:

	```sh
	Command (m for help): n
	Partition type
	p   primary (0 primary, 0 extended, 4 free)
	e   extended (container for logical partitions)
	Select (default p): p
	Partition number (1-4, default 1): 1
	First sector (0-4294967295, default 0): 40960
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (40960-4294967295, default 4294967295): +50M

	Created a new partition 1 of type 'Linux' and of size 50 MiB.
	```

	First sector là 40960 vì mỗi sector là 512 byte, 40960 x 512 byte = 20Mbyte. Last sector có thể ghi +50M, fdisk sẽ tự tính toán last sector.
- t: Đổi loại định dạng cho phân vùng từ Linux thành FAT32:

	```sh
	Command (m for help): t
	Selected partition 1
	Hex code or alias (type L to list all): c
	Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'.
	```

	Như vậy là hoàn tất tạo phân vùng. Làm tương tự để tạo phân vùng 2 ngay kế tiếp phân vùng 1. Phân vùng 2 để định dạng là ext4 (Linux). 
	Bước Last sector có thể để mặc định thì fdisk tự động lấy toàn bộ sector còn lại cho phân vùng 2:

	```sh
	Command (m for help): n
	Partition type
	p   primary (1 primary, 0 extended, 3 free)
	e   extended (container for logical partitions)
	Select (default p): p
	Partition number (2-4, default 2): 
	First sector (0-4294967295, default 0): 143360
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (143360-4294967295, default 4294967295): 

	Created a new partition 2 of type 'Linux' and of size 2 TiB.

	Command (m for help): p
	Disk /dev/sdb: 0 B, 0 bytes, 0 sectors
	Units: sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disklabel type: dos
	Disk identifier: 0xd3f4c7f0

	Device     Boot  Start        End    Sectors Size Id Type
	/dev/sdb1        40960     143359     102400  50M  c W95 FAT32 (LBA)
	/dev/sdb2       143360 4294967295 4294823936   2T 83 Linux
	```

- w: Ghi bảng phân vùng vừa tạo vào thẻ SD.
  
Lúc này thẻ SD đã có 2 phân vùng, một phân vùng FAT32 dành cho boot, phân vùng ext4 chứa root-filesytem. Ta định dạng các phân vùng bằng cách lệnh sau:

```sh
mkfs.vfat -F 32 /dev/sdX1
mkfs.ext4 /dev/sdX2
```

### Ghi dữ liệu vào phân vùng không được đánh dấu

Giả sử thẻ SD sau định dạng có bảng phân vùng như sau:

```sh
Device     Boot  Start        End    Sectors Size Id Type
/dev/sdb1        40960     143359     102400  50M  c W95 FAT32 (LBA)
/dev/sdb2       143360 4294967295 4294823936   2T 83 Linux
```

- Phân vùng 1: bắt đầu từ sector 40960 (20M) đến 143359 (70M).
- Phân vùng 2: bắt đầu từ sector 143360 (70M) đến cuối.

Trong đó còn một phân vùng không được đánh dấu, từ sector 0 (0M) tới sector 40959 (20M). Phân vùng này thường được để dành cho bootloader của board. Muốn ghi vào phân vùng này ta dùng lệnh `dd`. Ví dụ:

```sh
sudo dd if=hello.txt of=/dev/sdX bs=1k seek=100
```

Lệnh trên thực hiện ghi file `hello.txt` vào vị trí 100k trên thẻ nhớ.

Lệnh `dd` khá nguy hiểm vì nếu không dùng đúng cách, các phân vùng trên thẻ SD sẽ bị phá huỷ.