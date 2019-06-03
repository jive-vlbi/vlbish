##   vlbish 

control local and remote (== @stations) Mark5s, FlexBuffs, FiLa10Gs, sfxc cluster nodes from one place.

Usage:

```bash
$ vlbish [options] [-c "command" [-c "command"]] [path/to/script [arg1 arg2 ... argN]]
```

see 'vlbish --help' for [options] or 'vlbish --usage' for an extended help.
As always, anything in `[]` means it's optional.

`vlbish` can be scripted. This repository includes [some useful examples](examples/).
directory directory

##   Options:
            --help             show short help
            --usage            show this built-in help
            --version          print version and exit succesfully
            --nosql            do not attempt to connect to DB
            --debug            print debug output
            --queries          print queries as they are sent to the DB
            --testdb           connect to test database in stead of production

##  Description:

vlbish reads lines of input from stdin, command line arguments or a script file.

Each line of input either sends VSI/S commands via TCP/IP to a selection of hosts,
executes shell command(s) using ssh on a selection of hosts or is a vlbish built-in command. 

There are three (3) formats for a line of input:

- [`<selection>`] `/ commands`

  (attempts to) send the VSI/S commands to all machines in <selection> using TCP/IP

- [`<selection>`] `! shell commands`

  (attempts to) execute the shell commands on all machines in <selection> using ssh if no selection given uses last-known selection

- `commands`

  if recognized as vlbish built-in command, executes as that, otherwise interpreted as VSI/S and sent to last-known selection using TCP/IP

In this version of vlbish it is possible to put multiple 'logical' lines of
input on one physical line of input (all characters up to and including the
newline) by separating the logical lines with two or more semicolons:

```
       > 0/tstat?;; sleep 2;; 0!uname -a
```

is now interpreted and executed as if three separate lines of input were given: a VSI/S command, a vlbish builtin command and a shell command.

vlbish has lots of built-in knowledge to resolve <selection> into user/ip/port tuples. It also connects to the database @JIVE in order to resolve names and associations and equipment types. It is possible to set options in <selection> affecting the execution of <commands> (see below).


default ports used:

  `<VSI/S command>`  = 2620 [mark5 control port]

  `<shell command>`  = 22   [ssh]
