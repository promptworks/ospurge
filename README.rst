OpenStack project resources cleaner
===================================

* ``ospurge`` is a standalone, client-side, operators tool that aims at
  deleting all resources, taking into account their interdependencies,
  in a specified OpenStack project.

* ``ospurge`` ensures in a quick and automated way that no resource is
  left behind when a project is deleted.

* ``ospurge`` can be used by a cloud administrator, this means a user with the
  admin role, to cleanup any project or by a non-privileged user to cleanup his
  own project.


Supported resources
-------------------

At the moment it is possible to purge the following resources from a project:

* Ceilometer alarms
* floating IP addresses
* images / snapshots
* instances
* networks
* routers
* security groups
* Swift containers
* Swift objects
* volumes / snapshots


Error codes
-----------

The following error codes are returned when ``ospurge`` encounters
an error:

* ``Code 0``: Process exited sucessfully
* ``Code 1``: Unknown error
* ``Code 2``: Project doesn't exist
* ``Code 3``: Authentication failed (e.g. bad username or password)
* ``Code 4``: Resource deletion failed
* ``Code 5``: Connection error while deleting a resource (e.g. service not
  available)
* ``Code 6``: Connection to endpoint failed (e.g. wrong authentication URL)


Installation
------------

Create a Python virtual environment (requires the
`virtualenvwrapper <https://virtualenvwrapper.readthedocs.org/>`_):

.. code-block:: console

    $ mkvirtualenv ospurge

Install ``ospurge`` with ``pip``:

.. code-block:: console

    $ pip install ospurge

Available options can be displayed by using ``ospurge -h``:

.. code-block:: console

    $ ospurge -h
    usage: ospurge [-h] [--verbose] [--dry-run] [--dont-delete-project]
                   [--region-name REGION_NAME] [--endpoint-type ENDPOINT_TYPE]
                   --username USERNAME --password PASSWORD --admin-project
                   ADMIN_PROJECT [--admin-role-name ADMIN_ROLE_NAME] --auth-url
                   AUTH_URL [--cleanup-project CLEANUP_PROJECT] [--own-project]
                   [--insecure]

    Purge resources from an Openstack project.

    optional arguments:
      -h, --help            show this help message and exit
      --verbose             Makes output verbose
      --dry-run             List project's resources
      --dont-delete-project
                            Executes cleanup script without removing the project.
                            Warning: all project resources will still be deleted.
      --region-name REGION_NAME
                            Region to use. Defaults to env[OS_REGION_NAME] or None
      --endpoint-type ENDPOINT_TYPE
                            Endpoint type to use. Defaults to
                            env[OS_ENDPOINT_TYPE] or publicURL
      --username USERNAME   If --own-project is set : a user name with access to
                            the project being purged. If --cleanup-project is set
                            : a user name with admin role in project specified in
                            --admin-project. Defaults to env[OS_USERNAME]
      --password PASSWORD   The user's password. Defaults to env[OS_PASSWORD].
      --admin-project ADMIN_PROJECT
                            Project name used for authentication. This project
                            will be purged if --own-project is set. Defaults to
                            env[OS_TENANT_NAME].
      --admin-role-name ADMIN_ROLE_NAME
                                Name of admin role. Defaults to 'admin'.
      --auth-url AUTH_URL   Authentication URL. Defaults to env[OS_AUTH_URL].
      --cleanup-project CLEANUP_PROJECT
                            ID or Name of project to purge. Not required if --own-
                            project has been set. Using --cleanup-project requires
                            to authenticate with admin credentials.
      --own-project         Delete resources of the project used to authenticate.
                            Useful if you don't have the admin credentials of the
                            platform.
      --insecure            Explicitly allow all OpenStack clients to perform
                            insecure SSL (https) requests. The server's
                            certificate will not be verified against any
                            certificate authorities. This option should be used
                            with caution.


Example usage
-------------

To remove a project, credentials have to be
provided. The usual OpenStack environment variables can be used. When
launching the ``ospurge`` script, the project to be cleaned up has
to be provided, by using either the ``--cleanup-project`` option or the
``--own-project`` option. When the command returns, any resources associated
to the project will have been definitively deleted.

* Setting OpenStack credentials:

