
# Creating a Docker Swarm Cluster

Docker container technology was launched in 2013 as an open-source Docker Engine. It leveraged existing computing concepts around containers and specifically in the Linux world, primitives known as cgroups and namespaces. Docker’s technology is unique because it focuses on the requirements of developers and systems operators to separate application dependencies from infrastructure.

Docker swarm is a considerable addition to Docker. It’s designed to easily manage container scheduling over multiple hosts.

* Main point: It allows to connect multiple hosts with Docker together.
* It’s relatively simple. Compared with others solutions, starting with Docker Swarm is really easy.
* High availability — there are two node types in the cluster: master and worker. One of the masters is the leader. If the current leader fails, another master will become the leader. If the worker host fails, all containers will be rescheduled to other nodes.
* Declarative configuration. You tell what you want, how many replicas, and they’ll be automatically scheduled with respect to given constraints.
* Rolling updates — Swarm stores configuration for containers. If you update configuration, containers are updated in batches, so service by default will be available all the time.
* Build-in Service discovery and load balancing — similar to load balancing done by Docker-Compose. You can reference other services using their names, it doesn’t matter where containers are stored, they will receive requests in a round-robin fashion.
* Overlay network — if you expose a port from a service, it’ll be available on any node in the cluster. It really helps with external load balancing.

## What will we see in this article?

We will form a cluster of three machines with Docker Swarm

## Pre-conditions

* Machines with **Linux** installed. In the case of this example. I used the Ubuntu 20.04 distribution you can use what you want if the Docker is supported in your **Linux Distribution**.
* Machines are seeing between them and can send and received packets.

## What you should know before starting to setup a swarm?

* In your cluster, you can have one, two, three, or several servers.
* Each machine is called a node in your cluster.
* To take advantage of swarm mode’s fault-tolerance features, Docker recommends you implement an odd number of nodes according to your organization's high-availability requirements
* To set up a swarm, you need to have managers nodes/machines and workers nodes/machines.

The manager nodes handle cluster management tasks:

* Maintaining cluster state
* Scheduling services
* Serving swarm mode HTTP API endpoints

The Worker nodes don't participate in the Raft distributed state, make scheduling decisions, or serve the swarm mode HTTP API:

* An N manager cluster tolerates the loss of at most (N-1)/2 managers. Example: A three-manager swarm tolerates a maximum loss of one manager.
* You can create a swarm of one manager node, but you cannot have a worker node without at least one manager node. By default, all managers are also workers. In a single manager node cluster, you can run commands as docker service create, and the scheduler places all tasks on the local Engine.

## Installing the Docker

If you do not have the Docker installed yet please follow the next steps to get it ready.

Make sure that we don't have any old package that could raise any issue

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

Let's update the repositories

```bash
sudo apt-get update
```

Now we need to install the dependencies

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y
```

As we will install the packages from the Docker repository we need to import the GPG key

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Now we can check if the key was imported

```bash
sudo apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

We need to import the Docker repository

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Update the repositories

```bash
sudo apt-get update
```

Let's install the Docker packages

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

Now we need to start the Docker

```bash
sudo systemctl start docker
```

Let's enable it to start with the system

```bash
sudo systemctl enable docker
```

Now we need to add the common user into the Docker group to be able to handle the Docker Service

```bash
sudo usermod -aG docker douglas
```

Now exit from the shell

```bash
exit
```

Log again so you can run the docker ps as show

```bash
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

I will be using the VIM editor so let's install it

```bash
sudo apt-get install -y vim 
```

Now you can use a custom configuration that I like to feel free to use or not, I configured it to handle the YAML and the F7 key can remove the whitespaces at the end of the lines what sometimes can raise some issues.

```bash
curl -L https://raw.githubusercontent.com/douglasqsantos/DevOps/master/Misc/prep-vim.sh | bash
```

The default theme is Monokai but you can use the Gruvbox that is a great one as well.

## Form a cluster of machines

For this example, I am using 3 machines.

See below my machines roles and IP (All the three machines will be managers):

* My manager one has the IP 10.0.0.31 with the hostname node01
* My manager two has the IP 10.0.0.32 with the hostname node02
* My manager three has the IP 10.0.0.33 with the hostname node03

Make sure the following files are equal on each server.

```bash
vim /etc/hosts
[...]
# Please add this line in each node
10.0.0.31 node01
10.0.0.32 node02
10.0.0.33 node03
```

Now Make sure that you have access from each other, we can test this with ICMP

Let's check the node02 from node01

```bash
ping -c 2 node02
PING node02 (10.0.0.32) 56(84) bytes of data.
64 bytes from node02 (10.0.0.32): icmp_seq=1 ttl=64 time=1.98 ms
64 bytes from node02 (10.0.0.32): icmp_seq=2 ttl=64 time=0.950 ms

