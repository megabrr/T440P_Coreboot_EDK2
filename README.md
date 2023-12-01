# How-to Coreboot a Thinkpad T440p with the EDK2 Payload (UEFI).

This guide assumes that you have an unlocked Bios and a **backup of the original Bios** (i.e. able to use flashrom - p internal).
All the commands are run in MX Linux. They should work in other debian variant.

**WARNING: If the current OS installed on the laptop is working in legacy mode (i.e. not using uefi), the OS won't boot after flashing EDK2 and you will have to reinstall.**

## Step 1 - Install tools and libraries needed for coreboot
Make sure your linux distribution is up to date.
```
sudo apt update -y
sudo apt upgrade -y
sudo apt-get install -y bison build-essential curl flex git gnat libncurses5-dev m4 zlib1g-dev flashrom autoconf automake gettext autopoint pkg-config 
grub-common libfreetype6-dev unifont unifont-bin xfonts-unifont

```

## Step 2 - Use Libreboot script to install all the dependencies.
```
cd ~
sudo apt install -y python-is-python3
git clone https://codeberg.org/libreboot/lbmk
cd lbmk/
sudo ./build dependencies debian 
```
## Step 3 - Use Libreboot to get all the necessary Blobs.
Configure GIT
```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```
Build the T440P rom from lmbk
(We will not use the rom itself, but simply copy the blobs)
```
./build roms t440pmrc_12mb
```
If everything goes well, you should see the following message.
```
ROM images available in these directories:
* bin/t440pmrc_12mb
^^ ROM images available in these directories.

DO NOT flash ROM images from elf/ - please use bin/ instead.
```
Copy the blobs
```
mkdir ~/t440p
cp ~/lbmk/vendorfiles/t440p/me.bin ~/t440p
cp ~/lbmk/mrc/haswell/mrc.bin ~/t440p
cp ~/lbmk/config/ifd/t440p/gbe ~/t440p
cp ~/lbmk/config/ifd/t440p/ifd ~/t440p
```

## Step 4 - Configure Coreboot
Download and checkout Coreboot with GIT. I'm using version 4.22.01. If you want to use anything else you can find the tag or the branch here:
https://review.coreboot.org/plugins/gitiles/coreboot
```
cd ~
git clone https://review.coreboot.org/coreboot.git
cd coreboot/
git checkout d5e80fa9d22ae17ca80276e1d663333e0932c399
git submodule update --init --checkout
```
Compile Coreboot and the needed tools.
Can take several minutes depending on your hardware.
```
make crossgcc-i386 CPUS=$(nproc)
make -C payloads/coreinfo olddefconfig
make -C payloads/coreinfo
cd util/ifdtool && make
```
## Step 5 - Create the config and build the rom.
```
nano .config
```
(Copy the config here then ctrl+x > Y to save)
What I'm using (don't forget to change the path):
```
###Blobs section
CONFIG_HAVE_IFD_BIN=y
CONFIG_IFD_BIN_PATH="/home/user/t440p/ifd"
CONFIG_HAVE_ME_BIN=y
CONFIG_ME_BIN_PATH="/home/user/t440p/me.bin"
CONFIG_HAVE_GBE_BIN=y
CONFIG_GBE_BIN_PATH="/home/user/t440p/gbe"
CONFIG_HAVE_MRC=y
CONFIG_MRC_FILE="/home/user/t440p/mrc.bin"
#
###Config option
CONFIG_BOOTSPLASH=y
CONFIG_USE_OPTION_TABLE=y
CONFIG_TIMESTAMPS_ON_CONSOLE=y
CONFIG_VENDOR_LENOVO=y
CONFIG_CBFS_SIZE=0x800000
CONFIG_CONSOLE_CBMEM_BUFFER_SIZE=0x20000
CONFIG_BOARD_LENOVO_THINKPAD_T440P=y
CONFIG_UART_PCI_ADDR=0x0
CONFIG_VALIDATE_INTEL_DESCRIPTOR=y
CONFIG_H8_SUPPORT_BT_ON_WIFI=y
CONFIG_SUBSYSTEM_VENDOR_ID=0x0000
CONFIG_SUBSYSTEM_DEVICE_ID=0x0000
CONFIG_I2C_TRANSFER_TIMEOUT_US=500000
CONFIG_SMMSTORE_SIZE=0x40000
CONFIG_DRIVERS_PS2_KEYBOARD=y
CONFIG_TPM_DEACTIVATE=y
CONFIG_SECURITY_CLEAR_DRAM_ON_REGULAR_BOOT=y
CONFIG_POST_IO_PORT=0x80
#
###Payload section
CONFIG_PAYLOAD_EDK2=y
CONFIG_EDK2_UEFIPAYLOAD=y
CONFIG_EDK2_CUSTOM_BUILD_PARAMS=""
CONFIG_EDK2_BOOT_TIMEOUT=4
CONFIG_EDK2_FULL_SCREEN_SETUP=y
CONFIG_EDK2_DISABLE_TPM=y
CONFIG_EDK2_HAVE_EFI_SHELL=y
CONFIG_EDK2_PRIORITIZE_INTERNAL=y
CONFIG_EDK2_BOOT_MANAGER_ESCAPE=y
CONFIG_COMPRESSED_PAYLOAD_LZMA=y

```
Build the rom.
```
make savedefconfig
cat defconfig
(To make sure everything is OK)
make nconfig
(save with F6 and exit with F9)
make
```
If everthing went well you should see:
```
    HOSTCC     cbfstool/ifwitool.o
    HOSTCC     cbfstool/ifwitool (link)
    HOSTCC     cbfstool/cse_fpt.o
    HOSTCC     cbfstool/cse_helpers.o
    HOSTCC     cbfstool/fpt_hdr_20.o
    HOSTCC     cbfstool/fpt_hdr_21.o
    HOSTCC     cbfstool/cse_fpt (link)
    HOSTCC     cbfstool/cse_serger.o
    HOSTCC     cbfstool/bpdt_1_6.o
    HOSTCC     cbfstool/bpdt_1_7.o
    HOSTCC     cbfstool/subpart_hdr_1.o
    HOSTCC     cbfstool/subpart_hdr_2.o
    HOSTCC     cbfstool/subpart_entry_1.o
    HOSTCC     cbfstool/cse_serger (link)

Built lenovo/haswell (ThinkPad T440p)

```

## Step 6 - Flash Coreboot with Flashrom
Copy the rom you just build somewhere safe.
```
cp ~/coreboot/bin/coreboot.rom ~/t440/t440pmrc_12mb_coreboot_edk2.rom
```
Flash the rom.
```
sudo modprobe -r lpc_ich
sudo flashrom -p internal -w ~/t440/t440pmrc_12mb_coreboot_edk2.rom
```
Reboot...

## Step 7 - verify thar ME is both neutered and disabled
