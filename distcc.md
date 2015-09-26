Howto compile binaries for arm platform on remote x86_64 with arm-toolchain.

# distcc for armv7a-hardfloat-linux-gnueabi

### Links
1. http://archlinuxarm.org/developers/distcc-cross-compiling
2. https://wiki.archlinux.org/index.php/Distcc
3. http://crosstool-ng.org/
4. http://mynixworld.info/2013/02/24/distributed-compilation-with-distcc-x86-vs-r-pi/
5. http://distcc.googlecode.com/svn/trunk/doc/web/man/include_server_1.html

Terms

* **host / master**
  * arm device
  * The master is the computer which initiates and distributes the compilation to the configured slaves.
* **workers / slaves**
  * The slaves are running the distcc daemon which accepts incoming compilation requests send by the master.

## Master

````
gcc -dumpmachine
# armv7a-hardfloat-linux-gnueabi

emerge -vat distcc

# add distcc to inet group
usermod --append -G inet distcc

# 5=cores, lzo=LZO compression, cpp=distcc-pump mode
distcc-config --set-hosts 192.168.1.116/5,lzo,cpp
distcc-config --get-hosts
# 192.168.1.116/5,lzo,cpp

nano /etc/portage/make.conf
# FEATURES="-userfetch distcc distcc-pump"
# MAKEOPTS="-j12 -l1" # l=local cores, j=all cores * 2 + 1
````

## Slave

````
# edit distcc config
nano /etc/default/distcc
# ...

/etc/init.d/distcc restart
systemctl daemon-reload

````



## compiling toolchain on Slave (normal user _not_ root!)

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

### check cpu features on host
````
root@arm-host $ grep Features /proc/cpuinfo
Features        : swp half thumb fastmult vfp edsp neon vfpv3
````

### check gcc on host
````
root@arm-host $ gcc -dumpmachine
armv7a-hardfloat-linux-gnueabi
````

# change .config to match cpu/gcc
````
nano .config
````
* replace all ``vfpv3-d16`` with ``neon``
* set ``CT_TARGET_VENDOR`` to ``hardfloat``
* set ``CT_ARCH_ARM_TUPLE_USE_EABIHF`` to ``no`` (would be -gnueabihf)

### build
````
$ ./ct-ng build
[INFO ]  Performing some trivial sanity checks
[INFO ]  Build started 20150925.234252
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
[EXTRA]      target = arm-hardfloat-linux-gnueabi
[EXTRA]  Dumping internal crosstool-NG configuration: done in 0.09s (at 00:02)
....
````

The toolchain will install under "~/x-tools*".

### create links with proper names
````
cd ~/x-tools7h-new/arm-hardfloat-linux-gnueabi/bin/`
nano ./link.sh
````

````
#!/bin/bash

CHOST_GEN="arm-hardfloat-linux-gnueabi"
CHOST_REAL="armv7a-hardfloat-linux-gnueabi"

filename=$(basename "$0")

for file in `ls`; do
        if [[ "$file" == "$filename" ]]; then
                continue
        fi
        ln -s "$file" "$CHOST_REAL${file#$CHOST_GEN}"
done
````

````
chmod +x ./link.sh
# ./link.sh
````

Now the "bin" directory contains links with names that distcc will play nice with.

To get distcc to use these binaries instead of the default system ones, we need to place this directory into the path for the distcc daemon:

### change PATH in distcc daemon
````
nano /etc/init.d/distcc
# PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
PATH=/home/osboxes/x-tools7h-new/arm-hardfloat-linux-gnueabi/bin/:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
````

### restart/reload distcc
````
/etc/init.d/distcc restart
systemctl daemon-reload
````

````
# CC="distcc gcc" CXX="distcc g++" cmake
# CC="distcc gcc" CXX="distcc g++" pump emerge glibc -1
````
