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


---
---

# Spectre Attack related issue (currently unsolved)

## Description

The kernel module does not install.

Running command:

	sudo insmod /usr/local/xcap/drivers/x86_64/3.13.0-144-generic/pixci_x86_64.ko PIXCIPARM=-DM_1

returns:

	insmod: ERROR: could not insert module /usr/local/xcap/drivers/x86_64/3.13.0-144-generic/pixci_x86_64.ko: Invalid module format

dmseg command reports:

	pixci: version magic '3.13.0-144-generic SMP mod_unload modversions ' should be '3.13.0-144-generic SMP mod_unload modversions retpoline


Inspecting kernel module confirms it expects non-retpoline kernel vermagic. Command :

	/sbin/modinfo pixci_x86_64.ko

Returns:

	filename:       /usr/local/xcap/drivers/x86_64/3.13.0-144-generic/pixci_x86_64.ko
	license:        Proprietary
	description:    PIXCI(R) 32 Bit Driver. 3.8.00 [14.07.24.172802]. Copyright� 2013 EPIX, Inc.
	author:         EPIX, Inc.
	depends:        
	vermagic:       3.13.0-144-generic SMP mod_unload modversions 
	parm:           PIXCIPARM:charp

Verifying kernel retpoline. Running command:

	cat /sys/devices/system/cpu/vulnerabilities/spectre_v2

Returns:

	Mitigation: Full generic retpoline

Output of Spectre and Meltdown mitigation detection script:

```
Spectre and Meltdown mitigation detection tool v0.37+

Checking for vulnerabilities on current system
Kernel is Linux 3.13.0-144-generic #193-Ubuntu SMP Thu Mar 15 17:03:53 UTC 2018 x86_64
CPU is Intel(R) Xeon(R) CPU E5-2690 v2 @ 3.00GHz

Hardware check
* Hardware support (CPU microcode) for mitigation techniques
	* Indirect Branch Restricted Speculation (IBRS)
    * SPEC_CTRL MSR is available:  NO 
    * CPU indicates IBRS capability:  NO 
  * Indirect Branch Prediction Barrier (IBPB)
    * PRED_CMD MSR is available:  NO 
    * CPU indicates IBPB capability:  NO 
  * Single Thread Indirect Branch Predictors (STIBP)
    * SPEC_CTRL MSR is available:  NO 
    * CPU indicates STIBP capability:  NO 
  * Enhanced IBRS (IBRS_ALL)
    * CPU indicates ARCH_CAPABILITIES MSR availability:  NO 
    * ARCH_CAPABILITIES MSR advertises IBRS_ALL capability:  NO 
  * CPU explicitly indicates not being vulnerable to Meltdown (RDCL_NO):  NO 
  * CPU microcode is known to cause stability problems:  NO  (model 62 stepping 4 ucode 0x427 cpuid 0x306e4)
* CPU vulnerability to the three speculative execution attack variants
  * Vulnerable to Variant 1:  YES 
  * Vulnerable to Variant 2:  YES 
  * Vulnerable to Variant 3:  YES 

CVE-2017-5753 [bounds check bypass] aka 'Spectre Variant 1'
* Mitigated according to the /sys interface:  YES  (Mitigation: OSB (observable speculation barrier, Intel v6))
* Kernel has array_index_mask_nospec (x86):  NO 
* Kernel has the Red Hat/Ubuntu patch:  YES 
* Kernel has mask_nospec64 (arm):  NO 
> STATUS:  NOT VULNERABLE  (Mitigation: OSB (observable speculation barrier, Intel v6))

CVE-2017-5715 [branch target injection] aka 'Spectre Variant 2'
* Mitigated according to the /sys interface:  YES  (Mitigation: Full generic retpoline)
* Mitigation 1
  * Kernel is compiled with IBRS support:  YES 
    * IBRS enabled and active:  NO 
  * Kernel is compiled with IBPB support:  YES 
    * IBPB enabled and active:  NO 
* Mitigation 2
  * Kernel has branch predictor hardening (arm):  NO 
  * Kernel compiled with retpoline option:  YES 
    * Kernel compiled with a retpoline-aware compiler:  YES  (kernel reports full retpoline compilation)
> STATUS:  NOT VULNERABLE  (Full retpoline is mitigating the vulnerability)
IBPB is considered as a good addition to retpoline for Variant 2 mitigation, but your CPU microcode doesn't support it

CVE-2017-5754 [rogue data cache load] aka 'Meltdown' aka 'Variant 3'
* Mitigated according to the /sys interface:  YES  (Mitigation: PTI)
* Kernel supports Page Table Isolation (PTI):  YES 
  * PTI enabled and active:  UNKNOWN  (dmesg truncated, please reboot and relaunch this script)
  * Reduced performance impact of PTI:  YES  (CPU supports PCID, performance impact of PTI will be reduced)
* Running as a Xen PV DomU:  NO 
> STATUS:  NOT VULNERABLE  (Mitigation: PTI)

A false sense of security is worse than no security at all, see --disclaimer
```





## Trying to force module loading (failed)

Command:

	sudo cp pixci_x86_64.ko /lib/modules/3.13.0-144-generic/
	sudo depmod -a
	sudo modprobe --force pixci_x86_64

Returns:

	modprobe: ERROR: could not insert 'pixci_x86_64': Exec format error


## Editing Kernel headers (no effect)

Edit kernel header file:

	vim /usr/src/linux-headers-3.13.0-144/include/linux/vermagic.h

Change :

	#ifdef RETPOLINE
	#define MODULE_VERMAGIC_RETPOLINE "retpoline "
	#else
	#define MODULE_VERMAGIC_RETPOLINE ""
	#endif

To: 

	/* #ifdef RETPOLINE */
	#if 1
	#define MODULE_VERMAGIC_RETPOLINE "retpoline "
	#else
	#define MODULE_VERMAGIC_RETPOLINE ""
	#endif

Then reboot computer.



## Checking gcc version used to compile Kernel (fixed issue)

Command:
	
	cat /prov/version

Returns:

	Linux version 3.13.0-144-generic (buildd@lgw01-amd64-059) (gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.4) ) #193-Ubuntu SMP Thu Mar 15 17:03:53 UTC 2018

Noticed that default gcc version is older, returned by gcc -v:

	gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.3) 
	
So reinstalling gcc:

	sudo apt-get install --reinstall gcc-4.8


## Conclusion

On March 15 2018, automatic kernel update added the retpoline kernel feature against the meltdown/spectre attack. The gcc compiler was not automatically updated and the default gcc version did not have the retpoline feature.

xcap uses default gcc to create kernel module, and then tried to load it. Loading failed because of retpoline mismatch. Replacing gcc with retpoline-enabled gcc package fixed the issue.