.. code-block:: console

    $ export OS_USERNAME=admin
    $ export OS_PASSWORD=password
    $ export OS_TENANT_NAME=admin
    $ export OS_AUTH_URL=http://localhost:5000/v2.0

* Checking resources of the target project:

.. code-block:: console

    $ ./ospurge --dry-run --cleanup-project demo
    * Resources type: CinderSnapshots

    * Resources type: NovaServers
    server vm0 (id 8b0896d9-bcf3-4360-824a-a81865ad2385)

    * Resources type: NeutronFloatingIps

    * Resources type: NeutronInterfaces

    * Resources type: NeutronRouters

    * Resources type: NeutronNetworks

    * Resources type: NeutronSecgroups
    security group custom (id 8c13e635-6fdc-4332-ba19-c22a7a85c7cc)

    * Resources type: GlanceImages

    * Resources type: SwiftObjects

    * Resources type: SwiftContainers

    * Resources type: CinderVolumes
    volume vol0 (id ce1380ef-2d66-47a2-9dbf-8dd5d9cd506d)

    * Resources type: CeilometerAlarms

* Removing resources without deleting the project:

.. code-block:: console

    $ ./ospurge --verbose --dont-delete-project --cleanup-project demo
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone.usr.lab0.aub.cw-labs.net
    INFO:root:* Granting role admin to user e7f562a29da3492baba2cc7c5a1f2d84 on project demo.
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone-admin.usr.lab0.aub.cw-labs.net
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone-admin.usr.lab0.aub.cw-labs.net
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone-admin.usr.lab0.aub.cw-labs.net
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone.usr.lab0.aub.cw-labs.net
    INFO:root:* Purging CinderSnapshots
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone.usr.lab0.aub.cw-labs.net
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): cinder.usr.lab0.aub.cw-labs.net
    INFO:root:* Purging NovaServers
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone.usr.lab0.aub.cw-labs.net
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): nova.usr.lab0.aub.cw-labs.net
    INFO:root:* Deleting server vm0 (id 8b0896d9-bcf3-4360-824a-a81865ad2385).
    INFO:root:* Purging NeutronFloatingIps
    INFO:root:* Purging NeutronInterfaces
    INFO:root:* Purging NeutronRouters
    INFO:root:* Purging NeutronNetworks
    INFO:root:* Purging NeutronSecgroups
    INFO:root:* Deleting security group custom (id 8c13e635-6fdc-4332-ba19-c22a7a85c7cc).
    INFO:root:* Purging GlanceImages
    INFO:root:* Purging SwiftObjects
    INFO:root:* Purging SwiftContainers
    INFO:root:* Purging CinderVolumes
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): keystone.usr.lab0.aub.cw-labs.net
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): cinder.usr.lab0.aub.cw-labs.net
    INFO:root:* Deleting volume vol0 (id ce1380ef-2d66-47a2-9dbf-8dd5d9cd506d).
    INFO:requests.packages.urllib3.connectionpool:Starting new HTTP connection (1): cinder.usr.lab0.aub.cw-labs.net
    INFO:root:* Purging CeilometerAlarms

* Checking that resources have been correctly removed:

.. code-block:: console

    $ ./ospurge --dry-run --cleanup-project demo
    * Resources type: CinderSnapshots

    * Resources type: NovaServers

    * Resources type: NeutronFloatingIps

    * Resources type: NeutronInterfaces

    * Resources type: NeutronRouters

    * Resources type: NeutronNetworks

    * Resources type: NeutronSecgroups

    * Resources type: GlanceImages

    * Resources type: SwiftObjects

    * Resources type: SwiftContainers

    * Resources type: CinderVolumes

    * Resources type: CeilometerAlarms

* Removing project:

.. code-block:: console

    $ ./ospurge --cleanup-project demo
    $ ./ospurge --cleanup-project demo
    Project demo doesn't exist

* Example using this fork (where 'url' is the endpoint-type):

