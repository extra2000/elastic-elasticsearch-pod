Known Issues
============

Upgrade 7.17.x to 8.x
---------------------

Update this repository.

Upgrade keystore:

.. code:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --user=root --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    cp -v /tmp/secrets/es-master-01.keystore /usr/share/elasticsearch/config/elasticsearch.keystore
    ./bin/elasticsearch-keystore upgrade
    chown elasticsearch:elasticsearch /tmp/secrets/es-master-01.keystore

Stop Kibana and Logstash. Also disable routes at NGINX if any.

Temporary disable xpack security and comment all SSL related configs in ``configs/es-master-01.yml``:

.. code:: yaml

    xpack.security.enabled: false
    xpack.security.http.ssl.enabled: false
    #xpack.security.http.ssl.keystore.path: "elastic-http.p12"
    #xpack.security.http.ssl.verification_mode: certificate
    xpack.security.transport.ssl.enabled: false
    #xpack.security.transport.ssl.verification_mode: full
    #xpack.security.transport.ssl.client_authentication: required
    #xpack.security.transport.ssl.keystore.path: elastic-transport.p12
    #xpack.security.transport.ssl.truststore.path: elastic-transport.p12

Remove keystore mount lines in pod file if any.

Start pod and let the pod run until health is GREEN.

Copy keystore:

.. code:: bash

    podman cp ./secrets/es-master-01.keystore elk-es-master-01-pod-es-master-01:/usr/share/elasticsearch/config/elasticsearch.keystore

The old keystore can be safely deleted:

.. code:: bash

    rm ./secrets/es-master-01.keystore

Stop and destroy pod.

Re-enable xpack security and un-comment all SSL related configs in ``configs/es-master-01.yml``:

.. code:: yaml

    xpack.security.enabled: true
    xpack.security.http.ssl.enabled: true
    xpack.security.http.ssl.keystore.path: "elastic-http.p12"
    xpack.security.http.ssl.verification_mode: certificate
    xpack.security.transport.ssl.enabled: true
    xpack.security.transport.ssl.verification_mode: full
    xpack.security.transport.ssl.client_authentication: required
    xpack.security.transport.ssl.keystore.path: elastic-transport.p12
    xpack.security.transport.ssl.truststore.path: elastic-transport.p12

Start pod as usual.

Index Template ``.monitoring-logstash-mb`` missing field mapping
----------------------------------------------------------------

Define the following field ``logstash.node.stats.pipelines.queue.capacity.queue_size_in_bytes`` as ``long`` data type. This fixes an error with reason "``mapper [logstash.node.stats.pipelines.queue.capacity.queue_size_in_bytes] cannot be changed from type [float] to [long]``".
