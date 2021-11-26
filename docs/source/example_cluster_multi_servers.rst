Example Cluster Multiple Servers Deployment
===========================================

Example how to deploy a basic ELK cluster (the ``elk-cluster-01``) on multiple servers as shown in Figure below.

.. image:: _static/concept.svg

.. note::

    * The ``kibana-01`` is provided by https://github.com/extra2000/elastic-kibana-pod repository.
    * The ``logstash-01`` is provided by https://github.com/extra2000/elastic-logstash-pod repository.
    * The ``minio-01`` is provided by https://github.com/extra2000/minio-pod repository.

Getting Started
---------------

Make sure to create global secrets according to Chapter :ref:`creating_global_secrets`.

On every nodes, create directory and clone our repository:

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/elastic-elasticsearch-pod.git

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/

Build our Elasticsearch image:

.. code-block:: bash

    podman build -t extra2000/elastic/elasticsearch -f Dockerfile.amd64 .

Distribute CA
-------------

``cd`` into ``elastic-elasticsearch-pod/deployment/_global_secrets_/``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/_global_secrets_/

Then, distribute into all nodes:

.. code-block:: bash

    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-coord-01/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-01/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-02/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-03/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-01/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-02/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-01/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-02/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-01/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-02/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ml-01/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ingest-01/secrets/
    scp -P 22 elastic-ca.p12 USER@SERVER:/path/to/elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-transform-01/secrets/

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

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-master-01.yaml{.example,}
    cp -v configs/es-master-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

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
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-master-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-master-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_master_01_pod_es_master_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_master_01_pod_es_master_01"

Prerequisites for ``es-master-02``
----------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-master-02.yaml{.example,}
    cp -v configs/es-master-02.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

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
     - ``es-master-02``
   * - Enter name for directories and files of ``es-master-02``
     - ``es-master-02``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-master-02/es-master-02.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-master-02.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-master-02/es-master-02.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-master-02``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-master-02-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-master-02.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_master_02_pod_es_master_02.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_master_02_pod_es_master_02"

Prerequisites for ``es-master-03``
----------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-03``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-03

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-master-03.yaml{.example,}
    cp -v configs/es-master-03.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-master-03``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-master-03``
   * - Enter name for directories and files of ``es-master-03``
     - ``es-master-03``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-master-03/es-master-03.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-master-03.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-master-03/es-master-03.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-master-03``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-master-03-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-master-03.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_master_03_pod_es_master_03.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_master_03_pod_es_master_03"

Prerequisites for ``es-hot-01``
-------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-hot-01.yaml{.example,}
    cp -v configs/es-hot-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-hot-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-hot-01``
   * - Enter name for directories and files of ``es-hot-01``
     - ``es-hot-01``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-hot-01/es-hot-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-hot-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-hot-01/es-hot-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-hot-01``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-hot-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-hot-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_hot_01_pod_es_hot_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_hot_01_pod_es_hot_01"

Prerequisites for ``es-hot-02``
-------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-hot-02.yaml{.example,}
    cp -v configs/es-hot-02.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-hot-02``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-hot-02``
   * - Enter name for directories and files of ``es-hot-02``
     - ``es-hot-02``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-hot-02/es-hot-02.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-hot-02.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-hot-02/es-hot-02.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-hot-02``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``<ENTER>``
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

Create ``./secrets/es-hot-02-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-hot-02.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_hot_02_pod_es_hot_02.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_hot_02_pod_es_hot_02"

Prerequisites for ``es-warm-01``
--------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-warm-01.yaml{.example,}
    cp -v configs/es-warm-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-warm-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-warm-01``
   * - Enter name for directories and files of ``es-warm-01``
     - ``es-warm-01``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-warm-01/es-warm-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-warm-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-warm-01/es-warm-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-warm-01``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``<ENTER>``
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

Create ``./secrets/es-warm-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-warm-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_warm_01_pod_es_warm_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_warm_01_pod_es_warm_01"

Prerequisites for ``es-warm-02``
--------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-warm-02.yaml{.example,}
    cp -v configs/es-warm-02.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-warm-02``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-warm-02``
   * - Enter name for directories and files of ``es-warm-02``
     - ``es-warm-02``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-warm-02/es-warm-02.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-warm-02.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-warm-02/es-warm-02.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-warm-02``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-warm-02-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-warm-02.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_warm_02_pod_es_warm_02.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_warm_02_pod_es_warm_02"

Prerequisites for ``es-cold-01``
--------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-cold-01.yaml{.example,}
    cp -v configs/es-cold-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-cold-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-cold-01``
   * - Enter name for directories and files of ``es-cold-01``
     - ``es-cold-01``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-cold-01/es-cold-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-cold-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-cold-01/es-cold-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-cold-01``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-cold-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-cold-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_cold_01_pod_es_cold_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_cold_01_pod_es_cold_01"

Prerequisites for ``es-cold-02``
--------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-cold-02.yaml{.example,}
    cp -v configs/es-cold-02.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-cold-02``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-cold-02``
   * - Enter name for directories and files of ``es-cold-02``
     - ``es-cold-02``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-cold-02/es-cold-02.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-cold-02.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-cold-02/es-cold-02.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-02

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-cold-02``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-cold-02-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-cold-02.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_cold_02_pod_es_cold_02.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_cold_02_pod_es_cold_02"

Prerequisites for ``es-ml-01``
------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ml-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ml-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-ml-01.yaml{.example,}
    cp -v configs/es-ml-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-ml-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-ml-01``
   * - Enter name for directories and files of ``es-ml-01``
     - ``es-ml-01``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-ml-01/es-ml-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-ml-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-ml-01/es-ml-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-ml-01``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-ml-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-ml-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_ml_01_pod_es_ml_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_ml_01_pod_es_ml_01"

Prerequisites for ``es-ingest-01``
----------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ingest-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ingest-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-ingest-01.yaml{.example,}
    cp -v configs/es-ingest-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-ingest-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-ingest-01``
   * - Enter name for directories and files of ``es-ingest-01``
     - ``es-ingest-01``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-ingest-01/es-ingest-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-ingest-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-ingest-01/es-ingest-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create HTTP SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-ingest-01``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-ingest-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-ingest-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_ingest_01_pod_es_ingest_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_ingest_01_pod_es_ingest_01"

Prerequisites for ``es-transform-01``
-------------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-transform-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-transform-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-transform-01.yaml{.example,}
    cp -v configs/es-transform-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-transform-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-transform-01``
   * - Enter name for directories and files of ``es-transform-01``
     - ``es-transform-01``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-transform-01/es-transform-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-transform-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-transform-01/es-transform-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create HTTP SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-transform-01``'s ``elasticsearch-ssl-http.zip``
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-transform-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-transform-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_transform_01_pod_es_transform_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_transform_01_pod_es_transform_01"

