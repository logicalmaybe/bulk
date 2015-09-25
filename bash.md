# prompt

````
PS1="\[\033[1;30m\]$(date --iso-8601)|\[\033[0m\] \$? \[\033[1;30m\]|\[\033[0m\]\[\033[1;34m\] \w \[\033[0m\]\r\n\[\033[1;30m\] \t |\[\033[0m\] \[\033[1;31m\]\u\[\033[0m\]@\H \$ "
````

# history

````
HISTSIZE=10000
HISTCONTROL=ignoredups:erasedups
PROMPT_COMMAND="history -a"

# history -a # append?
# history -c # clear?
# history -r # reload?

````

# aliases

````
alias l='ls --color=auto -lap'
alias ll='ls --color=auto -lap'
````
