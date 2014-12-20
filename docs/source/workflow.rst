Cooking: How it all comes together
==================================

Build and deploy workflow
-------------------------
`These slides
<https://docs.google.com/a/yelp.com/presentation/d/1mtWoJUVevBrI7I2iCvZRiqKcLZudYLtrLV8kTkdP0jI/edit#>`_
provide a high level overview of the ingredients involved.

Cluster configuration
---------------------
Puppet does the server configuration work: installing packages, configuring
Mesos, scheduling crons to run the deployment scripts, etc. See the
`profile_mesos module
<https://opengrok.yelpcorp.com/xref/sysgit/puppet/modules/profile_mesos/>`_.

Service configuration
---------------------
`CEP 319 <http://y/cep319>`_ discusses how yelpsoa-configs are distributed to
``/nail/etc/services`` on machines in the cluster.

Contract with services
----------------------
The `Paasta Contract <http://y/paasta-contract>`_ describes the
responsibilities of services that wish to work with PaaSTA.

service_deployment_tools contains the implementation of several of these rules.
For example, `generate_deployments_json <generate_deployments_json.html>`_ is
the piece that checks each service's git repo for the specially-named branch
that tells PaaSTA which versions of the service should go to which clusters.

Deployment
----------
A yelpsoa-configs master runs `generate_deployments_json
<generate_deployments_json.html>`_ frequently. The generated
``deployments.json`` appears in ``/nail/etc/services`` throughout the cluster.

Marathon masters run `deploy_marathon_services
<deploy_marathon_services.html>`_, a thin wrapper around `setup_marathon_job
<setup_marathon_job.html>`_. These scripts parse ``deployments.json`` and the
current cluster state, then issue comands to Marathon to put the cluster into
the right state -- cluster X should be running version Y of service Z.

How PaaSTA runs Docker images
-----------------------------
Marathon launches the Docker containers that comprise a PaaSTA service. The
default configuration is managed by puppet in the `service_deployment_tools
module
<https://opengrok.yelpcorp.com/xref/sysgit/puppet/modules/service_deployment_tools/manifests/init.pp>`_.

Docker images are run by Mesos's native Docker executor. PaaSTA composes the
configuration for the running image:

* ``--attach``: stdout and stderr from running images are sent to logs that end
  up in the Mesos sandbox (currently unavailable).

* ``--cpu-shares``: This is the value set in ``marathon.yaml`` as "cpus".

* ``--memory``: This is the value set in ``marathon.yaml`` as "mem".

* ``--net``: PaaSTA uses bridge mode to enable random port allocation.

* ``--publish``: Mesos picks a random port on the host that maps to and exposes
  port 8888 inside the container. This random port is announced to Smartstack
  so that it can be used for load balancing.

* ``--privileged``: Containers run by PaaSTA are not privileged.

* ``--restart``: No restart policy is set on PaaSTA containers. Restarting
  tasks is left as a job for the Framework (Marathon).

* ``--rm``: Mesos containers are rm'd after they finish.

* ``--tty``: Mesos containers are *not* given a tty.

* ``--volume``: Volume mapping is controlled via the service_deployment_tools
  configuration. This is not user-controlled for security reasons. The default
  mappings include common configuration folders (like `srv-configs
  <https://trac.yelpcorp.com/wiki/HowToService/Configuration>`_) and key files
  in ``/nail/etc`` (``habitat``, ``ecosystem``, etc).

* ``--workdir``: Mesos containers are launched in a temporary "workspace"
  directory on disk. Use the workdir sparingly and try not to output files.

Monitoring
----------
`check_marathon_services_replication <check_marathon_services_replication.html>`_
runs periodically and sends an alert if the actual state of the cluster does
not match the desired state.

Cleanup
-------
`cleanup_marathon_jobs <cleanup_marathon_jobs.html>`_ gets rid of Marathon jobs
that don't need to be running anymore. This should be rare, like if you change
a service's name or manually delete a ``paasta-[clustername]`` git branch, but
is a useful safety net in case a task escapes.

`cleanup_marathon_orphaned_containers
<cleanup_marathon_orphaned_containers.html>`_ reaps containers that get lost in
the shuffle when we restart Mesos slaves too hard.