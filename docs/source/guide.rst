.. _ug:

User's guide
############

.. contents:: Table of Contents
   :local:
   :depth: 2

.. _ug_introduction:

Introduction
************

The goal of this configuration is to start *dhclient* and other system services (e.g. *routing*,
*ntpdate*, *ntpd*, ...) after the wifi interface connects to the network. The utility *wpa_cli*,
running in the background, will be notified by *wpa_supplicant* when the interface connects or
disconnects to/from the network. On such event *wpa_cli* executes the action file (-a
action_file). See `templates`_ what pre-configured scripts are available. For example,
`wpa_action-1.1.0.sh`_, after the connection, starts *dhclient*, restarts *routing*, and optionally
synchronizes date and time. This solves the potential problem of synchronizing date and time by
*settimeofday* at boot time of a wireless-only system. If *wpa_supplicant* doesn't manage to connect
to the network by the time *ntpdate* is executed *ntpdate* will time-out. Then, in most systems, the
*ntpd* service will start (see `rcorder /etc/rc.d/*`). When the hardware device has no battery and
no RTC, the offset might be huge. In this case `ntpd will reject the offset and will terminate
itself, believing something very strange must have happened`_.

* Ansible role: `freebsd_wpa_cli`_
* Supported systems: `FreeBSD`_
* Requirements: None

.. warning:: The role modifies scripts in /etc/rc.d

.. _ug_installation:

Installation
************

The most convenient way how to install an Ansible role is to use :index:`Ansible Galaxy` CLI
``ansible-galaxy``. The utility comes with the standard Ansible package and provides the user with a
simple interface to the Ansible Galaxy's services. For example, take a look at the current status of
the role ::

   shell> ansible-galaxy role info vbotka.freebsd_wpa_cli

and install it ::

    shell> ansible-galaxy role install vbotka.freebsd_wpa_cli

Install the collections `community.general`_ and `ansible.posix`_  ::

    shell> ansible-galaxy collection install ansible.posix
    shell> ansible-galaxy collection install community.general

.. seealso::
   * See `freebsd_postinstall`_
   * See `wpasupplicant.yml`_ how to configure wpa_supplicant by Ansible
   * To install specific versions from various sources see `Installing content`_.
   * Take a look at other roles ``shell> ansible-galaxy search --author=vbotka``

.. _ug_playbook:

Playbook
********

Create simple playbook that calls the role (10) at a single host srv.example.com (2)

.. code-block:: bash
   :emphasize-lines: 2,10
   :linenos:

   shell> cat playbook.yml
   - hosts: srv.example.com
     gather_facts: true
     connection: ssh
     remote_user: admin
     become: true
     become_user: root
     become_method: sudo
     roles:
       - vbotka.freebsd_wpa_cli

.. seealso::
   * For details see `Connection Plugins`_ (4-5)
   * See also `Understanding Privilege Escalation`_ (6-8)

.. _ug_debug:

Debug
*****

Some tasks will display additional information when the variable :index:`wpacli_debug` is
enabled. Enable debug output either in the configuration ::

  wpacli_debug: true

, or set the extra variable in the command ::

  shell> ansible-playbook playbook.yml -e wpacli_debug=true

.. hint::

   The debug output of this role is optimized for ``result_format=yaml``. See
   `result_format`_. Set it in the configuration

   .. code:: ini

      [defaults]
      callback_result_format = yaml

   , or in the environment

   .. code:: bash

      ANSIBLE_CALLBACK_RESULT_FORMAT=yaml

.. seealso::

   * `Playbook Debugger`_

.. _ug_tags:

Tags
****

The :index:`tags` provide the user with a very useful tool to run selected tasks of the role. To see
what tags are available list the tags of the role with the command

.. include:: tags-list.rst

Examples
========

* Display the list of the variables and their values uding the tag ``wpacli_debug`` (when the
  :index:`debug` is enabled ``wpacli_debug: true``) ::

    shell> ansible-playbook playbook.yml -t wpacli_debug

