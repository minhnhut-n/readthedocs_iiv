Filesystem
==========

FS là một mã nguồn code, trong đó phần lớn là các data struct (cấu trúc dữ liệu)
được dùng để tổ chức các dữ liệu của hệ thống, tổ chức ở đây bao gồm các hành động
lưu trữ, truy xuất, quản lý các hệ thống tệp tin. Tóm lại nó là một nền tảng trong 
việc tương tác với các dữ liệu. Ngoài ra có thể hiểu, FS là khâu trung gian giữa
các app/services và block devices.

File system có nhiều loại, tùy thuộc vào hệ thống và hệ điều hành mà nó hỗ trợ:
- ext4
- procfs
- sysfs
- devtmpfs
- tmpfs

Mỗi loại có những ưu nhược điểm khác nhau dùng để tối ưu về bộ nhớ hoặc tốc độ truy
xuất dữ liệu theo yêu cầu của target system. File system suy cho cùng cũng là một thuật
toán, mà thuật toán thì có giới hạn hoàn hảo của nó cho từng các trường hợp khác nhau.

Một hệ thống có thể có nhiều file system phục vụ cho nhiều mục đích khác nhu, việc mix
các thuật toán này theo usecase làm tăng tính linh hoạt và performance của system về
nhiều mặt. BIG-O là một phương pháp tiêu chuẩn để nói về độ phức tạp của thuật toán, trong
tình huống này sẽ xem xét các khía canh như 
-  tìm kiếm
-  đoc/ ghi file
-  định vị file
dựa trên số lượng dữ liệu mà thuật toán phải handle.

Cấu tạo cơ bản của một file system
----------------------------------

.. list-table:: Cấu trúc phân vùng file system
   :header-rows: 1
   :widths: 18 18 18 25 21

   * - ``SUPERBLOCK``
     - ``BLOCK BITMAP``
     - ``INODE BITMAP``
     - ``INODE TABLE``
     - ``DATA BLOCKS``
   * - Kích thước FS
     - Quản lý trạng thái sử dụng của Data Blocks
     - Quản lý trạng thái sử dụng của Inodes
     - Danh sách Inodes; mỗi Inode chứa metadata của file
     - Vùng chứa nội dung file thực tế / thư mục
   * - Số Inodes / Blocks
     - Dải bit, ví dụ: ``1011001...``
     - Dải bit, ví dụ: ``1110001...``
     - Mỗi inode lưu file type, permission, owner, size, timestamps
     - Nơi lưu dữ liệu thực tế của file và thư mục

Vai trò của từng Block
----------------------

- ``SUPERBLOCK``: Trái tim của FS. Lưu metadata tổng quan: tổng số block, số inode rảnh, kích thước block.
- ``BLOCK BITMAP``: Dải bit đánh dấu block nào đã dùng và block nào còn trống.
- ``INODE BITMAP``: Dải bit đánh dấu inode nào đã dùng và inode nào còn trống.
- ``INODE TABLE``: Tập hợp các cấu trúc ``struct inode``. Mỗi inode chứa thông tin về file như type, permission, kích thước, timestamps, và con trỏ tới data block.
- ``DATA BLOCKS``: Vùng lưu nội dung thực của file hoặc thư mục.

Một inode điển hình có thể mô tả như sau:

.. code-block:: text

   Inode #1024
   - File Type: Regular File
   - Permissions: rwxr-xr-- (0754)
   - Owner/Group: 1000/1000
   - File Size: 12,500 Bytes
   - Timestamps: atime, mtime, ctime
   - Data Pointers / Extents: Block #5001, #5002, #5003

Những gì có thể khác giữa các file system
-----------------------------------------

Ý của tôi là các file system đều khác nhau nhưng về cơ bản chúng nó được xây dựng theo một khuôn mẫu cụ thể
Vậy chúng nó khác nhau ở những điểm nào ?

SỰ KHÁC BIỆT CẤU TRÚC GIỮA CÁC LOẠI FILE SYSTEM

