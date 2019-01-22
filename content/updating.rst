Updating
========

.. note::

   It is highly recommended to perform a test update on a separate testing machine first.

Updating from an earlier version of OTRS 8
   You can update directly from any previous to the latest available patch level release.

Updating from OTRS 7
   You can update from any OTRS 7 patch level release to the latest available OTRS 8 patch level release.

Updating from OTRS 6 or earlier
   You cannot update from OTRS 6 or earlier directly to OTRS 8. Full updates to all available minor versions have to be made sequentially instead. For example, if you come from OTRS 5, you first have to perform a full update to OTRS 6, then to OTRS 7 and finally to OTRS 8.

   .. seealso::

      See the admin manual of the previous versions of OTRS for the update instructions.


Step 1: Stop All Relevant Services and the OTRS Daemon
------------------------------------------------------

Please make sure there are no more running services or cron jobs that try to access OTRS. This will depend on your service configuration and OTRS version.

.. code-block:: bash

   root> systemctl stop postfix
   root> systemctl stop apache2
   root> systemctl stop otrs-daemon
   root> systemctl stop otrs-webserver


Step 2: Backup Files and Database
---------------------------------

Create a backup of the following files and folders:

- ``Kernel/Config.pm``
- ``Kernel/WebApp.conf`` (only in case of a patch level update of OTRS 8, and only if the file was modified)
- ``var/*``
- as well as the database

.. warning::

   Don't proceed without a complete backup of your system. Use the :ref:`backup` script for this.


Step 3: Install the New Release
-------------------------------

.. note::

   With OTRS 8 RPMs are no longer provided. RPM based installations need to switch by uninstalling the RPM (this will not drop your database) and using the source archives instead.

You can obtain either ``otrs-x.y.z.tar.gz`` or ``otrs-x.y.z.tar.bz2``. Unpack the source archive (for example, using ``tar``) into the directory ``/opt``, and create a symbolic link ``/opt/otrs`` that points to ``/opt/otrs-x.y.z``. **Do not forget** to replace the version numbers!

.. note::

   Package ``bzip2`` is not installed in some systems by default. Make sure, that ``bzip2`` is installed before unpacking ``otrs-x.y.z.tar.bz2``.

Unpack command for ``otrs-x.y.z.tar.gz``:

.. code-block:: bash

   root> tar -xzf otrs-x.y.z.tar.gz -C /opt

Unpack command for ``otrs-x.y.z.tar.bz2``:

.. code-block:: bash

   root> tar -xjf otrs-x.y.z.tar.bz2 -C /opt

It is recommended to create a symbolic link named ``/opt/otrs`` that always points to the latest OTRS version. Using symbolic link makes easy to manage the OTRS updates, because you can leave untouched the directory of the previous version, only the symbolic link needs to change. If you need to revert the update, you can change the target of the symbolic link back.

Execute this command to create a symbolic link:

.. code-block:: bash

   root> ln -fns /opt/otrs-x.y.z /opt/otrs


Restore Old Configuration Files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``Kernel/Config.pm``
- ``Kernel/WebApp.conf``


Restore Article Data
~~~~~~~~~~~~~~~~~~~~

If you configured OTRS to store article data in the file system you have to restore the ``article`` folder to ``/opt/otrs/var/`` or the folder specified in the system configuration.


Restore Already Installed Default Statistics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have additional packages with default statistics you have to restore the stats XML files with the suffix ``*.installed`` to ``/opt/otrs/var/stats``.

.. code-block:: bash

   root> cd OTRS-BACKUP/var/stats
   root> cp *.installed /opt/otrs/var/stats


Set File Permissions
~~~~~~~~~~~~~~~~~~~~

Please execute the following command to set the file and directory permissions for OTRS. It will try to detect the correct user and group settings needed for your setup.

.. code-block:: bash

   root> /opt/otrs/bin/otrs.SetPermissions.pl


Install Required Programs and Perl Modules
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please refer to the section :ref:`Step 2: Install Additional Programs and Perl Modules` in the installation guide that explains how to verify external dependencies such as Perl modules and Node.js.

In addition to that, OTRS 8 also requires an active cluster of Elasticsearch 6.0 or higher. Please refer to the :ref:`Step 8: Setup Elasticsearch Cluster` section in the installation guide.


Step 4: Run the Migration Script
--------------------------------

The migration script will perform many checks on your system and give you advice on how to install missing Perl modules etc., if that is required. If all checks succeeded, the necessary migration steps will be performed. Please also run this script in case of patch level updates.

Run the migration script:

.. code-block:: bash

   otrs> /opt/otrs/scripts/DBUpdate-to-8.pl

.. warning::

   Do not continue the upgrading process if this script did not work properly for you. Otherwise malfunction or data loss may occur.


Step 5: Update Installed Packages
---------------------------------

.. note::

   Packages for OTRS 7 are not compatible with OTRS 8 and have to be updated.

You can use the command below to update all installed packages. This works for all packages that are available from online repositories. You can update other packages later via the package manager (this requires a running OTRS daemon).

.. code-block:: bash

   otrs> /opt/otrs/bin/otrs.Console.pl Admin::Package::UpgradeAll


Step 6: Restart your Services
-----------------------------

.. code-block:: bash

   root> systemctl stop postfix
   root> systemctl stop apache2

.. note::

   The OTRS daemon is required for correct operation of OTRS such as sending emails. Please activate it as described in the next step.


Step 7: Start the OTRS Daemon, Web Server and Cron Job
------------------------------------------------------

OTRS comes with example systemd configuration files that can be used to make sure that the OTRS daemon and web server are started automatically after the system starts.

.. code-block:: bash

   root> cd /opt/otrs/scripts/systemd
   root> for UNIT in *.service; do cp -vf $UNIT /usr/lib/systemd/system/; systemctl enable $UNIT; done
   root> systemctl start otrs-daemon
   root> systemctl start otrs-webserver

Now you can log into your system.


Step 8: Manual Migration Tasks and Changes
------------------------------------------

.. warning::

   This step is required only for major update from OTRS 7.

...
