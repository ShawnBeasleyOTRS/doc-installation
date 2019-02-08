Installation
============

This chapter describes the installation and basic configuration of the central OTRS framework.

Follow the detailed steps in this chapter to install OTRS on your server. You can then use its web interface to login and administer the system.


Preparation: Disable SELinux
----------------------------

.. note::

   If your system uses SELinux, you should disable it, otherwise OTRS will not work correctly.

Here's how to disable SELinux for RHEL/CentOS/Fedora.

1. Configure ``SELINUX=disabled`` in the ``/etc/selinux/config`` file:

   .. code-block:: none

      # This file controls the state of SELinux on the system.
      # SELINUX= can take one of these three values:
      #       enforcing - SELinux security policy is enforced.
      #       permissive - SELinux prints warnings instead of enforcing.
      #       disabled - No SELinux policy is loaded.
      SELINUX=disabled
      # SELINUXTYPE= can take one of these two values:
      #       targeted - Targeted processes are protected,
      #       mls - Multi Level Security protection.
      SELINUXTYPE=targeted

2. Reboot your system. After reboot, confirm that the ``getenforce`` command returns *Disabled*:

   .. code-block:: bash

      root> getenforce
      Disabled


Step 1: Unpack and Install the Application
------------------------------------------

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


Step 2: Install Additional Programs and Perl Modules
----------------------------------------------------

Use the following script to get an overview of all installed and required CPAN modules and other external dependencies.

.. code-block:: none

   root> perl /opt/otrs/bin/otrs.CheckEnvironment.pl
   Checking for Perl Modules:
     o Archive::Tar.....................ok (v1.90)
     o Archive::Zip.....................ok (v1.37)
     o Crypt::Eksblowfish::Bcrypt.......ok (v0.009)
   ...

.. note::

   Please note that OTRS requires a working Perl installation with all *core* modules such as the module ``version``. These modules are not explicitly checked by the script. You may need to install a ``perl-core`` package on some systems like RHEL that do not install the Perl core packages by default.

To install the required and optional packages, you can use either CPAN or the package manager of your Linux distribution.

Execute this command to get an install command to install the missing dependencies:

.. code-block:: bash

   root> /opt/otrs/bin/otrs.CheckEnvironment.pl --list

OTRS requires a supported stable version of Node.js to be installed. Please refer to the `Node.js installation instructions <https://nodejs.org/en/download/package-manager/>`__.


Step 3: Create the OTRS User
----------------------------

Create a dedicated user for OTRS within its own group:

.. code-block:: bash

   root> useradd -r -U -d /opt/otrs -c 'OTRS user' otrs -s /bin/bash


Step 4: Activate the Default Configuration File
-----------------------------------------------

There is an OTRS configuration file bundled in ``$OTRS_HOME/Kernel/Config.pm.dist``. You must activate it by copying it without the ``.dist`` filename extension.

.. code-block:: bash

   root> cp /opt/otrs/Kernel/Config.pm.dist /opt/otrs/Kernel/Config.pm


Step 5: Configure the Apache Web Server
---------------------------------------

OTRS comes with an own built-in web server that is used behind Apache as a reverse proxy (or any other reverse proxy server). A few Apache modules are needed for correct operation: ``proxy_module``, ``proxy_http_module`` and ``proxy_wstunnel_module``.

On some systems like Debian and SuSE, these modules need to be specifically enabled:

.. code-block:: bash

   root> a2enmod proxy
   root> a2enmod proxy_http
   root> a2enmod proxy_wstunnel

Most Apache installations have a ``conf.d`` directory included. On Linux systems you can usually find this directory under ``/etc/apache`` or ``/etc/apache2``. Log in as root, change to the ``conf.d`` directory and
link the appropriate template in ``/opt/otrs/scripts/apache2-httpd.include.conf`` to a file called
``zzz_otrs.conf`` in the Apache configuration directory (to make sure it is loaded after the other configurations).

.. code-block:: bash

   # Debian/Ubuntu:
   root> ln -s /opt/otrs/scripts/apache2-httpd.include.conf /etc/apache2/sites-enabled/zzz_otrs.conf

