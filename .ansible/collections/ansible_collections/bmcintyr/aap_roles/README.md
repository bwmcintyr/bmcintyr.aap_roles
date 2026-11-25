# `bmcintyr.aap_roles`

[![Ansible Lint](https://img.shields.io/badge/ansible--lint-passed-green.svg)](https://docs.ansible.com/ansible-lint/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A specialized Ansible Collection designed to simplify organizational onboarding and enable automated configuration management ("Configuration as Code") within **Red Hat Ansible Automation Platform (AAP)**.

---

## Overview

The `aap_roles` collection standardizes and simplifies the administration of Ansible Automation Platform by dividing management responsibilities into two distinct, high-value roles:

1. **`config_as_code`**: Enables a GitOps workflow by cloning variable repositories and invoking the `infra.aap_configuration` collection to dynamically provision and manage AAP resources at scale.

---

## Requirements

Ansible & AAP Versions

* Ansible Core >= 2.15.0
* Ansible Automation Platform 2.5+


```yaml
collections:
  - name: infra.aap_configuration
```

## Roles Overview

### 1. `config_as_code`

The `config_as_code` role implements a Infrastructure-as-Code / GitOps pattern for AAP. It pulls standard configuration definitions (YAML variables) from a Git repository and executes the official infra.aap_configuration collection to declare target state for AAP objects.

#### Key Capabilities
* **Git Sync**: Automatically clones or pulls the latest configuration repository containing AAP variables.
* **Variable Resolution**: Automatically loads and parses structural YAML configs (e.g., inventories, credentials, job templates, execution environments).
* **Automated Provisioning**: Orchestrates infra.aap_configuration roles to apply the declarative desired state to the AAP instance.

#### Basic Usage Example

```yaml
- name: Apply Configuration as Code from Git
  hosts: localhost
  gather_facts: false
  roles:
    - role: aap_roles.config_as_code
      vars:
        config_repo_url: "[https://github.com/example/aap-config-as-code.git](https://github.com/example/aap-config-as-code.git)"
        config_repo_version: "main"
        config_dir: "./vars"
        aap_hostname: "[https://aap.example.com](https://aap.example.com)"
        aap_validate_certs: true