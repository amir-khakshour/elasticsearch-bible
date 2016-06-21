### Shards and replicas in Elasticsearch

When you download elasticsearch and start it up you create an elasticsearch node which tries to join an existing cluster if available or creates a new one. Let's say you created your own new cluster with a single node, the one that you just started up. We have no data, therefore we need to create an index.

When you create an index (an index is automatically created when you index the first document as well) you can define how many shards it will be composed of. If you don't specify a number it will have the default number of shards: 5 primaries. What does it mean? 

It means that elasticsearch will create 5 primary shards that will contain your data:

     ____    ____    ____    ____    ____
    | 1  |  | 2  |  | 3  |  | 4  |  | 5  |
    |____|  |____|  |____|  |____|  |____|


Every time you index a document elasticsearch will decide which primary shard is supposed to hold that document and will index it there. Primary shards are not copy of the data, they are the data! Having multiple shards does help taking advantage of parallel processing on a single machine, but the whole point is that if we start another elasticsearch instance on the same cluster, the shards will be distributed in an even way over the cluster.

Node 1 will then hold for example only three shards:

     ____    ____    ____ 
    | 1  |  | 2  |  | 3  |
    |____|  |____|  |____|


Since the remaining two shards have been moved to the newly started node:

     ____    ____
    | 4  |  | 5  |
    |____|  |____|


Why does this happen? Because elasticsearch is a distributed search engine and this way you can make use of multiple nodes/machines to manage big amounts of data.

Every elasticsearch index is composed of at least one primary shard, since that's where the data is stored. Every shard comes at a cost though, therefore if you have a single node and no foreseeable growth, just stick with a single primary shard.

Another type of shard is replica. The default is 1, meaning that every primary shard will be copied to another shard that will contain the same data. Replicas are used to increase search performance and for fail-over. A replica shard is never going to be allocated on the same node where the related primary is (it would pretty much be like putting a backup on the same disk as the original data).


Back to our example, with 1 replica we'll have the whole index on each node, since 3 replica shards will be allocated on the first node and they will contain exactly the same data as the primaries on the second node:

```
     ____    ____    ____    ____    ____
    | 1  |  | 2  |  | 3  |  | 4R |  | 5R |
    |____|  |____|  |____|  |____|  |____|
```
Same for the second node, which will contain a copy of the primary shards on the first node:
```
     ____    ____    ____    ____    ____
    | 1R |  | 2R |  | 3R |  | 4  |  | 5  |
    |____|  |____|  |____|  |____|  |____|
```

With a setup like this, if a node goes down you still have the whole index. The replica shards will automatically become primaries and the cluster will work properly despite the node failure, as follows:

```
     ____    ____    ____    ____    ____
    | 1  |  | 2  |  | 3  |  | 4  |  | 5  |
    |____|  |____|  |____|  |____|  |____|
```

Since you have `"number_of_replicas":1`, the replicas cannot be assigned anymore as they are never allocated on the same node where their primary is. That's why you'll have 5 unassigned shards, the replicas, and the cluster status will be `YELLOW` instead of `GREEN`. No data loss, but it could be better as some shards cannot be assigned.

As soon as the node that had left is back up, it'll join the cluster again and the replicas will be assigned again. The existing shard on the second node can be loaded but they need to be synchronized with the other shards, as write operations most likely happened while the node was down. At the end of this operation the cluster status will become `GREEN`.

### How many shards and replicas do i need?

How many shards and replicas you use really depends on your data, the way you access them and the number of available nodes/servers. It's best practice to overallocate shards a little in order to redistribute them in case you add more nodes to your cluster, **since you can't (for now) change the number of shards once you created the index**. Otherwise you can always change the number of shards if you are willing to do a complete reindex of your data.

Every additional shard comes with a cost since each shard is effectively a Lucene instance. The maximum number of shards that you can have per machine really depends on the hardware available and your data as well. Good to know that having 100 indexes with each one shard or one index with 100 shards is really the same since you'd have 100 lucene instances in both cases.

Of course at query time if you want to query a single elasticsearch index composed of 100 shards elasticsearch would need to query them all in order to get proper results (unless you used a specific routing for your documents to then query only a specific shard). This would have a performance cost.

You can easily check the state of your cluster and nodes using the [Cluster Nodes Info API][2] through which you can check a lot of useful information, all you need in order to know whether your nodes are running smoothly or not. Even easier, there are a couple of plugins to check those information through a nice user interface (which internally uses the elasticsearch APIs anyway): [paramedic][3] and [bigdesk][4].

###### summary
- **Node**: an Elasticsearch instance running (a java process). Usually every node runs on its own machine.
- **Cluster**: one or more nodes with the same cluster name.
- **Index**: more or less like a database.
- **Type**: more or less like a database table.
- **Shard**: effectively a lucene index. Every index is composed of one or more shards. A shard can be a primary shard (or simply shard) or a **replica**.

Source: [shards-and-replicas-in-elasticsearch](http://stackoverflow.com/questions/15694724/shards-and-replicas-in-elasticsearch)
