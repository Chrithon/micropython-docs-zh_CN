.. _pyboard_general:

pyboard 的基本资料
=====================================

.. contents::

本地文件系统和SD卡文件系统
----------------------------

There is a small internal filesystem (a drive) on the pyboard, called ``/flash``,
which is stored within the microcontroller's flash memory.  If a micro SD card
is inserted into the slot, it is available as ``/sd``. '\n\r'
pyboard的内部有一个很小的文件系统（驱动器），叫做``/flash``，它使用单片机自带的flash来存储。
如果将microSD卡插入卡槽，则可以使用``/sd``里面的文件系统。

When the pyboard boots up, it needs to choose a filesystem to boot from.  If
there is no SD card, then it uses the internal filesystem ``/flash`` as the boot
filesystem, otherwise, it uses the SD card ``/sd``. After the boot, the current
directory is set to one of the directories above.
当pyboard启动后，它需要选择一个文件系统来用作启动。如果没有SD卡插入，那么它将使用内部文件系统
``/flash`` 作为启动系统来源。如果有SD卡插入，则自动使用SD卡 ``/sd``作为启动系统来源。成功启
动后，看到目录将是以上两个文件系统的目录之一。

If needed, you can prevent the use of the SD card by creating an empty file
called ``/flash/SKIPSD``.  If this file exists when the pyboard boots
up then the SD card will be skipped and the pyboard will always boot from the
internal filesystem (in this case the SD card won't be mounted but you can still
mount and use it later in your program using ``os.mount``).
如果有需要，你可以通过在创建一个空文件``/flash/SKIPSD``来避免通过SD卡启动。如果pyboard文件系统
内存在这个文件则启动时候SD卡会被跳过检测，总是从内部文件系统启动。（在这种情况下SD卡不会被挂在但你
仍然可以通过程序里使用``os.mount``来挂在并使用它。）

(Note that on older versions of the board, ``/flash`` is called ``0:/`` and ``/sd``
is called ``1:/``).
（注意：在旧版本的板子上，``/flash``称为``0:/`` 以及 ``/sd``被称为``1:/``）

The boot filesystem is used for 2 things: it is the filesystem from which
the ``boot.py`` and ``main.py`` files are searched for, and it is the filesystem
which is made available on your PC over the USB cable.

The filesystem will be available as a USB flash drive on your PC.  You can
save files to the drive, and edit ``boot.py`` and ``main.py``.

*Remember to eject (on Linux, unmount) the USB drive before you reset your
pyboard.*

Boot modes
----------

If you power up normally, or press the reset button, the pyboard will boot
into standard mode: the ``boot.py`` file will be executed first, then the
USB will be configured, then ``main.py`` will run.

You can override this boot sequence by holding down the user switch as
the board is booting up.  Hold down user switch and press reset, and then
as you continue to hold the user switch, the LEDs will count in binary.
When the LEDs have reached the mode you want, let go of the user switch,
the LEDs for the selected mode will flash quickly, and the board will boot.

The modes are:

1. Green LED only, *standard boot*: run ``boot.py`` then ``main.py``.
2. Orange LED only, *safe boot*: don't run any scripts on boot-up.
3. Green and orange LED together, *filesystem reset*: resets the flash
   filesystem to its factory state, then boots in safe mode.

If your filesystem becomes corrupt, boot into mode 3 to fix it.
If resetting the filesystem while plugged into your compute doesn't work,
you can try doing the same procedure while the board is plugged into a USB
charger, or other USB power supply without data connection.

Errors: flashing LEDs
---------------------

There are currently 2 kinds of errors that you might see:

1. If the red and green LEDs flash alternatively, then a Python script
    (eg ``main.py``) has an error.  Use the REPL to debug it.
2. If all 4 LEDs cycle on and off slowly, then there was a hard fault.
   This cannot be recovered from and you need to do a hard reset.

Guide for using the pyboard with Windows
----------------------------------------

The following PDF guide gives information about using the pyboard with Windows,
including setting up the serial prompt and downloading new firmware using
DFU programming:
`PDF guide <http://micropython.org/resources/Micro-Python-Windows-setup.pdf>`__.

.. _hardware_index:

.. include:: hardware/index.rst
