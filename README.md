# mx4-wlink8-backport

## Build steps

Clone the wlink8 build utils

	git clone git://git.ti.com/wilink8-wlan/build-utilites.git

	cd build-utilites
	# checkout a working git revision
	git checkout a45a32efb204941a18f06e1fd20c3374dfd7f359

Create an `setup-env` file containing paths to tool-chain and target Linux kernel (ROOTFS path can be left as DEFAULT in which case everything will end up in build-utils/fs). This is what it looked like when I built:

	#                            \\\//
	#                           -(o o)-
	#========================oOO==(_)==OOo=======================
	# This file contains the exports needed for automating the
	# build process of WLAN components.
	# Place this file in the same directory with wl18xx_build.sh
	# build scripts. No need to run 'source setup-env', the build
	# scripts will perform it internally.
	#===========================================================
	# User specific environment settings - use full PATH

	# if DEFAULT toolchain path is set toolchain will be downloaded to ./toolchain.
	export TOOLCHAIN_PATH=/home/mirza/git/mx4/t20/oe-core/build/out-eglibc/sysroots/x86_64-linux/usr/bin/armv7ahf-vfp-angstrom-linux-gnueabi/

	# if DEFAULT path to root filesystem is set ./fs folder will be used.
	export ROOTFS=DEFAULT

	#if DEFAULT kernel path is set - kernel will be downloaded (set branch to match kernel version)
	export KERNEL_PATH=/home/mirza/git/mx4/t20/colibri-t20-L4T-linux-kernel
	#export KERNEL_PATH=/home/mirza/git/mx4/t20/linux_vf

	# if KERNEL_VARIANT below is set the build script will look for kernel specific
	# patches under the patches directory:
	# - patches under the pathces/driver_patches/$KERNEL_VARIANT directory would be
	#   applied during "modules" build.
	# - patches under the patches/kernel_patches/$/$KERNEL_VARIANT directory would
	#   be applied to the kernel pointed by KERNEL_PATH in case the "patch_kernel"
	#   command is used.
	#   Note: the kernel is not built automatically after the patches are applied
	export KERNEL_VARIANT=DEFAULT

	export CROSS_COMPILE=arm-angstrom-linux-gnueabi-
	export ARCH=arm
	[ "$TOOLCHAIN_PATH" != "DEFAULT" ] && export PATH=$TOOLCHAIN_PATH:$PATH


Setup the environment by running

	./build_wl18xx.sh init R8.7_SP1


Apply patches to respective "module" that you can find in the `patches` directory in this repository. Directory structure should match the one in build-utilites/src.

	$ cd src/backports
	$ git am ../../../patches/backports/*
	Applying: fix build with L4T 3.1.10 kernel
	$ cd ../driver/
	$ git am ../../../patches/driver/*
	Applying: net: wireless: remove trace support
	Applying: net: wlcore: hardware IRQ flags
	Applying: net: wlcore: add an hard-coded interrupt number


Make sure that you have built you target Linux kernel and that the `.config` contains the following (`build-utilities/verify_kernel_config.sh` can be used to sanity check the `.config` file)

	CONFIG_CFG80211=N
	CONFIG_MAC80211=N

and

	#
	# Network testing
	#

	# MAC 80211
	CONFIG_WLAN=y
	CONFIG_WIRELESS=y
	CONFIG_WIRELESS_EXT=y
	CONFIG_WL12XX_PLATFORM_DATA=y

	CONFIG_KEYS=y
	CONFIG_SECURITY=y
	CONFIG_CRYPTO=y

	CONFIG_CRYPTO_ARC4=y
	CONFIG_CRYPTO_ECB=y
	CONFIG_CRYPTO_AES=y
	CONFIG_CRYPTO_MICHAEL_MIC=y
	CONFIG_CRYPTO_RNG=y
	CONFIG_CRYPTO_AEAD=y
	CONFIG_CRYPTO_CCM=y
	CONFIG_CRYPTO_GCM=y

	CONFIG_RFKILL=y

	CONFIG_REGULATOR_FIXED_VOLTAGE=y
	CONFIG_CRC7=y

	# The following are needed for soft AP
	CONFIG_NETFILTER=y
	CONFIG_NETFILTER_ADVANCED=y
	CONFIG_NF_CONNTRACK=y
	CONFIG_NETFILTER_XTABLES=y
	CONFIG_NF_DEFRAG_IPV4=y
	CONFIG_NF_CONNTRACK_IPV4=y
	CONFIG_NF_CONNTRACK_PROC_COMPAT=y
	CONFIG_IP_NF_IPTABLES=y
	CONFIG_IP_NF_FILTER=y
	CONFIG_IP_NF_TARGET_LOG=y
	CONFIG_NF_NAT=y
	CONFIG_NF_NAT_NEEDED=y
	CONFIG_IP_NF_TARGET_MASQUERADE=y

	CONFIG_INPUT_UINPUT=y

	# Enable Ethernet-WLAN Bridge
	CONFIG_NETFILTER=y
	CONFIG_NETFILTER_ADVANCED=y
	CONFIG_BRIDGE_NETFILTER=y
	CONFIG_STP=y
	CONFIG_BRIDGE=y
	CONFIG_BRIDGE_IGMP_SNOOPING=y
	CONFIG_LLC=y
	CONFIG_INPUT_UINPUT=y
	CONFIG_HAS_IOMEM=y
	CONFIG_HAS_IOPORT=y
	CONFIG_HAS_DMA=y
	CONFIG_NLATTR=y
	CONFIG_AVERAGE=y


Start the build

	# This will build everything
	./build_wl18xx.sh

Output of build can be found in `build-utilites/fs` (if ROOTFS=DEFAULT) and `build-utilites/outputs`. The file in `build-utilites/outputs` you can transfer to the device and unpack in `/`


## Useful links (where I got the most of my information)

- http://processors.wiki.ti.com/index.php/WL18xx_Platform_Integration_Guide
- http://processors.wiki.ti.com/index.php/WL18xx_System_Build_Scripts
- http://software-dl.ti.com/ecs/WiLink8/SP/latest/index_FDS.html
