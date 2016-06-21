Unassigned Shards, how to fix?
==============================

::

    curl -XPUT 'localhost:9200/_cluster/settings' -d '{
        "transient" : {
            "cluster.routing.allocation.enable" : "all"
        }
    }'



