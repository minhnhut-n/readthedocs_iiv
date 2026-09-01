Filesystem
==========

- ext4
- procfs
- sysfs
- devtmpfs
- tmpfs

ext4
----

Là filesystem mặc định phổ biến nhất trên Linux, hỗ trợ dung lượng lớn lên đến 1 exabyte,
kích thước file tối đa 16 TB (với block size 4KB). ext4 là phiên bản kế thừa của ext3
với các cải tiến về hiệu suất, hỗ trợ journaling (phục hồi dữ liệu khi crash) và
extents (giảm fragmentation).

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
