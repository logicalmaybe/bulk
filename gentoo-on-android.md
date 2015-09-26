# Pass 1

````

export SDCARD=/data/gentoo
export ROOT=$SDCARD/mnt/gentoo_chroot/
export LD_LIBRARY_PATH=/system/lib:$ROOT/lib:$ROOT/usr/lib:$ROOT/usr/armv7a-hardfloat-linux-gnueabi/lib
export PATH=$ROOT/usr/local/bin:$ROOT/usr/local/sbin:$ROOT/usr/bin:$ROOT/usr/sbin:$ROOT/bin:$ROOT/sbin:$PATH


CHROOT GENTOO on ANDROID
https://wiki.gentoo.org/wiki/Chroot/de


1. Stock Firmware flashen
2. rooten 
3. "rooted ssh/sftp daemon" installieren


8. CPU wird runtergetaktet und bleibt dort?
# cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
# echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# # echo ondemand > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# # echo powersave > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# cat /proc/cpuinfo


9. switching IO scheduler
https://www.kernel.org/doc/Documentation/block/switching-sched.txt
cat /sys/block/mmcblk0/queue/scheduler
echo noop > /sys/block/mmcblk0/queue/scheduler
echo deadline > /sys/block/mmcblk0/queue/scheduler
cat /sys/block/mmcblk0/queue/scheduler


### stop some services to free up memory
/system/bin/stop
setprop ctl.stop media
setprop ctl.stop adbd
setprop ctl.stop drm
setprop ctl.stop dbus

### prepare sd card

