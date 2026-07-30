
# bmcintyr.aap_roles.config_as_code


## Description

This Ansible project provides a GitOps-style workflow for managing Ansible Automation Platform (AAP) resources. It retrieves resource definitions from a Git repository and uses the `infra.aap_configuration` collection to synchronize the definitions with an AAP instance.


## Overview

This project is the automation "engine" that enables the developer workflow. It is typically triggered by a CI/CD pipeline (like an EDA rulebook) after a merge to a protected branch.

This role is designed to:
1.  Be given a "definitions" repository (`config_as_code_repo`) and branch (`config_as_code_branch`).
2.  Clone that repository and inspect the `aap_configuration/` directory.
3.  Read all `.yml` resource definitions within that directory.
4.  Connect to a target AAP instance and use the `infra.aap_configuration` collection to reconcile the resources in AAP, ensuring they match the definitions in Git.

---

## Requirements

Before running this project, you will need:

* collections/requirements.yml
  ```yaml
  collections:
    - name: infra.aap_configuration
    - name: ansible.platform
    - name: ansible.controller
    - name: ansible.hub
    - name: ansible.eda
  ```

## Variables

|Variable Name|Default Value|Required|Type|Description|Example|
|:---|:---:|:---:|:---|:---|:---|
|`config_as_code_repo`||yes|str|The HTTP URL to the source git repository|'https://git.mycompany.com/project.git'|
|`config_as_code_branch`||yes|str|The branch of the source git repository|'dev'|
|`config_as_code_directory`|aap_configuration|yes|str|The directory at the root of the project where resource definitions are found.|'aap_configuration'|
|`config_as_code_environment`|dev|yes|str|The current environment where the job is being executed.|'dev'|
|`config_as_code_user_login`||no|str|Git username. Used to find user in AAP when executed from git pipeline.|firstname.lastname|
|`config_as_code_user_email`||no|str|Git email. Used to find user in AAP when executed from git pipeline.|firstname.lastname@example.com|
|`config_as_code_allowed_objects`||no|list|AAP resources that will be created for normal users.  Does not apply to 'superuser'.|config_as_code_allowed_objects: ['controller_projects', 'controller_templates']|
|`config_as_code_secure_logging`|`true`|no|boolean|Whether or not to include sensitive role tasks in the log|



## Playbook Examples

While this project can be run directly from its own repository, it is intended to be the Job Template that EDA launches. The playbook in *this* repository should be simple, designed only to call the `config_as_code` role.

### Example `config_as_code.yml` Playbook

```yaml
---
- name: Synchronize AAP Configuration from Git
  hosts: localhost
  connection: local
  gather_facts: false

  roles:
    - name: config_as_code
      # The config_as_code_repo and config_as_code_branch variables are expected
      # to be passed in as Extra Variables by the EDA rulebook.
      # We provide defaults here for testing.
      vars:
        config_as_code_repo: "{{ config_as_code_repo }}"
        config_as_code_branch: "{{ config_as_code_branch }}"
```

## Directory Structure

To add resources to AAP, developers must create `.yml` files inside an `aap_configuration` in *their* repository.

The `config_as_code` role will parse any file ending in `.yml` inside this folder.  The *names* of the files do not matter, only their contents.

Example Directory
```
my-app-repo/
├── aap_configuration/
│   ├── all/
│   │   ├── 01-project.yml
│   │   ├── 02-inventory.yml
│   │   ├── 03-credentials.yml
│   │   └── 04-job-templates.yml
│   ├── dev/
│   │   ├── 05-project.yml
│   │   ├── 06-inventory.yml
│   ├── tpp/
│   │   ├── 07-project.yml
│   │   ├── 08-inventory.yml
│   ├── prod/
│   │   ├── 09-project.yml
│   │   ├── 10-inventory.yml
├── playbooks/
│   └── deploy_app.yml
└── README.md
```

Alternate Example
```
my-app-repo/
├── aap_configuration/
│   ├── 01-project.yml
│   ├── 02-inventory.yml
│   ├── 03-credentials.yml
│   ├── 04-job-templates.yml
│   ├── 05-project.yml
│   ├── 06-inventory.yml
├── playbooks/
│   └── deploy_app.yml
└── README.md
```


## Example Resource Definition (`04-job-templates.yml`)

The YAML structure must match what the `infra.controller_configuration` collection expects.

```yaml
---
# This file can contain one or many job templates
controller_templates:
  - name: "My App - Deploy (Dev)"
    organization: "Default"
    project: "My App Project"
    playbook: "playbooks/deploy_app.yml"
    inventory: "My App Inventory (Dev)"
    credentials:
      - "My App SSH Key"
    ask_variables_on_launch: true
    survey_enabled: true
    survey:
      spec:
        - variable: "app_version"
          question_name: "Application Version"
          question_description: "Enter the version tag to deploy (e.g., v1.2.3)"
          type: "text"
          required: true
          default: "latest"

  - name: "My App - Health Check (Dev)"
    organization: "Default"
    project: "My App Project"
    playbook: "playbooks/health_check.yml"
    inventory: "My App Inventory (Dev)"
```
