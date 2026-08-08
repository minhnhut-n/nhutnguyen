.. _general_setup_to_develop_espidf_platform:

===================================================================
General Setup to Develop ESP-IDF Platform
===================================================================

.. contents:: Mục Lục Bài Viết
   :depth: 2
   :local:

---

.. _section-introduction:

I. Giới Thiệu
=============

Thông thường khi download các file dự án platform io thường không đi kèm với file config của esp-idf được khai báo trong file .gitignore,
nên khi clone về ta cần thực hiện vài bước config để có thể sử dụng một cách hoàn chỉnh các tính năng như đề xuất code bằng intellisense.

Vấn đề trên chủ yếu là do symlink của project không tìm được các package info của project esp-idf, tuy nhiên BẮT BUỘC là phải có file 
platform.init và các file sdk config của board, nhưng không có cũng không sao vì ta có thể tạo lại được, và sửa dần.

---

.. _section-prerequisites:

II. Điều Kiện Tiên Quyết (Prerequisites)
========================================

Trước khi bắt đầu, cần chuẩn bị các điều kiện sau:
* **PlatformIO Extension:** Cài đặt extension **PlatformIO** trên Visual Studio Code.
* **Python:** Cài đặt python trong việc support platform io chạy build (optional, nhưng nên có).

---

.. _section-guide:

III. Hướng Dẫn Các Bước (Guide)
===============================

Bước 1: Mở Project tại nơi có file platform.init hoặc ít nhất là có các folder như src/, build/, include/
---------------------------------------------------------------------------------------------------------

+ Thực hiện build trước 1 lần bằng PlatformIO build icon ở thanh công cụ, hoặc chạy lệnh ``pio run`` trong terminal, để PlatformIO tự động tạo ra các file config cần thiết.
+ Xác nhận lại đã có thư mục /.pio và /.vscode chưa?

Bước 2: Tạo các file config cần thiết cho workspace để làm việc
---------------------------------------------------------------
Trên Windows, vào file ``platformio.ini``, paste các config sau vào:

* **c_cpp_properties.json:** 

.. code-block:: json

   {
      "configurations": [
         {
               "name": "ESP-IDF",
               "compileCommands": "${workspaceFolder}/.pio/build/esp32doit-devkit-v1/compile_commands.json",
               "includePath": [
                  "${workspaceFolder}/**"
               ],
               "defines": [
                  "ESP_PLATFORM",
                  "__cplusplus"
               ],
               "compilerPath": "C:/Users/Aelius_Nguyen/.platformio/packages/toolchain-xtensa-esp-elf/bin/xtensa-esp32-elf-gcc.exe",
               "cStandard": "c17",
               "cppStandard": "c++17",
               "intelliSenseMode": "gcc-x64"
         }
      ],
      "version": 4
   }

.. note::
   Ở trên là một ví dụ về USER là ``Aelius_Nguyen``, bạn cần thay bằng tên user của bạn trên Windows, và đường dẫn đến thư mục ``toolchain-xtensa-esp-elf`` của bạn.
   Trong mục ''compilerCommand'' để đúng với đường dẫn đến file ``compile_commands.json`` của project, nếu không có thì build 1 lần để PlatformIO tự tạo ra. (hướng dẫn properties.json cho intellisense)
   Trong folder /.pio chứa các object library nên includePath search tất cả library trong workspace ngụ ý là lấy các header của các thư mục đó để kiểm tra.

* **extensions.json** 

.. code-block:: json

   {
      // See http://go.microsoft.com/fwlink/?LinkId=827846
      // for the documentation about the extensions.json format
      "recommendations": [
         "platformio.platformio-ide",
         "ms-vscode.cpptools-extension-pack"
      ],
      "unwantedRecommendations": []
   }

* **settings.json** 

.. code-block:: json

   {
      "C_Cpp.default.configurationProvider": "platformio.platformio-ide",
      "C_Cpp.intelliSenseEngine": "default",
      "C_Cpp.default.compileCommands": "${workspaceFolder}/.pio/build/esp32doit-devkit-v1/compile_commands.json",
      "C_Cpp.errorSquiggles": "enabled",
      "files.associations": {
         "*.h": "c"
      }
   }

* **platformio.ini** 

.. code-block:: ini

   [env:esp32doit-devkit-v1]
   platform = espressif32
   board = esp32doit-devkit-v1
   framework = espidf
   monitor_speed = 115200


Bước 3: Refresh lại/ rescan intellisense, build project
--------------------------------------------------------
+ Mở một file code bất kì, có include các file header bị underline.
+ Bấm vào cái Chữ C hoặc tùy ngôn ngữ mà vscode dedect được ở dưới cuối của màn hình, chọn rescan intellisense, hoặc bấm tổ hợp phím Ctrl + Shift + P, gõ rescan intellisense, chọn rescan intellisense.
+ Vào mục problems kiểm tra lại có còn bị lỗi nào không.
