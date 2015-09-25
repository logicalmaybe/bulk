http://blog.sanctum.geek.nz/putty-configuration/

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


# UTF-8

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
