2. Setup
=============================================================================

.. rubric:: Yêu cầu phần cứng

- Banana Pi M4
- MicroSD card (16GB trở lên)
- Nguồn 5V/3A
- Cáp HDMI
- Bàn phím và chuột

---

.. rubric:: 1. Image Installation

**1.1 Download Recommended Image:**

- **Latest stable image**: `Google Drive - Banana Pi Images <https://drive.google.com/drive/folders/19xi4l9xX34r1EY3TGjs1guxnvt2qUIsr>`_
- **Recommended**: Ubuntu by mikeylinux (Ubuntu 18.04)

**1.2 Required Tools:**

- `balenaEtcher <https://www.balena.io/etcher/>`_ (for flashing SD card)

**1.3 Installation Steps:**

1. Run balenaEtcher with **Administrator mode**
2. Extract the downloaded image file
3. In balenaEtcher:
   - Select the extracted image file
   - Select the SD card as target
   - Flash the image

.. warning::

   If balenaEtcher reports an error when loading the image, restart the application with **Administrator mode** and select the image again.

**1.4 Copy Image to eMMC (from SD card):**

.. code-block:: bash

   sudo dd if=/dev/mmcblk0 of=/dev/mmcblk1 bs=4M status=progress && sync

.. note::

   Adjust device names (``/dev/mmcblk0`` = SD card, ``/dev/mmcblk1`` = eMMC) based on your setup.

---

.. rubric:: 2. Useful System Commands

.. list-table::
   :header-rows: 1

   * - Command
     - Description
   * - ``cat /proc/meminfo``
     - Check memory information
   * - ``lsblk``
     - List all block devices on the board

---

.. rubric:: 3. Change Username

**3.1 Change root Password:**

1. Connect to the board via UART serial console
2. Change root password:

   .. code-block:: bash

      sudo passwd root

   → Enter new password for root

3. Change user password:

   .. code-block:: bash

      sudo passwd <username>

   → Enter new password

4. Reboot the board
5. Login with the root account (using the newly changed password)

**3.2 Kill Services of Old Username:**

.. code-block:: bash

   sudo killall -u <old_username>

Verify no remaining processes:

.. code-block:: bash

   pgrep -u <old_username>

(Should return nothing)

**3.3 Rename User:**

.. code-block:: bash

   sudo usermod -l new_username -d /home/new_username -m old_username

**3.4 Reboot:**

.. code-block:: bash

   sudo reboot

**3.5 Fix Post-Rename Issues:**

After rebooting, you may encounter two errors:

**Error 1**: Login display still shows "Banana Pi"

Edit the LightDM auto-login config:

.. code-block:: bash

   sudo nano /etc/lightdm/lightdm.conf

Change ``autologin-user=`` to the new username.

**Error 2**: Blueman service warning

(Investigate and fix blueman service configuration)

---

.. rubric:: 4. Disable GUI (Headless / Terminal-Only Mode)

**4.1 Set Default Target to Multi-User (No GUI):**

.. code-block:: bash

   sudo systemctl set-default multi-user.target

**4.2 Stop GUI Immediately (No Reboot Required):**

.. code-block:: bash

   sudo systemctl isolate multi-user.target

Or:

.. code-block:: bash

   sudo systemctl stop display-manager

**4.3 Reboot:**

.. code-block:: bash

   sudo reboot

After booting, you will see::

   Ubuntu 18.04 LTS minion-pi tty1

   minion-pi login:

Login with your user credentials.

---

.. rubric:: 5. Re-enable GUI

**Option A: Set Default Target to Graphical:**

.. code-block:: bash

   sudo systemctl set-default graphical.target
   sudo reboot

**Option B: Start GUI Session Only (Without Changing Default):**

.. code-block:: bash

   sudo systemctl start display-manager

---

.. rubric:: 6. SSH Setup (Do This Before Disabling GUI)

**6.1 Enable and Start SSH:**

.. code-block:: bash

   sudo systemctl enable ssh
   sudo systemctl start ssh
   systemctl status ssh

**6.2 Connect from PC:**

.. code-block:: bash

   ssh aelius@<board-ip-address>

.. note::

   Replace ``aelius`` with your actual username and ``<board-ip-address>`` with the IP of your Banana Pi board.
