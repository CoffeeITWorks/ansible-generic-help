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

    pip install --user molecule[docker]
    # If you want to test in local machine in fedora:
    pip install --user docker[podman]

* Install docker to use with docker driver.

* On fedora/centos also install:

   sudo dnf install libselinux-python podman

  sudo systemctl start podman

Testing with molecule+docker: 
-----------------------------

Install docker-engine and requirements shown at: https://molecule.readthedocs.io/en/latest/driver/index.html#usage

Example `molecule/local/molecule.yml` file working with centos/systemd, debian, Ubuntu/latest: 

Those files are automatically created with: https://molecule.readthedocs.io/en/latest/usage.html#init

```yaml
---
dependency:
  name: galaxy
  options:
    ignore-certs: True
    ignore-errors: True
    role-file: dev_requirements.yml  # this file is at the root of the git project same place as molecule is executed
driver:
  name: podman
platforms:

  - name: ansible_burp2_server-04
    image: "geerlingguy/docker-centos8-ansible"
    command: /usr/sbin/init
    #privileged: True
    pre_build_image: true
    capabilities:
      - SYS_ADMIN
    tmpfs:
      - /run
      - /tmp
    volumes:
      - "/sys/fs/cgroup:/sys/fs/cgroup:ro"
    groups:
      - use_pip_package

provisioner:
  name: ansible
  config_options:
    defaults:
      callback_whitelist: profile_tasks
    ssh_connection:
      pipelining: false
      ssh_args: -o ControlMaster=auto -o ControlPersist=60s
  inventory:
    group_vars:
      master:
        burpsrcext: "zip"
        burp_version: "master"
        burp_server_port_per_operation_bool: true

```

Requirement `molecule/local/converge.yml`
-----------------------------------------

Playbook file to run with ansible for the test.

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

Testing with github actions your `molecule` scenarios: 
------------------------------------------------------

See my example files in role: https://github.com/CoffeeITWorks/ansible_burp2_server/tree/master/.github/workflows

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

If you are using containers ensure `docker` or `podman` service is **running**

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
