# Setup Indexer Clustering

References:
- [Cluster Deployment Overview](https://docs.splunk.com/Documentation/Splunk/8.0.2/Indexer/Clusterdeploymentoverview)
- [About Indexers and Clusters of Indexers](https://docs.splunk.com/Documentation/Splunk/8.0.2/Indexer/Aboutclusters)
- [Search Factor](https://docs.splunk.com/Documentation/Splunk/8.0.2/Indexer/Thesearchfactor)
- [Replication Factor](https://docs.splunk.com/Documentation/Splunk/8.0.2/Indexer/Thereplicationfactor)


Steps:
1. [Enable Cluster Master](https://docs.splunk.com/Documentation/Splunk/8.0.2/Indexer/Enablethemasternode)
2. [Enable Peer Nodes](https://docs.splunk.com/Documentation/Splunk/8.0.2/Indexer/Enablethepeernodes)
3. Verify Status


## Enable Cluster Master
You can setup the cluster master by changing a configuration file or by using the CLI.
These changes should be applied to the `splunk-mgt` instance from the reference architecture.

Important:
- Remember the `pass4SymmKey` passphrase as you need to set it up on the indexers as well.
- Defining proper values for RF/SF is depending on availability requirements & available infrastructure/budget.

Configure the cluster master using the following `/opt/splunk/etc/system/local/server.conf` settings:
```
[clustering]
mode = master
replication_factor = 2
search_factor = 1
pass4SymmKey = whatever
cluster_label = cluster1
```

Alternatively you can use the CLI to setup the cluster master:
`splunk edit cluster-config -mode master -replication_factor 2 -search_factor 1 -secret whatever -cluster_label cluster1`

After applying the settings you need to restart Splunk: 
`systemctl restart splunkd` or `/etc/init.d/splunk restart`	 


## Enable Peer Nodes
All the indexer instances (`splunk-idxN`) from the reference architecture need to be configured to pair with with cluster master.
This task can be accomplished by editing a configuration file or by using CLI commands.

Configure indexer by changing the `/opt/splunk/etc/system/local/server.conf` configuration file:
```
[replication_port://9100]

[clustering]
master_uri = https://splunk-mgt:8089
mode = slave
pass4SymmKey = whatever
cluster_label = cluster1
```

Alternatively you can use the CLI to setup the peer nodes (indexers):
`splunk edit cluster-config -mode slave -master_uri https://splunk-mgt:8089 -replication_port 9100 -secret whatever`

After applying the settings you need to restart Splunk:
`systemctl restart splunkd` or `/etc/init.d/splunk restart`


## Verify Status
Some useful commands to verify the health and/or configuration of the indexer cluster:
- splunk cluster commands: `splunk help cluster`
- cluster health, run from the cluster master: `splunk show cluster-status`
- list cluster peers, run from the cluster master: `splunk list cluster-peers`
- cluster configuration: `splunk list cluster-config`
