# Ansible Role: Hetzner Cloud hcloud

[![CI](https://github.com/ngine-io/ansible-role-hcloud/workflows/CI/badge.svg?event=push)](https://github.com/ngine-io/ansible-role-hcloud/actions?query=workflow%3ACI)

Manages compute resources on [Hetzner Cloud](https://www.hetzner.com/cloud).

## Dependencies

See `requirements.txt` for python library dependencies.

## Examples

### Variables

The following variables are meant to be set in a top level group, e.g.`group_vars/all.yml`.

API key:

> NOTE: Recommended to encrypt this token with Ansible Vault or use a lookup to your secrets management system!

```yaml
hcloud__api_token: ...
```

Ensures a private network is created:

```yaml
hcloud__networks:
  - name: internal
    network: 10.10.10.0/24
```

Ensure some firewalls and rules are created:

```yaml
hcloud__firewalls:
  - name: default
    rules:
      - direction: in
        protocol: icmp
        source_ips:
          - 0.0.0.0/0
          - ::/0
      - direction: in
        protocol: tcp
        port: "22"
        source_ips:
          - 0.0.0.0/0
  - name: web
    rules:
      - direction: in
        protocol: tcp
        port: "80"
        source_ips:
          - 0.0.0.0/0
          - ::/0
      - direction: in
        protocol: tcp
        port: "443"
        source_ips:
          - 0.0.0.0/0
          - ::/0
```

The following configs are server specific and meant to be set in `group_vars` of `host_vars`, e.g. `group_vars/web.yml`

Server image to use:

```yaml
hcloud__server_image: debian-11
```

Location to deploy into:

```yaml
hcloud__server_location: fsn1
```

Ensure web servers have these firewalls configured:

```yaml
hcloud__server_firewalls:
  - default
  - web
```

Ensure web servers are attached to these networks:

```yaml
hcloud__server_networks:
  - name: internal
```

Ensure web server have this type:

```yaml
hcloud__server_type: cpx11
```

User data to be executed on boot, e.g.:

```yaml
hcloud__server_user_data: |
  #cloud-config
  users:
    - name: admin
      groups: users, admin
      sudo: ALL=(ALL) NOPASSWD:ALL
      shell: /bin/bash
      ssh_authorized_keys:
        - <public_ssh_key>
  packages:
    - fail2ban
  package_update: true
  package_upgrade: true
```

Ensure a volume is created and attached to the server:

> HINT: use the special variable `inventory_hostname_short` as prefix to see which volume is attached to which server.

```yaml
hcloud__server_volumes:
  - name: "{{ inventory_hostname_short }}-vol1"
    format: ext4
    size: 10
```

### Playbook

A typical playbook would look like this.

```yaml
---
- name: Provision Cloud servers
  hosts: all
  serial: 5
  gather_facts: false
  roles:
    - role: ngine_io.hcloud
      delegate_to: localhost

  post_tasks:
    - name: Wait for SSH access
      delegate_to: localhost
      wait_for:
        host: "{{ ansible_host }}"
        port: 22
        timeout: 60
```

## License

MIT

## Author Information

René Moser (@resmo)
