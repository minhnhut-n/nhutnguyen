Level 3 – Kernel Build
======================

Đây là lúc bắt đầu "đụng kernel".

Build kernel
------------

- make menuconfig
- make
- make dtbs
- make modules

Hiểu
----

- Kconfig
- Makefile
- Image
- zImage
- uImage
- modules

Cross Compile
-------------

::

   PC
   ↓
   ARM64
   ↓
   BPI-M4

Kernel Config
-------------

- CONFIG\_*
- enable
- disable
- module
- builtin

.. include:: ../../../../_includes/contact_info.rst
