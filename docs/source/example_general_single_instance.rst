Single Instance Deployment
==========================

Example how to deploy a single instance Elastic Stack cluster for general purpose.

Getting Started
---------------

Create directory and clone our repository:

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/elastic-elasticsearch-pod.git

``cd`` into project's root directory:

.. code-block:: bash

    cd elastic-elasticsearch-pod

From the project's root directory, ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/``:

.. code-block:: bash

    cd deployment/examples/

Build our Elasticsearch image:

.. code-block:: bash

    podman build -t extra2000/elastic/elasticsearch -f Dockerfile.amd64 .

Deploy MinIO
------------

Deploy MinIO project from `extra2000/minio-pod`_ and use the following credentials for testing purpose:

.. _extra2000/minio-pod: https://github.com/extra2000/minio-pod

* ``minio_root_user``: ``minio``
* ``minio_root_password``: ``minio123``

.. note::

    Later, ``s3.client.default.access_key`` refers to ``minio_root_user`` and ``s3.client.default.secret_key`` refers to ``minio_root_password``.

Prerequisites for ``es-master``
-------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/general-single-instance/es-master``:

.. code-block:: bash

    cd deployment/examples/general-single-instance/es-master

Create Config Files
~~~~~~~~~~~~~~~~~~~

Create config files and fix permissions:

.. code-block:: bash

    cp -v configmaps/es-master.yaml{.example,}
    cp -v configs/es-master.yml{.example,}
    cp -v configs/log4j2.properties{.example,}
    chmod -R o+r ./configs/*

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-master-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-master-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-master-config
    podman run -it --rm -v es-master-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create SELinux Security Policy:

.. code-block:: bash

    cp -v selinux/es_master_podman.cil{.example,}

Load the security policy:

.. code-block:: bash

    sudo semodule -i selinux/es_master_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_master_podman"

Deployment
----------

Deploy ``es-master``
~~~~~~~~~~~~~~~~~~~~

From the project's root directory, ``cd`` into ``deployment/examples/general-single-instance/es-master``:

.. code-block:: bash

    cd deployment/examples/general-single-instance/es-master

.. code-block:: bash

    podman play kube --configmap configmaps/es-master.yaml --seccomp-profile-root ./seccomp es-master-pod.yaml

Reset Built-in User Passwords
-----------------------------

Reset password for ``elastic`` user:

.. code-block:: bash

    podman exec -it es-master-pod-srv01 elasticsearch-reset-password --interactive --username elastic --url "https://es-master.mydomain:9200"

Reset password for ``kibana_system`` user:

.. code-block:: bash

    podman exec -it es-master-pod-srv01 elasticsearch-reset-password --interactive --username kibana_system --url "https://es-master.mydomain:9200"

Reset password for ``remote_monitoring_user`` user:

.. code-block:: bash

    podman exec -it es-master-pod-srv01 elasticsearch-reset-password --interactive --username remote_monitoring_user --url "https://es-master.mydomain:9200"

.. note::

    Replace ``es-master.mydomain`` with your ``es-master``'s FQDN. For testing purpose, use password ``abcde12345`` for all users.

Check Cluster Health
--------------------

.. code-block:: bash

    curl --cacert ./secrets/elastic-ca.crt --user elastic:abcde12345 https://es-master.mydomain:9200/_cluster/health/?pretty

If success, the command above should produce the following output:

.. code-block:: json

    {
      "cluster_name" : "elk-cluster-01",
      "status" : "green",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 3,
      "active_shards" : 3,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 0,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 100.0
    }