--- node02 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.950/1.463/1.976/0.513 ms
```

Let's check the node03 from node01

```bash
ping -c 2 node03
PING node03 (10.0.0.33) 56(84) bytes of data.
64 bytes from node03 (10.0.0.33): icmp_seq=1 ttl=64 time=1.64 ms
64 bytes from node03 (10.0.0.33): icmp_seq=2 ttl=64 time=0.572 ms

--- node03 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.572/1.108/1.644/0.536 ms
```

Another thing that you can do as well is to configure the ssh-key to communicate between the nodes.

Let's create the key on the node01

```bash
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/douglas/.ssh/id_rsa): # ENTER
Enter passphrase (empty for no passphrase): # ENTER
Enter same passphrase again:
Your identification has been saved in /home/douglas/.ssh/id_rsa
Your public key has been saved in /home/douglas/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:wuH7qWZ/aJf87U3waVWjnCxonbbK45J4KF5o/Amat3w douglas@node02
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|      .        ..|
|     o .  o + o o|
|      + So = =. .|
|  . .  o. . o  oo|
|   = .+ .o o   oo|
|  =o+E+=+o*  ..o |
| oo++=o+**....o .|
+----[SHA256]-----+
```

Now let's copy the key to the other servers

```bash
ssh-copy-id douglas@node02
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/douglas/.ssh/id_rsa.pub"
The authenticity of host 'node02 (10.0.0.32)' can't be established.
ECDSA key fingerprint is SHA256:QNCeK5fUjcs6cueotnic8FQoqTNAlXw3vIRdw6s5OAU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
douglas@node02's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'douglas@node02'"
and check to make sure that only the key(s) you wanted were added.
```

Do the same process for each server, or use your own private key to access or the password does in your way.

Do the same process on the other nodes to make sure that each server can see each other.

***Possible issues:*** If packets are not received, please check your firewall settings.

## Initialize docker swarm

Initialize docker swarm and let’s form the cluster

on the node01 machine (10.0.0.31) to initialize our swarm

```bash
docker swarm init --advertise-addr 10.0.0.31
Swarm initialized: current node (w0vmnb799i80g1h9rb8fi9x81) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5j27cut3rf06dos879sg49pw4t07l4qtkw6ha0b3nxplokly0o-7xv15vwt6deykkqgp686zdqwi 10.0.0.31:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

After initializing our swarm, always on node01 machine, run this command:

```bash
docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5j27cut3rf06dos879sg49pw4t07l4qtkw6ha0b3nxplokly0o-5bx9m090z6ekdqun1e2lngz14 10.0.0.31:2377
```

Please copy the output command from this confirmation text. The command will be used to add our other servers to the swarm.

On the node02 machine (10.0.0.32) to add it to our swarm

To add the node02 machine to the swarm as manager, follow the instructions after you run this command:

```bash
docker swarm join --token SWMTKN-1-5j27cut3rf06dos879sg49pw4t07l4qtkw6ha0b3nxplokly0o-5bx9m090z6ekdqun1e2lngz14 10.0.0.31:2377
This node joined a swarm as a manager.
```

On the node03 machine (10.0.0.33) to add it to our swarm

To add the node03 machine to the swarm as manager, follow the instructions after you run this command:

```bash
docker swarm join --token SWMTKN-1-5j27cut3rf06dos879sg49pw4t07l4qtkw6ha0b3nxplokly0o-5bx9m090z6ekdqun1e2lngz14 10.0.0.31:2377
This node joined a swarm as a manager.
```

Check nodes status

```bash
docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
w0vmnb799i80g1h9rb8fi9x81 *   node01              Ready               Active              Leader              19.03.12
t8aanmv2pcqomr5es19dcrsfc     node02              Ready               Active              Reachable           19.03.12
x7vt9qok5xwelt3h22rqr9blk     node03              Ready               Active              Reachable           19.03.12
```

We could have added workers instead of managers after the initialization of the swarm. To do this, you must copy from the confirmation text of the swarm initialization the command indicating how to add workers. In my case, the confirmation text says:

```bash
To add a worker to this swarm, run the following command:
docker swarm join --token SWMTKN-1-5j27cut3rf06dos879sg49pw4t07l4qtkw6ha0b3nxplokly0o-7xv15vwt6deykkqgp686zdqwi 10.0.0.31:2377
```

## References

* [Docker Swarm Init](https://docs.docker.com/engine/reference/commandline/swarm_init/)
* [Docker Service](https://docs.docker.com/engine/reference/commandline/service/)
* [Docker Stack](https://docs.docker.com/engine/reference/commandline/stack/)
* [Overview of docker-compose CLI](https://docs.docker.com/compose/reference/overview/)
* [Docker run reference](https://docs.docker.com/engine/reference/run/)