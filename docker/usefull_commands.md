# Container
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

W-I-P
