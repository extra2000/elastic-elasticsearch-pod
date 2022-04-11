Cluster Multiple Servers Deployment
===================================

Example how to deploy a basic ELK cluster (the ``elk-cluster-01``) on multiple servers as shown in Figure below.

.. image:: _static/concept.svg

.. note::

    * The ``kibana-01`` is provided by https://github.com/extra2000/elastic-kibana-pod repository.
    * The ``logstash-01`` is provided by https://github.com/extra2000/elastic-logstash-pod repository.
    * The ``minio-01`` is provided by https://github.com/extra2000/minio-pod repository.

Getting Started
---------------

On every nodes, create directory and clone our repository:

.. code-block:: bash

    mkdir ~/extra2000
    cd ~/extra2000
    git clone https://github.com/extra2000/elastic-elasticsearch-pod.git

``cd`` into project's root directory:

    cd elastic-elasticsearch-pod

From the project's root directory, ``cd`` into ``deployment/examples/``:

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

Prerequisites for ``es-master-01``
----------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-master-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-master-01

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

    cp -v es-master-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-master-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-master-01-config
    podman run -it --rm -v es-master-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

On the node, don't forget to label the ``secrets`` directory as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./secrets

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_master_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_master_01_podman"

Prerequisites for ``es-master-02``
----------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-master-02``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-master-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-master-02.yaml{.example,}
    cp -v configs/es-master-02.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-master-02-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-master-02-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-master-02-config
    podman run -it --rm -v es-master-02-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -storepass changeit -alias minio-ca -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_master_02_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_master_02_podman"

Prerequisites for ``es-master-03``
----------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-master-03``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-master-03

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-master-03.yaml{.example,}
    cp -v configs/es-master-03.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-master-03-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-master-03-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-master-03-config
    podman run -it --rm -v es-master-03-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_master_03_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_master_03_podman"

Prerequisites for ``es-hot-01``
-------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-hot-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-hot-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-hot-01.yaml{.example,}
    cp -v configs/es-hot-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-hot-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-hot-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-hot-01-config
    podman run -it --rm -v es-hot-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_hot_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_hot_01_podman"

Prerequisites for ``es-hot-02``
-------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-hot-02``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-hot-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-hot-02.yaml{.example,}
    cp -v configs/es-hot-02.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-hot-02-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-hot-02-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-hot-02-config
    podman run -it --rm -v es-hot-02-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_hot_02_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_hot_02_podman"

Prerequisites for ``es-warm-01``
--------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-warm-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-warm-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-warm-01.yaml{.example,}
    cp -v configs/es-warm-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-warm-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-warm-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-warm-01-config
    podman run -it --rm -v es-warm-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_warm_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_warm_01_podman"

Prerequisites for ``es-warm-02``
--------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-warm-02``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-warm-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-warm-02.yaml{.example,}
    cp -v configs/es-warm-02.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-warm-02-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-warm-02-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-warm-02-config
    podman run -it --rm -v es-warm-02-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_warm_02_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_warm_02_podman"

Prerequisites for ``es-cold-01``
--------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-cold-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-cold-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-cold-01.yaml{.example,}
    cp -v configs/es-cold-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-cold-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-cold-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-cold-01-config
    podman run -it --rm -v es-cold-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_cold_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_cold_01_podman"

Prerequisites for ``es-cold-02``
--------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-cold-02``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-cold-02

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-cold-02.yaml{.example,}
    cp -v configs/es-cold-02.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-cold-02-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-cold-02-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-cold-02-config
    podman run -it --rm -v es-cold-02-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create
    ./bin/elasticsearch-keystore add s3.client.default.access_key
    ./bin/elasticsearch-keystore add s3.client.default.secret_key

Create JVM CA Certs
~~~~~~~~~~~~~~~~~~~

On the node, create ``./secrets/jdk-cacerts`` file:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch cp -v /usr/share/elasticsearch/jdk/lib/security/cacerts /secrets/jdk-cacerts

Import MinIO CA Cert
~~~~~~~~~~~~~~~~~~~~

Copy your MinIO CA certificate (for example the ``minio-ca.crt`` file) into ``./secrets/`` directory and label it with ``container_file_t``:

.. code-block:: bash

    chcon -v -t container_file_t ./secrets/minio-ca.crt

Then, convert the certificate to DER format:

.. code-block:: bash

    openssl x509 -outform der -in ./secrets/minio-ca.crt -out ./secrets/minio-ca.der

On the node, import MinIO CA cert into JDK ``cacerts``:

.. code-block:: bash

    podman run -it --rm -v ./secrets:/secrets:rw extra2000/elastic/elasticsearch jdk/bin/keytool -import -alias your-alias -keystore /secrets/jdk-cacerts -file /secrets/minio-ca.der

