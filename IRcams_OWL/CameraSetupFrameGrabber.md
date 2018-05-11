# IR viewing cameras setup

Raptor Photonics OWL cameras (InGaAs), 320x256 pix

EPIX frame grabber.


---
---



# Fresh Install: Drivers Installation

Create directory ./src/OWLcam
These notes assume a Linux kernel version 3.11 or later

## DOWNLOAD BINARY INSTALL SCRIPTS

Download the most recent versions of the sdk and tool:
- ftp://ftp.epixinc.com/software/xclib_v38/xcliblnx_x86_64.bin
- ftp.epixinc.com/downloads/xcaplnx_x86_64.bin

For this, you can type from the command line:

	wget ftp://ftp.epixinc.com/software/xclib_v38/xcliblnx_x86_64.bin
	wget ftp://ftp.epixinc.com/downloads/xcaplnx_x86_64.bin


## VERIFY ELF INTERPRETER (optional)

	readelf -a xcliblnx_x86_64.bin | grep interpreter:
[
Requesting program interpreter: /lib64/ld-lsb-x86-64.so.3]

Check that the ELF interpreter exists. If not, establish a symbolic link as follows:

	cd /lib64
	sudo ln -s ld-linux-x86-64.so.2 ld-lsb-x86-64.so.3


## RUN BINARY INSTALL SCRIPTS


### install xcap

	chmod +x *.bin
	sudo ./xcaplnx_x86_64.bin

Answer yes to: "Create executable stub  /usr/local/bin/xcap  to start XCAP for for Linux 64 bit"
Choose "Individual files" option

Start XCAP
enter XCAP license code

When first restarting xcap, it will ask for the driver. In the
provided menu, select the kernel version and click on the compile
buton.
Select 3.8 binary blob (it will work with kernels > 3.8)

Exit xcap

Change permissions as follows:

	cd ~/src/OWLcam
	chown -R scexao xcap
	chgrp -R scexao xcap


### install xclib

	chmod +x xcliblnx_x86_64.bin
	sudo ./xcliblnx_x86_64.bin

Enter XClib licence code




### RUN AND CONFIGURE XCAP FOR BOTH CAMERAS

	xcap

If the camera isn't automatically selected at startup (and I think
that depends on whether the camera is connected to the PIXCI board
when you first start it), follow this procedure:

use the main xcap menu and select:
-> "PIXCI", "PIXCI" "Open/Close", "Close", "Camera & Format"
Select the "Raptor Photonics OWL CL-High speed" in the menu.

Check the option that starts xcap with the last coniguration used (as
opposed to the "generic" option that doesn't work with the OWL).

Finally click "Open".

By clicking on the Live button of the tool that communicates with the
camera, you should now see what the camera sends.

---

The second PIXCI board was installed in the extension chassis and the
camera connected to it. In XCAP, select:
-> "PIXCI", "PIXCI" "Open/Close", "Close", "Multiple devices"

Selec the two cameras. After that, click "Open".

You now get the simultaneous display for both cameras

	cp ~/src/OWLcam/xcap/settings/xcvidset.fmt /usr/local/xcap/




---
---


# LOAD KERNEL MODULES

Series of commands to execute with super user prileges:

	sudo insmod /usr/local/xcap/drivers/x86_64/3.13.0-144-generic/pixci_x86_64.ko PIXCIPARM=-DM_3
	sudo rm -f /dev/pixci
	sudo mknod /dev/pixci c $(cat /proc/devices | grep PIXCI\(R\) | awk '{print $1}') 0
	sudo chmod 666 /dev/pixci

Notes: "-DM 3" loads the two cameras, "-DM 1" loads cam#1, "-DM 2" loads cam#2.
Note: replace "3.13.0-144-generic" by the adequate kernel name,



	sudo ln -s /usr/local/xcap/drivers/x86_64/3.13.0-144-generic/pixci_x86_64.ko /lib/modules/3.13.0-144-generic/
	sudo depmod -a



