# Description

This session describes basic sysadmin tasks

# Analyze disk usage

System users very often do not care about housekeeping of filesystems.
Therefore it is a regular task to analyze filesystem usage after the monitoring complained that the filesystem is full.

Get a handy overview about filesystem usage
```
apt install ncdu
ncdu -x /filesystem
```
Use arrow keys for navigation.

Alternative solution:
```
cd /filesystem
find . -type d -exec du -sxm {} \; | sort -nr 2>&1 | tee /tmp/stats.txt
less -n /tmp/stats.txt
```

This command prints a list of the largest directories in descending order.
Due to the "-x" parameter of "du", this procedure does not cross filesystem borders.
Calculating directory sizes in random order might look a bit inefficent, the unix/linux pagecache helps you to do this in a efficent way

# Control ressource usage of processes

Io- or cpu-intenvise tasks sometimes consume the ressosurces of more important processes.
Therefore it is a good thing to limit the ressources

```
# to start a process with limited resources
ionice -c3 nice -n 20 resource_hungy_script.sh

$ ps auxww|grep the-process |grep -v grep
root      1011  0.0  0.0  31672  1812 ?        Ss   Mai04   0:00 /usr/sbin/the-process -f

$ renice -n 20 1011
$ ionice -c3 -p 1011
```

This reduces the cpu priority to the lowest priority (20) and adds the prcess to the idle io scheduling class (3).
The process will only get the ressources which are not needed by other processes (which are in a better priority/class)

# Pause processes

Sometimee it is useful to pause processes because more important work needs to be executed with higher priority.
Unix provides a possibility to pause processes.

```
# Identify the process
$ ps auxww|grep the-process |grep -v grep
root      1011  0.0  0.0  31672  1812 ?        Ss   Mai04   0:00 /usr/sbin/the-process -f
                                               ^

$ kill -SIGSTOP 1011
$ ps auxww|grep the-process |grep -v grep
root      1011  0.0  0.0  31672  1812 ?        Ts   Mai04   0:00 /usr/sbin/the-process -f
                                               ^
$ kill -SIGCONT 1011

$ ps auxww|grep the-process |grep -v grep
root      1011  0.0  0.0  31672  1812 ?        Ss   Mai04   0:00 /usr/sbin/the-process -f
                                               ^
```

Another possibility is to use the bash process management.
Pause the running process by pressing STRG+z

```
$ iostat -d -x 2 -p /dev/sda
Linux 4.15.0-20-generic (l-099) 	09.05.2018 	_x86_64_	(4 CPU)

Device            r/s     w/s     rkB/s     wkB/s   rrqm/s   wrqm/s  %rrqm  %wrqm r_await w_await aqu-sz rareq-sz wareq-sz  svctm  %util
sda              2,03    2,71     53,93    248,15     0,22     1,35   9,96  33,28    0,58    4,52   0,01    26,52    91,40   0,23   0,11

^Z
[1]+  Angehalten              iostat -d -x 2 -p /dev/sda

$ jobs
[1]+  Angehalten              iostat -d -x 2 -p /dev/sda

# run in background, run in foreground (if this is the only process, you can suppress the identifier
$ bg %1
$ fg %1

# Alternatively
$ kill -SIGCONT %1
```
