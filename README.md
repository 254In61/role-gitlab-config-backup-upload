# role-gitlab
Role contains tasks that enable storage of device running configurations on gitlab.

# Requirements
- Role reads and already created file containing the device configurations.
- Minimum ansible 2.11

# Role Variables
- Variables are fed into the role during the play.
- Varibles are set in playbooks/vars/all.yml file
- See example below

# How to use
- See example below

# Example vars file

all-vars.yml
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
-- name: Backup cisco configurations
  hosts: appc_cisco_mgmt_sw # From inventory/staging.ini
  gather_facts: false
  connection: network_cli

  vars:
    ansible_network_os: ios

  pre_tasks:
    - name: Include specific project variables
      ansible.builtin.include_vars:
        dir: group_vars
      delegate_to: localhost

  tasks:
    - name: Set file operations facts
      ansible.builtin.set_fact:
        tmp_config_store: "{{ root_dir }}/{{ config_backup_file }}"
        git_repo_vendor_dir: "{{ cisco_git_repo_dir }}"
      delegate_to: localhost

    - name: Import role-cisco-config-backup role
      ansible.builtin.import_role:
        name: role-cisco-config-backup
      vars:
        cisco: true
     
    - name: Import role-gitlab-config-backup-upload
      ansible.builtin.import_role:
        name: role-gitlab-config-backup-upload
      vars:
        gitlab: true   # bool for role use
        lfs: false     # bool for gitlab lfs

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
