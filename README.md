# How-to Coreboot a Thinkpad T440p with the the EDK2 Payload (UEFI).

This guide assumes that you have an unlucked Bios (i.e. able to use flashrom - p internal).
All the command are run in MX Linux. They should work in other debian variant.

**WARNING: If the current OS installed on the laptop is working in legacy bios mode (i.e. not using uefi), the OS won't boot after flashing EDK2 and you will have to reinstall.**

## Before we begin
Make sure your linux distribution is up to date.
```
sudo apt update -y
sudo apt upgrade -y
```

## Step 1 - Use Libreboot script to install all the dependencies.
```
sudo apt install -y python-is-python3
git clone https://codeberg.org/libreboot/lbmk
cd lbmk/
sudo ./build dependencies debian 
```
## Step 2 - Use Libreboot to get all the necessary Blobs.


## Step 3 - Configure Coreboot
Download and checkout Coreboot with GIT. I'm using version 4.22.01. If you want to use anything else you can find the tag or the branch here:
https://review.coreboot.org/plugins/gitiles/coreboot
```
sudo apt-get install -y bison build-essential curl flex git gnat libncurses5-dev m4 zlib1g-dev
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
## Step 4 - Create the config and build the rom.
```
nano .config
```
(Copy the config here then ctrl+x > Y to save)
What I'm using (don't forget to change the path):
```
CONFIG_USE_OPTION_TABLE=y
CONFIG_TIMESTAMPS_ON_CONSOLE=y
CONFIG_VENDOR_LENOVO=y
CONFIG_CBFS_SIZE=0x800000
CONFIG_IFD_BIN_PATH="/home/user/t440p/blobs/ifd"
CONFIG_ME_BIN_PATH="/home/user/t440p/blobs/me.bin"
CONFIG_GBE_BIN_PATH="/home/user/t440p/blobs/gbe"
CONFIG_EDK2_BOOT_TIMEOUT=4
CONFIG_HAVE_IFD_BIN=y
CONFIG_HAVE_MRC=y
CONFIG_MRC_FILE="/home/user/t440p/blobs/mrc.bin"
CONFIG_VALIDATE_INTEL_DESCRIPTOR=y
CONFIG_H8_SUPPORT_BT_ON_WIFI=y
CONFIG_HAVE_ME_BIN=y
CONFIG_HAVE_GBE_BIN=y
CONFIG_BOOTSPLASH=y
CONFIG_DRIVERS_PS2_KEYBOARD=y
CONFIG_TPM_DEACTIVATE=y
CONFIG_SECURITY_CLEAR_DRAM_ON_REGULAR_BOOT=y
CONFIG_PAYLOAD_EDK2=y
CONFIG_EDK2_BOOT_MANAGER_ESCAPE=y
CONFIG_EDK2_DISABLE_TPM=y
CONFIG_EDK2_CUSTOM_BUILD_PARAMS=""
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


## Step 5
Flash
