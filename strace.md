# howto strace other terminal

http://unix.stackexchange.com/questions/93892/screen-is-terminating

1. Open 2 new terminals
2. In the first terminal, log in to the remote machine as ``root``
3. In the second terminal, log in to the remote machine as ``normal user``
4. Use ``ps`` to get the ``PID`` of the normal user's shell process in the second terminal
5. In the first terminal, run ``strace -f -p SHELLPID``
6. In the second terminal, run screen and watch it fail
7. In the first terminal, you now have the strace log you need to find out what's really wrong.

The key addition to the strace command is the -f option, which tells it to trace child processes. You need it to trace the screen that will be a child of the shell process you specified with -p.

I also like to use -ff and specify an output file with -o, as in

``strace -ff -o /tmp/screentrace -p SHELLPID``

which will create a separate output file for each child process. Afterward you read them with ``less /tmp/screentrace*`` and the result is usually cleaner than what you get using a single ``-f``.
