# Docker Deep-dive

## Namespace Demonstration

### Experiment with process namespace
- To create process in a new namespace run `sudo unshare --fork --pid --mount-proc /bin/sh`.
- In the new namespcae run `ps aux` to see the running process. 
- To see the pid namespace check of a process `sudo ls -l /proc/{pid}/ns`.
- To see the other pid namespace currently exists run `sudo lsns -t pid`, to check the number of process running in a particular pid namespace follow the 3rd column of `sudo lsns -t pid`.
- To enter to existing namespace run `sudo nsenter -t {pid} --pid  --mount --net /bin/sh`.

### Experiment with network namespace

To create process in a new network namespace run `sudo unshare --fork --net /bin/bash`. (We wont use this as that it creates anonymous NS)

- `sudo ip netns list`
- `sudo ip netns add ns1`
- `sudo ip netns exec ns1 ip addr`
- `sudo ip netns exec ns1 ping 127.0.0.1`
- `sudo ip netns exec ns1 ip link set dev lo up`
- `sudo ip netns exec ns1 ping 127.0.0.1`
- `sudo ip netns exec ns1 ip addr`
- `sudo ip netns exec ns1 ip route show` - To see the routing table
- `sudo ip link add v-eth type veth peer name v-ethc` - add virtual ethernet
- `sudo ip link set v-ethc netns ns1` - Move one end of veth to new NS
- `sudo ip addr add 10.200.1.1/24 dev v-eth` - add ip
- `sudo ip link set v-eth up` - Up interface
- `sudo ip netns exec ns1 ip addr add 10.200.1.2/24 dev v-ethc`
- `sudo ip netns exec ns1 ip link set v-ethc up`               
- `sudo ip netns exec ns1 ip link set lo up`
- `sudo ip netns exec ns1 ip route add default via 10.200.1.1`
- `sudo iptables -t nat -A POSTROUTING -s 10.200.1.0/255.255.255.0 -o eno1 -j MASQUERADE`
- `sudo iptables -A FORWARD -i eno1 -o v-eth -j ACCEPT`
- `sudo iptables -A FORWARD -o eno1 -i v-eth -j ACCEPT`

- `sudo iptables -t nat -D POSTROUTING -s 10.200.1.0/255.255.255.0 -o eno1 -j MASQUERADE`
- `sudo iptables -D FORWARD -i eno1 -o v-eth -j ACCEPT`
- `sudo iptables -D FORWARD -i v-eth -o eno1 -j ACCEPT`
- `sudo ip netns delete ns1`


### Experiment with mount namespace
- Run `sudo unshare --fork --pid --mount-proc /bin/sh`.
- From the container run `ls /proc` and from the host run `ls /proc` and observe the difference.
- Run `findmnt` to find the mounts currently available.
- To create a new mount `mount --bind /usr/bin/ /mnt`

## Cgroup Demonstration

### Experiment with cgroups
- `sudo cgcreate -a avik -g memory:containergroup`
- `ls -l /sys/fs/cgroup/memory/containergroup/`
- `sudo echo 10000000 >  /sys/fs/cgroup/memory/containergroup/memory.kmem.limit_in_bytes`
- `sudo cgexec  -g memory:containergroup bash`
- `sudo cgdelete memory:containergroup`

cgcreate, cgexec, cgdelete, cgclassify

### Experiment with seccomp




## Demonstrating Docker Architecture
- Watch - `watch 'ps fxa | grep "docker\|containerd\|runc" -A 3'`
- run - `docker run -it alpine /bin/sh`
- Observe the hierarchical structure of the processes and understand the process id map. Ensuring the isolation.
- Observe all the processes running under namespace `sudo lsns -t pid`, see total process count changes if we - run `sleep 60` inside the container.
- Observe output of below commands to see docker creates NS around pid,network,ipc,mnt etc.
```
Cgroup      `sudo lsns -t cgroup`   Cgroup root directory                 
IPC         `sudo lsns -t ipc`      System V IPC, POSIX message queues
Network     `sudo lsns -t net`      Network devices, stacks, ports, etc.
Mount       `sudo lsns -t mnt`      Mount points
PID         `sudo lsns -t pid`      Process IDs
User        `sudo lsns -t user`     User and group IDs
UTS         `sudo lsns -t uts`      Hostname and NIS domain name
```
Observe cgroup and user namespace not listed here and may be handled in a different way.


### Wrap up




### Other important command
`docker export $(docker create busybox) | tar -C rootfs -xvf -`



