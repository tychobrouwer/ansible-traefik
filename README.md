# Traefik Ansible Role

This Ansible role installs and configures [Traefik](https://traefik.io/), a modern reverse proxy and load balancer, on Debian-based systems. It automates the deployment of Traefik, including static and dynamic configuration files, and sets it up as a systemd service. The role is designed to be flexible, supporting custom installation paths, architectures, and configuration templates.

---

## Role Overview

The role performs the following tasks:
- Installs Traefik on a Debian-based system.
- Configures Traefik using static and dynamic configuration files (e.g., `traefik_static.toml` and `traefik_dynamic.toml`).
- Sets up Traefik as a systemd service for reliable operation and management.

This role supports Traefik's split configuration model, where static settings (e.g., entrypoints, providers) are defined in one file, and dynamic settings (e.g., routing rules, services) are defined in another. 

---

## Requirements

- **Operating System**: Debian-based system (e.g., Debian 11/12, Ubuntu 20.04/22.04).
- **Ansible**: Version 2.9 or higher.
- **Privileges**: Root or sudo access on the target host(s).
- **Dependencies**: 
  - `wget` or `curl` (for downloading Traefik binaries).
  - `tar` (for extracting the Traefik archive).
  - A working systemd installation (default on most modern Debian systems).
- **Network**: Ports 80 (HTTP) and/or 443 (HTTPS) must be accessible if Traefik is used as a web proxy.

No additional Python modules or external roles are required beyond a standard Ansible setup.

---

## Role Variables

The role uses several variables to customize the installation and configuration of Traefik. Below are the available variables with their defaults (if applicable):

| Variable                     | Description                                                                 | Default Value           |
|------------------------------|-----------------------------------------------------------------------------|-------------------------|
| `traefik_arch`              | Target architecture for Traefik (e.g., `amd64`, `arm64`).                   | `amd64`                |
| `traefik_dir`               | Directory where the Traefik binary will be installed.                       | `/usr/local/bin`       |
| `traefik_static_template`   | Path to a custom Jinja2 template for the static configuration file.         | `templates/traefik_static.toml.j2` |
| `traefik_dynamic_template`  | Path to a custom Jinja2 template for the dynamic configuration file.        | `templates/traefik_dynamic.toml.j2` |

### Notes:
- The role assumes Traefik uses TOML format for configuration files (`.toml`), but you can adapt the templates to YAML if preferred by modifying the role logic.
- Static configuration defines Traefik’s core settings (e.g., entrypoints, providers), while dynamic configuration handles runtime changes (e.g., routing rules).
- Ensure the templates (`traefik_static_template` and `traefik_dynamic_template`) exist in your playbook’s `templates/` directory or provide absolute paths.

---

## Example Playbook

Here’s an example Ansible playbook demonstrating how to use this role:

```yaml
---
- hosts: servers
  roles:
    - role: tychobrouwer.traefik  # Default installation
    
    - role: tychobrouwer.traefik
      traefik_arch: "amd64"
      traefik_dir: "/usr/local/bin"
      traefik_static_template: "/templates/traefik_static.toml.j2"
      traefik_dynamic_template: "/templates/traefik_dynamic.toml.j2"
```

---

## Installation Steps (Performed by the Role)

1. **Download Traefik**: Fetches the specified version of Traefik for the target architecture from the official release page.
2. **Extract and Install**: Extracts the Traefik binary and places it in `traefik_dir` (e.g., `/usr/local/bin`).
3. **Configure Traefik**: 
   - Copies the rendered `traefik_static_template` to `traefik_config_dir/traefik.toml`.
   - Copies the rendered `traefik_dynamic_template` to `traefik_config_dir/dynamic.toml`.
4. **Set Up Systemd Service**: Creates and enables a systemd service to manage Traefik, pointing to the static configuration file.
5. **Start Service**: Ensures Traefik is running and configured to load dynamic settings.

---

## Usage Notes

- **Custom Configuration**: 
  - Create `traefik_static.toml.j2` for static settings (e.g., entrypoints, file provider for dynamic config).
  - Create `traefik_dynamic.toml.j2` for dynamic settings (e.g., routers, services, middleware).
  - Example static config:
    ```toml
    [entryPoints]
      [entryPoints.web]
        address = ":80"
    [providers]
      [providers.file]
        filename = "{{ traefik_config_dir }}/dynamic.toml"
    ```
  - Example dynamic config:
    ```toml
    [http.routers]
      [http.routers.myapp]
        rule = "Host(`myapp.example.com`)"
        service = "myapp-service"
    [http.services]
      [http.services.myapp-service]
        [http.services.myapp-service.loadBalancer]
          [[http.services.myapp-service.loadBalancer.servers]]
            url = "http://backend:8080"
    ```
- **Verification**: After running the playbook, verify Traefik is operational by checking the logs (`journalctl -u traefik`) or accessing a configured route.
- **Dynamic Updates**: Modify the dynamic configuration file (`dynamic.toml`) to update routing rules without restarting Traefik.

