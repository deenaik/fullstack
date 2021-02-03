# Container
This document outlines useful commands and working with containers as well as debugging issues related to operating system.

Containers are just normal Linux Processes with additional configuration applied.

## Processes

### Run command
```sh
docker run -d --name=db redis:alpine
```

### See the process started by docker.
```sh
ps aux | grep redis-server
```

### Get PID (Process ID) and PPID (Parent Process ID)
```sh
docker top db
```

### Get the name of the Parent Process
```sh
ps aux | grep <ppid>
```

### list all of the sub processes of docker deamon.
```sh
pstree -c -p -A $(pgrep dockerd)
```

## Process directory

### Configuration of running container
```sh
DBPID=$(pgrep redis-server)
ls /proc/$DBPID
```

### view and update the environment variables defined to that process
* Linux specific command
```sh
cat /proc/$DBPID/environ
```
* Docker specific command
```sh
docker exec -it db env
```

## Namespaces

### The available namespaces are:
* Mount (mnt)
* Process ID (pid)
* Network (net)
* Interprocess Communication (ipc)
* UTS (hostnames)
* User ID (user)
* Control group (cgroup)

### Launching containerd process with unshare
```sh
sudo unshare --fork --pid --mount-proc
exit
```

### List all the namespaces
```sh
ls -lha /proc/$DBPID/ns/
```
### NSEnter is used to attach processes to existing Namespaces. 
Useful for debugging purposes.
```sh
nsenter --target $DBPID --mount --uts --ipc --net --pid ps aux
```

### Sharing namespace 
namespaces can be shared using the syntax container:<container-name>
```sh
docker run -d --name=web --net=container:db nginx:alpine
```

### verifing the sharing
```sh
ls -lha /proc/$WEBPID/ns/ | grep net
ls -lha /proc/$DBPID/ns/ | grep net
```

## Chroot
Chroot provides the ability for a process to start with a different root directory to the parent OS. This allows different files to appear in the root.

## Cgroups (Control Groups)
CGroups limit the amount of resources a process can consume.
```sh
cat /proc/$DBPID/cgroup
```
These are mapped to other cgroup directories on disk at:
```sh
ls /sys/fs/cgroup/
```

### CPU stats for a process
```sh
cat /sys/fs/cgroup/cpu,cpuacct/docker/$DBID/cpuacct.stat
cat /sys/fs/cgroup/cpu,cpuacct/docker/$WEBID/cpuacct.stat
```

### CPU shares limit
```sh
cat /sys/fs/cgroup/cpu,cpuacct/docker/$DBID/cpu.shares
cat /sys/fs/cgroup/cpu,cpuacct/docker/$WEBID/cpu.shares
```

### Docker cgroups for the container's memory configuration
```sh
ls /sys/fs/cgroup/memory/docker/
```
Each of the directory is grouped based on the container ID assigned by Docker.
```sh
DBID=$(docker ps --no-trunc | grep 'db' | awk '{print $1}')
WEBID=$(docker ps --no-trunc | grep 'nginx' | awk '{print $1}')
ls /sys/fs/cgroup/memory/docker/$DBID
ls /sys/fs/cgroup/memory/docker/$WEBID
```

### view Container limits via docker stats command.
```sh
docker stats db --no-stream
```

### change the memory limits of a process
The memory quotes are stored in a file called memory.limit_in_bytes
```sh
echo 8000000 > /sys/fs/cgroup/memory/docker/$DBID/memory.limit_in_bytes
docker stats db --no-stream
```

## Seccomp / AppArmor

### view the current AppArmor profile assigned to a process
```sh
cat /proc/$DBPID/attr/current
```

### The status of SecComp is also defined within a file.
```sh
cat /proc/$DBPID/status | grep Seccomp
```
The flag meaning are: 0: disabled 1: strict 2: filtering

## Capabilities

### to see status file containers the Capabilities flag
```sh
cat /proc/$DBPID/status | grep ^Cap
```

### The flags are stored as a bitmask that can be decoded with
```sh
capsh --decode=00000000a80425fb
```
