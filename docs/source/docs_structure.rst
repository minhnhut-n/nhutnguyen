Documentation Structure
=======================

This page shows the documentation tree used by the Read the Docs site.

.. code-block:: text

   docs/
   └── source/
       ├── index.rst
       ├── embedded_linux/
       │   ├── index.rst
       │   ├── board/
       │   ├── general/
       │   └── services/
       ├── embedded_mcu/
       │   ├── index.rst
       │   ├── board/
       │   ├── general/
       │   └── helper/
       ├── embedded_robot/
       │   ├── index.rst
       │   ├── controller/
       │   ├── theory/
       │   └── type/
       ├── projects/
       │   ├── index.rst
       │   ├── linux/
       │   ├── mcu/
       │   └── robot/
       ├── topics/
       ├── _includes/
       └── _static/

This structure is organized so that the documentation can be published cleanly on Read the Docs with separate sections for Linux, MCU, robotics, and projects.

.. include:: _includes/contact_info.rst
