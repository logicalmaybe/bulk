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

# on Slave
````

````
