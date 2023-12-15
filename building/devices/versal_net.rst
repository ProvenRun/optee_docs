.. _versal_net:

###############################
AMD-Xilinx Versal Net VNX B2197
###############################
Instructions below show how to run OP-TEE on the Versal Net VNX B2197 development board.
Details of the Versal Net can be found in the Versal Net Technical Reference Manual.

Setting up the toolchain
************************
This build chain relies on Petalinux 2023.2, therefore the first step will be to
download and `install`_ it from the AMD-Xilinx website (`Downloads`_).

Then, you will also need to download the board support package (BSP) from the
AMD-Xilinx website (`Downloads`_). It contains prebuilt firmwares and hardware
definition files required to assemble a bootable image.

.. note::
   You will need a free AMD-Xilinx account to proceed with the two previous
   steps.

Configuring and building for Versal Net
***************************************
Lets summarize the steps taken so far; these are common to all boards.

.. code-block:: bash

	$ mkdir ~/optee-project
	$ cd ~/optee-project
	$ repo init -u https://github.com/ProvenRun/optee_manifest.git -m versal_net.xml -b versal_net_port
	$ repo sync -j4 --no-clone-bundle
	$ cd build
	$ make -j8 toolchains
	$ make -j8

At this point we have a working directory ``~/optee-project`` with all the
repositories required with the exception of the Versal Net board support
package. A pre-requisite to unpacking the BSP file is installing Petalinux
(`install`_) as previously mentioned.

Having done that, now is the time to unpack the BSP:

.. code-block:: bash

	$ cd ~/optee-project
	$ cp ~/Downloads/${versal_net_bsp_file}.bsp .
	$ source /path/to/petalinux.2023.2/settings.sh
	$ petalinux-create --type project -s ${versal_net_bsp_file}.bsp

In order for the Versal OP-TEE port to work correctly, the PLM needs to be
updated to add the XilNvm and XilPuf libraries. This can be accomplished by the
following steps within the PetaLinux workspace created above:

.. code-block:: bash

   $ mkdir project-spec/meta-user/recipes-bsp/embeddedsw
   $ cp ~/optee-project/build/versal/plm-firmware_%.bbappend project-spec/meta-user/recipes-bsp/embeddedsw
   $ petalinux-build -c plm

The newly created PLM will be located in the folder ``images/linux/plm.elf``.

Before building the release, you will need to edit the Boot Image File (BIF)
``build/versal/bootImage-versal-net-vnx-b2197-revA.bif`` to point to the required BSP files.
The paths for the following files in the BIF will need to be updated *before*
proceeding:

- vpl_gen_fixed.pdi
- plm.elf
- psmfw.elf

.. note::
   The default PLM **only** contains the xilsecure library. If you would like to
   take advantage of all of hardware cryptographic features implemented for
   Versal, you **must** enable the xilpuf and xilnvm libraries by following the
   steps above for customizing the PLM (`PLM_Customization`_).

The xilpuf library enables support of the physically unclonable function (PUF)
and the xilnvm library enables support of reading and writing to eFUSEs. Once
these libraries are enabled, be sure to point to the updated PLM firmware in the
previously mentioned BIF file.

After you have done that you can build the images as follows:

.. code-block:: bash

	$ cd ~/optee-project
	$ cd build
	$ make -f versal_net.mk image
	$ ls versal | grep -E 'BIN|ub'
	  BOOT.BIN
	  versal-net-vnx-b2197-revA.ub


JTAG boot to U-Boot shell
*************************
To run the bootable image ``BOOT.BIN`` via JTAG, configure the boot switches as
seen below and then power up the board.

TODO: Update image with Versal Net picture

.. figure:: /images/boards/vck190-jtag-boot.png
	:width: 400
	:align: center

Then run the boot_jtag.sh script.

This script will first ask for the path of the Petalinux installation; once
entered, it will download and execute the image on the Versal ACAP platform.

.. code-block:: bash

	$ cd ~/optee-project/build/versal/
	$ ./boot_jtag.sh



.. _Downloads: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2023-2.html

.. _install: https://docs.xilinx.com/r/en-US/ug1144-petalinux-tools-reference-guide/Installing-the-PetaLinux-Tool

.. _PLM_Customization: https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/2037088327/Versal+Platform+Loader+and+Manager#PLM-Feature-Configuration-for-PetaLinux
