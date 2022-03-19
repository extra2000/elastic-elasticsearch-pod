Creating SSL Certs
==================

.. code-block:: bash

    git clone https://github.com/extra2000/openssl-templates.git
    cd openssl-templates/
    cp -rv certs/{example-elasticstack,elasticstack-mycluster}
    cd certs/elasticstack-mycluster

Create CA
---------

Create private key and certificate for CA:

.. code-block:: bash

    openssl genrsa -aes128 -out output/elastic-ca.key 2048
    openssl req -new -x509 -key output/elastic-ca.key -out output/elastic-ca.crt -config configs/ca-crt.conf -days 1825

Prerequisites before signing CSR later:

.. code-block:: bash

    touch index.txt
    echo '01' > serial

Create and Sign Elasticsearch Certs
-----------------------------------

.. note::

    For non single master deployment, change ``es-master-http`` and ``es-master-transport`` according to node name.

Create private key:

.. code-block:: bash

    openssl genrsa -out output/es-master-http.key 2048
    openssl genrsa -out output/es-master-transport.key 2048

.. note::

    Optionally, use ``-aes128`` to protect the private key with passphrase.

Create CSR:

.. code-block:: bash

    openssl req -new -key output/es-master-http.key -out output/es-master-http.csr -config configs/es-master-http-csr.conf
    openssl req -new -key output/es-master-transport.key -out output/es-master-transport.csr -config configs/es-master-transport-csr.conf

Sign the CSR:

    openssl ca -config configs/ca-policy.conf -out output/es-master-http.crt -infiles output/es-master-http.csr
    openssl ca -config configs/ca-policy.conf -out output/es-master-transport.crt -infiles output/es-master-transport.csr

Distribute the following certs into your Elasticsearch instance:

.. code-block:: bash

    cd output
    scp -P 22 elastic-ca.crt es-master-http.crt es-master-http.key es-master-transport.crt es-master-transport.key USER@es-master:extra2000/elastic-elasticsearch-pod/deployment/examples/general-single-instance/es-master/secrets/

Create and Sign Kibana Cert
---------------------------

Create private key:

.. code-block:: bash

    openssl genrsa -out output/kibana.key 2048

.. note::

    Optionally, use ``-aes128`` to protect the private key with passphrase.

Create CSR:

.. code-block:: bash

    openssl req -new -key output/kibana.key -out output/kibana.csr -config configs/kibana-csr.conf

Sign the CSR:

    openssl ca -config configs/ca-policy.conf -out output/kibana.crt -infiles output/kibana.csr

Distribute the following certs into your instance:

.. code-block:: bash

    cd output
    scp -P 22 elastic-ca.crt kibana.crt kibana.key USER@kibana:extra2000/elastic-kibana-pod/deployment/production/kibana/secrets/

Create and Sign Logstash Cert
-----------------------------

Create private key:

.. code-block:: bash

    openssl genrsa -out output/logstash.key 2048

.. note::

    Optionally, use ``-aes128`` to protect the private key with passphrase.

For unknown reason, `the PEM key needed to be converted to PKCS8`_:

.. _the PEM key needed to be converted to PKCS8: https://discuss.elastic.co/t/logstash-ssl-file-does-not-contain-a-valid-private-key-with-beats/173229/2

.. code-block:: bash

    openssl pkcs8 -in output/logstash.key -topk8 -out output/logstash-pkcs8.key -nocrypt

Create CSR:

.. code-block:: bash

    openssl req -new -key output/logstash.key -out output/logstash.csr -config configs/logstash-csr.conf

Sign the CSR:

    openssl ca -config configs/ca-policy.conf -out output/logstash.crt -infiles output/logstash.csr

Distribute the following certs into your instance:

.. code-block:: bash

    cd output
    scp -P 22 elastic-ca.crt logstash.crt logstash-pkcs8.key USER@logstash:extra2000/elastic-logstash-pod/deployment/production/logstash/secrets/

Create and Sign Beats Cert
--------------------------

Create private key:

.. code-block:: bash

    openssl genrsa -out output/beats.key 2048

.. note::

    Optionally, use ``-aes128`` to protect the private key with passphrase.

Create CSR:

.. code-block:: bash

    openssl req -new -key output/beats.key -out output/beats.csr -config configs/beats-csr.conf

Sign the CSR:

    openssl ca -config configs/ca-policy.conf -out output/beats.crt -infiles output/beats.csr

Distribute the following certs into your instance:

.. code-block:: bash

    cd output
    scp -P 22 elastic-ca.crt beats.crt beats.key USER@beats:extra2000/beats-metricbeat-pod/deployment/production/secrets/
    scp -P 22 elastic-ca.crt beats.crt beats.key USER@beats:extra2000/beats-filebeat-pod/deployment/production/secrets/
