# controller_configuration.instance_groups

## Description

An Ansible Role to create/update/remove instance groups on Ansible Controller.

## Requirements

ansible-galaxy collection install -r tests/collections/requirements.yml to be installed
Currently:
  awx.awx
  or
  ansible.controller

## Variables

|Variable Name|Default Value|Required|Description|Example|
|:---|:---:|:---:|:---|:---|
|`controller_state`|"present"|no|The state all objects will take unless overridden by object default|'absent'|
|`controller_hostname`|""|yes|URL to the Ansible Controller Server.|127.0.0.1|
|`controller_validate_certs`|`True`|no|Whether or not to validate the Ansible Controller Server's SSL certificate.||
|`controller_username`|""|no|Admin User on the Ansible Controller Server. Either username / password or oauthtoken need to be specified.||
|`controller_password`|""|no|Controller Admin User's password on the Ansible Controller Server. This should be stored in an Ansible Vault at vars/controller-secrets.yml or elsewhere and called from a parent playbook. Either username / password or oauthtoken need to be specified.||
|`controller_oauthtoken`|""|no|Controller Admin User's token on the Ansible Controller Server. This should be stored in an Ansible Vault at or elsewhere and called from a parent playbook. Either username / password or oauthtoken need to be specified.||
|`controller_request_timeout`|`10`|no|Specify the timeout in seconds Ansible should use in requests to the controller host.||
|`controller_instance_groups`|`see below`|yes|Data structure describing your instance groups Described below.||

### Enforcing defaults

The following Variables compliment each other.
If Both variables are not set, enforcing default values is not done.
Enabling these variables enforce default values on options that are optional in the controller API.
This should be enabled to enforce configuration and prevent configuration drift. It is recomended to be enabled, however it is not enforced by default.

Enabling this will enforce configurtion without specifying every option in the configuration files.

'controller_configuration_instance_groups_enforce_defaults' defaults to the value of 'controller_configuration_enforce_defaults' if it is not explicitly called. This allows for enforced defaults to be toggled for the entire suite of controller configuration roles with a single variable, or for the user to selectively use it.

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`controller_configuration_instance_groups_enforce_defaults`|`False`|no|Whether or not to enforce default option values on only the applications role|
|`controller_configuration_enforce_defaults`|`False`|no|This variable enables enforced default values as well, but is shared across multiple roles, see above.|

### Secure Logging Variables

The following Variables compliment each other.
If Both variables are not set, secure logging defaults to false.
The role defaults to False as normally the add instance groups task does not include sensitive information.
controller_configuration_instance_groups_secure_logging defaults to the value of controller_configuration_secure_logging if it is not explicitly called. This allows for secure logging to be toggled for the entire suite of controller configuration roles with a single variable, or for the user to selectively use it.

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`controller_configuration_instance_groups_secure_logging`|`False`|no|Whether or not to include the sensitive instance groups role tasks in the log. Set this value to `True` if you will be providing your sensitive values from elsewhere.|
|`controller_configuration_secure_logging`|`False`|no|This variable enables secure logging as well, but is shared across multiple roles, see above.|

### Asynchronous Retry Variables

The following Variables set asynchronous retries for the role.
If neither of the retries or delay or retries are set, they will default to their respective defaults.
This allows for all items to be created, then checked that the task finishes successfully.
This also speeds up the overall role.

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`controller_configuration_async_retries`|30|no|This variable sets the number of retries to attempt for the role globally.|
|`controller_configuration_instance_groups_async_retries`|`{{ controller_configuration_async_retries }}`|no|This variable sets the number of retries to attempt for the role.|
|`controller_configuration_async_delay`|1|no|This sets the delay between retries for the role globally.|
|`controller_configuration_instance_groups_async_delay`|`controller_configuration_async_delay`|no|This sets the delay between retries for the role.|
|`controller_configuration_loop_delay`|0|no|This sets the pause between each item in the loop for the roles globally. To help when API is getting overloaded.|
|`controller_configuration_instance_groups_loop_delay`|`controller_configuration_loop_delay`|no|This sets the pause between each item in the loop for the role. To help when API is getting overloaded.|
|`controller_configuration_async_dir`|`null`|no|Sets the directory to write the results file for async tasks. The default value is set to `null` which uses the Ansible Default of `/root/.ansible_async/`.|

## Data Structure

### Instance Group Variables

|Variable Name|Default Value|Required|Type|Description|
|:---:|:---:|:---:|:---:|:---:|
|`name`|""|yes|str|Name of this instance group.|
|`new_name`|""|str|no|Setting this option will change the existing name (looked up via the name field).|
|`credential`|""|no|str|Credential to authenticate with Kubernetes or OpenShift. Must be of type "Kubernetes/OpenShift API Bearer Token". Will make instance part of a Container Group.|
|`is_container_group`|False|no|bool|Signifies that this InstanceGroup should act as a ContainerGroup. If no credential is specified, the underlying Pod's ServiceAccount will be used.|
|`policy_instance_percentage`|""|no|int|Minimum percentage of all instances that will be automatically assigned to this group when new instances come online.|
|`policy_instance_minimum`|""|no|int|Static minimum number of Instances that will be automatically assign to this group when new instances come online.|
|`policy_instance_list`|""|no|list|List of exact-match Instances that will be assigned to this group.|
|`max_concurrent_jobs`|0|no|int|Maximum number of concurrent jobs to run on this group. Zero means no limit.|
|`max_forks`|0|no|int|Max forks to execute on this group. Zero means no limit.|
|`pod_spec_override`|""|no|str|A custom Kubernetes or OpenShift Pod specification.|
|`instances`|""|no|list|The instances associated with this instance_group.|
|`state`|`present`|no|str|Desired state of the resource.|

### Standard Instance Group Data Structure

#### Yaml Example

```yaml
---
controller_instance_groups:
  - name: test_instance_group
```

## Playbook Examples

### Standard Role Usage

```yaml
---
- name: Playbook to configure ansible controller post installation
  hosts: localhost
  connection: local
  # Define following vars here, or in controller_configs/controller_auth.yml
  # controller_hostname: ansible-controller-web-svc-test-project.example.com
  # controller_username: admin
  # controller_password: changeme
  pre_tasks:
    - name: Include vars from controller_configs directory
      ansible.builtin.include_vars:
        dir: ./yaml
        ignore_files: [controller_config.yml.template]
        extensions: ["yml"]
  roles:
    - {role: infra.controller_configuration.instance_groups, when: controller_instance_groups is defined}
```

## License

[GPL-3.0](https://github.com/redhat-cop/infra.controller_configuration#licensing)

## Author

[Sean Sullivan](https://github.com/sean-m-sullivan)
