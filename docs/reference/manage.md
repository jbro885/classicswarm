---
description: Create a Swarm manager.
keywords: swarm, create, manage
---

# manage  — Create a Swarm manager

Prerequisite: Before using `manage` to create a Swarm manager, establish a discovery backend as described in [this discovery topic](../discovery.md).

The `manage` command creates a Swarm manager whose purpose is to receive commands on behalf of the cluster and assign containers to Swarm nodes. You can create multiple Swarm managers as part of a high-availability cluster.

To create a Swarm manager, use the following syntax:

    $ docker run swarm manage [OPTIONS] <discovery>

For example, you can use `manage` to create a Swarm manager in a high-availability cluster with other managers:

    $ docker run -d -p 4000:4000 swarm manage -H :4000 --replication --advertise 172.30.0.161:4000 consul://172.30.0.165:8500

Or, for example, you can use it to create a Swarm manager that uses Transport Layer Security (TLS) to authenticate the Docker Client and Swarm nodes:

    $ docker run -d -p 3376:3376 -v /home/ubuntu/.certs:/certs:ro swarm manage --tlsverify --tlscacert=/certs/ca.pem --tlscert=/certs/cert.pem --tlskey=/certs/key.pem --host=0.0.0.0:3376 token://$TOKEN


## Argument

The `manage` command has only one argument:

### `<discovery>` — Discovery backend

Before you create a Swarm manager, [create a discovery token](create.md) or [set up a discovery backend](../discovery.md) for your cluster.

When you create the swarm node, use the `<discovery>` argument to specify one of the following discovery backends:

* `token://<token>`
* `consul://<ip1>/<path>`
* `etcd://<ip1>,<ip2>,<ip3>/<path>`
* `file://<path/to/file>`
* `zk://<ip1>,<ip2>/<path>`
* `[nodes://]<iprange>,<iprange>`

Where:

* `<token>` is a discovery token generated by Docker Hub's hosted discovery service. To generate this discovery token, use the [`create`](create.md) command.
    > Warning: Docker Hub's hosted discovery backend is not recommended for production use. It’s intended only for testing/development.

* `ip1`, `ip2`, `ip3` are each the IP address and port numbers of a discovery backend node.
* `path` (optional) is a path to a key-value store on the discovery backend. When you use a single backend to service multiple clusters, you use paths to maintain separate key-value stores for each cluster.
* `path/to/file` is the path to a file that contains a static list of the Swarm managers and nodes that are members the cluster. <!--tbd - can the file contain ipranges?-->
* `iprange` is an IP address or a range of IP addresses followed by a port number.

Here are a pair of `<discovery>` argument examples:

* A discovery token: `token://0ac50ef75c9739f5bfeeaf00503d4e6e`
* A Consul node: `consul://172.30.0.165:8500`

The environment variable for `<discovery>` is `$SWARM_DISCOVERY`.

For more information and examples, see the [Docker Swarm Discovery](../discovery.md) topic.

## Options

The `manage` command has the following options:

### `--strategy` — Scheduler placement strategy

Use `--strategy "<value>"` to tell the Docker Swarm scheduler which placement strategy to use.

Where `<value>` is:

  * `spread` — Assign each container to the Swarm node with the most available resources.
  * `binpack` - Assign containers to one Swarm node until it is full before assigning them to another one.
  * `random` - Assign each container to a random Swarm node.

By default, the scheduler applies the `spread` strategy.

For more information and examples, see [Docker Swarm strategies](../scheduler/strategy.md).

### `--filter`, `-f` — Scheduler filter

Use `--filter <value>` or `-f <value>` to tell the Docker Swarm scheduler which nodes to use when creating and running a container.

Where `<value>` is:

  * `health` — Use nodes that are running and communicating with the discovery backend.
  * `port` — For containers that have a static port mapping, use nodes whose corresponding port number is available and not occupied by another container or process.
  * `dependency` — For containers that have a declared dependency, use nodes that already have a container with the same dependency.
  * `affinity` — For containers that have a declared affinity, use nodes that already have a container with the same affinity.
  * `constraint` — For containers that have a declared constraint, use nodes that already have a container with the same constraint.

You can use multiple scheduler filters, like this:

`--filter <value> --filter <value>`

For more information and examples, see [Swarm filters](../scheduler/filter.md).

### `--host`, `-H` — Listen to IP/port

Use `--host <ip>:<port>` or `-H <ip>:<port>` to specify the IP address and port number to which the manager listens for incoming messages. If you replace `<ip>` with zeros or omit it altogether, the manager uses the default host IP. For example, `--host=0.0.0.0:3376` or `-H :4000`.

The environment variable for `--host` is `$SWARM_HOST`.
<!--tbd - Provide more information and examples of using environment variables.-->
<!--tbd - What happens if you omit the port number?-->

### `--replication` — Enable Swarm manager replication

Enable Swarm manager replication between the *primary* and *secondary* managers in a high-availability cluster. Replication mirrors cluster information from the primary to the secondary managers so that, if the primary manager fails, a secondary can become the primary manager.

### `--replication-ttl` — Leader lock release time on failure

Use `--replication-ttl "<delay>s"` to specify the delay, in seconds, before notifying secondary managers that the primary manager is down or unreachable. This notification triggers an election in which one of the secondary managers becomes the primary manager. By default, the delay is 15 seconds.

### `--advertise`, `--addr` — Advertise Docker Engine's IP and port number

Use `--advertise <ip>:<port>` or `--addr <ip>:<port>` to advertise the IP address and port number of the Docker Engine. For example, `--advertise 172.30.0.161:4000`. Other Swarm managers MUST be able to reach this Swarm manager at this address.

The environment variable for `--advertise` is `$SWARM_ADVERTISE`.

