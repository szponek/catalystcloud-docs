.. _launching-your-first-instance-using-ansible:

*******************************************
Launching your first instance using Ansible
*******************************************

`Ansible`_ is a popular open source configuration management and application
deployment tool. Ansible provides a set of core modules for interacting with
OpenStack. This makes Ansible an ideal tool for providing both OpenStack
orchestration and instance configuration, letting you use a single tool to
setup the underlying infrastructure and configure instances. As such Ansible
can substitute for Heat for OpenStack orchestration and Puppet for instance
configuration.

.. _Ansible: http://www.ansible.com/

Comprehensive documentation of the Ansible OpenStack modules is available at
https://docs.ansible.com/ansible/list_of_cloud_modules.html#openstack

Install Ansible
===============

We have written a script to install the required Ansible and OpenStack
libraries within a Python virtual environment. This script is part of the
`catalystcloud-ansible`_ git repository. Lets clone this repository and run the
install script in order to install Ansible.

.. _catalystcloud-ansible: https://github.com/catalyst/catalystcloud-ansible

.. code-block:: bash

 $ git clone https://github.com/catalyst/catalystcloud-ansible.git && CC_ANSIBLE_DIR="$(pwd)/catalystcloud-ansible" && echo $CC_ANSIBLE_DIR
 $ cd catalystcloud-ansible
 $ ./install-ansible.sh
 Installing stable version of Ansible
 ...
 $ source $CC_ANSIBLE_DIR/ansible/bin/activate
 $ ansible --version
 ansible 2.0.1.0
   config file = /etc/ansible/ansible.cfg
   configured module search path = Default w/o overrides

.. note::

  Catalyst recommends customers use Ansible >= 2.0 and Shade >= 1.4 with the
  Catalyst Cloud.

OpenStack credentials
=====================

Before we can run the playbooks we need to setup our OpenStack credentials, the
easiest way to achieve this is to make use of environment variables. We will
make use of the standard variables provided by an OpenStack RC file as
described at :ref:`source-rc-file`. These variables are read by the Ansible
``os_auth`` module to provide Ansible with permissions to access the Catalyst
Cloud APIs.

.. note::

 If you do not source an OpenStack RC file, you will need to set a few
 mandatory authentication attributes in the playbooks. See the vars section of
 the playbooks for details.

Now we have an Ansible installation that includes the required OpenStack
modules and have setup our OpenStack credentials we can proceed to build our
first instance. We have split the first instance playbooks into two playbook
files, the first playbook ``create-network.yml`` creates the required network
components and the second playbook ``launch-instance.yml`` launches the
instance.

Run the create network playbook
===============================

Lets take a look at what tasks the create network playbook is going to
complete:

.. code-block:: bash

 $ grep ' \- name' create-network.yml
     - name: Connect to the Catalyst Cloud
     - name: Create a network
     - name: Create a subnet
     - name: Create a router
     - name: Create a security group
     - name: Create a security group rule for SSH access
     - name: Import an SSH keypair

We are going to need to provide the path to a valid SSH key in order for this
playbook to work. You can edit ``create-network.yml`` and update the
``ssh_public_key`` variable, or we can override the variable when we run the
playbook as shown below:

.. note::

 Bash does not perform tilde expansion within quotes so you need to provide the
 full path to your SSH public key in the ssh_public_key extra var below.

.. code-block:: bash

 $ ansible-playbook --extra-vars 'ssh_public_key=/home/youruser/.ssh/id_rsa.pub' create-network.yml

 PLAY [Deploy a cloud instance in OpenStack] ************************************

 TASK [setup] *******************************************************************
 ok: [localhost]

 TASK [Connect to the Catalyst Cloud] *******************************************
 ok: [localhost]

 TASK [Create a network] ********************************************************
 changed: [localhost]

 TASK [Create a subnet] *********************************************************
 changed: [localhost]

 TASK [Create a router] *********************************************************
 changed: [localhost]

 TASK [Create a security group] *************************************************
 changed: [localhost]

 TASK [Create a security group rule for SSH access] *****************************
 changed: [localhost]

 TASK [Import an SSH keypair] ***************************************************
 changed: [localhost]

 PLAY RECAP *********************************************************************
 localhost                  : ok=8    changed=6    unreachable=0    failed=0


Run the launch instance playbook
================================

Now we have a network setup we can run the launch instance playbook:

.. code-block:: bash

 $ ansible-playbook launch-instance.yml

 PLAY [Deploy a cloud instance in OpenStack] ************************************

 TASK [setup] *******************************************************************
 ok: [localhost]

 TASK [Connect to the Catalyst Cloud] *******************************************
 ok: [localhost]

 TASK [Create a compute instance on the Catalyst Cloud] *************************
 changed: [localhost]

 TASK [Assign a floating IP] ****************************************************
 changed: [localhost]

 TASK [Output floating IP] ******************************************************
 ok: [localhost] => {
     "floating_ip_info.floating_ip.floating_ip_address": "150.242.41.75"
 }

 PLAY RECAP *********************************************************************
 localhost                  : ok=4    changed=2    unreachable=0    failed=1

We can now connect to our new instance via SSH using the IP address output by
the ``Output floating IP`` task:

.. code-block:: bash

 $ ssh ubuntu@150.242.41.75

We can now write playbooks to configure the instance we have created as
required.
