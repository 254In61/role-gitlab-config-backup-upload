# role-gitlab

Role contains tasks that enable storage of device running configurations on gitlab.

# Requirements

- Role reads and already created file containing the device configurations.
- Minimum ansible 2.11

# Role Variables


- Variables are fed into the role during the play.
- Varibles are set in playbooks/vars/all.yml file
- See example below:

# Dependencies
- One of these roles has to run to build the file(s) that role-gitlab operates on.
- Roles expected :
    - role-cisco-config-backup
    - role-junos-config-backup
    - role-fortios-config-backup
    - role-f5-config-backup
    - role-linux-config-backup

# Example Playbook

## playbook/vars/all.yml
---
tmp_root_dir: /tmp
file_name: "{{ inventory_hostname }}-running-config.txt"
tmp_configs_store: "{{ tmp_root_dir }}/{{ file_name }}"

git_user: <git username>
git_user_email: <git user email address>


backups_git_repo_url: https://<git url>/<git username>/network-devices-configs-backup-store.git
backups_git_repo_dir: network-devices-configs-backup-store


cisco_dir: cisco      # cisco backups
junos_dir: junos      # junos backups
f5_dir: f5            # f5 backups
fortios_dir: fortios  # fortios backups
linux_dir: linux      # linux backups

...

## /playbook/linux-config-backup 

---
- name: Backup linux configurations
  hosts: linux-devices
  gather_facts: false

  pre_tasks:
    - name: Include Default Vars - environment agnostic
      ansible.builtin.include_vars:
        file: vars/all.yml

  tasks:
    - name: Import role-linux-config-backup role
      ansible.builtin.import_role:
        name: role-linux-config-backup
      vars:
        linux: true

    - name: Import gitlab role
      ansible.builtin.import_role:
        name: role-gitlab
      vars:
        git: true
        vendor_dir: "{{ linux_dir }}"

  post_tasks:
    - name: Debug out results
      ansible.builtin.debug:
        msg:
          - "configuration backup tasks completed"
...

# License
BSD

# Author Information
Name : Allan Maseghe

Company: Red Hat

Role: Consultant - Ansible
