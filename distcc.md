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

### Install crosstool-ng

````
mkdir /home/osboxes/crosstool-ng
cd /home/osboxes/crosstool-ng
mkdir -p cross/src
cd cross
git clone https://github.com/crosstool-ng/crosstool-ng.git
cd crosstool-ng
./bootstrap
./configure --prefix=/home/osboxes/crosstool-ng/cross
make
make install
````

### create .config file

````
cd /home/osboxes/crosstool-ng/cross/bin/
wget http://archlinuxarm.org/builder/xtools/xtools-dotconfig-v7 -O .config
./ct-ng oldconfig
````

### check cpu features
````
grep Features /proc/cpuinfo
# Features        : swp half thumb fastmult vfp edsp neon vfpv3
````

### check gcc
````
root@localhost $ gcc -dumpmachine
armv7a-hardfloat-linux-gnueabi
````

# change .config to match cpu/gcc
* replace all ``vfpv3-d16`` with ``neon``
* set ``CT_TARGET_VENDOR`` to ``hardfloat``
* set ``CT_ARCH_ARM_TUPLE_USE_EABIHF`` to ``no`` (would be -gnueabihf)
````
nano .config
````

### build
````
$ ./ct-ng build
[INFO ]  Performing some trivial sanity checks
[INFO ]  Build started 20150925.232805
[INFO ]  Building environment variables
[WARN ]  Directory '/home/osboxes/cross/src' does not exist.
[WARN ]  Will not save downloaded tarballs to local storage.
[EXTRA]  Preparing working directories
[EXTRA]  Installing user-supplied crosstool-NG configuration
[EXTRA]  =================================================================
[EXTRA]  Dumping internal crosstool-NG configuration
[EXTRA]    Building a toolchain for:
[EXTRA]      build  = x86_64-unknown-linux-gnu
[EXTRA]      host   = x86_64-unknown-linux-gnu
[EXTRA]      target = arm-unknown-linux-gnueabihf
....
````