### `--tls` — Enable transport layer security (TLS)

Use `--tls` to enable transport layer security (TLS). If you use `--tlsverify`, you do not need to use `--tls`.

### `--tlscacert` — Path to a CA's public key file

Use `--tlscacert=<path/file>` to specify the path and filename of the public key (certificate) from a Certificate Authority (CA). For example, `--tlscacert=/certs/ca.pem`. When specified, the manager trusts only remotes that provide a certificate signed by the same CA.

### `--tlscert` — Path to the node's TLS certificate file

Use `--tlscert` to specify the path and filename of the manager's certificate (signed by the CA). For example, `--tlscert=/certs/cert.pem`.

### `--tlskey` — Path to the node's TLS key file

Use `--tlskey` to specify the path and filename of the manager's private key (signed by the CA). For example, `--tlskey=/certs/key.pem`.

### `--tlsverify` — Use TLS and verify the remote

Use `--tlsverify` to enable transport layer security (TLS) and accept connections from only those managers, nodes, and clients that have a certificate signed by the same CA. If you use `--tlsverify`, you do not need to use `--tls`.

### `--engine-refresh-min-interval` — Set engine refresh minimum interval

Use `--engine-refresh-min-interval "<interval>s"` to specify the minimum interval, in seconds, between Engine refreshes. By default, the interval is 30 seconds.

> When the primary manager in performs an *Engine refresh*, it gets updated information about an Engine in the cluster. The manager uses this information to, among other things, determine whether the Engine is *healthy*. If there is a connection failure, the manager determines that the node is *unhealthy*. The manager *retries* an Engine refresh a specified number of times. If the Engine responds to one of the retries, the manager determines that the Engine is healthy again. Otherwise, the manager stops retrying and ignores the Engine. <!--TBD - Use this note as the basis for a new topic called something like "Node Lifecycle". Then replace this note with a link to that topic.-->

### `--engine-refresh-max-interval` — Set engine refresh maximum interval

Use `--engine-refresh-max-interval "<interval>s"` to specify the minimum interval, in seconds, between Engine refresh. By default, the interval is 60 seconds.

### `--engine-failure-retry` — Set engine failure retry count

Use `--engine-failure-retry "<number>"` to specify the number of retries to attempt if the engine fails. By default, the number is 3 retries.

### `--engine-refresh-retry` — Deprecated

Deprecated; Use `--engine-failure-retry` instead of `--engine-refresh-retry "<number>"`. The default number is 3 retries.

### `--heartbeat` — Period between each heartbeat

Use `--heartbeat "<interval>s"` to specify the interval, in seconds, between heartbeats the manager sends to the primary manager. These heartbeats indicate that the manager is healthy and reachable. By default, the interval is 60 seconds.

### `--api-enable-cors`, `--cors` — Enable CORS headers in the Engine API

Use `--api-enable-cors` or `--cors` to enable cross-origin resource sharing (CORS) headers in the Engine API.

### `--cluster-driver`, `-c` — Cluster driver to use

Use `--cluster-driver "<driver>"`, `-c "<driver>"` to specify a cluster driver to use. Where `<driver>` is one of the following:

* `swarm` is the Docker Swarm driver.
* `mesos-experimental` is the Mesos cluster driver.

By default, the driver is `swarm`.

For more information about using Mesos driver, see [Using Docker Swarm and Mesos](https://github.com/docker/swarm/blob/master/cluster/mesos/README.md).

### `--cluster-opt` — Cluster driver options

You can enter multiple cluster driver options, like this:

`--cluster-opt <value> --cluster-opt <value>`

Where `<value>` is one of the following:

  * `swarm.overcommit=0.05` — Set the fractional percentage by which to overcommit resources. The default value is `0.05`, or 5 percent.
  * `swarm.createretry=0` — Specify the number of retries to attempt when creating a container fails.  The default value is `0` retries.
  * `mesos.address=` — Specify the Mesos address to bind on. The environment variable for this option is  `$SWARM_MESOS_ADDRESS`.
  * `mesos.checkpointfailover=false` — Enable Mesos checkpointing, which allows a restarted slave to reconnect with old executors and recover status updates, at the cost of disk I/O. The environment variable for this option is `$SWARM_MESOS_CHECKPOINT_FAILOVER`.  The default value is `false` (disabled).
  * `mesos.port=` — Specify the Mesos port to bind on. The environment variable for this option is `$SWARM_MESOS_PORT`.
  * `mesos.offertimeout=30s` — Specify the Mesos timeout for offers, in seconds. The environment variable for this option is `$SWARM_MESOS_OFFER_TIMEOUT`. The default value is `30s`.
  * `mesos.offerrefusetimeout=5s` — Specify timeout for Mesos to consider unused resources refused, in seconds. The environment variable for this option is `$SWARM_MESOS_OFFER_REFUSE_TIMEOUT`. The default value is `5s`.
  * `mesos.tasktimeout=5s` — Specify the timeout for Mesos task creation, in seconds. The environment variable for this option is `$SWARM_MESOS_TASK_TIMEOUT`. The default value is `5s`.
  * `mesos.user=` — Specify the Mesos framework user name. The environment variable for this option is `$SWARM_MESOS_USER`.

### `--discovery-opt` — Discovery options

Use `--discovery-opt <value>` to discovery options, such as paths to the TLS files; the CA's public key certificate, the certificate, and the private key of the distributed K/V store on a Consul or etcd discovery backend. You can enter multiple discovery options. For example:

    --discovery-opt kv.cacertfile=/path/to/mycacert.pem \
    --discovery-opt kv.certfile=/path/to/mycert.pem \
    --discovery-opt kv.keyfile=/path/to/mykey.pem \

For more information, see [Use TLS with distributed key/value discovery](../discovery.md)