* See what packages will be installed ::

    shell> ansible-playbook playbook.yml -t wpacli_packages --check

* Install packages and exit the play ::

    shell> ansible-playbook playbook.yml -t wpacli_packages

.. _ug_tasks:

Tasks
*****

<TODO>

.. seealso::

   * Source code :ref:`as_main.yml`

Development and testing
=======================

In order to deliver an Ansible project it's necessary to test the code and configuration. The tags
provide the administrators with a tool to test single tasks' files and single tasks. For example, to
test a single task at a single remote host *test_01*, create the playbook

.. code-block:: yaml
   :emphasize-lines: 1

   shell> cat playbook.yml
   - hosts: test_01
     become: true
     roles:
       - vbotka.freebsd_wpa_cli

Customize configuration in ``host_vars/test_01/wpa-cli.yml`` and :index:`check the syntax`

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml --syntax-check

Then :index:`dry-run` the selected task and see what will be changed. Replace <tag> with valid tag
or with a comma-separated list of tags

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml -t <tag> --check --diff

When all seems to be ready run the command. Run the command twice and make sure the playbook and the
configuration is :index:`idempotent`

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml

.. _ug_task_wpacli:
.. include:: task-wpacli.rst

.. _ug_task_rc:
.. include:: task-rc.rst

.. _ug_vars:

Variables
*********

.. seealso::

   * `Ansible variable precedence. Where should I put a variable?`_

.. _ug_vars_defaults:
.. include:: vars-defaults.rst

.. _ug_bp:

Best practice
*************

Test the syntax

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml --syntax-check

See what variables will be used

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml -t wpacli_debug -e wpacli_debug=true

:index:`Dry-run`, display differences and display variables

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml -e wpacli_debug=true --check --diff

Install packages

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml -t wpacli_packages -e wpacli_install=true

Run the playbook

.. code-block:: sh
   :emphasize-lines: 1

   shell> ansible-playbook playbook.yml

Test the :index:`idem-potency`. The role and the configuration data shall be idempotent. Once the
installation and configuration have passed there should be no changes reported by *ansible-playbook*
when running the playbook repeatedly. Disable debug, and install to speedup the playbook and run the
playbook again.

.. code-block:: sh
   :emphasize-lines: 1

    shell> ansible-playbook playbook.yml

.. _templates: https://github.com/vbotka/ansible-freebsd-wpa-cli/tree/master/templates
.. _wpa_action-1.1.0.sh: https://raw.githubusercontent.com/vbotka/ansible-freebsd-wpa-cli/master/templates/wpa_action-1.1.0.sh.j2
.. _ntpd will reject the offset and will terminate itself, believing something very strange must have happened: http://www.ntp.org/ntpfaq/NTP-s-algo.htm#Q-ALGO-BASIC-STEP-SLEW)
.. _freebsd_wpa_cli: https://galaxy.ansible.com/vbotka/freebsd_wpa_cli
.. _FreeBSD: https://www.freebsd.org/releases

.. _community.general: https://docs.ansible.com/ansible/latest/collections/community/general
.. _ansible.posix: https://docs.ansible.com/ansible/latest/collections/ansible/posix/index.html#plugins-in-ansible-posix
.. _freebsd_postinstall: https://galaxy.ansible.com/vbotka/freebsd_postinstall
.. _wpasupplicant.yml: https://raw.githubusercontent.com/vbotka/ansible-freebsd-postinstall/master/tasks/wpasupplicant.yml
.. _Installing content: https://galaxy.ansible.com/docs/using/installing.html

.. _Connection Plugins: https://docs.ansible.com/ansible/latest/plugins/connection.html
.. _Understanding Privilege Escalation: https://docs.ansible.com/ansible/latest/user_guide/become.html#understanding-privilege-escalation

.. _result_format: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/default_callback.html#parameter-result_format
.. _Playbook Debugger: https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html

.. _Ansible variable precedence. Where should I put a variable?: <https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable
