Single Instance Deployment
==========================

Example how to deploy a basic ELK cluster (the ``elk-cluster-01``) on AMD64 machines for general purpose.

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

Create Podman Network (the ``elknet``)
--------------------------------------

Create ``/etc/cni/net.d/elknet.conflist`` file:

.. code-block:: yaml

    {
      "cniVersion": "0.4.0",
      "name": "elknet",
      "plugins": [
        {
          "type": "bridge",
          "bridge": "cni-podman1",
          "isGateway": true,
          "ipMasq": true,
          "hairpinMode": true,
          "ipam": {
            "type": "host-local",
            "routes": [{ "dst": "0.0.0.0/0" }],
            "ranges": [
              [
                {
                  "subnet": "192.168.124.0/24",
                  "gateway": "192.168.124.1"
                }
              ]
            ]
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        },
        {
          "type": "firewall"
        },
        {
          "type": "tuning"
        },
        {
          "type": "dnsname",
          "domainName": "elknet"
        }
      ]
    }

.. note::

    If ``/etc/cni/net.d/`` is not exists, create the directory using ``sudo mkdir -pv /etc/cni/net.d/``.

Distribute CA
-------------

From the project's root directory, ``cd`` into ``deployment/``:

.. code-block:: bash

    cd deployment/

Then, distribute into ``es-master-01``:

.. code-block:: bash

    cp -v _global_secrets_/elastic-ca.p12 examples/general-single-instance/es-master-01/secrets/

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

Prerequisites for ``es-master-01``
----------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/general-single-instance/es-master-01``:

.. code-block:: bash

    cd deployment/examples/general-single-instance/es-master-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-master-01.yaml{.example,}
    cp -v configs/es-master-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v elk-es-master-01-pod.yaml{.example,}

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-master-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-master-01``
   * - Enter name for directories and files of ``es-master-01``
     - ``es-master-01``
   * - Enter IP Addresses for instance
     - ``127.0.0.1``
   * - Enter DNS names for instance
     - ``elk-es-master-01-pod.elknet``, ``es-master-01.yourhostname.lan``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-master-01/es-master-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-master-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-master-01/es-master-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-master-01``'s ``elasticsearch-ssl-http.zip``
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
     - ``elk-es-master-01-pod.elknet``, ``es-master-01.yourhostname.lan``, ``localhost``
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

Create ``elk-es-master-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create elk-es-master-01-config
    podman run -it --rm -v elk-es-master-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
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

    scp -r -P 22 secrets/certificate-bundle secrets/elasticsearch-ssl-http USER@ES-MASTER-01:extra2000/elastic-elasticsearch-pod/deployment/examples/general-single-instance/es-master-01/secrets/

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/elk_es_master_01_pod_es_master_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "elk_es_master_01_pod_es_master_01"

Deployment
----------

Deploy ``es-master-01``
~~~~~~~~~~~~~~~~~~~~~~~

From the project's root directory, ``cd`` into ``deployment/examples/general-single-instance/es-master-01``:

.. code-block:: bash

    cd deployment/examples/general-single-instance/es-master-01

.. code-block:: bash

    podman play kube --network elknet --configmap configmaps/es-master-01.yaml --seccomp-profile-root ./seccomp elk-es-master-01-pod.yaml

Setup Elasticsearch REST API Credentials
----------------------------------------

.. code-block:: bash

    podman exec -it elk-es-master-01-pod-es-master-01 elasticsearch-setup-passwords interactive --url "https://fqdn-es-master-01:9200"

.. note::

    Replace ``fqdn-es-master-01`` with your ``es-master-01``'s FQDN. For testing purpose, use password ``abcde12345`` for all.

Check Cluster Health
--------------------

.. code-block:: bash

    podman run -it --rm --network elknet docker.io/curlimages/curl --insecure --user elastic:abcde12345 https://elk-es-master-01-pod.elknet:9200/_cluster/health/?pretty

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
