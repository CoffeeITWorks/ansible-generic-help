Testing for developers
======================

https://molecule.readthedocs.io/en/latest/

References:

https://werner-dijkerman.nl/2017/09/05/using-molecule-v2-to-test-ansible-roles/amp/

https://dzone.com/articles/ansible-tdd-development-using-molecule?utm_medium=feed&utm_source=feedpress.me&utm_campaign=Feed%253A%20dzone%252Fdevops

http://molecule.readthedocs.io/en/latest/porting.html

The following examples are to test with molecule, but you can also use your own servers and setup an [Ansible inventory](http://docs.ansible.com/ansible/intro_inventory.html)

Molecule resolves some good things for a dev environment, like: automatic provision new hosts, automatic inventory for ansible, automatic deploy, automatic destroy, automatic test idempotence, etc. 

* Install molecule

    pip install --user molecule
    pip install --user docker

* Install docker to use with docker driver.

* On fedora/centos also install:

   sudo yum install libselinux-python

Also stop firewall and start docker on Fedora:

  sudo systemctl stop firewalld
  sudo systemctl start docker

Testing with molecule+docker: 
-----------------------------

Install docker-engine and requirements shown at: https://molecule.readthedocs.io/en/latest/driver/index.html#usage

Example `molecule/default/molecule.yml` file working with centos/systemd, debian/8, Ubuntu/latest: 

Those files are automatically created with: https://molecule.readthedocs.io/en/latest/usage.html#init

```yaml

---
---
dependency:
  name: galaxy
  options:
    ignore-certs: True
    ignore-errors: True
    # role-file: dev_requirements.yml  # this file is at the root of the git project same place as molecule is executed
driver:
  name: docker
platforms:

  - name: ansible_burp_reports-01
    image: "geerlingguy/docker-ubuntu1804-ansible:latest"
    #privileged: True
    command: /sbin/init
    pre_build_image: true
    capabilities:
      - SYS_ADMIN
    tmpfs:
      - /run
      - /tmp
    volumes:
      - "/sys/fs/cgroup:/sys/fs/cgroup:ro"
    groups:
      - group1

  - name: ansible_burp_reports-02
    image: "geerlingguy/docker-debian9-ansible"
    #privileged: True
    command: /sbin/init
    pre_build_image: true
    capabilities:
      - SYS_ADMIN
    tmpfs:
      - /run
      - /tmp
    volumes:
      - "/sys/fs/cgroup:/sys/fs/cgroup:ro"
    groups:
      - group1

  - name: ansible_burp_reports-03
    image: docker.io/pycontribs/centos:7
    pre_build_image: true
    command: /sbin/init
    capabilities:
      - SYS_ADMIN
    volumes:
      - "/sys/fs/cgroup:/sys/fs/cgroup:ro"
    #privileged: True
    groups:
      - group1

provisioner:
  name: ansible
  config_options:
    defaults:
      callback_whitelist: profile_tasks
  inventory:
    group_vars:
      master:
        burpsrcext: "zip"
        burp_version: "master"
        burp_remove_clients:
          - name: client_to_remove
          - name: other_client_to_remove
        burp_server_port_per_operation_bool: true
```

Requirement `molecule/default/playbook.yml`
--------------------------

```yaml

---
- hosts: all
  # You can change or remove this serial line to allow more parallel work
  # I have added it to use with travisci with less load to the travis
  serial: 1
  vars:
    burp_module_test_client: true
  roles:
    - role: ansible_burp2_server
```

https://molecule.readthedocs.io/en/latest/examples.html

Testing with travisci your `molecule.yml` file: 
-----------------------------------------------

```yaml
# http://www.jeffgeerling.com/blog/testing-ansible-roles-travis-ci-github
sudo: required
language: python
services:
  - docker
before_install:
  - sudo apt-get -qq update

install:
  - sudo apt-get install -y python-pip libssl-dev libffi-dev
  - pip install ansible
  - pip install "molecule[docker]"
  #- pip install docker-py
  # https://docs.ansible.com/ansible/latest/scenario_guides/guide_docker.html#requirements
    #- ansible-galaxy install -r requirements.yml

script:
  - molecule --debug create
  - molecule converge
  - molecule syntax
  - molecule idempotence

notifications:
    webhooks: https://galaxy.ansible.com/api/v1/notifications/
```

Testing with molecule+vagrant
-----------------------------

It's very useful for local test and ansible development, also to test burp with ansible in multiple environments/distribution. 

* have installed the role (see getting started)
* [Install vagrant](https://www.vagrantup.com/docs/installation/)


Choose your provider, example for libvirt: 

[vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)

See https://molecule.readthedocs.io/en/latest/configuration.html#vagrant

Run molecule
------------

```shell
sudo molecule test
```

Or first run:

```shell
sudo molecule --debug test
```

If you are using containers ensure `docker` service is **running**

* test command runs all the steps, so maybe you could use only converge and syntax, then destroy when you are done

Run only some parts of the test with molecule
---------------------------------------------

See: https://molecule.readthedocs.io/en/latest/usage.html

You can use the command line with other options, check `molecule --help` outpu: 

```bash

Usage: molecule [OPTIONS] COMMAND [ARGS]...

Options:
  --debug / --no-debug  Enable or disable debug mode. Default is disabled.
  --version             Show the version and exit.
  --help                Show this message and exit.

Commands:
  check        Performs a check ("Dry Run") on the current...
  converge     Provisions all instances defined in...
  create       Creates all instances defined in...
  dependency   Perform dependent actions on the current...
  destroy      Destroys all instances created by molecule.
  idempotence  Provisions instances and parses output to...
  init         Creates the scaffolding for a new role...
  list         Prints a list of currently available...
  login        Initiates an interactive ssh session with the...
  status       Prints status of configured instances.
  syntax       Performs a syntax check on the current role.
  test         Runs a series of commands (defined in config)...
  verify       Performs verification steps on running...
```

So for example to only test syntax and `ansible-lint` with molecule: 

```bash
molecule syntax
molecule lint
```

Accessing interactivery to docker containers
--------------------------------------------

with docker:

You can see the containers running:

```shell
sudo docker ps
```

And access interactivery

```shell
sudo docker exec -it image-name /bin/bash
```

You can also use `sudo molecule login --host hostname`, see https://molecule.readthedocs.io/en/latest/usage.html#login

Testing docker nested in docker
-------------------------------

See [docker.yml](docker.md)
