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

```bash
    python3 -m pip install --user molecule[docker]
    # If you want to test in local machine in fedora:
    python3 -m pip install --user docker[podman]
```

* Install docker to use with docker driver or podman.

* On fedora/centos also install:

```bash
  sudo dnf install libselinux-python podman
  sudo systemctl start podman
```

Testing with molecule+docker: 
-----------------------------

---

Install docker-engine and requirements shown at: https://molecule.readthedocs.io/en/latest/driver/index.html#usage

Example `molecule/local/molecule.yml` file working with centos/systemd, debian, Ubuntu/latest: 

Those files are automatically created with: https://molecule.readthedocs.io/en/latest/usage.html#init

Check an updated structure scenario at: 

https://github.com/CoffeeITWorks/ansible_burp2_server/tree/master/molecule/ubuntu-2004

And good example for local run at:

https://github.com/CoffeeITWorks/ansible_burp2_server/tree/master/molecule/local

Requirement `molecule/local/converge.yml`
-----------------------------------------

---

Playbook file to run with ansible for the test.

```yaml

---
- name: Converge
  hosts: all
  tasks:
    - name: Include ansible_burp2_server
      include_role: 
        name: ansible_burp2_server
```

https://molecule.readthedocs.io/en/latest/examples.html

Testing with github actions your `molecule` scenarios: 
------------------------------------------------------

---

See my example files in role: https://github.com/CoffeeITWorks/ansible_burp2_server/tree/master/.github/workflows

There are 2 files in this example: 

Normal tests for every push:

    molecule-test.yml

Normal test for master and tags:

    galaxy-notify.yml

You will need to add a `secret` with your `galaxy-api-key` in your repository `settings` -> `secrets` -> `actions`

Testing with molecule+vagrant
-----------------------------

---

It's very useful for local test and ansible development, also to test burp with ansible in multiple environments/distribution. 

* have installed the role (see getting started)
* [Install vagrant](https://www.vagrantup.com/docs/installation/)


Choose your provider, example for libvirt: 

[vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)

See https://molecule.readthedocs.io/en/latest/configuration.html#vagrant

Run molecule
------------

---

I recommend to alway use scenario name, also very useful with github actions to use one scenario per OS
It will improve debugging and speed.

```shell
sudo molecule test -s local
```

Or first run:

```shell
sudo molecule --debug test -s local
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

You can see the containers running (you can also use podman):

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
