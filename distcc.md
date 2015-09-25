# distcc for armv7a-hardfloat-linux-gnueabi

http://archlinuxarm.org/developers/distcc-cross-compiling


* Host: arm device
* Slave: all workers

# on Host
````
root@localhost $ gcc -dumpmachine
armv7a-hardfloat-linux-gnueabi

root@localhost $ nano /etc/portage/make.conf

FEATURES="-userfetch distcc distcc-pump"
MAKEOPTS="-j12 -l1" # l=local cores, j=all cores * 2 + 1
````

# on Slave (normal user _not_ root!)
````
mkdir /home/osboxes/crosstool-ng
   15  cd /home/osboxes/crosstool-ng
   16  mkdir -p cross/src
   17  cd cross
   18  git clone https://github.com/crosstool-ng/crosstool-ng.git
   19  cd crosstool-ng
   20  bootstrap
   21  ./bootstrap
   22  ./configure --prefix=/home/osboxes/crosstool-ng/cross
   23  make
   24  make install
   25  cd /home/osboxes/crosstool-ng/cross/bin/
   26  wget http://archlinuxarm.org/builder/xtools/xtools-dotconfig-v7 -O .config
   27  ls .config
   28  make oldconfig
   29  ./ct-ng oldconfig
   30  cd /home/your_user/cross/bin
   31  ./ct-ng build
   32  nano .config
   33  cp .config .config_GUT
   34  nano .config
   35  su
   36  nano .config
   37  ./ct-ng build
   38  make menuconfig
   39  ./ct-ng menuconfig
   40  nano .config
   41  ./ct-ng build
   42  nano .config
   43  ./ct-ng build
   44  nano .config
   45  ./ct-ng build
   46  nano .config
   47  ./ct-ng build
   48  nano .config
   49  ./ct-ng build
   50  history

````
