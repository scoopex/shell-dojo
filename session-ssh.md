# Description

This session describes probably useful ssh things

# Using the SSH agent

Your desktop environment might start the implementation of a ssh-agent automatically after login.
"ssh-agent" acts as a daemon process which provides access to unlocked private ssh-keys.

Review your process list:
```
# Gnome keyring agent
$ env|grep -i SSH
SSH_AUTH_SOCK=/run/user/1000/keyring/ssh
SSH_AGENT_LAUNCHER=gnome-keyring
$ pstree -p |less -p gnome-keyring

# standard ssh-agent
$ env|grep -i SSH
SSH_AUTH_SOCK=/tmp/ssh-yLtkCf9HPk8D/agent.29086
SSH_AGENT_PID=29087
$ pstree -p |less -p 29087
```
As you can see these environment variables might be set by the ssh-agent or gnome-keyring and inherited to child processes

You can add multiple ssh-key by invoking "ssh-add"
```
$ ssh-add
Enter passphrase for /home/mschoechlin/.ssh/id_rsa:
Identity added: /home/mschoechlin/.ssh/id_rsa (/home/mschoechlin/.ssh/id_rsa)
mschoechlin@ntbk0435(2017-05-08 17:02:40) ~/src/git/github/shell-dojo [master]
$ ssh-add /home/mschoechlin/.ssh/id_rsa
id_rsa          id_rsa.jenkins  id_rsa.pub      id_rsa.thebest
mschoechlin@ntbk0435(2017-05-08 17:02:40) ~/src/git/github/shell-dojo [master]
$ ssh-add /home/mschoechlin/.ssh/id_rsa.thebest
Enter passphrase for /home/mschoechlin/.ssh/id_rsa.thebest:
Identity added: /home/mschoechlin/.ssh/id_rsa.thebest (/home/mschoechlin/.ssh/id_rsa.thebest)
mschoechlin@ntbk0435(2017-05-08 17:02:54) ~/src/git/github/shell-dojo [master]
$ ssh-add -l
4096 SHA256:i1cp79XvQWw/N9/IOd17gQdXosycFKJi49eSgg2PbUI /home/mschoechlin/.ssh/id_rsa.thebest (RSA)
4096 SHA256:i1cp79XvQWw/N9/IOd17gQdXosycFKJi49eSgg2PbUI marc.schoechlin@256bit.org (RSA)
```


# Simple SSH Tunnels

HINT: Use highports (>1024) for forwards

 * Connect to 256bit.org and create a tunnel to the remote mysql server
   ```
   # Local system, no listen  port
   $ netstat -nl|grep 13306
   # Local system, invoke tunnel
   $ ssh -L 13306:127.0.0.1:3306 256bit.org

   # Remote system, listen port of the service
   $ netstat -nl|grep 3306
   tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN    

   # Local system, voilà ssh client opened a tunnel
   $ sudo netstat -nlp|grep 3306
   tcp        0      0 127.0.0.1:13306         0.0.0.0:*               LISTEN      29419/ssh           
   tcp6       0      0 ::1:13306               :::*                    LISTEN      29419/ssh    
   ```
 * Connect to 256bit.org and create a tunnel from remote to local system
   ```
   # Local system, invoke tunnel
   $ nc -v -w 2 10.1.1.1 3306
   Connection to 10.1.1.1 3306 port [tcp/mysql] succeeded!

   $ ssh -R 13306:10.1.1.1:3306 256bit.org

   # Remote system, listen port is open
   $ sudo netstat -nlp|grep 13306
   tcp        0      0 127.0.0.1:13306         0.0.0.0:*               LISTEN      2251/11         
   tcp6       0      0 ::1:13306               :::*                    LISTEN      2251/11  
   ```

# Advanced ssh tunnel, socks 4 tunnel

Socks v4 provides a very comfortable method for automatic ssh tunnels.

Preconditions/Environment:

 * system with ssh client is not capable to connecto webserver
 * ssh_server is capable to connec to webserver

Example:
```
   [Client/TSocks]----->[SSH_Server]------>[Webserver]
```

How to use a transparent socks tunnel:

 * Install Tsocks
   ```
   $ apt-get install tsocks
   ```
 * Configure /etc/tsocks.conf
   ```
   # Change the follwing values
   server = 127.0.0.1
   server_type = 4
   server_port = 1080
   ```
 * Connect to remote server and enable the ssh socks feature
   ```
   $ ssh 256bit.org -D 1080
   ```
 * Invoke "tsocks" to set the LD_PRELOAD connect wrapper
   ```
   # Invokes a bash shell, and set LD_PRELOAD to wrap connect libc functions
   $ tsocks
   $ nc -v -w 2 10.1.1.1 3306
   Connection to 10.1.1.1 3306 port [tcp/mysql] succeeded!
   ```

# Configure SSH Client

Creating a ssh config is useful for the following reasons:

 * you can use bash completion for the configured hosts
 * you can set useful options for specific connections
 * you can set useful options for any connection
 * this can be used for ssh and scp


Some probably useful settings:
```
##########################################################################################################
####
#### SSH CONFIG: ~/.ssh/config
####
#### See manpage: man ssh_config
####
#### For each parameter, the first obtained value will be used.

# just defined these Hostnames
Host foo.bar.baz.org
Host foo2.bar.baz.org
Host foo2.bar1.baz.org

# define a host with alias "apache" and some defualt settings
Host people.apache.org apache
   ForwardX11 no
   ForwardAgent no
   User scoopex
   Hostname people.apache.org

# define a host, do fast connects in environemnts with broken dns *sigh* and use specific ssh key and user
Host apt.pain.de
   Hostname apt.pain.de
   GSSAPIAuthentication no
   IdentityFile ~/.ssh/id_rsa.jenkins
   User aptly

# define a default connection to local test vms (i.e. kitchen vms) 
# and prevent hassle with the known_hosts, might be o.k. for local vms
Host vm 127.0.0.1 localhost
   Hostname 127.0.0.1
   User root
   Compression yes
   StrictHostKeyChecking no
   UserKnownHostsFile /dev/null
   Port 2222

# connect to app01.backend.foobar.de which is only reachable by doing a ssh to jumper.foobar.de
# also really useful for scp
Host app01.backend.foobar.de
   Hostname app01.backend.foobar.de
   ProxyJump marc@jumper.foobar.de
   User root

# set userful default options for all hosts which match to *.aws.crap.de
Host *.aws.crap.de
   GSSAPIAuthentication no
   IdentityFile ~/.ssh/home/packer_id_rsa
   User ubuntu

# - set default options for all hosts
# - pass useful GIT envrionment variables to target systems
#   (see https://git-scm.com/book/gr/v2/Git-Internals-Environment-Variables
#   GIT_AUTHOR_NAME, GIT_AUTHOR_EMAIL, ...)
Host *
   Compression yes
   TCPKeepAlive yes
   User marc
   ForwardAgent yes
   ServerAliveInterval 15
   ForwardX11 yes
   SendEnv GIT_*
```

# Restrict access by authorized_keys

The "authorized_keys" files provides useful restrictions to limit the power of logins.
This is especially useful for automated processes - i.e. if jenkins publishes deployments.
```
~/.ssh/authorized_keys
# allow to open a specific ssh tunnel, set options
command="sleep 600",permitopen="192.168.2.15:5900",no-X11-forwarding,no-pty ssh-rsa AAAAB3...3lw== john@example.net
# allow only specific command, limit to client ip, set options
command="rsync...." from="192.168.2.31",no-port-forwarding,no-X11-forwarding,no-pty ssh-rsa AAAAB3...Vpw== script@standby
```
