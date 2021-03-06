Installation
============

Common Installation Tasks
-------------------------

These tasks are required by all installation methods.

You will need to provide an initial configuration files for FAUCET and Gauge, and create directores for FAUCET and Gauge to log to.

.. code:: console

  mkdir -p /etc/ryu/faucet
  mkdir -p /var/log/ryu/faucet
  mkdir -p /var/log/ryu/gauge
  $EDITOR /etc/ryu/faucet/faucet.yaml
  $EDITOR /etc/ryu/faucet/gauge.yaml

This example ``faucet.yaml`` file creates an untagged VLAN between ports 1 and 2 on DP 0x1. See :doc:`configuration` for
more advanced configuration. See :doc:`vendors/index` for how to configure your switch.

.. code:: yaml

  vlans:
      100:
          description: "dev VLAN"
  dps:
      switch-1:
          dp_id: 0x1
          interfaces:
              1:
                  native_vlan: 100
              2:
                  native_vlan: 100


This example ``gauge.yaml`` file instructs Gauge to poll the switch at 10s intervals and make the results available to Prometheus.
See :doc:`configuration` for more advanced configuration.

.. code:: yaml

  faucet_configs:
      - '/etc/ryu/faucet/faucet.yaml'
  watchers:
    port_stats:
        dps: ['switch-1']
        type: 'port_stats'
        interval: 10
        db: 'prometheus'
    flow_table:
        dps: ['switch-1']
        type: 'flow_table'
        interval: 10
        db: 'prometheus'
  dbs:
    prometheus:
        type: 'prometheus'
        prometheus_port: 9303
        prometheus_addr: ''


Installation with Docker
------------------------

We provide official automated builds on `Docker Hub <https://hub.docker.com/r/faucet/>`_ so that you can easily
run Faucet and it's components in a self-contained environment without installing on the main host system.

See our :doc:`docker` section for detauls on how to install and start the Faucet and Gauge docker images.

You can check that Faucet and Gauge are running via systemd or via docker:

.. code:: console

    service faucet status
    service gauge status
    docker ps

Installation with pip
---------------------

You can install the latest pip package, or you can install directly from git via pip.

First, install some python dependencies:

.. code:: console

  apt-get install python3-dev python3-pip
  pip3 install setuptools
  pip3 install wheel

Then install the latest stable release of faucet from pypi, via pip:

.. code:: console

  pip3 install faucet

Or, install the latest development code from git, via pip:

.. code:: console

  pip3 install git+https://github.com/faucetsdn/faucet.git

Starting Faucet Manually
~~~~~~~~~~~~~~~~~~~~~~~~

Faucet includes a start up script for starting Faucet and Gauge easily from the
command line.

To run Faucet manually:

.. code:: console

  faucet --verbose

To run Gauge manually:

.. code:: console

  gauge --verbose

There are a number of options that you can supply the start up script for
changing various options such as OpenFlow port and setting up and encrypted
control channel. You can find a list of the additional arguments by running:

.. code:: console

  faucet --help


Starting Faucet With Systemd
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Systemd can be used to start Faucet and Gauge at boot automatically:

.. code:: console

    $EDITOR /etc/systemd/system/faucet.service
    $EDITOR /etc/systemd/system/gauge.service
    systemctl daemon-reload
    systemctl enable faucet.service
    systemctl enable gauge.service
    systemctl restart faucet
    systemctl restart gauge

``/etc/systemd/system/faucet.service`` should contain:

.. literalinclude:: ../etc/systemd/system/faucet.service
  :language: shell
  :caption: faucet.service
  :name: faucet.service

``/etc/systemd/system/gauge.service`` should contain:

.. literalinclude:: ../etc/systemd/system/gauge.service
  :language: shell
  :caption: gauge.service
  :name: gauge.service

Virtual Machine Image
---------------------

We provide a VM image for running FAUCET for development and learning purposes.
The VM comes pre-installed with FAUCET, GAUGE, prometheus and grafana.

Openstack's `diskimage-builder <https://docs.openstack.org/diskimage-builder/latest/>`_
(DIB) is used to build the VM images in many formats (qcow2,tgz,squashfs,vhd,raw).

We provide `DIB elements <elements>`_ for configuring each component installed in the VM.

Pre-built images are available on our build host `<https://builder.faucet.nz>`_.

Building the images
~~~~~~~~~~~~~~~~~~~

If you don't want to use our `pre-built images <https://builder.faucet.nz>`_, you can build them yourself:

1. `Install the latest disk-image-builder <https://docs.openstack.org/diskimage-builder/latest/user_guide/installation.html>`_
2. `Install a patched vhd-util <https://launchpad.net/~openstack-ci-core/+archive/ubuntu/vhd-util>`_
3. Run build-faucet-vm.sh

Security Considerations
~~~~~~~~~~~~~~~~~~~~~~~

This VM is not secure by default, it includes no firewall and has a number of
network services listening on all interfaces with weak passwords. It also
includes a backdoor user (faucet) with weak credentials.

**Services**

The VM exposes a number of ports listening on all interfaces by default:

======================== ====
Service                  Port
======================== ====
SSH                      22
Faucet OpenFlow Channel  6653
Gauge OpenFlow Channel   6654
Grafana Web Interface    3000
Prometheus Web Interface 3000
======================== ====

**Default Credentials**

===================== ======== ========
Service               Username Password
===================== ======== ========
VM TTY Console        faucet   faucet
SSH                   faucet   faucet
Grafana Web Interface admin    admin
===================== ======== ========

Post-Install Steps
~~~~~~~~~~~~~~~~~~

Grafana comes installed but unconfigured, you will need to login to the grafana
web interface at ``http://VM_IP:3000`` and configure a data source and some dashboards.

After logging in with the default credentials shown above, the first step is to add a `prometheus data source <https://prometheus.io/docs/visualization/grafana/#creating-a-prometheus-data-source>`_,
please add ``http://localhost:9090`` as your data source.
Next step is to configure some dashboards, you can add some we have `prepared earlier <https://monitoring.redcables.wand.nz/grafana-dashboards/>`_
or `create your own <http://docs.grafana.org/features/datasources/prometheus/>`_.

You will need to supply your own faucet.yaml and gauge.yaml configuration in the VM.
There are samples provided at /etc/ryu/faucet/faucet.yaml and /etc/ryu/faucet/gauge.yaml.

Finally you will need to point one of the supported OpenFlow vendors at the controller VM,
port 6653 is the Faucet OpenFlow control channel and 6654 is the Gauge OpennFlow control channel for monitoring.
