Testing for developers
======================

https://molecule.readthedocs.io/en/latest/

The following examples are to test with molecule, but you can also use your own servers and setup an [Ansible inventory](http://docs.ansible.com/ansible/intro_inventory.html)

Molecule resolves some good things for a dev environment, like: automatic provision new hosts, automatic inventory for ansible, automatic deploy, automatic destroy, automatic test idempotence, etc. 

* Install molecule

    sudo pip install molecule
    

Testing with molecule+docker: 
-----------------------------

Install docker-engine and requirements shown at: https://molecule.readthedocs.io/en/latest/driver/index.html#usage

Example `molecule.yml` file working with centos/systemd, debian/8, Ubuntu/latest: 

```yaml

---

driver:
  name: docker

docker:
  containers:

      - name: ansible_burp2_server-01
        image: ubuntu
        image_version: latest
        ansible_groups:
          - group1

      - name: ansible_burp2_server-master2
        image: debian
        image_version: '8'
        
        # check the example group below (you can safetly remove these lines)
        ansible_groups:
          - group_master

      # Use with centos/systemd to ensure systemd will be available with ansible service command.
      - name: ansible_burp2_server-03
        image: centos/systemd
        image_version: latest
        volume_mounts:
          - "/sys/fs/cgroup:/sys/fs/cgroup:ro"
        privileged: True
        ansible_groups:
          - group1

verifier:
  name: testinfra

ansible:
  playbook: playbook.yml
  
  # Some example group
  group_vars:
    group_master:
      burpsrcext: "zip"
      burp_version: "master"
```

Requirement `playbook.yml`
--------------------------

Add this file to use with `molecule.yml`

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

Testing with travisci your `molecule.yml` file: 
-----------------------------------------------

```yaml

sudo: required
dist: trusty
#group: edge
env:
  global:
  - DOCKER_VERSION="1.9.1-0~trusty"

language: python
services:
  - docker
before_install:
  - sudo apt-get -qq update
  #- sudo apt-get install -o Dpkg::Options::="--force-confold" --force-yes -y docker-engine
  - sudo apt-get remove docker-engine -yq
  - sudo apt-get install docker-engine=$DOCKER_VERSION -yq --no-install-suggests --no-install-recommends --force-yes -o Dpkg::Options::="--force-confnew"
install:
  - pip install molecule
  # - pip install required driver (e.g. docker, python-vagrant, shade)
  - pip install docker
script:
  - molecule test
notifications:
    webhooks: https://galaxy.ansible.com/api/v1/notifications/
```

Testing with molecule+vagrant: 
------------------------------

It's very useful for local test and ansible development, also to test burp with ansible in multiple environments/distribution. 

* have installed the role (see getting started)
* [Install vagrant](https://www.vagrantup.com/docs/installation/)


Choose your provider, example for libvirt: 

[vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)

modify ´molecule.yml´ file and change driver, example: 

    driver:
      name: vagrant

Also ensure you are using same provider as choosen:


    providers:
      - name: libvirt
        type: libvirt


Run molecule
------------

    sudo molecule test 

If you are using containers ensure `docker` service is **running**

Run only some parts of the test with molecule
---------------------------------------------

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
molecule verify
```

Accessing interactivery to docker containers
--------------------------------------------

You can see the containers running: 

    sudo docker ps

And access interactivery

    sudo docker exec -it image-name /bin/bash
