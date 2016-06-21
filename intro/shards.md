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

###### summary
- **Node**: an Elasticsearch instance running (a java process). Usually every node runs on its own machine.
- **Cluster**: one or more nodes with the same cluster name.
- **Index**: more or less like a database.
- **Type**: more or less like a database table.
- **Shard**: effectively a lucene index. Every index is composed of one or more shards. A shard can be a primary shard (or simply shard) or a **replica**.

Source: [shards-and-replicas-in-elasticsearch](http://stackoverflow.com/questions/15694724/shards-and-replicas-in-elasticsearch)