Now you can restart your web server to load the new configuration settings. On most systems you can do that with the command:

.. code-block:: bash

   root> systemctl restart apache2.service


Step 6: Set File Permissions
----------------------------

Please execute the following command to set the file and directory permissions for OTRS. It will try to detect the correct user and group settings needed for your setup.

.. code-block:: bash

   root> /opt/otrs/bin/otrs.SetPermissions.pl


Step 7: Setup the Database
--------------------------

The following steps need to be taken to setup the database for OTRS properly:

- Create a dedicated database user and database.
- Create the database structure.
- Insert the initial data.
- Configure the database connection in ``Kernel/Config.pm``.

.. note::

   Please note that OTRS requires ``utf8`` as database storage encoding.

MySQL or MariaDB
~~~~~~~~~~~~~~~~

Log in to MySQL console as database admin user:

.. code-block:: bash

   root> mysql -uroot -p

Create a database:

.. code-block:: bash

   mysql> CREATE DATABASE otrs CHARACTER SET utf8;

Special database user handling is needed for MySQL 8, as the default ``caching_sha2_password`` can only be used over secure connections. Create a database user in MySQL 8:

.. code-block:: bash

   mysql> CREATE USER 'otrs'@'localhost' IDENTIFIED WITH mysql_native_password BY 'choose-your-password';

Create a database user in older MySQL versions:

.. code-block:: bash

   mysql> CREATE USER 'otrs'@'localhost' IDENTIFIED BY 'choose-your-password';

Assign user privileges to the new database:

.. code-block:: bash

   mysql> GRANT ALL PRIVILEGES ON otrs.* TO 'otrs'@'localhost';
   mysql> FLUSH PRIVILEGES;
   mysql> quit

Run the following commands on the shell to create schema and insert data:

.. code-block:: bash

   root> mysql -uroot -p otrs < /opt/otrs/scripts/database/otrs-schema.mysql.sql
   root> mysql -uroot -p otrs < /opt/otrs/scripts/database/otrs-initial_insert.mysql.sql
   root> mysql -uroot -p otrs < /opt/otrs/scripts/database/otrs-schema-post.mysql.sql

Configure database settings in ``Kernel/Config.pm``:

.. code-block:: perl

   $Self->{DatabaseHost} = '127.0.0.1';
   $Self->{Database}     = 'otrs';
   $Self->{DatabaseUser} = 'otrs';
   $Self->{DatabasePw}   = 'choose-your-password';
   $Self->{DatabaseDSN}  = "DBI:mysql:database=$Self->{Database};host=$Self->{DatabaseHost};";

.. note::

   The following configuration settings are recommended for MySQL setups. Please add the following lines to ``/etc/my.cnf`` under the ``[mysqld]`` section:

   .. code-block:: ini

      max_allowed_packet   = 64M
      query_cache_size     = 32M
      innodb_log_file_size = 256M


PostgreSQL
~~~~~~~~~~

.. note::

   We assume, that OTRS and PostgreSQL server run on the same machine and PostgreSQL uses *Peer* authentication method. For more information see the `Client Authentication <https://www.postgresql.org/docs/current/client-authentication.html>`__ section in the PostgreSQL manual.

Switch to ``postgres`` user:

.. code-block:: bash

   root> su - postgres

Create a database user:

.. code-block:: bash

   postgres> createuser otrs

Create a database:

.. code-block:: bash

   postgres> createdb --encoding=UTF8 --owner=otrs otrs

Run the following commands on the shell to create schema and insert data:

.. code-block:: bash

   otrs> psql < /opt/otrs/scripts/database/otrs-schema.postgresql.sql
   otrs> psql < /opt/otrs/scripts/database/otrs-initial_insert.postgresql.sql
   otrs> psql < /opt/otrs/scripts/database/otrs-schema-post.postgresql.sql

Configure database settings in ``Kernel/Config.pm``:

