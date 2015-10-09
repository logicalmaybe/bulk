http://blog.sanctum.geek.nz/putty-configuration/

# font
Check out "Consolas".

# terminfo

1. set ``Connection => Data => Terminal-type string`` in putty to ``putty-256color``
2. merge ``ncurses`` to get ``terminfo`` files
    (``minimal`` useflag may prevent installing all terminfo files... so remove it)

````
$ USE="-minimal" emerge -uvat ncurses

$ ls -lap /usr/share/terminfo/* | wc -l
2898
````

### test

````
for ((color = 0; color <= 255; color++)); do
tput setaf "$color"
printf "test"
done

````

Use ``reset`` after test, if needed.

#### use screen scroll back buffer in putty
http://unix.stackexchange.com/questions/18006/can-mouse-wheel-scrolling-work-in-a-usr-bin-screen-session

Add this to .screenrc: ``termcapinfo putty-256color ti@:te@``
In tmux, it'd be something like (.tmux.conf): ``set -g terminal-overrides 'xterm*:smcup@:rmcup@'``



# UTF-8

https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Configure_locales

### insert ``de_DE.UTF-8 UTF-8`` into ``/etc/locale.gen``
````
# echo "de_DE.UTF-8 UTF-8" >> /etc/locale.gen
````

### regenerate locales
````
locale-gen
````

### select proper locale
````
eselect locale list
eselect locale set 2
eselect locale list
````

### insert ``LC_MESSAGES="C"`` into ``/etc/env.d/02locale`` to revert messages to default lang
````
nano /etc/env.d/02locale
# LANG="de_DE.utf8"
# LC_MESSAGES="C"

````

### update env
````
env-update && source /etc/profile
````

### check all vars
````
locale

# LANG=de_DE.utf8
# LC_CTYPE="de_DE.utf8"
# LC_NUMERIC="de_DE.utf8"
# LC_TIME="de_DE.utf8"
# LC_COLLATE="de_DE.utf8"
# LC_MONETARY="de_DE.utf8"
# LC_MESSAGES=C
# LC_PAPER="de_DE.utf8"
# LC_NAME="de_DE.utf8"
# LC_ADDRESS="de_DE.utf8"
# LC_TELEPHONE="de_DE.utf8"
# LC_MEASUREMENT="de_DE.utf8"
# LC_IDENTIFICATION="de_DE.utf8"
# LC_ALL=
````


### set keymap to ``de``
````
/etc/conf.d/keymaps

# keymap="de"
````

### set languages globaly
````
nano /etc/portage/make.conf
# LINGUAS="de en"
````
