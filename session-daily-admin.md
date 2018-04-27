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


