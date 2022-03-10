Single Instance Deployment
==========================

Example how to deploy a single instance Elastic Stack cluster for general purpose.

Getting Started
---------------

Make sure to create global secrets according to Chapter :ref:`creating_global_secrets`.

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

Distribute CA
-------------

From the project's root directory, ``cd`` into ``deployment/``:

.. code-block:: bash

    cd deployment/

Then, distribute into ``es-master``:

.. code-block:: bash

    cp -v _global_secrets_/elastic-ca.p12 examples/general-single-instance/es-master/secrets/

.. warning::

    You should create certificates on your laptop and then distribute the created certificates into your nodes. Again, never distribute the ``elastic-ca.p12``.

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

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-master``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-master``
   * - Enter name for directories and files of ``es-master``
     - ``es-master``
   * - Enter IP Addresses for instance
     - ``127.0.0.1``
   * - Enter DNS names for instance
     - ``es-master.mydomain``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-master/es-master.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-master.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-master/es-master.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-master``'s ``elasticsearch-ssl-http.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Do you wish to generate a Certificate Signing Request (CSR)?
     - ``n``
   * - Do you have an existing Certificate Authority (CA) key-pair that you wish to use to sign your certificate?
     - ``y``
   * - What is the path to your CA?
     - ``/tmp/secrets/elastic-ca.p12``
   * - Password for ``elastic-ca.p12``
     - ``abcde12345``
   * - How long should your certificates be valid?
     - ``5y``
   * - Generate a certificate per node? [y/N]
     - ``n``
   * - Which hostnames will be used to connect to your nodes?
     - ``es-master.mydomain``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``127.0.0.1``
   * - Other certificate options. Do you wish to change any of these options? [y/N]
     - ``n``
   * - What password do you want for your private key(s)? Provide a password for the "http.p12" file:
     - ``abcde12345``
   * - Where should we save the generated files?
     - ``/tmp/secrets/elasticsearch-ssl-http.zip``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/elasticsearch-ssl-http.zip -d ./secrets/elasticsearch-ssl-http

Verify the ``http.p12`` and ``elasticsearch-ca.pem`` certificates:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/elasticsearch-ssl-http/elasticsearch/http.p12 -nodes | openssl x509 -noout -text | less
    cat ./secrets/elasticsearch-ssl-http/kibana/elasticsearch-ca.pem | openssl x509 -noout -text | less

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-master-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-master-config
    podman run -it --rm -v es-master-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Distribute Secrets
~~~~~~~~~~~~~~~~~~

Copy the created certificates to the node:

.. code-block:: bash

    scp -r -P 22 secrets/certificate-bundle secrets/elasticsearch-ssl-http USER@es-master:extra2000/elastic-elasticsearch-pod/deployment/examples/general-single-instance/es-master/secrets/

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

Setup Elasticsearch REST API Credentials
----------------------------------------

.. code-block:: bash

    podman exec -it es-master-pod-srv01 elasticsearch-setup-passwords interactive --url "https://es-master.mydomain:9200"

.. note::

    Replace ``es-master.mydomain`` with your ``es-master``'s FQDN. For testing purpose, use password ``abcde12345`` for all.

Check Cluster Health
--------------------

.. code-block:: bash

    podman run -it --rm docker.io/curlimages/curl --insecure --user elastic:abcde12345 https://es-master.mydomain:9200/_cluster/health/?pretty

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
