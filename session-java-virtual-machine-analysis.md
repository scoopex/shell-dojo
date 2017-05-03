# Description

This session describes basic things to analyze the behavior of java virtual machines

# The Garbage Collection Log

* The following settings/arguments are useful to analyze the behavior of java virtual machines
  ```
  java -XX:+PrintGCDetails -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationConcurrentTime \
     -XX:+PrintGCApplicationStoppedTime -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC \
     -XX:+UseGCLogFileRotation -XX:+PrintTenuringDistribution -XX:NumberOfGCLogFiles=5 \
     -XX:GCLogFileSize=5M -Xloggc:/var/log/foo/bar/java_gc.log -verbose:gc ....
  ```
* How to find the most recent GC Log
  ```
  cd /var/log/foo/bar/
  ls -latr
  ```
* Add a timestamp to the logfile name
  ```
  -Xloggc:/var/log/foo/bar/java_gc-$(date --date="today" "+%Y-%m-%d_%H-%M-%S").log 
  ```

GC Logs can by analyzed by:

* http://gceasy.io/
* https://www.ibm.com/developerworks/community/groups/service/html/communityview?communityUuid=22d56091-3a7b-4497-b36e-634b51838e11

# Using heapdumps find memory leaks or to size the heap for performance

Its a good idea to have enough space (minimal = -Xmx + 50%) for creating heapdumps.

 * Automatic heapdumps
   Create a heapdump when a out of memory situation appears and stop server immediately.
   ```
   java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/log/tomcat/ -XX:OnOutOfMemoryError="kill -9 %p" ....
   ```
 * Create a manual heapdump
   ```
   su - <java process user>
   jmap -heap:file=heap.dump.out,format=b [PID]
   time jmap -F -dump:file=$(hostname -f)_$(date --date="today" "+%Y-%m-%d_%H-%M-%S").hprof,format=b `pgrep java`
   ```

Heapdumps can be analyzed with: 
(change heap of this tools to load huge heapdumps)
* http://www.eclipse.org/mat/
* https://visualvm.github.io/


# Interacting with a running jvm

*NOTE: If you call tools from the jdk/jre invoke them with the same user and with the same jdk/jre like the running jvm*

 * Create threadump to stdout
   ```
   kill -QUIT $(pgrep java)
   ```
 * Which java are we running currently and with which user?
   ```
   ps auxwwww|grep java
   readlink -f <java process>
   ```
 * Print current state of the Heap
   ```
   jmap -heap <pid>
   ```
 * Get a histogramm of all loaded classes
   ```
   jmap -histo 29655
   ```
 * Invoke a manual GC, useful to get a idea how much memory is really needed
   ```
   jcmd 12572 GC.run
   ```
 * Get a Threadump
   ```
   jstack <pid>
   ```

# Analyzing deployments

 * Get a list of all classes
   ```
   find . -name "*.jar" |while read A; do unzip -l $A; done|grep class|awk '{print $4}'|grep -e "\.class$"|sort -u|sed '~s,\.class$,,;~s,/,.,g'
   ```
 * Analyze a deployment if there are duplicate classes
   ```
   find . -name "*.jar" |while read A; do unzip -l $A; done|grep class|awk '{print $4}'|sort|uniq -c |sort -n|awk '{if ($1 > 1){print $0}}' |tee duplicate-classes
   ```


