Preparation: service_deployment_tools and yelpsoa-configs
=========================================================

service_deployment_tools reads configuration about services from several YAML
files in `yelpsoa-configs <http://y/cep319>`_:

marathon-[clustername].yaml
---------------------------

e.g. ``marathon-norcal-prod.yaml``, ``marathon-mesosstage.yaml``. The
clustername is usually the same as the ``superregion`` in which the cluster
lives (``norcal-prod``), but not always (``mesosstage``).

The yaml where marathon jobs are actually defined.

Top level keys are namespaces, e.g. ``main`` and ``canary``. Each namespace MAY have:

  * ``cpu``: Number of CPUs an instance needs

  * ``mem``: Memory (in MB) an instance needs

  * ``instances``: Marathon will attempt to run this many instances of the Service

  * ``nerve_ns``: Specifies that this namespace should be routed to by another namespace. E.g. ``canary`` instances have a different configuration but traffic from the ``main`` pool reaches them.

  * ``bounce_method``: future placeholder; see `bounce_lib <bounce_lib.html>`_

  * ``constraints``: future placeholder; see https://github.com/mesosphere/marathon/wiki/Constraints

  * ``args``: docker args if you use the "entrypoint" functionality

smartstack.yaml
---------------

The yaml where nerve namespaces are defined and bound to ports.

See `CEP 319 <http://y/cep319>`_

monitoring.yaml
---------------

The yaml where monitoring for the service is defined.

See `the wiki
<https://trac.yelpcorp.com/wiki/HowToService/Monitoring/monitoring.yaml>`_


Where does service_deployment_tools look for yelpsoa-configs?
-------------------------------------------------------------

By default, service_deployment_tools uses the system yelpsoa-configs dir,
``/nail/etc/services``. Scripts should allow this to be overridden with ``-d``
or ``--soa-dir``. Normally you would only do this for testing or debugging.