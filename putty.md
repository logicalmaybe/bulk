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