.. code-block:: perl

   $Self->{DatabaseHost} = '127.0.0.1';
   $Self->{Database}     = 'otrs';
   $Self->{DatabaseUser} = 'otrs';
   $Self->{DatabasePw}   = 'choose-your-password';
   $Self->{DatabaseDSN}  = "DBI:Pg:dbname=$Self->{Database};host=$Self->{DatabaseHost};";


Finishing the Database Setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To verify your database setup, run the following command:

.. code-block:: none

   otrs> /opt/otrs/bin/otrs.Console.pl Maint::Database::Check
   Trying to connect to database 'DBI:Pg:dbname=otrs;host=localhost' with user 'otrs'...
   Connection successful.

Once the database is configured correctly, please initialize the system configuration with the following command:

.. code-block:: none

   otrs> /opt/otrs/bin/otrs.Console.pl Maint::Config::Rebuild
   Rebuilding the system configuration...
   Done.

.. note::

   For security reasons, please change the default password of the admin user ``root@localhost`` by generating a random password:

   .. code-block:: none

      otrs> /opt/otrs/bin/otrs.Console.pl Admin::User::SetPassword root@localhost
      Generated password 'rtB98S55kuc9'.
      Successfully set password for user 'root@localhost'.

   You can also choose to set your own password:

   .. code-block:: none

      otrs> /opt/otrs/bin/otrs.Console.pl Admin::User::SetPassword root@localhost your-own-password
      Successfully set password for user 'root@localhost'


Step 8: Setup Elasticsearch Cluster
-----------------------------------

OTRS requires an active cluster of Elasticsearch 6.0 or higher. The easiest way is to `setup Elasticsearch <https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html>`__ on the same host as OTRS and binding it to its default port. With that, no further configuration in OTRS is needed.

Additionally, OTRS requires plugins to be installed into Elasticsearch:

.. code-block:: bash

   root> /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch ingest-attachment
   root> /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch analysis-icu

.. note::

   Restart Elasticsearch afterwards, or indexes will not be built.

To verify the Elasticsearch installation, you can use the following command:

.. code-block:: none

   otrs> /opt/otrs/bin/otrs.Console.pl Maint::DocumentSearch::Check
   Trying to connect to cluster...
     Connection successful.


Step 9: Start the OTRS Daemon and Web Server
--------------------------------------------

The new OTRS daemon is responsible for handling any asynchronous and recurring tasks in OTRS. The built-in OTRS web server process handles the web requests handed over from Apache. Both processes must be started from the ``otrs`` user.

.. code-block:: bash

   otrs> /opt/otrs/bin/otrs.Daemon.pl start
   otrs> /opt/otrs/bin/otrs.WebServer.pl


Step 10: First Login
--------------------

Now you are ready to login to your system at http://localhost/otrs/index.pl as user ``root@localhost`` with the password that was generated (see above).

Use http://localhost to access the external interface.


Step 11: Setup Systemd Files
----------------------------

OTRS comes with example systemd configuration files that can be used to make sure that the OTRS daemon and web server are started automatically after the system starts.

.. code-block:: bash

   root> cd /opt/otrs/scripts/systemd
   root> for UNIT in *.service; do cp -vf $UNIT /usr/lib/systemd/system/; systemctl enable $UNIT; done

With this step, the basic system setup is finished.


Step 12: Setup Bash Auto-Completion (optional)
----------------------------------------------

All regular OTRS command line operations happen via the OTRS console interface. This provides an auto completion for the bash shell which makes finding the right command and options much easier.

You can activate the bash auto-completion by installing the package ``bash-completion``. It will automatically detect and load the file ``/opt/otrs/.bash_completion`` for the ``otrs`` user.

After restarting your shell, you can just type this command followed by TAB, and it will list all available commands:

.. code-block:: bash

   otrs> /opt/otrs/bin/otrs.Console.pl

If you type a few characters of the command name, TAB will show all matching commands. After typing a complete command, all possible options and arguments will be shown by pressing TAB.

.. note::

   If you have problems, you can add the following line to your ``~/.bashrc`` to execute the commands from the file.

   .. code-block:: bash

      source /opt/otrs/.bash_completion


Step 13: Further Information
----------------------------

We advise you to read the OTRS :doc:`performance-tuning` chapter.
