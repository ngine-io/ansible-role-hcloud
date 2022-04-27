# Ansible Role: Hetzner Cloud hcloud

[![CI](https://github.com/ngine-io/ansible-role-hcloud/workflows/CI/badge.svg?event=push)](https://github.com/ngine-io/ansible-role-hcloud/actions?query=workflow%3ACI)

Manages compute resources on [Hetzner Cloud](https://www.hetzner.com/cloud).

## Dependencies

See `requirements.txt` for python library dependencies.

## Examples

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
