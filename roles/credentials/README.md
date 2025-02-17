# controller_configuration.credentials
## Description
An Ansible Role to create Credentials on Ansible Controller.

## Requirements
ansible-galaxy collection install  -r tests/collections/requirements.yml to be installed
Currently:
  awx.awx
  or
  ansible.controller

## Variables

### Authentication
|Variable Name|Default Value|Required|Description|Example|
|:---:|:---:|:---:|:---:|:---:|
|`controller_state`|"present"|no|The state all objects will take unless overriden by object default|'absent'|
|`controller_hostname`|""|yes|URL to the Ansible Controller Server.|127.0.0.1|
|`controller_validate_certs`|`True`|no|Whether or not to validate the Ansible Controller Server's SSL certificate.||
|`controller_username`|""|yes|Admin User on the Ansible Controller Server.||
|`controller_password`|""|yes|Controller Admin User's password on the Ansible Controller Server.  This should be stored in an Ansible Vault at vars/controller-secrets.yml or elsewhere and called from a parent playbook.||
|`controller_oauthtoken`|""|yes|Controller Admin User's token on the Ansible Controller Server.  This should be stored in an Ansible Vault at or elsewhere and called from a parent playbook.||
|`controller_credentials`|`see below`|yes|Data structure describing your credentials Described below.||

### Secure Logging Variables
The following Variables compliment each other.
If Both variables are not set, secure logging defaults to false.
The role defaults to False as normally the add credentials task does not include sensitive information.
controller_configuration_credentials_secure_logging defaults to the value of controller_configuration_secure_logging if it is not explicitly called. This allows for secure logging to be toggled for the entire suite of configuration roles with a single variable, or for the user to selectively use it.

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`controller_configuration_credentials_secure_logging`|`False`|no|Whether or not to include the sensitive Credential role tasks in the log.  Set this value to `True` if you will be providing your sensitive values from elsewhere.|
|`controller_configuration_secure_logging`|`False`|no|This variable enables secure logging as well, but is shared accross multiple roles, see above.|

### Asynchronous Retry Variables
The following Variables set asynchronous retries for the role.
If neither of the retries or delay or retries are set, they will default to their respective defaults.
This allows for all items to be created, then checked that the task finishes successfully.
This also speeds up the overall role.

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`controller_configuration_async_retries`|30|no|This variable sets the number of retries to attempt for the role globally.|
|`controller_configuration_credentials_async_retries`|30|no|This variable sets the number of retries to attempt for the role.|
|`controller_configuration_async_delay`|1|no|This sets the delay between retries for the role globally.|
|`controller_configuration_credentials_async_delay`|1|no|This sets the delay between retries for the role.|

## Data Structure
### Variables
|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`name`|""|yes|Name of Credential|
|`new_name`|""|no|Setting this option will change the existing name (looked up via the name field.|
|`copy_from`|""|no|Name or id to copy the credential from. This will copy an existing credential and change any parameters supplied.|
|`description`|`False`|no|Description of  of Credential.|
|`organization`|""|no|Organization this Credential belongs to. If provided on creation, do not give either user or team.|
|`credential_type`|""|no|Name of credential type. See below for list of options. More information in Ansible controller documentation. |
|`inputs`|""|no|Credential inputs where the keys are var names used in templating. Refer to the Ansible controller documentation for example syntax. Individual examples can be found at /api/v2/credential_types/ on an controller.|
|`user`|""|no|User that should own this credential. If provided, do not give either team or organization. |
|`team`|""|no|Team that should own this credential. If provided, do not give either user or organization. |
|`state`|`present`|no|Desired state of the resource.|
|`update_secrets`|true|no|bool| True will always change password if user specifies password, even if API gives $encrypted$ for password. False will only set the password if other values change too.|

### Credential types
|Credential types|
|:---:|
|Amazon Web Services|
|Controller|
|GitHub Personal Access Token|
|GitLab Personal Access Token|
|Google Compute Engine|
|Insights|
|Machine|
|Microsoft Azure Resource Manager|
|Network|
|OpenShift or Kubernetes API Bearer Token|
|OpenStack|
|Red Hat CloudForms|
|Red Hat Satellite 6|
|Red Hat Virtualization|
|Source Control|
|Vault|
|VMware vCenter|

### Standard Organization Data Structure
#### Json Example
```json
---
{
    "controller_credentials": [
      {
        "name": "gitlab",
        "description": "Credentials for GitLab",
        "organization": "Default",
        "credential_type": "Source Control",
        "inputs": {
          "username": "person",
          "password": "password"
        }
      }
    ]
}
```
#### Yaml Example
```yaml
---
controller_credentials:
- name: gitlab
  description: Credentials for GitLab
  organization: Default
  credential_type: Source Control
  inputs:
    username: person
    password: password
- name: localuser
  description: Machine Credential example with become_method input
  credential_type: Machine
  inputs:
    username: localuser
    password: password
    become_method: sudo
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
      include_vars:
        dir: ./yaml
        ignore_files: [controller_config.yml.template]
        extensions: ["yml"]
  roles:
    - {role: redhat_cop.controller_configuration.credentials, when: controller_credentials is defined}
```
## License
[MIT](LICENSE)

## Author
[Andrew J. Huffman](https://github.com/ahuffman)
[Sean Sullivan](https://github.com/sean-m-sullivan)
