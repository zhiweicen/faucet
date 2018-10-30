Docker
======

.. _docker-install:

Installing docker
-----------------

We recommend installing Docker Community Edition (CE) according to the official
`docker engine installation guide <https://docs.docker.com/engine/installation>`_.

Initial configuration
---------------------

.. code:: console

  sudo mkdir -p /etc/faucet
  sudo vi /etc/faucet/faucet.yaml
  sudo vi /etc/faucet/gauge.yaml

See :doc:`installation` and :doc:`configuration` for configuration options.

In particular, see vendor specific docs for additional files that may be
necessary in /etc/faucet to configure the switch pipeline.

Official builds
---------------

We provide official automated builds on Docker Hub so that you can run Faucet
easily without having to build your own.

We use Docker tags to differentiate between versions of Faucet. The latest
tag will always point to the latest stable release of Faucet. All tagged
versions of Faucet in git are also available to use, for example using the
``faucet/faucet:1.8.0`` Docker will run the released version 1.8.0 of Faucet.

By default the Faucet and Gauge images are run as the `faucet` user under
UID 0, GID 0. If you need to change that it can be overridden at runtime with
the Docker flags: ``-e LOCAL_USER_ID`` and ``-e LOCAL_GROUP_ID``.

To pull and run the latest version of Faucet:

.. code:: console

  mkdir -p /var/log/faucet/
  docker pull faucet/faucet:latest
  docker run -d \
      --name faucet \
      --restart=always \
      -v /etc/faucet/:/etc/faucet/ \
      -v /var/log/faucet/:/var/log/faucet/ \
      -p 6653:6653 \
      -p 9302:9302 \
      faucet/faucet

Port 6653 is used for OpenFlow, port 9302 is used for Prometheus - port 9302
may be omitted if you do not need Prometheus.

To pull and run the latest version of Gauge:

.. code:: console

  mkdir -p /var/log/faucet/gauge/
  docker pull faucet/gauge:latest
  docker run -d \
      --name gauge \
      --restart=always \
      -v /etc/faucet/:/etc/faucet/ \
      -v /var/log/faucet/:/var/log/faucet/ \
      -p 6654:6653 \
      -p 9303:9303 \
      faucet/gauge

Port 6654 is used for OpenFlow, port 9303 is used for Prometheus - port 9303
may be omitted if you do not need Prometheus.

Additional Arguments
--------------------

You may wish to run faucet under docker with additional arguments, for example:
setting certificates for an encrypted control channel. This can be done by
overriding the docker entrypoint like so:

.. code:: console

  docker run -d \
      --name faucet \
      --restart=always \
      -v /etc/faucet/:/etc/faucet/ \
      -v /etc/ryu/ssl/:/etc/ryu/ssl/ \
      -v /var/log/faucet/:/var/log/faucet/ \
      -p 6653:6653 \
      -p 9302:9302 \
      faucet/faucet \
      faucet \
      --ctl-privkey /etc/ryu/ssl/ctrlr.key \
      --ctl-cert /etc/ryu/ssl/ctrlr.cert  \
      --ca-certs /etc/ryu/ssl/sw.cert

You can get a list of all additional arguments faucet supports by running:

.. code:: console

  docker run -it faucet/faucet faucet --help

Docker compose
--------------

This is an example docker-compose file that can be used to set up gauge to talk
to Prometheus and InfluxDB with a Grafana instance for dashboards and visualisations.

It can be run with:

.. code:: console

  docker-compose pull
  docker-compose up

The time-series databases with the default settings will write to
``/opt/prometheus/`` ``/opt/influxdb/shared/data/db`` you can edit these locations
by modifying the ``docker-compose.yaml`` file.

On OSX, some of the default shared paths are not accessible, so to overwrite
the location that volumes are written to on your host, export an environment
varible name ``FAUCET_PREFIX`` and it will get prepended to the host paths.
For example:

.. code:: bash

  export FAUCET_PREFIX=/opt/faucet

When all the docker containers are running we will need to configure Grafana to
talk to Prometheus and InfluxDB. First login to the Grafana web interface on
port 3000 (e.g http://localhost:3000) using the default credentials of
``admin:admin``.

Then add two data sources. Use the following settings for prometheus:

::

  Name: Prometheus
  Type: Prometheus
  Url: http://prometheus:9090

And the following settings for InfluxDB:

::

  Name: InfluxDB
  Type: InfluxDB
  Url: http://influxdb:8086
  With Credentials: true
  Database: faucet
  User: faucet
  Password: faucet

Check the connection using test connection.

From here you can add a new dashboard and a graphs for pulling data from the
data sources. Hover over the ``+`` button on the left sidebar in the web
interface and click ``Import``.

We will import the following dashboards, just download the following
links and upload them through the grafana dashboard import screen:

* `Instrumentation <_static/grafana-dashboards/faucet_instrumentation.json>`_
* `Inventory <_static/grafana-dashboards/faucet_inventory.json>`_
* `Port Statistics <_static/grafana-dashboards/faucet_port_statistics.json>`_
