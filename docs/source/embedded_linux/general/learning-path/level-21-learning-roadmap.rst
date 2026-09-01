Level 21 – Embedded Linux Learning Roadmap
==========================================

A project-driven learning roadmap from basic Linux to professional embedded Linux,
applied on the Banana Pi M4 and i.MX6.

Prerequisites
-------------

This roadmap is a hands-on, project-driven path using two boards: Banana Pi M4 and i.MX6.
Follow the steps in small incremental changes, from basic Linux up to professional
embedded Linux.

1. Comfortable with Linux Operations
------------------------------------

- install armbian/debian on Banana Pi
- use ssh, user/groups, permissions, systemd, journalctl, ip, ss, mount, lsblk, top, htop, dmesg
- run an Nginx server and a small C program as systemd services

2. Learn Networking Using Both Boards
-------------------------------------

- Give each board a static DHCP lease
- SSH from one to the other using keys
- Run iperf3, transfer files with rsync, capture traffic with tcpdump
- Make Banana Pi host MQTT, make the i.MX6 publish sensor/status messages

3. Learn the Boot Process on i.MX6
----------------------------------

- Connect the serial console
- Stop in U-Boot, inspect environment variables, boot targets, memory, and boot commands
- Boot Linux, save the full boot log, then identify bootloader -> kernel -> device tree -> rootfs
- Do not modify bootloader storage until you can reliably boot from a known-good SD card

4. Learn Hardware Through Device Tree
-------------------------------------

- Find the active board model: ``cat /proc/device-tree/model``
- Inspect hardware: ``ls /sys/class``, ``lspci`` (if available), ``lsusb``, ``i2cdetect``, ``gpioinfo``, ``dmesg -w``
- Start with a GPIO led/button, then add I2C sensor, SPI display, and UART device
- Make small incremental changes: make one small Device Tree change—such as enabling a peripheral or defining an LED—then rebuild/deploy only the DTB (Device Tree Blob)

5. Build Your Own Embedded Image
--------------------------------

- Start with Buildroot: build a minimal root filesystem, cross-compile a "hello + GPIO" program, add it as a package, and boot it.
- Transition to Yocto: move to Yocto once Buildroot feels familiar. NXP's BSP documentation centers its image-building workflow on Yocto, making it valuable for professional i.MX work.

6. Finish with One Real Project
-------------------------------

- On i.MX6: C service reads GPIO/I²C data and publishes via MQTT.
- On Banana Pi: MQTT broker + SQLite/InfluxDB + simple dashboard.
- Add production features: implement systemd auto-start, log rotation, reconnect handling, watchdog behavior, and a deployment script.

Context & Prerequisites
-----------------------

Integration with Embedded C
....................................

- Use opaque driver objects and virtual APIs on the i.MX6.
- Use callbacks/queues for events.
- Use the Banana Pi to observe and test the system remotely.

Important Caveat
.................

- "Realtek" may refer to the Wi-Fi/Ethernet chip or a different board variant. Before following board-specific instructions, identify the exact hardware.

.. include:: ../../../_includes/contact_info.rst
