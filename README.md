Role: ansible-pull
=========

This role is used to configure an ansible pull service for automated code updates on a scheduled basis. This method reverses the push architecture of ansible which then pushes the code to the localhost.

What it does:
--------------
* Ensures that a stable release of Ansible is installed to meet the requirement (ansible >= 2.15)
  * For Debian based systems this involves configuring an apt repository for ```ppa:ansible``` and the package is then installed.
  * Red Hat systems only install ```ansible-core```.
* Creates a files based off the template provided in ```templates``` in /etc/systemd/system for .service .time to run on a systemd timer when prompted
* Creates a cron file to run the ansible pull command when promoted.
* Creates a logrotate file to rotate the logs generated by the cron based run.

Role Variables
--------------
The Following varibles may be applied at the group,host,role,or task level to deploy the ansible-pull mechanism to managed hosts. This allows an administrator to deploy the service is they do not have supporting infracturcture from Satellite/Foreman or an Automation platform such as AWX/AAP/OLAM but still have a git repository that the code base can be housed at.

Service Enabling Variables
| Variable                    |Type     | Description                                                                   |Default       | Required |
|:----------------------------|:--------|:------------------------------------------------------------------------------|:-------------|:---------|
| ANSIBLE_SERVICE             | bool    | Enables systemd service for ansible pull mechanism to run on boot.            | true         | No       |
| ANSIBLE_TIMER               | bool    | Enables a systemd timer file to be used for polling the ansible-pull service  | true         | No       |
| ANSIBLE_SERVICE_SCRIPT      | bool    | Creates a shell script utilized by the systemd serice                         | true         | No       |
| ANSIBLE_CRON                | bool    | Creates a cron file to execute ansible-pulls in lieu of the systemd option    | false        | No       |
| ANSIBLE_LOGROTATE           | bool    | Creats a file within logrotate to ensure that pull logs are rotated regularly | true         | No       |

