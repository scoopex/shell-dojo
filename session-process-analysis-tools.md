# Description

This session describes how to analyze the behavior of a current process

# Tracing process behavior - STrace

Strace is a diagnostic, debugging and instructional userspace utility for Linux. It is used to monitor and tamper with interactions between processes and the Linux kernel, which include system calls, signal deliveries, and changes of process state. The operation of strace is made possible by the kernel feature known as ptrace.

Strace might be useful:
- for debugging library, class or configfile loading order
- for debugging non obvious permission problems which
- for debugging process interaction
- ....

The following commands can help to find a first detailed impression whats going wrong with a running software:

 * Create statistical data of the characteristics of syscalls, executiontimes and errors of a running (java) process an subprocesses/threads
   Wait a few seconds and stop tracing by hitting STRG+c
   ```
   strace -f -c -p <PID> 
   strace -f -c -p $(pgrep java)
   ```
 
 * Trace a running process and write all information to /tmp/trace.out
   ```
   strace -o /tmp/trace.out -f <PID>
   ```

 * Analyze logged data
   ``` 
   # normally vim discovers automatically strace files
   vim -c "set filetype=strace" -c "syntax on" /tmp/trace.out
   
   # get more details about the syscalls
   apt-get install manpages-dev
   man 2 open

   # Filter for a specific PID
   grep -e "^19239" /tmp/foo
   ```

 * Run the command "curl" and store reduced trace information to /tmp/trace.out
   ```
   strace -o /tmp/trace.out -f -e trace=file curl https://www.google.de # File operations 
   strace -o /tmp/trace.out -f -e trace=network https://www.google.de   # Network operations 
   ...
   # see strace manpage
   ```

 * Extensively log the behavior of a process : syscall, parameters, environment variables, ...
   ```
   strace -frvT -s128 -o /tmp/foo -p <PID>
   ```
  
 * Trace only open() syscalls of this process and all of its subprocesses
   ```
   strace -f -o fileio.dump. -e trace=open tweaks
   ```


# Tracing/Analyzing network communication

Snooping network communication uncovers the details the communication with your webserver and application server.
See also: https://danielmiessler.com/study/tcpdump/#protocol

 * Snoop on a remotehost any network interfaces for interaction with that system and exclude ssh communication
   ```
   tcpdump -i any -w /tmp/tcpdump.snoop "not ( port 22 and tcp )"
   tcpdump -i any -n -w /tmp/tcpdump.snoop "not ( port 22 and tcp )" # suppress dns resolving
   ```

 * Only output new tcp connection attempts
   If we want to match packets with only the SYN flag set, the 14th byte would have a binary value of 00000010 which equals 2 in decimal
   ```
   tcpdump -i any 'host 10.6.254.1 and port 1418 and (tcp[13] = 2)'
   ```

 * Trace only every 7th mysql connection (tcp port 3306) on systems with a very high troughput.
   Tcpdump misses pakets on systems with a high network utilisation. Reducing traced connections by a modulus 7 reduces this problem dramatically.
   ```
   tcpdump -i eth0 -s 65535 -x -n -q -tttt 'port 3306 and tcp[1] & 7 == 2 and tcp[3] & 7 == 2' > mysql.tcp.txt
   ```

 * Extract cleartext information from tcpdump and store communication in dedicated files for every connection and direction
   ```
   cd /tmp/
   tcpflow -r <tcpdumpfile>
   ls -l *.txt
   ```

 * Execute tcpdump on a remote host a feed traced data to a locally running wireshark
   ```
   ssh root@HOST sudo "tcpdump -U -i any -s0 -w - 'not port 22'" | wireshark -k -i -
   ```