.. code-block:: console

    $ ospurge --verbose --cleanup-project purgeme_746 --endpoint-type url

    $ ospurge --cleanup-project=purgeme_130 --endpoint-type=url --verbose
    INFO:root:* Granting role admin to user 53de9bfd6a6d4ce8ac6bd3ca576f2983 on project 6d72e53cebce4b529bf5af04c2db9e43.
    INFO:root:* Purging CinderSnapshots
    INFO:root:* Deleting snapshot (none) (id 552e1314-8d13-4eeb-820a-32870dac2045).
    INFO:root:* Deletion failed - Retrying in 5 seconds - Retry count 1
    INFO:root:* Deleting snapshot (none) (id 552e1314-8d13-4eeb-820a-32870dac2045).
    INFO:root:* Deletion failed - Retrying in 5 seconds - Retry count 2
    INFO:root:* Deleting snapshot (none) (id 552e1314-8d13-4eeb-820a-32870dac2045).
    Deletion failed, but continuing on.
    INFO:root:* Purging CinderBackups
    INFO:root:* Purging NeutronFireWall
    INFO:root:* Deleting Firewall ospurge_test_firewall_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id b7b69573-a4ad-45f6-8150-8f4a9adc313b).
    INFO:root:* Purging NeutronFireWallPolicy
    INFO:root:* Deleting Firewall policy ospurge_test_policy_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id d76a3264-98bb-40b8-95fb-d81cc7e64011).
    INFO:root:* Deletion failed - Retrying in 5 seconds - Retry count 1
    INFO:root:* Deleting Firewall policy ospurge_test_policy_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id d76a3264-98bb-40b8-95fb-d81cc7e64011).
    INFO:root:* Purging NeutronFireWallRule
    INFO:root:* Deleting Firewall rule ospurge_test_rule_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id 4a4b1c1f-0ae8-4d3e-935f-9f55ed7abb0e).
    Skipping neutron resource
    Skipping neutron resource
    Skipping neutron resource
    Skipping neutron resource
    INFO:root:* Purging NovaServers
    INFO:root:* Deleting server ospurge_test_vm_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id 66de6d01-f6de-4c43-94ef-4162f2972565).
    INFO:root:* Purging NeutronFloatingIps
    INFO:root:* Deleting floating ip 192.168.0.206 (id 56175347-ecfe-487d-b6f8-45bf22a6a90e).
    INFO:root:* Purging NeutronMeteringLabel
    INFO:root:* Deleting meter-label ospurge_test_meter_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id e096d2df-f187-4939-b766-73364787f1d6).
    INFO:root:* Purging NeutronInterfaces
    INFO:root:* Purging NeutronRouters
    INFO:root:* Deleting router ospurge_test_rout_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id 17dc995c-0252-4ba4-bb30-b9b99727e62c).
    INFO:root:* Purging NeutronPorts
    INFO:root:* Deleting port  (id 7779bc2a-61e3-44f2-afc2-7e6a38f5885f).
    INFO:root:* Purging NeutronNetworks
    INFO:root:* Deleting network ospurge_test_net_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id 64a57c9b-fa82-4059-9056-399899df3794).
    INFO:root:* Purging NeutronSecgroups
    INFO:root:* Deleting security group ospurge_test_secgroup_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id aa5639ce-ff09-45c3-bf8d-323fb85bdd2d).
    INFO:root:* Purging GlanceImages
    INFO:root:* Deleting image ospurge_test_image_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8 (id f00fd42b-6750-4ac8-8b4d-bf759861d5e9).
    INFO:root:* Purging SwiftObjects
    INFO:root:* Purging SwiftContainers
    INFO:root:* Purging CinderVolumes
    INFO:root:* Deleting volume (none) (id c228b3fa-25c1-46c0-90fc-d09f95fcf42d).
    INFO:root:* Purging CeilometerAlarms
    INFO:root:* Deleting alarm ospurge_test_alarm_e0fca56c-84e7-43bb-b7c2-b150c11ef5f8.
    INFO:root:* Purging HeatStacks
    exception (continuing anyway): ERROR: Access was denied to this resource.
    INFO:root:* Deleting project 6d72e53cebce4b529bf5af04c2db9e43.

* Users can be deleted by using the ``python-openstackclient`` command-line
  interface:

.. code-block:: console

   $ openstack user delete <user>


How to contribute
-----------------

OSpurge is hosted on the OpenStack infrastructure and is using
`Gerrit <https://review.openstack.org>`_ to manage contributions. You can
contribute to the project by following the
`OpenStack Development workflow <http://docs.openstack.org/infra/manual/developers.html#development-workflow>`_.