Ansible Pull Options
| Variable                    |Type     | Description                                                                   |Default       | Required |
|:----------------------------|:--------|:------------------------------------------------------------------------------|:-------------|:---------|
| ANSIBLE_PULL_USER           | string  | Connect as this user                                                          | Current User | No       |
| ANSIBLE_PULL_KEY            | string  | Use this file to authenticate the connect                                     | none         | No       |
| ANSIBLE_SSH_COMMON_ARGS     | string  | Use this file to authenticate the connection                                  | none         | No       |
| ANSIBLE_SFTP_EXTRA_ARGS     | string  | Specify extra arguments to pass to sftp only                                  | none         | No       |
| ANSIBLE_SCP_EXTRA_ARGS      | string  | Specify extra arguments to pass to scp only                                   | none         | No       |
| ANSIBLE_SSH_EXTRA_ARGS      | string  | Specify extra arguments to pass to ssh only                                   | none         | No       |
| ANSIBLE_VAULT_IDS           | string  | The vault identity to use                                                     | none         | No       |
| ANSIBLE_VAULT_PASSWORD_FILE | string  | Define vault password file to use                                             | none         | No       |
| ANSIBLE_EXTRA_VARS          | string  | Set additional variables as key=value or YAML/JSON, if filename prepend with @| none         | No       |
| ANSIBLE_TAGS                | string  | only run plays and tasks tagged with these values                             | none         | No       |
| ANSIBLE_SKIP_TAGS           | string  | only run plays and tasks whose tags do not match these values                 | none         | No       |
| ANSIBLE_INVENTORY           | string  | specify inventory host path or comma separated host list                      | none         | No       |
| ANSIBLE_LIST_HOSTS          | string  | outputs a list of matching hosts; does not execute anything else              | none         | No       |
| ANSIBLE_LIMIT               | string  | further limit selected hosts to an additional pattern                         | none         | No       |
| ANSIBLE_MODULE_PATH         | string  | prepend colon-separated path(s) to module library                             | ANSIBLE_HOME | No       |
| ANSIBLE_PURGE               | bool    | purge checkout after playbook run                                             | true         | No       |
| ANSIBLE_CHANGE_ONLY         | bool    | only run the playbook if the repository has been updated                      | false        | No       |
| ANSIBLE_SLEEP               | int     | sleep for random interval (between 0 and n number of seconds) before starting | none         | No       |
| ANSIBLE_FORCE               | bool    | run the playbook even if the repository could not be updated                  | false        | No       |
| ANSIBLE_DESTINATION_PATH    | string  | absolute path of repository checkout directory. Reliative paths not supported | ~/.ansible        | No       |
| ANSIBLE_REPO_URL            | string  | URL of the playbook repository                                                | none         | Yes      |
| ANSIBLE_FULL_CLONE          | bool    | Do a full clone, instead of a shallow one.                                    | false        | Yes      |
| ANSIBLE_CHECKOUT            | string  | branch/tag/commit to checkout. Defaults to behavior of repository module.     | main         | No       |
| ANSIBLE_ACCEPT_HOST_KEY     | bool    | adds the hostkey for the repo url if not already added                        | false        | No       |
| ANSIBLE_MODULE_NAME         | string  | Repository module name, which ansible will use to check out the repo.         | git          | No       |
| ANSIBLE_VERIFY_COMMIT       | bool    | verify GPG signature of checked out commit, if it fails abort running the playbook. | false        | No       |
| ANSIBLE_REPO_CLEAN          | bool    | modified files in the working repository will be discarded                    | false        | No       |
| ANSIBLE_TRACK_SUBS          | bool    | submodules will track the latest changes.                                     | false        | No       |
| ANSIBLE_CHECK_MODE          | bool    | don't make any changes                                                        | false        | No       |
| ANSIBLE_DIFF                | bool    | when changing (small) files and templates, show the differences in those files| false        | No       |
| ANSIBLE_PLAYBOOK            | string  | Define the playbook to run on pull. Defaults to local.yml. If none then the pull fails.| local.yml    | No       |
| ANSIBLE_PULL_TLS_IGNORE     | bool    | This option sets the ENV variable GIT_SSL_NO_VERIFY=true. If HTTPS is using self-signed certificates | none         | No       |

Output Variables
| Variable                    |Type     | Description                                                                   |Default       | Required |
|:----------------------------|:--------|:------------------------------------------------------------------------------|:-------------|:---------|
| ANSIBLE_LOG_PATH            | string  | Sets the path and file for the job output generated by either systemd or cron | /var/log/ansible-pull.log | No |

Configuring ansible-pull.timer
| Variable                    |Type     | Description                                                                   |Default       | Required |
|:----------------------------|:--------|:------------------------------------------------------------------------------|:-------------|:---------|
| ANSIBLE_TIMER_SCHEDULE      | string  | This variable set the OnCalandar directive for the ansible-pull.time poller. (Default: 5 minutes) | *:0/05:00    | No  |

Configuring ansible-pull cron entry
| Variable                    |Type     | Description                                                                   |Default       | Required |
|:----------------------------|:--------|:------------------------------------------------------------------------------|:-------------|:---------|
| ANSIBLE_CRON_SCHEDULE       | string  | Dictates the schedule to execute the ansible-pull command via cron. (Default: 15 minutes) | */15 * * * * | No |
| ANSIBLE_CRON_USER           | string  | Dictates who the cron job execute as. (Default: root)                         | root         | No       |
Dependencies
------------
ansible >= 2.15.0


Example Playbook
----------------

```yml
---
 - name: Implement ansible pull on managed hosts
   hosts: all

   roles:
    - role: ansible-pull
      state: present
      vars:
        ANSIBLE_TAGS:
          - tag1
          - tag2
        ANSIBLE_SKIP_TAGS:
          - tag3
          - tag4
```
Removing Ansible-Pull from hosts
```yml
---
 - name: Implement ansible pull on managed hosts
   hosts: all

   roles:
    - role: ansible-pull
      state: absent
```
-------

BSD

Author Information
------------------

David Lewellyn <david@thebarbariccrayon.com>
