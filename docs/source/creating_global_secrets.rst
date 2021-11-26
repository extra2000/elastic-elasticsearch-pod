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

``cd`` into the project root's directory:

.. code-block:: bash

    cd elastic-elasticsearch-pod

Build our Elasticsearch image
-----------------------------

From the project's root directory, ``cd`` into ``deployment/examples/``:

.. code-block:: bash

    cd deployment/examples/

Build our Elasticsearch image:

.. code-block:: bash

    podman build -t extra2000/elastic/elasticsearch -f Dockerfile.amd64 .

.. _creating-ca-certificate:

Creating ``elastic-ca.p12`` :term:`CA` Certificate
--------------------------------------------------

From the project's root directory ``cd`` into ``deployment``:

.. code-block:: bash

    cd deployment

Ensure the ``./_global_secrets_`` directory is labeled as ``container_file_t``:

.. code-block:: bash

    chcon -R -v -t container_file_t ./_global_secrets_

.. code-block:: bash

    podman run -it --network none --rm -v ./_global_secrets_:/tmp/secrets:rw localhost/extra2000/elastic/elasticsearch ./bin/elasticsearch-certutil ca --ca-dn "CN=Extra2000 Elastic Stack" --out /tmp/secrets/elastic-ca.p12

.. note::

    You can use password ``abcde12345`` for the testing purpose.

Check and make sure the CA certificate is correct:

.. code-block:: bash

    openssl pkcs12 -in _global_secrets_/elastic-ca.p12 -nodes | openssl x509 -noout -text | less
