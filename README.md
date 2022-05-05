# Deploying Solana Validator Clusters

This orchestration set, which uses [Directord](directord.com) will setup and
deploy a Solana cluster. The orchestration files provide for both binary and
source based builds.

> The base OS used for example purposes is CentOS 9, however, there's nothing
  in the core orchestrations that limit the deployment options to only CentOS.

### Why might this be useful?

Directord was built to help operators meet SLA/SLO requirements at
[massive scale](https://directord.com/analysis.html). A lot of tools out there
aim to do this very thing, however, Directord achieves it's goals by remaining
hyper simple while leveraging lower level protocols. This means, you, as an
operator of a Solana clsuter, can build any scale environments and not have
worry about the tool chain becoming the blocker. The driving montra of
Directord is to get in, get done, and get out of the way.

### Typical Deployment Process

1. Bootstrap the environment.

Install Directord on your server node. While the server node can exist within
your application cluster it is advisable to place the server outside the core
application.

Using the `catalog.yaml` file as an example, fill in the details for your
cluster. Most of what you, the deployer, needs to define is contained within
the `host` entries.

Once your `catalog.yaml` contains all of your required information, run the
bootstrap operation.

``` shell
$ directord bootstrap --catalog catalog.yaml
```

2. Validate your cluster.

With the Directord bootstrap complete, validate your cluster is operational.

``` shell
$ sudo /opt/directord/bin/directord manage --list-nodes
```

The output should provide information on all reporting nodes within the
cluster.

3. Deploy your Solana Validator Cluster (Binary)

``` shell
$ sudo /opt/directord/bin/directord orchestrate solana-binary-base-c9.yaml \
                                                solana-binary-install.yaml \
                                                solana-key-creation.yaml \
                                                solana-key-distribution.yaml \
                                                solana-vote-account.yaml \
                                                solana-validator-deploy.yaml
```

If you already have keys and/or a vote account, you can omit the
`solana-key-creation.yaml`, `solana-key-distribution.yaml` and
`solana-vote-account.yaml` files.

> NOTE: This deployment method assumes that the key files are located at
  `/var/lib/solana`. If you already have keys, and do not need to directord
  to generate them (as would be the case in production), it is
  recommended that you mount the key files in place using some form of
  shared storage, or seed the key files into your operating system through
  other means.

3. Deploy your Solana Validator Cluster (Source)

Source based builds provide some flexibility that binary deployments don't. If
you need a source based build, simply include the source files into your
orchestration instead of the binary ones.

``` shell
$ sudo /opt/directord/bin/directord orchestrate solana-source-base-c9.yaml \
                                                solana-source-install.yaml \
                                                solana-key-creation.yaml \
                                                solana-key-distribution.yaml \
                                                solana-vote-account.yaml \
                                                solana-validator-deploy.yaml
```

4. Understanding the deployment.

Once the cluster is operational, you can login to the nodes and poke around.
You'll see that the `solana-validator` and `solana-sys-tuner` are both
running within the `solana` slice, from the `solana` user, with the group
`solana-application` using systemd. Check the solana log files for more
information as well as the systemd journal to ensure everything is running
to your liking.

5. Post deployment operations.

With the Directord server, you can monitor the application health and perform
cluster wide updates and upgrades in pseudo realtime. While Directord can double
as an excellent uptime and monitoring tool, it can also serve in an ephemeral
use-case. If Directord is not something you wish to be persistent, you can
freely shutdown the `directord-client` and `directord-server` at any time.

To reestablish the Directord topology, simply follow the bootstrap process
from step one to re-enable directord.
