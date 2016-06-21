Unassigned Shards, how to fix?
==============================
.. code-block:: json
curl -XPUT 'localhost:9200/_cluster/settings' -d '{
    "transient" : {
        "cluster.routing.allocation.enable" : "all"
    }
}'
::