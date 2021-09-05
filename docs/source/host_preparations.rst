Host Preparations
=================

Preparing hosts for deploying Elasticsearch Cluster.

Podman Installations
--------------------

Execute the following command to install Podman:

.. code-block:: bash

    sudo dnf install podman

SELinux Utilities
-----------------

Install ``udica`` which is required for simplifying SELinux for containers:

.. code-block:: bash

    sudo dnf install udica

Configure Podman
----------------

Create ``/etc/containers/containers.conf`` if not exists:

.. code-block:: bash

    sudo cp -v /usr/share/containers/containers.conf /etc/containers/containers.conf

Then, in ``/etc/containers/containers.conf``, make sure ``ulimits`` is set to at least ``65535`` and make ``memlock`` unlimited:

.. code-block:: text

    [containers]

    default_ulimits = [ 
      "nofile=65535:65535",
      "memlock=-1:-1"
    ]

Since the ``ulimit`` config above is applied globally, it will cause a permission error when Podman is executed as rootless. To prevent this error, create an empty ``default_ulimits`` in ``~/.config/containers/containers.conf`` file:

.. code-block:: text

    [containers]

    default_ulimits = []

Configure ``sysctl``
--------------------

Create ``/etc/sysctl.d/vm-max-map-counts.conf`` with the following line:

.. code-block:: text

    vm.max_map_count=262144

To apply ``vm.max_map_count`` without reboot, execute the following command:

.. code-block:: text

    sudo sysctl -w vm.max_map_count=262144
