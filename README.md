##   vlbish 

control local and remote (== @stations) Mark5s, FlexBuffs, FiLa10Gs, sfxc cluster nodes from one place.

Usage:

```bash
$ vlbish [options] [-c "command" [-c "command"]] [path/to/script [arg1 arg2 ... argN]]
```

see `vlbish --help` for [options] or `vlbish --usage` for an extended help.
As always, anything in `[]` means it's optional.

`vlbish` can be scripted. This repository includes some useful examples in the [examples](examples/) directory.


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

  (attempts to) send the VSI/S commands to all machines in \<selection> using TCP/IP. If no selection specified sends the command(s) to the last-used selection (if any).

- [`<selection>`] `! shell commands`

  (attempts to) execute the shell commands on all machines in \<selection> using ssh. If no selection specified sends the command(s) to the last-known selection (if any).

- `commands`

  if recognized as vlbish built-in command, executes as that, otherwise interpreted as VSI/S and sent to last-known selection using TCP/IP

`vlbish` remembers/reuses the last-known selection and displays it in the prompt. The selection remains 'active' until it is modified using a version of the syntax which specifies a (new) selection.

In this version of vlbish it is possible to put multiple 'logical' lines of input on one physical line of input (all characters up to and including the newline) by separating the logical lines with two or more semicolons:

```
       > 0/tstat?;; sleep 2;; 0!uname -a
```

is now interpreted and executed as if three separate lines of input were given: a VSI/S command, a vlbish builtin command and a shell command.

vlbish has lots of built-in knowledge to resolve \<selection> into user/ip/port tuples. It also connects to the database @JIVE in order to resolve names and associations and equipment types. It is possible to set options in \<selection> affecting the execution of \<commands> (see below).


default ports used:

  `<VSI/S command>`  = 2620 [mark5 control port]

  `<shell command>`  = 22   [ssh]


## template commands

With the possibility of setting options on hosts, generic scripting becomes possible.

Define aliases for two hosts. Both have the same `mtu` option set, but to different values:

```bash
alias sender host1.domain.com:4004,mtu=2500
alias receiver 192.168.1.2:8008,mtu=9000
```

Suppose the `set_mtu=...` command exists that can be sent to a host to set a specific MTU value, it can now be done for both (or individual) hosts like below; the command engine inside `vlbish` replaces strings of the form `{key}` in the command with the value for that `key` from the options set for a particular host:

```basn
> sender,receiver/set_mtu={mtu}
```

An elaborate use case can be found in the [rdbe_10g_init.scr](examples/rdbe_10g_init.scr) example - using that script several machines can be configured with their respective IP/ethernet configuration.

Note that in script files double `{{ ... }}` braces are needed to escape the first level of string substitution. 
Lines in scripts are "template" commands for the script parser inside `vlbish` - arguments can be passed to scripts and strings of the form `{N}` (N is a number >= 0) are replaced by the scripts' arguments, with the first *argument* having index 0.

The engine is not yet smart enough to disambiguate between `{N}` and `{key}`