Prerequisites for ``es-coord-01``
---------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-coord-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-coord-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-coord-01.yaml{.example,}
    cp -v configs/es-coord-01.yml{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Creating Transport SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the ``./secrets`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Create transport SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil cert --ca /tmp/secrets/elastic-ca.p12 --multiple

.. list-table:: Questions and answers for creating ``es-coord-01``'s ``certificate-bundle.zip``
   :widths: 50 50
   :header-rows: 1

   * - Question
     - Answer
   * - Enter password for CA (``/tmp/secrets/elastic-ca.p12``)
     - ``abcde12345``
   * - Enter instance name
     - ``es-coord-01``
   * - Enter name for directories and files of ``es-coord-01``
     - ``es-coord-01``
   * - Enter IP Addresses for instance
     - ``SERVER_IP``, ``127.0.0.1``
   * - Enter DNS names for instance
     - ``SERVER_FQDN``, ``localhost``
   * - Would you like to specify another instance?
     - ``n``
   * - Please enter the desired output file
     - ``/tmp/secrets/certificate-bundle.zip``
   * - Enter password for ``es-coord-01/es-coord-01.p12``
     - ``abcde12345``

Extract the certificate archive:

.. code-block:: bash

    unzip ./secrets/certificate-bundle.zip -d ./secrets/certificate-bundle

Verify the ``es-coord-01.p12`` certificate:

.. code-block:: bash

    openssl pkcs12 -in ./secrets/certificate-bundle/es-coord-01/es-coord-01.p12 -nodes | openssl x509 -noout -text | less

Creating HTTP SSL Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create HTTP SSL certificate:

.. code-block:: bash

    podman run -it --network none --rm -v ./secrets:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil http

.. list-table:: Questions and answers for creating ``es-coord-01``'s ``elasticsearch-ssl-http.zip``.
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
     - ``SERVER_FQDN``, ``localhost``
   * - Which IP addresses will be used to connect to your nodes?
     - ``SERVER_IP``, ``127.0.0.1``
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

Create ``./secrets/es-coord-01-pod.keystore`` file to store certificate passwords:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/tmp/secrets:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
    ./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
    cp -v /usr/share/elasticsearch/config/elasticsearch.keystore /tmp/secrets/es-coord-01.keystore

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_coord_01_pod_es_coord_01.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_coord_01_pod_es_coord_01"

Deployment
----------

Deploy ``es-master-01``
~~~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-master-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-master-01.yaml --seccomp-profile-root ./seccomp es-master-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-master-01-pod
    systemctl --user enable pod-es-master-01-pod.service container-es-master-01-pod-es-master-01.service

Deploy ``es-master-02``
~~~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-master-02`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-02

.. code-block:: bash

    podman play kube --configmap configmaps/es-master-02.yaml --seccomp-profile-root ./seccomp es-master-02-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-master-02-pod
    systemctl --user enable pod-es-master-02-pod.service container-es-master-02-pod-es-master-02.service

Deploy ``es-master-03``
~~~~~~~~~~~~~~~~~~~~~~~


``ssh`` into ``es-master-03`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-03``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-master-03

.. code-block:: bash

    podman play kube --configmap configmaps/es-master-03.yaml --seccomp-profile-root ./seccomp es-master-03-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-master-03-pod
    systemctl --user enable pod-es-master-03-pod.service container-es-master-03-pod-es-master-03.service

Deploy ``es-hot-01``
~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-hot-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-hot-01.yaml --seccomp-profile-root ./seccomp es-hot-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-hot-01-pod
    systemctl --user enable pod-es-hot-01-pod.service container-es-hot-01-pod-es-hot-01.service

Deploy ``es-hot-02``
~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-hot-02`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-hot-02

.. code-block:: bash

    podman play kube --configmap configmaps/es-hot-02.yaml --seccomp-profile-root ./seccomp es-hot-02-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-hot-02-pod
    systemctl --user enable pod-es-hot-02-pod.service container-es-hot-02-pod-es-hot-02.service

Deploy ``es-warm-01``
~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-warm-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-warm-01.yaml --seccomp-profile-root ./seccomp es-warm-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-warm-01-pod
    systemctl --user enable pod-es-warm-01-pod.service container-es-warm-01-pod-es-warm-01.service

Deploy ``es-warm-02``
~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-warm-02`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-warm-02

.. code-block:: bash

    podman play kube --configmap configmaps/es-warm-02.yaml --seccomp-profile-root ./seccomp es-warm-02-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-warm-02-pod
    systemctl --user enable pod-es-warm-02-pod.service container-es-warm-02-pod-es-warm-02.service

Deploy ``es-cold-01``
~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-cold-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-cold-01.yaml --seccomp-profile-root ./seccomp es-cold-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-cold-01-pod
    systemctl --user enable pod-es-cold-01-pod.service container-es-cold-01-pod-es-cold-01.service

Deploy ``es-cold-02``
~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-cold-02`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-02``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-cold-02

.. code-block:: bash

    podman play kube --configmap configmaps/es-cold-02.yaml --seccomp-profile-root ./seccomp es-cold-02-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-cold-02-pod
    systemctl --user enable pod-es-cold-02-pod.service container-es-cold-02-pod-es-cold-02.service

Deploy ``es-ml-01``
~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-ml-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ml-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ml-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-ml-01.yaml --seccomp-profile-root ./seccomp es-ml-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-ml-01-pod
    systemctl --user enable pod-es-ml-01-pod.service container-es-ml-01-pod-es-ml-01.service

Deploy ``es-ingest-01``
~~~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-ingest-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ingest-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-ingest-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-ingest-01.yaml --seccomp-profile-root ./seccomp es-ingest-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-ingest-01-pod
    systemctl --user enable pod-es-ingest-01-pod.service container-es-ingest-01-pod-es-ingest-01.service

Deploy ``es-transform-01``
~~~~~~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-transform-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-transform-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-transform-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-transform-01.yaml --seccomp-profile-root ./seccomp es-transform-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-transform-01-pod
    systemctl --user enable pod-es-transform-01-pod.service container-es-transform-01-pod-es-transform-01.service

Deploy ``es-coord-01``
~~~~~~~~~~~~~~~~~~~~~~

``ssh`` into ``es-coord-01`` and ``cd`` into ``elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-coord-01``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/cluster-multi-servers/es-coord-01

.. code-block:: bash

    podman play kube --configmap configmaps/es-coord-01.yaml --seccomp-profile-root ./seccomp es-coord-01-pod.yaml

Generate ``systemd`` files and enable on ``boot``:

.. code-block:: bash

    mkdir -pv ~/.config/systemd/user
    cd ~/.config/systemd/user
    podman generate systemd --files --name es-coord-01-pod
    systemctl --user enable pod-es-coord-01-pod.service container-es-coord-01-pod-es-coord-01.service

Setup Elasticsearch REST API Credentials
----------------------------------------

``ssh`` into ``es-coord-01`` and execute the following command:

.. code-block:: bash

    podman exec -it es-coord-01-pod-es-coord-01 elasticsearch-setup-passwords interactive

.. note::

    For testing purpose, use password ``abcde12345`` for all.

Check Cluster Health
--------------------

.. code-block:: bash

    podman run -it --rm --network host docker.io/curlimages/curl --insecure --user elastic:abcde12345 https://ES-COORD-01-IP-ADDRESS:9200/_cluster/health/?pretty

If success, the command above should produce the following output:

.. code-block:: json

    {
      "cluster_name" : "elk-cluster-01",
      "status" : "green",
      "timed_out" : false,
      "number_of_nodes" : 13,
      "number_of_data_nodes" : 6,
      "active_primary_shards" : 12,
      "active_shards" : 24,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 0,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 100.0
    }
