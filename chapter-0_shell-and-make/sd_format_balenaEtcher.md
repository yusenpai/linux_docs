# Sử dụng balenaEtcher để ghi ảnh đĩa lên thẻ SD

- [Sử dụng balenaEtcher để ghi ảnh đĩa lên thẻ SD](#sử-dụng-balenaetcher-để-ghi-ảnh-đĩa-lên-thẻ-sd)
	- [balenaEtcher](#balenaetcher)
	- [Sử dụng](#sử-dụng)
	- [Ví dụ: Tạo thẻ SD chứa hệ điều hành Raspberry Pi OS.](#ví-dụ-tạo-thẻ-sd-chứa-hệ-điều-hành-raspberry-pi-os)

## balenaEtcher

balenaEtcher là một phần mềm dùng để ghi ảnh đĩa lên thẻ SD, với giao diện rất dễ sử dụng. Tải phần mềm tại https://etcher.balena.io

## Sử dụng

Đầu tiên bạn cần chuẩn bị:

- Một thẻ nhớ SD dung lượng đủ lớn (thường thì 8G trở lên)
- Ảnh đĩa cần ghi.
  
Gồm 3 bước thực hiện:

1. Chọn ảnh đĩa
2. Chọn thẻ SD
3. Bắt đầu ghi

![alt text](<./assets/Screenshot 2025-04-19 at 22.19.06.png>)

## Ví dụ: Tạo thẻ SD chứa hệ điều hành Raspberry Pi OS.

Raspberry Pi OS là hệ điều hành được phát triển dành riêng cho board Raspberry Pi. Tải về tại: https://www.raspberrypi.com/software/operating-systems/

Ví dụ mình chọn bản Raspberry Pi OS Lite 64bit. Phiên bản lite không có giao diện đồ hoạ, nên kích thước nhẹ hơn nhiều so với những phiên bản khác. Sau khi tải về, làm theo 3 bước hướng dẫn như trên để ghi ảnh đĩa vào thẻ SD.

Bên trong thẻ SD có hai phân vùng:

- `boot` (hoặc `bootfs`): Chiếm từ 50M - 1G, định dạng `FAT32`. Chứa những file cần thiết cho việc khởi tạo hệ thống, gồm: Linux kernel, device tree, initramfs,... Đối với Raspberry Pi 4 còn có thêm một số file khác như `bootcode.bin` `start4.elf` `fixup4.data`, tham khảo thêm ở https://www.raspberrypi.com/documentation/computers/configuration.html#boot-folder-content.

- `rootfs`: Chiếm phần lớn thẻ nhớ, định dạng `ext4`. Phân vùng này chứa Root Filesystem.