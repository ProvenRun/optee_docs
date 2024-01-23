.. _versal_net:

###############################
AMD-Xilinx Versal Net VNX B2197
###############################
Instructions below show how to run OP-TEE on the Versal Net VNX B2197 development board.
Details of the Versal Net can be found in the Versal Net Technical Reference Manual.

Prerequisites
*************

Public AMD-Xilinx U-Boot and Linux git repository do not allow to properly boot Versal Net
in this configuration. So you will need to obtain the following:

* U-Boot source tree archive compatible with this OP-TEE port
* Linux source tree archive compatible with this OP-TEE port
* versal-net-bsp folder with:
  * ksb-hw-patched-raw_20231113_115133.pdi
  * system_20231113_115133_optee.dtb

Configuring and building for Versal Net
***************************************

Fetching source code
====================

The default build is mostly automated and follow generic OP-TEE build procedure:

.. code-block:: bash

	$ mkdir ~/optee-project
	$ cd ~/optee-project
	$ repo init -u https://github.com/ProvenRun/optee_manifest.git -m versal_net.xml -b versal_net_port
	$ repo sync -j4 --no-clone-bundle

Adding AMD-Xilinx specific components
=====================================

Copy the versal-net-bsp folder to the ``optee-project`` directory:

.. code-block:: bash

	$ cp -a <path-to>/versal-net-bsp .

Then extract Linux and u-boot sources in their respective ``linux`` and ``u-boot`` folder
in ``optee-project`` as well. We now have all the sources needed to build.

Building the bootimage
======================

Let's prepare the toolchains:

.. code-block:: bash

	$ cd build
	$ make -j8 toolchains

At this point we have a working directory ``~/optee-project`` with all the required toolchains.

.. code-block:: bash

	$ make -j8
	$ make -j8 bootimage

A JTAG bootable image is now available at ``versal/BOOT.BIN``.

Booting the image
*****************

The ADKv2 debug module exposes 4 USB interfaces to the Linux host:

* The third one, usually ``/dev/ttyUSB2`` is used by U-Boot and Linux for their console
* The fourth one, usually ``/dev/ttyUSB3`` displays PLM, TF-A and OP-TEE traces
  (OP-TEE traces will be moved to the other UART in a future updage)

JTAG Boot to U-Boot
===================

.. note::
	This section assumes that PetaLinux 2023.2 tools such as ``hw_server`` and ``xsdb`` are
	available in the ``PATH``. They can be downloaded and `installed`_ from the AMD-Xilinx
	website (`Downloads`_).
	
	The user these executables are run with should also have the correct UNIX access rights
	to open the underlying USB device nodes. Most of the time adding said user to the
	``dialout`` UNIX group is enough on Ubuntu/Debian-based systems. Otherwise, run
	``hw_server`` as root (see below).

To run the bootable image ``BOOT.BIN`` via JTAG, configure the boot switches for JTAG boot
then power up the board.

In one terminal; start ``hw_server``:

.. code-block:: bash

	$ sudo hw_server

Then in another terminal, run the following commands:

.. code-block:: bash

	$ xsdb
	rlwrap: warning: your $TERM is 'xterm-256color' but rlwrap couldn't find it in the terminfo database. Expect some problems.
	
	****** System Debugger (XSDB) v2023.2
	  **** Build date : Oct 10 2023-17:54:17
	    ** Copyright 1986-2022 Xilinx, Inc. All Rights Reserved.
	    ** Copyright 2022-2023 Advanced Micro Devices, Inc. All Rights Reserved.

	
	xsdb% connect                                                                                        
	tcfchan#0
	xsdb% device program BOOT.BIN

It will download and execute the image on the Versal Net platform.

Booting Linux and running tests
===============================

To properly boot Linux with the current configuration, stop automatic boot by pressing the spacebar to get to
U-Boot prompt, the run the following command:

.. code-block:: bash

	U-Boot 2023.01 (Jan 23 2024 - 10:26:16 +0100)
	 
	Model: Xilinx Versal Net VNX
	DRAM:  2 GiB (effective 32 GiB)
	EL Level:EL2
	Core:  40 devices, 23 uclasses, devicetree: board
	MMC:   mmc@f1050000: 1
	Loading Environment from nowhere... OK
	In:    serial@f1930000
	Out:   serial@f1930000
	Err:   serial@f1930000
	Bootmode: JTAG_MODE
	Timeout waiting MAC address publication.
	Net:   
	ZYNQ GEM: f19f0000, mdio bus f19f0000, phyaddr 4, interface rmii
	
	Warning: ethernet@f19f0000 (eth0) using random MAC address - aa:f7:8b:a9:3e:1b
	eth0: ethernet@f19f0000
	Autoboot in 5 seconds
	(press space bar to interrupt)
	Versal NET> booti 0x27200000 0x40000000 0x27100000

When Linux has completed its boot sequence, you can login as ``root`` without any password. All
OP-TEE services should have been started at this point and you run the ``xtest`` tool to run OP-TEE tests:

.. code-block:: bash

	OP-TEE embedded distrib for versal-net-vnx-b2197-revA
	buildroot login: root
	# xtest

.. _Downloads: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2023-2.html

.. _installed: https://docs.xilinx.com/r/en-US/ug1144-petalinux-tools-reference-guide/Installing-the-PetaLinux-Tool
