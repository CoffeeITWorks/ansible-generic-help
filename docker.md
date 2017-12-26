Testing docker inside docker with molecule
==========================================

Adding to molecule/default/molecule.yml
---------------------------------------

Add volume to use docker:

`- "/var/run/docker.sock:/var/run/docker.sock"`

```yaml
platforms:
  - name: ansible_test-01
    image: solita/ubuntu-systemd:16.04
    privileged: True
    command: /sbin/init
    capabilities:
      - SYS_ADMIN    
    volumes:
      - "/sys/fs/cgroup:/sys/fs/cgroup:ro"
      - "/var/run/docker.sock:/var/run/docker.sock"
    groups:
      - group1
```

Add dependency example to install docker in the container

```yaml
dependency:
  name: galaxy
  options:
    ignore-certs: True
    ignore-errors: True
    role-file: tests/requirements.yml
```

Example tests/requirements.yml

```yaml
---
- src: geerlingguy.docker
  name: geerlingguy.docker
```

Adding to molecule/default/playbook.yml
---------------------------------------

```yaml
---
- name: Converge
  hosts: all
  vars:
    burp_module_test_client: False
    burpui_standalone: True
  roles:
    - role: geerlingguy.docker
    - role: ansible-role-docker-nginx
```
