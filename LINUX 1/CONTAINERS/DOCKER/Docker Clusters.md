
Ports:
- `2377/tcp` - cluster management
- `7946/tcp,udp` - communication between nodes
- `4789/udp` - for overlay network traffic

#### Cluster Creation and Joining

Initialize the first node in the cluster

`manager`
```bash
docker swarm init --advertise-addr "MANAGER IP"
```

A join token is generated. To recover the token

```bash
docker swarm join-token -q worker
```

Join a worker to the cluster

`worker`
```bash
docker swarm join \
> --token "TOKEN" \
> --advertise-addr "WORKER_IP" "MANAGER_IP":2377
```

`manager`
```
dockerd[984]: ... msg="worker ilkp...7p0rm was successfully registered" method="(*Dispatcher)
dockerd[984]: ... msg="Node 995d5f7a2c72/192.168.99.102, joined gossip cluster"
dockerd[984]: ...msg="Node 995d5f7a2c72/192.168.99.102, added to nodes list"
```

List nodes 

`manager`
```bash
docker node ls
```

> Number of managers must be kept odd. Min of three is required for fault tolerance. Should be spread across different hosts and even switches.

It is not uncommon to put the manager in `drain` state so as not to accept tasks
#### Services

Deploy a service

`manager`
```bash
docker service create [--mode global] --name hello --replicas 1 nginx
```

Global services run on every active node in the cluster.  If a drained node is brought back to active, the service starts immediately on that node.

List and inspect

```bash
docker service [OPTIONS]
```

- `ls`: List services
- `inspect --pretty "SERVICE"`: Service info
- `ps "SERVICE`: See which nodes a service is running on

Replicas 1 means the service is running only on the host. To distribute it among the worker nodes:

```bash
docker service scale "SERVICE"="NUMBER_OF_NODES"
```

```bash
docker service ps "SERVICE"
```

```
ID             NAME       IMAGE           NODE              CURRENT STATE
cb3uwlnk2ifh   pinger.1   alpine:latest   docker1.do1.lab   Running         
6uytefvjifbh   pinger.2   alpine:latest   docker2.do1.lab   Running         
wdqu7v7bi9rc   pinger.3   alpine:latest   docker3.do1.lab   Running         
```

##### Constraints

Constraints define which nodes a service can run on. Multiple constraints can be set on a service with the final result being the intersection of all of them.

To constraint a service to run on nodes with specific labels:

```bash
docker service create --constraint "node.labels.env == dev" --name ...
```

##### Stopping Services

Docker swarm does not have means to stop a service. It assumes that it is no longer needed, thus it can only be removed.

```bash
docker service rm "SERVICE"
```

To work around this, set the number of replicas to 0

```bash
docker service update --replicas 0 'SERVICE'
```
#### Node Maintenance

```bash
docker node update --availability "STATE" "NODE-ID"
```

States:
- `pause`: Prevents the scheduler from assigning new tasks to the node. Existing tasks continue to run.
- `drain`: Do not assign new tasks. Running containers are stopped and moved to other available nodes. Useful when upgrading a node.
- `active`: The node is ready to accept tasks.

> Tasks are only assigned when a new scheduling event, such as starting a service, happens, so reactivating a node might not immediately bring moved containers back.

If a node must be put into maintenance mode (drained) it transfers the containers to other nodes.

```bash
docker node update --availability drain "NODE HOSTNAME'
```

```
Availability:          Drain
```

Inspect a node

```bash
docker node inspect --pretty "NODE HOSTNAME"
```

Bring the node back to active state

```bash
docker node update --availability active "NODE"
```

> Draining a node moves tasks away, reactivating it doesn't automatically move them back
> Swarm only schedules tasks according to **desired state, placement and available resources**.

To redeploy tasks

```bash
docker service update --force "SERVICE"
```

Group nodes with labels

```bash
docker node update --availability --label-add 'KEY=VALUE' 'NODE'
```
##### Removing Nodes

First drain the node to move running containers elsewhere

Leave the swarm

```bash
docker swarm leave
```

> Leaving managers must first be demoted to workers. Quorum must be maintained!

Once the node status has been marked `Down`, it can be removed from the swarm

```bash
docker node rm [--force] "NODE"
```

#### Updating Services

A **rolling update** is when Swarm updates a service gradually, one task at a time (or batches), instead of stopping everything at once. This ensure 0 downtime - users keep being served while new constraints replace old ones.

```bash
docker service update OPTIONS
```

To update an image

```bash
docker service create --name web --replicas 3 nginx:1.24
```

```bash
docker service update --image nginx:1.25 web
```

Update options:

- `--update-parallelism 1`: Update 1 task at a time - default. `0` - all at once. SQL
- `--update-delay 10s`: Wait 10 seconds between each update

Control what happens if tasks are failing during the rolling update

- `--update-max-failure-ratio [RATIO]`: Controls failure ratio tolerance
	- number of failed tasks / total number of tasks updated.
		- `0.0`: Zero failure tolerance (default)
		- `0.5`: Up to half of the tasks can fail before rollback/pause
		- `1.0`: All can fail (not very useful)
- `--update-failure-action rollback`: Rollback to previous version if failed ratio is reached.

If updating breaks the service, it can rolled back

```bash
docker service rollback 'SERVICE'
```

If an image is not found on a node, it will be pulled from the registry, slowing down the process. Make sure to have the images everywhere prior to updating.
#### Disaster Recovery

##### Restarting the Cluster

Shutdown the workers first, then the managers. Start the managers first, then the workers. Managers must have static IPs.

##### Backup and Recovery

- swarm state data
- application data

##### Backing up the Swarm

Each manager keeps the cluster information in:

`/var/lib/docker/swarm/raft`

Back the `raft` directory.

##### Recovering a Swarm

1. Stop Docker
2. Copy the `raft` data back to `/var/lib/docker/swarm/raft`. 
3. Start Docker with:

```bash
docker swarm init --force-new-cluster --advertise-addr manager
```

##### Backing up Services

The `raft` database contains service info.

`Dockerfiles` and `docker compose` files should be in version control.