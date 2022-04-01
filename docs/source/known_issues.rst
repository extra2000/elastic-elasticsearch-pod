Known Issues
============

Upgrade 7.17.x to 8.x
---------------------

Update this repository.

Create config volume, put keystore, and upgrade keystore:

.. code:: bash

    podman volume create es-master-config
    podman run -it --rm -v es-master-config:/usr/share/elasticsearch/config:rw --user elasticsearch --name es-tmp localhost/extra2000/elastic/elasticsearch bash
    podman cp ./secrets/es-master-01.keystore es-tmp:/usr/share/elasticsearch/config/elasticsearch.keystore
    ./bin/elasticsearch-keystore upgrade
    exit

Start new pod as usual.

Index Template ``.monitoring-logstash-mb`` missing field mapping
----------------------------------------------------------------

Define the following field ``logstash.node.stats.pipelines.queue.capacity.queue_size_in_bytes`` as ``long`` data type. This fixes an error with reason "``mapper [logstash.node.stats.pipelines.queue.capacity.queue_size_in_bytes] cannot be changed from type [float] to [long]``".
