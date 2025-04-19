# Quá trình khởi động của một board chạy Linux

## Mục lục

- [Quá trình khởi động của một board chạy Linux](#quá-trình-khởi-động-của-một-board-chạy-linux)
	- [Mục lục](#mục-lục)
	- [Boot sequence](#boot-sequence)
	- [ROM Code](#rom-code)
	- [Secondary Program Loader - SPL](#secondary-program-loader---spl)
	- [Tertiary Program Loader - TPL](#tertiary-program-loader---tpl)

## Boot sequence

Quá trình khởi động của hệ thống trải qua 3 giai đoạn:

- ROM code
- SPL
- TPL

## ROM Code

Khi vừa khởi động hoặc nhấn nút RESET, một đoạn chương trình được chạy. Đoạn chương trình này được lập trình trong quá trình sản xuất chip, được lưu trữ ở vùng nhớ ghi một lần (ROM) hoặc bộ nhớ FLASH nội, nên nó được gọi là **ROM code**. Đoạn chương trình này có nhiệm vụ tìm kiếm và nạp SPL từ thẻ SD, bộ nhớ SPI Flash, NAND Flash, EEPROM I2C; hoặc thậm chí nạp qua giao tiếp UART, Ethernet hay USB.

ROM Code thường không khởi tạo bộ điều khiển DRAM. Bộ nhớ RAM mà chương trình này sử dụng đến từ `SRAM`, là RAM nội bên trong con chip, kích thước rất bé (tối đa vài MB).

Ở cuối giai đoạn ROM code, SPL đã được nạp sẵn vào SRAM và chờ thực thi.

## Secondary Program Loader - SPL

SPL phải khởi tạo bộ điều khiển DRAM để truy cập được tới vùng nhớ DRAM lớn hơn nhiều so với SRAM. SPL có thể đọc các chương trình TPL nằm trong thẻ SD, bộ nhớ,... rồi nạp nó vào DRAM.

SPL thường được cung cấp sẵn từ hãng.

## Tertiary Program Loader - TPL

Đây là bootloader hoàn chỉnh, ví dụ như U-boot. TPL thường có giao diện command-line đơn giản để giao tiếp với nó. Ta có thể nạp Linux kernel, device tree, initramfs vào DRAM để chuẩn bị khởi động Linux.

Ở cuối giai đoạn này, TPL phải truyền cho kernel Linux một số thông tin:

- Machine number: dành cho lõi ARM cũ không hỗ trợ device tree
- Thông tin cơ bản về board, bao gồm kích thước và vị trí của RAM vật lý (DRAM) và tốc độ CPU.
- Kernel command line
- Device tree (tuỳ chọn)
- initramfs (tuỳ chọn)