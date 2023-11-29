# T440P_Coreboot_EDK2
How-to Coreboot a Thinkpad T440p with the EDK2 Payload.

This guide assumes that you have an unlucked Bios (i.e. able to use flashrom - p internal).
All the command are run in MX Linux. They should work in other debian variant.

# Befor we begin
Make sure your linux distribution is up to date
``
sudo apt update -y··
sudo apt upgrade -y
``
# Step 1
Use Libreboot script to install all the dependencies.
``
  sudo apt install -y python-is-python3  
  git clone https://codeberg.org/libreboot/lbmk  
  cd lbmk/  
  sudo ./build dependencies debian  
``
# Step 2
Use Libreboot to get all the necessary Blobs.


# Step 3
Configure Coreboot
``
  sudo apt-get install -y bison build-essential curl flex git gnat libncurses5-dev m4 zlib1g-dev  
  git clone https://review.coreboot.org/coreboot.git  
  cd coreboot/  
  git checkout d5e80fa9d22ae17ca80276e1d663333e0932c399  
  git submodule update --init --checkout  
  make crossgcc-i386 CPUS=$(nproc)  
  make -C payloads/coreinfo olddefconfig  
  make -C payloads/coreinfo  
  cd util/ifdtool && make  
  nano .config  
  (Copy the config here then ctrl+x > Y to save)  
  make savedefconfig  
  cat defconfig  
  (To make sure everything is OK)  
  make nconfig  
  (save with F6 and exit with F9)  
  make
``


# Step 4
Flash
