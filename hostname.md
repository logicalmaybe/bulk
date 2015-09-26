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


###### script
````

#/bin/bash 
# Copyright Kalin KOZHUHAROV <kalin@ThinRope.net> 

HOST="me" 
DOMAIN="zo.net" 
# the IP address of our DNS cache (some call it server, WRONG!) 
CACHE="192.168.1.1" 

# If you have tried before, clean up 
echo "Cleaning..." 
for f in /etc/{{host,{dns,nis}domain}name,resolv.conf,hosts}; do mv $f $f-bAcKUP 2>/dev/null;done 
rc-update del domainname >/dev/null 2>&1 

# Create the files needed: 
echo "Creating the files needed..." 
echo ${HOSTNAME} >/etc/hostname 
echo -e "127.0.0.1\t\t${HOST}.${DOMAIN} ${HOST} localhost" >/etc/hosts 
echo -e "domain ${DOMAIN}\nsearch ${DOMAIN}\nnameserver ${CACHE}"    >/etc/resolv.conf 

echo "Updating your config..." 
hostname -F /etc/hostname 
rc-update add hostname boot >/dev/null 2>&1 

# check what we have done 
echo "Checking..." 
set -x 
hostname 
dnsdomainname 
hostname -s 
hostname -d 
hostname -f 
#set +x 
echo 
echo "=====================================================================" 
echo "If you are satisfied with the results, delete the old files by doing:" 
echo -e '\trm -f /etc/*-bAcKUP' 
echo 
echo "You can also see how did your config change by:" 
echo -e '\tfor f in /etc/{{host,{dns,nis}domain}name,resolv.conf,hosts}; do diff -u $f-bAcKUP $f;done' 
echo "=====================================================================" 

````