.. note::

    The default password for the JDK ``cacerts`` is ``changeit``.

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_cold_02_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_cold_02_podman"

Prerequisites for ``es-ml-01``
------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-ml-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-ml-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-ml-01.yaml{.example,}
    cp -v configs/es-ml-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-ml-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-ml-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-ml-01-config
    podman run -it --rm -v es-ml-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_ml_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_ml_01_podman"

Prerequisites for ``es-ingest-01``
----------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-ingest-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-ingest-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-ingest-01.yaml{.example,}
    cp -v configs/es-ingest-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-ingest-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-ingest-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-ingest-01-config
    podman run -it --rm -v es-ingest-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_ingest_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_ingest_01_podman"

Prerequisites for ``es-transform-01``
-------------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-transform-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-transform-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-transform-01.yaml{.example,}
    cp -v configs/es-transform-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-transform-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-transform-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-transform-01-config
    podman run -it --rm -v es-transform-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_transform_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_transform_01_podman"

Prerequisites for ``es-coord-01``
---------------------------------

From the project's root directory, ``cd`` into ``deployment/examples/cluster-multi-servers/es-coord-01``:

.. code-block:: bash

    cd deployment/examples/cluster-multi-servers/es-coord-01

Create Config Files
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    cp -v configmaps/es-coord-01.yaml{.example,}
    cp -v configs/es-coord-01.yml{.example,}
    cp -v configs/log4j2.properties{.example,}

Allow config files to be mounted into container:

.. code-block:: bash

    chcon -R -v -t container_file_t ./configs

Create pod file:

.. code-block:: bash

    cp -v es-coord-01-pod.yaml{.example,}

Creating Keystore
~~~~~~~~~~~~~~~~~

Create ``es-coord-01-config`` volume and then create keystore into the volume:

.. code-block:: bash

    podman volume create es-coord-01-config
    podman run -it --rm -v es-coord-01-config:/usr/share/elasticsearch/config:rw --entrypoint=bash localhost/extra2000/elastic/elasticsearch
    ./bin/elasticsearch-keystore create

Load SELinux Security Policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    sudo semodule -i selinux/es_coord_01_podman.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}

Verify that the SELinux module exists:

.. code-block:: bash

    sudo semodule --list | grep -e "es_coord_01_podman"

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
    podman generate systemd --files --name es-master-01-pod-srv01
    systemctl --user enable container-es-master-01-pod-srv01.service

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
    podman generate systemd --files --name es-master-02-pod-srv01
    systemctl --user enable container-es-master-02-pod-srv01.service

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
    podman generate systemd --files --name es-master-03-pod-srv01
    systemctl --user enable container-es-master-03-pod-srv01.service

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
    podman generate systemd --files --name es-hot-01-pod-srv01
    systemctl --user enable container-es-hot-01-pod-srv01.service

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
    podman generate systemd --files --name es-hot-02-pod-srv01
    systemctl --user enable container-es-hot-02-pod-srv01.service

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
    podman generate systemd --files --name es-warm-01-pod-srv01
    systemctl --user enable container-es-warm-01-pod-srv01.service

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
    podman generate systemd --files --name es-warm-02-pod-srv01
    systemctl --user enable container-es-warm-02-pod-srv01.service

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
    podman generate systemd --files --name es-cold-01-pod-srv01
    systemctl --user enable container-es-cold-01-pod-srv01.service

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
    podman generate systemd --files --name es-cold-02-pod-srv01
    systemctl --user enable container-es-cold-02-pod-srv01.service

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
    podman generate systemd --files --name es-ml-01-pod-srv01
    systemctl --user enable container-es-ml-01-pod-srv01.service

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
    podman generate systemd --files --name es-ingest-01-pod-srv01
    systemctl --user enable container-es-ingest-01-pod-srv01.service

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
    podman generate systemd --files --name es-transform-01-pod-srv01
    systemctl --user enable container-es-transform-01-pod-srv01.service

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
    podman generate systemd --files --name es-coord-01-pod-srv01
    systemctl --user enable container-es-coord-01-pod-srv01.service

Setup Elasticsearch REST API Credentials
----------------------------------------

``ssh`` into ``es-coord-01`` and execute the following command:

.. code-block:: bash

    podman exec -it es-coord-01-pod-srv01 elasticsearch-setup-passwords interactive --url "https://fqdn-es-coord-01:9200"

.. note::

    Replace ``fqdn-es-coord-01`` with your ``es-coord-01``'s FQDN. For testing purpose, use password ``abcde12345`` for all.

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