.. list-table:: So sánh các loại file system
   :header-rows: 1
   :widths: 32 32 36

   * - Lưu trữ truyền thống
     - Lưu trữ hiện đại
     - Pseudo / In-Memory
   * - ``ext4``, ``FAT32``
     - ``Btrfs``, ``ZFS``
     - ``procfs``, ``sysfs``, ``tmpfs``
   * - Dùng bảng cố định
     - Cấu trúc cây B-Tree
     - Không có đĩa thật
   * - Cấp phát inode tĩnh
     - Copy-on-Write (CoW)
     - Dữ liệu nằm trên RAM
   * - Cần quét phục lỗi
     - Snapshots tức thì
     - Tạo động bởi kernel

ext4
----

Fourth extended file system, là gen 4th của loại storage file system extX, vài trò của
nó chính là để tương tác với các hệ thống lưu trữ như eMMC, EEPROM, SSD, HDD,...

Ưu điểm:

- Tìm kiếm file: Sử dụng cấu trúc H-Tree (variant của B-Tree) -> Olog(N)
- Cấp phát không gian lưu trữ cho file lớn theo cơ chế extents thay vì block-to-block -> O(1)
- Metadata look-up, mỗi file được gắn cho một inode, việc đọc các thuộc tính của file diễn ra nhanh -> O(1)
- Phục hồi sự cố dữ liệu, journaling, được ghi vào trong metadata trước khi lưu xuống ổ đĩa thật -> O(J)

Nhược điểm:

- Khi đĩa gần tiến đến hết dung lượng, các block lớn liên tiếp sẽ không được cấp phát, dẫn đến dữ liệu
  sẽ bị chi thành K extents khác nhau làm phân mảnh dữ liệu.
- Số lượng inode là cố định khi vừa format ổ đĩa, không có cơ chế dynamic allocator, do vậy không thể cấp
  thêm inode khi cần dù đôi khi ổ đĩa còn trên 50% dung lượng.
- Không có cơ chế CoW, khi một file được tạo nó sẽ có thời gian O(k) để backup lại file đó mà không trả về
  ngay lập tức như các file system có copy on write.

.. code-block:: bash

   # Xem thông tin filesystem
   $ df -hT

   # Kiểm tra dung lượng ổ đĩa
   $ df -h

   # Kiểm tra dung lượng thư mục
   $ du -sh /path/to/dir

procfs
------

Là filesystem ảo (pseudo filesystem) mount tại ``/proc``, cung cấp thông tin về
kernel và processes đang chạy. Mỗi process có một thư mục riêng với PID tương ứng
bên trong ``/proc``.

.. code-block:: bash

   # Xem thông tin CPU
   $ cat /proc/cpuinfo

   # Xem thông tin memory
   $ cat /proc/meminfo

   # Xem thông tin process cụ thể (PID=1 là init/systemd)
   $ cat /proc/1/status

sysfs
-----

Là filesystem ảo mount tại ``/sys``, cung cấp thông tin về hardware devices,
drivers và kernel objects. sysfs xuất hiện từ kernel 2.6 để thay thế cho các
file rác rưởi trong procfs.

.. code-block:: bash

   # Xem danh sách block devices
   $ ls /sys/block/

   # Xem thông tin về một device cụ thể (vd: sda)
   $ cat /sys/block/sda/size

   # Xem class devices
   $ ls /sys/class/

devtmpfs
--------

Là filesystem ảo mount tại ``/dev``, tự động tạo các device nodes cho hardware
được kernel phát hiện. Trước đây, việc tạo device nodes phải làm thủ công với
``mknod``, devtmpfs tự động hóa quá trình này.

.. code-block:: bash

   # Xem các device nodes
   $ ls /dev/

   # Một số device nodes phổ biến:
   # /dev/sda   - ổ cứng SCSI/SATA đầu tiên
   # /dev/ttyUSB0 - USB-to-Serial adapter
   # /dev/mmcblk0 - thẻ nhớ SD/MMC
   # /dev/null  - "black hole" device

tmpfs
-----

Là filesystem ảo lưu trữ dữ liệu trong RAM (volatile memory). Dữ liệu trên tmpfs
sẽ mất khi reboot. Thường được dùng cho các file tạm thời, cache, hoặc shared memory.

.. code-block:: bash

   # Xem các tmpfs đang mounted
   $ df -hT | grep tmpfs

   # Mount tmpfs thủ công
   $ sudo mount -t tmpfs -o size=100M tmpfs /mnt/mytmp
