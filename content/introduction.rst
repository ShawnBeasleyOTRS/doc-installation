Introduction
============

OTRS (Open Technology – Real Service) is an open source ticket request system with many features to manage customer telephone calls and emails. It is distributed under the GNU General Public License (GPL) and tested on various Linux platforms. Do you receive many e-mails and want to answer them with a team of agents? You're going to love OTRS!


About This Manual
-----------------

This manual is intended for use by system administrators. The chapters describe the installation and updating of the OTRS software.

There is no graphical user interface for installation and updating. System administrators have to follow the steps described in the following chapters.

All console commands look like ``username> command-to-execute``. Username indicates the user account of the operating system, which need to use to execute the command. If a command starts with ``root>``, you have to execute the command as a user who has root permissions. If a command starts with ``otrs>``, you have to execute the command as the user created for OTRS.

.. warning::

   Don't select ``username>`` when you copy the command and paste it to the shell. Otherwise you will get an error.

We supposed that OTRS will be installed to ``/opt/otrs``. If you want to install OTRS to a different directory, then you have to change the path in the commands or create a symbolic link to this directory.

.. code-block:: bash

   root> ln -s /path/to/otrs /opt/otrs
