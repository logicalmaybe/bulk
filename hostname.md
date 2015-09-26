https://forums.gentoo.org/viewtopic.php?p=519288

# defaults
````
nano /etc/hosts
# 127.0.0.1       localhost

nano /etc/hostname
# st15i

nano /etc/resolv.conf
# search mydomain.topdomain 
# nameserver 8.8.8.8

rc-update add hostname default
````

Logout an login again.

###### last resort
````
$ find /etc -type f | xargs grep -i "oldhostname"
$ find / -type f | xargs grep -i "oldhostname"
````