# busybox fdisk /dev/block/mmcblk0 -l
	o - to create a new dos partition table
	n -> new partition
		=> p -> primary
		=> 1 -> partition nr. 1
		=> from start - to end
		(no need for swap as it's on internal flash)
	w -> write / persist changes

### check changes with:
busybox fdisk /dev/block/mmcblk0 -l


### create ext2 filesystem

busybox mkfs.ext2 /dev/block/mmcblk0p1 -b 2048 -L ANDROID-GENTOO-8GB
# there should be something like this in output:
# > 237 block groups
# > 2048 inodes per group

2048 * 237 = 485376 maximal inodes on this partition
inodes can't be changed after creation!
this number is important as portage has about 200k files! adjust this accordingly with -i/-I parameter (-I 128 -i 8192)

# # busybox mkfs.ext2 /dev/block/mmcblk0p1 -b 2048 -i 2048 -L ANDROID-GENTOO
> 237 block groups
> 16376 inodes per group

### mount sd card
mkdir /data/gentoo
mount -t ext2 /dev/block/mmcblk0p1 /data/gentoo
cd /data/gentoo


/system/bin/mount -o remount,rw /system

### add DNS entries to resolv.conf for wget to work
echo 'nameserver 8.8.8.8' > /system/etc/resolv.conf
echo 'nameserver 8.8.4.4' >> /system/etc/resolv.conf



### load latest busybox
busybox wget http://www.busybox.net/downloads/binaries/latest/busybox-armv7l
chmod 0755 ./busybox-armv7l
busybox mv busybox-armv7l /system/xbin/busybox

### load latest toybox
busybox wget http://www.landley.net/toybox/bin/toybox-armv6l
chmod 0755 ./toybox-armv6l
busybox mv toybox-armv6l /system/xbin/toybox



### load stage3 tarball and portage
busybox wget http://mirror.switch.ch/ftp/mirror/gentoo/releases/arm/autobuilds/current-stage3-armv7a_hardfp/stage3-armv7a_hardfp-20150730.tar.bz2.DIGESTS
busybox wget http://mirror.switch.ch/ftp/mirror/gentoo/releases/arm/autobuilds/current-stage3-armv7a_hardfp/stage3-armv7a_hardfp-20150730.tar.bz2
busybox wget http://mirror.switch.ch/ftp/mirror/gentoo/snapshots/portage-latest.tar.bz2.md5sum
busybox wget http://mirror.switch.ch/ftp/mirror/gentoo/snapshots/portage-latest.tar.bz2

extract both

 failed with busyboy bzcat and default tar!!!!
 
# # tar xjpf stage3-*.tar.bz2 -C /data/gentoo
# # tar xjf portage-*.tar.bz2 -C /data/gentoo/usr


# toybox bzcat stage3-*.tar.bz2 | busybox tar -xf - -C /data/gentoo
# toybox bzcat portage-*.tar.bz2 | busybox tar -xf - -C /data/gentoo/usr/


### chroot
mount -o remount,rw /
mkdir /tmp
busybox mount -t tmpfs tmpfs /tmp

mkdir /data/gentoo/tmp
mkdir /data/gentoo/sys
mkdir /data/gentoo/dev
mkdir /data/gentoo/proc

busybox mount -o bind /tmp /data/gentoo/tmp
busybox mount -o bind /sys /data/gentoo/sys
busybox mount -o rbind /dev /data/gentoo/dev
busybox mount -t proc none /data/gentoo/proc

rm /data/gentoo/dev/fd
ln -s /proc/self/fd /data/gentoo/dev/fd
ln -s /proc/mounts /data/gentoo/etc/mtab


busybox cp /system/etc/resolv.conf /data/gentoo/etc/resolv.conf

### clean PATH
PATH=/bin:/usr/bin:/sbin:/usr/sbin

### start chroot
/system/xbin/busybox chroot /data/gentoo /bin/bash
/usr/sbin/env-update
source /etc/profile
export PS1="(chroot) $PS1"

### copy make.conf
emerge --sync






#######################################




am start -a android.settings.AIRPLANE_MODE_SETTINGS

#adb shell am start -a android.intent.action.VIEW -d   "file:///mnt/sdcard/DCIM/Camera/test.3gp" -t "video/*" 


# start gentoo
#    mount -t ext2 /dev/block/mmcblk0p1 /data/gentoo
#    /data/gentoo/init.sh > /data/gentoo/fail-init.log


busybox mkfs.ext2 /dev/block/mmcblk0p1 -b 2048 -i 2048 -L ANDROID-GENTOO-2GB
mount -t ext2 /dev/block/mmcblk0p1 /data/gentoo
cd /mnt/gentoo

busybox bzip2 -dc rap-stage3-armv7a_hardfp-20141210.tar.bz2 | busybox tar -xf - -C /data
/data/gentoo/startprefix
emerge --sync
layman -s heroxbd


mkdir /data/gentoo/proc
mkdir /data/gentoo/sys
mkdir /data/gentoo/dev

busybox mount -t proc proc /data/gentoo/proc
busybox mount --rbind /dev /data/gentoo/dev
busybox mount --rbind /sys /data/gentoo/sys

busybox chroot /data/gentoo /bin/bash && source /etc/profile


/sys/class/power_supply/bq24185

cd /sys/devices/i2c-0/0-0040/leds/
cd blue
echo 255 > brightness

echo 0 > /sys/devices/i2c-0/0-0040/leds/red/brightness
echo 0 > /sys/devices/i2c-0/0-0040/leds/green/brightness
echo 0 > /sys/devices/i2c-0/0-0040/leds/blue/brightness

mount -o remount,rw /
mount -o remount,rw /system
nano /system/etc/hw_config.sh
/system/bin/sh
# change hostname (dhcpcd)
# /system/build.prop
# net.hostname = UrNewHostname


# start gentoo
    mount -t ext2 /dev/block/mmcblk0p1 /data/gentoo
	/data/gentoo/startprefix
	/data/gentoo/sbin/rc default
	
	/data/gentoo/sbin/rc default &> /data/gentoo/fail.log
	
	cat /data/gentoo/fail.log

	
	
	cd /system/etc; find . | grep -v cacert |grep -v .xml |grep -v .jpg | grep -v .png
	cat ./vold.fstab

	
[    8.638580] mmc1: host does not support reading read-only switch. assuming write-enable.
[    8.648559] mmc1: new high speed SDHC card at address e624
[     8.648803] sdio_al mmc1:e624: Probing..
[    8.649291] mmcblk0: mmc1:e624 SU08G 7.40 GiB
[    8.649536]  mmcblk0: p1


./platform/msm_sdcc.3/mmc_host
./msm_sdcc.4/mmc_host/mmc1/mmc1:e624/block/mmcblk0/mmcblk0p1

/sys/devices/platform/msm_sdcc.4/mmc_host/mmc1/mmc1\:e624/block/mmcblk0/mmcblk0p1/
	
dev_mount gentoo /data/gentoo auto /devices/platform/msm_sdcc.4/mmc_host/mmc1



	
### hardcoded uid/gid: https://android.googlesource.com/platform/system/core/+/master/include/private/android_filesystem_config.h
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1000 system
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1001 radio
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1002 bluetooth
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1003 graphics
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1004 input
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1005 audio
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1006 camera
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1007 log
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1008 compass
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1009 mount
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1010 wifi
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1011 adb
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1012 install
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1013 media
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1014 dhcp
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1015 sdcard_rw
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1016 vpn
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1017 keystore
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1018 usb
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1019 drm
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1020 mdnsr
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1021 gps
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1022 unused1
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1023 media_rw
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1024 mtp
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1025 unused2
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1026 drmrpc
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1027 nfc
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1028 sdcard_r
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1029 clat
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1030 loop_radio
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1031 media_drm
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1032 package_info
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1033 sdcard_pics
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1034 sdcard_av
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1035 sdcard_all
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1036 logd
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1037 shared_relro
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 1038 dbus
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 2000 shell
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 2001 cache
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 2002 diag
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3001 net_bt_admin
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3002 net_bt
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3003 inet
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3004 net_raw
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3005 net_admin
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3006 net_bw_stats
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3007 net_bw_acct
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 3008 net_bt_stack
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 9997 everybody
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 9998 misc
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 9999 nobody
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 10000 app
useradd --user-group --no-create-home --home-dir /dev/null --shell /data/gentoo/bin/false --uid 100000 user






/system/bin/stop
/system/bin/setprop ctl.stop media				# media
/system/bin/setprop ctl.stop adbd				# adb
/system/bin/setprop ctl.stop zygote				# system
/system/bin/setprop ctl.stop drm				# drm
/system/bin/setprop ctl.stop qmuxd				# Qualcomm Management Interface Multiplexer
/system/bin/setprop ctl.stop vold				# Android Vold (Volume Daemon) sub-system
/system/bin/setprop ctl.stop iddd
/system/bin/setprop ctl.stop debuggerd
/system/bin/setprop ctl.stop servicemanager
/system/bin/setprop ctl.stop installd
/system/bin/setprop ctl.stop keystore

/system/bin/setprop ctl.stop mltlusbd
/system/bin/setprop ctl.stop atfwd-daemon
/system/bin/setprop ctl.stop wpa_supplicant
/system/bin/setprop ctl.stop netd # wooot?
/system/bin/setprop ctl.stop netmgrd # wooot?




# Pos1 und Home Keys funzen nicht!
http://blog.florian-felgenhauer.de/2012/10/pos1-und-ende-tasten-mit-putty-nutzen/
set terminal type (unter "Connection" => "Data") in Putty to "putty"


1. charger
- set up charger threashold
- show current charge in bash prompt
cat /sys/devices/i2c-0/0-0055/power_supply/bq27520/capacity

2. camera
- connect both cams (sub?) per ip

3. setup network over USB

4. compile kernelmono
5. install mono

6. fix bash history from multiple sessions
http://nodsw.com/blog/leeland/2011/10/26-all-bash-history-forever-and-across-multiple-sessions



7. Uhr/Timezone => UTC => Berlin
hwclock --show
emerge -vta sys-libs/timezone-data
echo "Europe/Berlin" > /data/gentoo/etc/timezone
emerge --config timezone-data

rap-stage3 had Europe/* timezones missing!
https://wiki.gentoo.org/wiki/System_time


8. CPU wird runtergetaktet und bleibt dort?
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo ondemand > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
echo powersave > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
cat /proc/cpuinfo


9. switching IO scheduler
https://www.kernel.org/doc/Documentation/block/switching-sched.txt
cat /sys/block/mmcblk0/queue/scheduler
echo noop > /sys/block/mmcblk0/queue/scheduler
echo deadline > /sys/block/mmcblk0/queue/scheduler
cat /sys/block/mmcblk0/queue/scheduler


10. look for strage things in dmesg
cat /proc/kmsg
suspend goes wild?
find /sys | grep suspend
````


# Pass 2
````
rm /data/gentoo/mnt/gentoo_chroot/etc/mtab
ln -s /proc/mounts /data/gentoo/mnt/gentoo_chroot/etc/mtab

busybox mount -t proc proc /data/gentoo/mnt/gentoo_chroot/proc
busybox mount --rbind /sys /data/gentoo/mnt/gentoo_chroot/sys
busybox mount --rbind /dev /data/gentoo/mnt/gentoo_chroot/dev
#busybox mount -o bind /tmp /data/gentoo/mnt/gentoo_chroot/tmp
busybox mount -t tmpfs tmpfs /data/gentoo/mnt/gentoo_chroot/tmp

#PATH=/bin:/usr/bin:/sbin:/usr/sbin /system/xbin/busybox chroot /data/gentoo/mnt/gentoo_chroot /bin/bash
PATH=/bin:/usr/bin:/sbin:/usr/sbin:/data/gentoo/bin:/data/gentoo/usr/bin chroot /data/gentoo/mnt/gentoo_chroot /bin/bash
/usr/sbin/env-update
source /etc/profile
export PS1="(chroot) $PS1"



### install layman
layman --sync-all
bash: layman: command not found

### set proper timezone
# emerge --config sys-libs/timezone-data

### set locale
# locale-gen
# eselect locale list
# env-update && source /etc/profile


# lspci
emerge sys-apps/pciutils

# install some stuff
emerge sys-apps/pciutils layman app-misc/screen



export PATH=/bin:/usr/bin:/sbin:/usr/sbin:/data/gentoo/bin:/data/gentoo/usr/bin chroot /data/gentoo/mnt/gentoo_chroot /bin/bash

export PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
````


# Pass 3
````
rm /data/gentoo/mnt/gentoo_chroot/etc/mtab
ln -s /proc/mounts /data/gentoo/mnt/gentoo_chroot/etc/mtab


busybox mount -t proc proc /data/gentoo/mnt/gentoo_chroot/proc
busybox mount --rbind /sys /data/gentoo/mnt/gentoo_chroot/sys
busybox mount --rbind /dev /data/gentoo/mnt/gentoo_chroot/dev
busybox mount -t tmpfs tmpfs /data/gentoo/mnt/gentoo_chroot/tmp

export PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
/system/xbin/busybox chroot /data/gentoo/mnt/gentoo_chroot /bin/bash


/usr/sbin/env-update
source /etc/profile
export PS1="(chroot) $PS1"

rc default

````


