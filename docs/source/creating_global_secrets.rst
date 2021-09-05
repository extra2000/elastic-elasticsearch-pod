.. _creating_global_secrets:

Creating Global Secrets
=======================

Instructions how to create CA and Elasticsearch Transport certificates. This Chapter shall be executed on sysadmin's laptop only.

Getting Started
---------------

Clone repository `extra2000/elastic-elasticsearch-pod`_ on your laptop:

.. _extra2000/elastic-elasticsearch-pod: https://github.com/extra2000/elastic-elasticsearch-pod

.. code-block:: bash

    git clone https://github.com/extra2000/elastic-elasticsearch-pod

Build our Elasticsearch image
-----------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment/examples/``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment/examples/

Build our Elasticsearch image:

.. code-block:: bash

    sudo podman build -t extra2000/elastic/elasticsearch -f Dockerfile.amd64 .
    podman build -t extra2000/elastic/elasticsearch -f Dockerfile.amd64 .

.. _creating-ca-certificate:

Creating ``elastic-ca.p12`` :term:`CA` Certificate
--------------------------------------------------

``cd`` into ``elastic-elasticsearch-pod/deployment``:

.. code-block:: bash

    cd elastic-elasticsearch-pod/deployment

Ensure the ``./_global_secrets_`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./_global_secrets_

.. code-block:: bash

    podman run -it --network none --rm -v ./_global_secrets_:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil ca --out /tmp/secrets/elastic-ca.p12

.. note::

    You can use password ``abcde12345`` for the testing purpose.
