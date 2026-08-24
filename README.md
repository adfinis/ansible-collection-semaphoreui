# Ansible Collection - adfinis.semaphoreui

![License](https://img.shields.io/github/license/adfinis/semaphoreui-ansible-collection)
![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/adfinis/semaphoreui-ansible-collection/ansible-lint.yml)
[![adfinis.semaphoreui on Ansible Galaxy](https://img.shields.io/badge/collection-adfinis.semaphore-blue)](https://galaxy.ansible.com/ui/repo/published/adfinis/semaphoreui/)

Ansible Collection to manage and deploy Semaphore UI in Docker.

## Install

The Collection can be installed from Ansible Galaxy:
``` bash
ansible-galaxy collection install adfinis.semaphoreui
```

Alternatively put the collection into a `requirements.yml` file:
``` yaml
---
collections:
  - name: adfinis.semaphoreui
```

## Roles

### `adfinis.semaphoreui.semaphore`

Ansible role to deploy and manage Semaphore UI servers and runners using Docker Compose.
* Works with the free version of Semaphore UI and Semaphore UI Pro.
* Optional Caddy reverse proxy for easy HTTPS
* Supports multiple database backends (sqlite (default), postgres, mysql).
* Ability to run different Ansible versions on the Semaphore UI Runner for backwards compatibility with older systems.

#### Usage
##### Inventory

The role does not assume any host group names. Whether a host is configured as a
Semaphore UI server or a runner is controlled by the variable `semaphore_type`
(`server` (default) or `runner`). Set it per host or per group, however your
inventory is organized.

A simple example:

**INI:**
```ini
[semaphore_servers]
vm-semaphore-server-01.example.com

[semaphore_runners]
vm-semaphore-runner-01.example.com
vm-semaphore-runner-02.example.com

[semaphore_runners:vars]
semaphore_type=runner
```

**YAML:**
``` yaml
---
semaphore_servers:
  hosts:
    vm-semaphore-server-01.example.com:

semaphore_runners:
  vars:
    semaphore_type: runner
  hosts:
    vm-semaphore-runner-01.example.com:
    vm-semaphore-runner-02.example.com:
```

##### Role variables

All arguments can be found in `roles/semaphore/meta/arguments_specs.yml`. You can display them with `ansible-doc`:
``` bash
ansible-doc -t role adfinis.semaphoreui.semaphore
```

The following variables are required:
* `semaphore_address`: Public domain or IP address for the Semaphore instance. 
* `semaphore_admin_username`: Username for the Semaphore UI admin account.
* `semaphore_admin_password`: Password for the Semaphore UI admin account.

##### Encryption secrets

Semaphore UI needs three persistent secrets: `semaphore_cookie_hash`,
`semaphore_cookie_encryption` and `semaphore_access_key_encryption`.
If you don't set them, the role generates them **deterministically** from
`semaphore_address`. This keeps them stable across runs and identical on all
servers of one instance, but it also means anyone who knows your
`semaphore_address` can reproduce them. You should therefore set all three
explicitly, e.g. stored in `ansible-vault` or fetched from an external secrets
provider:

``` yaml
semaphore_cookie_hash: "..."
semaphore_cookie_encryption: "..."
semaphore_access_key_encryption: "..."
```

> [!WARNING]
> `semaphore_access_key_encryption` is used to encrypt the access
> keys stored in Semaphore. Changing it on an existing deployment makes those
> keys unreadable. If you currently rely on the auto-generated defaults, copy
> the existing values from `<semaphore_path>/.env` on your server into your
> variables **before** changing `semaphore_address` or upgrading to a role
> version that changes the seed.

##### Encryption keyring

Secrets can be encrypted with a [keyring](https://semaphoreui.com/docs/admin-guide/security/encryption)
that supports key rotation:

``` yaml
semaphore_encryption_keys:
  key1: "..."  # openssl rand -base64 32
semaphore_encryption_secret_key: key1
```

After adding or rotating a key, run `semaphore vault rekey` inside the container.
See the upstream docs for details.

##### Caddy with DNS-01 challenge

By default Caddy uses the HTTP-01 challenge, which require ports 80/443 to be
publicly reachable. Alternatively, set a [caddy-dns](https://github.com/caddy-dns) provider to use
the DNS-01 challenge. The role then builds a custom Caddy image with the provider module.
The Caddy images are pinned by tag and digest (`semaphore_caddy_*_version`/`_digest`).

**Example:**

``` yaml
semaphore_use_caddy: true
semaphore_caddy_dns_provider: azure
semaphore_caddy_dns_provider_config:
  tenant_id: "{env.AZURE_TENANT_ID}"
  client_id: "{env.AZURE_CLIENT_ID}"
  client_secret: "{env.AZURE_CLIENT_SECRET}"
  subscription_id: "{env.AZURE_SUBSCRIPTION_ID}"
  resource_group_name: "{env.AZURE_RESOURCE_GROUP_NAME}"
semaphore_caddy_env:
  AZURE_TENANT_ID: "..."
  AZURE_CLIENT_ID: "..."
  AZURE_CLIENT_SECRET: "..."
  AZURE_SUBSCRIPTION_ID: "..."
  AZURE_RESOURCE_GROUP_NAME: "..."
```

##### Playbook
The Ansible Collection features a playbook to call the role `adfinis.semaphoreui.semaphore` without having to write one yourself:
``` bash
ansible-playbook adfinis.semaphoreui.semaphore_playbook
```

## License

[GPL-3.0-or-later](https://github.com/adfinis/semaphoreui-ansible-collection/blob/main/LICENSE)

## Author Information

The Ansible collection `adfinis.semaphore` was written by:

* Adfinis AG | [Website](https://www.adfinis.com/) | [GitHub](https://github.com/adfinis)

