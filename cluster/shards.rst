Unassigned Shards, how to fix?
==============================

::

    # v0.90.x and earlier
    curl -XPUT 'localhost:9200/_settings' -d '{
        "index.routing.allocation.disable_allocation": false
    }'
    
    # v1.0+
    curl -XPUT 'localhost:9200/_cluster/settings' -d '{
        "transient" : {
            "cluster.routing.allocation.enable" : "all"
        }
    }'



