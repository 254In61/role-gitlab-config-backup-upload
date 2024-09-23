# role-gitlab
- Role contains tasks that enable storage of device running configurations on gitlab.

# Requirements
- Role reads and already created file containing the device configurations.

- Minimum ansible 2.11

# Role Variables
- Variables are fed into the role during the play.

- Varibles are set in group_vars/all.yml file

- See example below

# How to use
- See example below

# Example vars file

! all-vars.yml
---
git_user: "{{ lookup('env', 'ANSIBLE_NET_USERNAME') }}"

git_pwd: "{{ lookup('env', 'ANSIBLE_NET_PASSWORD') }}"

git_user_email: amaseghe@redhat.com

root_dir: /tmp

repo_dir: network-devices-configs-backup-store

namespace: redhat

git_repo: "https://{{ git_user }}:{{ git_pwd }}@gitlab.vodafone.com.au/{{ namespace }}/{{ repo_dir }}.git"

cisco_git_repo_dir: cisco

...

## example playbook file
---
- name: Backup f5 Configuration

  hosts: appc_bigip_lb

  gather_facts: false

  connection: local

  vars:

    provider:

      server: "{{ ansible_host }}"

      user: "{{ f5_user }}"

      password: "{{ f5_password }}"

      validate_certs: false

      server_port: "{{ f5_access_port }}"

      timeout: 600

  pre_tasks:
    - name: Include specific project variables

      ansible.builtin.include_vars:

        dir: group_vars

      delegate_to: localhost

  tasks:

    - name: Set file operations facts

      ansible.builtin.set_fact:
        tmp_config_store: "{{ root_dir }}/{{ config_backup_file }}"

        git_repo_vendor_dir: "{{ f5_git_repo_dir }}"

        ucs_filename: "{{ f5_ucs_file_name }}"

      delegate_to: localhost

    - name: Import role-f5-config-backup role - config backup tasks

      ansible.builtin.import_role:

        name: role-f5-config-backup

      vars:

        f5_config: true

        f5_ucs: false
   
    - name: Import role-gitlab-config-backup-upload - upload running config backup

      ansible.builtin.import_role:

        name: role-gitlab-config-backup-upload

      vars:

        normal_file: true

        large_file: false

    - name: Import role-f5-config-backup role - ucs file backup tasks

      ansible.builtin.import_role:

        name: role-f5-config-backup

      vars:

        f5_config: false

        f5_ucs: true

    - name: Import role-gitlab-config-backup-upload - upload ucs file

      ansible.builtin.import_role:

        name: role-gitlab-config-backup-upload

      vars:

        normal_file: false

        large_file: true
    
  post_tasks:

    - name: Debug out results

      ansible.builtin.debug:

        msg:

          - "configuration backup tasks completed"
          
      delegate_to: localhost
...

# License
BSD

# Author Information
Name : Allan Maseghe

Company: Red Hat

Role: Consultant - Ansible
