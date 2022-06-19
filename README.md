# Ansible Role: Hetzner Cloud hcloud

[![CI](https://github.com/ngine-io/ansible-role-hcloud/workflows/CI/badge.svg?event=push)](https://github.com/ngine-io/ansible-role-hcloud/actions?query=workflow%3ACI)

Manages compute resources on [Hetzner Cloud](https://www.hetzner.com/cloud).

## Dependencies

See `requirements.txt` for python library dependencies.

## Examples

### Inventory

Given is a inventory similar as:

```ini
[web]
web-1
web-2
web-3
```

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

Ensure placment groups are created:

```yaml
hcloud__placement_groups:
  - name: my placement group
    labels:
       project: genesis
```

#### Loadbalancers

Configure a simple load balancer and add servers as targets later (see server configs):

```yaml
hcloud__loadbalancers:
  - name: my load balancer
    type: lb11
    services:
      - protocol: http
        port: 80
      - protocol: tcp
        port: 443
        destination_port: 443
```

Configure a load balancer and use label selector hcloud feature:

```yaml
hcloud__loadbalancers:
  - name: my load balancer
    type: lb11
    targets:
      - type: label_selector
        label_selector: app=public
    services:
      - protocol: http
        port: 80
        health_check:
          interval: 15
          port: 80
          timeout: 3
          retries: 3
          protocol: http
          http:
            # Single wildcard char '?' or glob wildcard '*'
            status_codes:
              - 2*
              - 3*
              - 40?
            # Optional
            path: /
            domain: www.example.com
            response: hello world
      - protocol: tcp
        port: 443
        destination_port: 443
```

#### Server Configs

The following configs are server specific and meant to be set in a lower level `group_vars` or `host_vars`, e.g. `group_vars/web.yml`

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

Ensure web servers are in a placement group:

```yaml
hcloud__server_placement_group: my placement group
```

Add server to an existing loadbalancer as server target (optionally use private IP):

```yaml
hcloud__server_loadbalancer_name: my load balancer
hcloud__server_loadbalancer_use_private_ip: true
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

> HINT: Use the special variable `inventory_hostname_short` as prefix to easily identify which volume is attached to which server.

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
