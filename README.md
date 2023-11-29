# T440P_Coreboot_EDK2
How-to Coreboot a Thinkpad T440p with the EDK2 Payload.

This guide assumes that you have an unlucked Bios (i.e. able to use flashrom - p internal).
All the command are run in MX Linux. They should work in other debian variant.

# Befor we begin
Make sure your linux distribution is up to date
sudo apt update
sudo apt upgrade
s
# Step 1
Use Libreboot script to install all the dependencies.
# Step 2
Use Libreboot to get all the necessary Blobs.
sudo apt install python-is-python3
git clone https://codeberg.org/libreboot/lbmk
cd lbmk/
sudo ./build dependencies debian

# Step 3
Configure Coreboot
git clone https://review.coreboot.org/coreboot.git
cd ~/coreboot
git checkout d5e80fa9d22ae17ca80276e1d663333e0932c399
git submodule update --init --checkout

cd util/ifdtool && make


# Step 4
Flash
