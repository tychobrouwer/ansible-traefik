Install and configure Traefik
=========

This role installs and configures Traefik as a reverse proxy.

Role Variables
--------------

Requirements
----------------

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
    - role: tychobrouwer.traefik
    
    - role: tychobrouwer.traefik
      traefik_dynamic_template: /templates/traefik_dynamic.toml.j2
      traefik_static_template: /templates/traefik_static.toml.j2
      traefik_arch: amd64
      traefik_dir: /usr/local/bin
      
```

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
