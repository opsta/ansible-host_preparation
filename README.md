Host preparation
=========

Ansible role to prepare Ubuntu host.
- Configure hostname (optional)
- Configure apt-cacher-ng (optional)
- Update all packages include kernel to latest
- Install some tools (see [Role Variables](#Role-Variables))
- Tuning system (see [Role Variables](#Role-Variables))
- Configure NTP
- Configure user to can use sudo without password
- Configure authorized_keys

You can see example how to make playbook, configuration and sample commands here https://github.com/winggundamth/ansible-wing-playbook

Requirements
------------

Before using this role. Please prepare your public key to be authorized_keys and configure ```host_preparation_authorized_keys_path``` to point to your authorized_keys files.

Role Variables
--------------

```yaml
# This is default variables
host_preparation_reboot_timeout: 600
host_preparation_ntp_server: ntp.ubuntu.com
host_preparation_sshd_config: /etc/ssh/sshd_config
host_preparation_sshd_port: 22
host_preparation_group_authorized_keys_path: "{{ playbook_dir }}/files/groups/{{ group_names[-1] }}/authorized_keys"
host_preparation_global_authorized_keys_path: "{{ playbook_dir }}/files/authorized_keys"
host_preparation_base_packages:
  - htop
  - iotop
  - sysstat
  - iftop
host_preparation_is_config_hostname: false
host_preparation_is_tuning: false
host_preparation_sysctl_vars:
  - { regexp: '^fs\.file-max \= ', line: 'fs.file-max = 1000000' }
  - { regexp: '^net\.ipv4\.tcp_max_syn_backlog \= ', line: 'net.ipv4.tcp_max_syn_backlog = 65535' }
  - { regexp: '^net\.ipv4\.tcp_tw_reuse \= ', line: 'net.ipv4.tcp_tw_reuse = 1' }
  - { regexp: '^net\.ipv4\.tcp_tw_recycle \= ', line: 'net.ipv4.tcp_tw_recycle = 1' }
  - { regexp: '^net\.ipv4\.ip_local_port_range \= ', line: 'net.ipv4.ip_local_port_range = 1024 65000' }
  - { regexp: '^net\.ipv4\.tcp_max_tw_buckets \= ', line: 'net.ipv4.tcp_max_tw_buckets = 400000' }
  - { regexp: '^net\.ipv4\.tcp_no_metrics_save \= ', line: 'net.ipv4.tcp_no_metrics_save = 1' }
  - { regexp: '^net\.ipv4\.tcp_rmem \= ', line: 'net.ipv4.tcp_rmem = 4096 87380 16777216' }
  - { regexp: '^net\.ipv4\.tcp_syn_retries \= ', line: 'net.ipv4.tcp_syn_retries = 2' }
  - { regexp: '^net\.ipv4\.tcp_synack_retries \= ', line: 'net.ipv4.tcp_synack_retries = 2' }
  - { regexp: '^net\.ipv4\.tcp_wmem \= ', line: 'net.ipv4.tcp_wmem = 4096 65536 16777216' }
  - { regexp: '^net\.core\.somaxconn \= ', line: 'net.core.somaxconn = 65535' }
  - { regexp: '^net\.core\.netdev_max_backlog \= ', line: 'net.core.netdev_max_backlog = 4096' }
  - { regexp: '^net\.core\.rmem_max \= ', line: 'net.core.rmem_max = 16777216' }
  - { regexp: '^net\.core\.wmem_max \= ', line: 'net.core.wmem_max = 16777216' }
  - { regexp: '^net\.nf_conntrack_max \= ', line: 'net.nf_conntrack_max = 1048576' }
  - { regexp: '^vm\.min_free_kbytes \= ', line: 'vm.min_free_kbytes = 65536' }
  - { regexp: '^vm\.overcommit_memory \= ', line: 'vm.overcommit_memory = 1' }
  - { regexp: '^vm\.swappiness \= ', line: 'vm.swappiness = 0' }
host_preparation_rc_vars:
  - { regexp: '^echo never > /sys/kernel/mm/transparent_hugepage/enabled', line: 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' }
  - { regexp: '^echo never > /sys/kernel/mm/transparent_hugepage/defrag', line: 'echo never > /sys/kernel/mm/transparent_hugepage/defrag' }
  - { regexp: '^ip link set eth0 txqueuelen ', line: 'ip link set eth0 txqueuelen 5000' }
host_preparation_limits_vars:
  - { regexp: '^\* soft nofile ', line: '* soft nofile 1000000' }
  - { regexp: '^\* hard nofile ', line: '* hard nofile 1000000' }
  - { regexp: '^\* soft nproc ', line: '* soft nproc 393216' }
  - { regexp: '^\* hard nproc ', line: '* hard nproc 393216' }

# This is optional variables
host_preparation_apt_cacher_ng: http://apt-cacher-ng.example.com:3142
```

Dependencies
------------

NA

Example Playbook
----------------

Since Ubuntu Xenial did't come with python 2 by default. So playbook need to install python 2 first without gathering facts.

```yaml
- hosts: all
  gather_facts: no
  become: true
  pre_tasks:
    - name: Install Python 2 first
      raw: python --version || apt-get update && apt-get install -y python
  roles:
    - winggundamth.host_preparation
  vars_files:
    - "{{ host_preparation_vars_file }}"
```

List of useful tags
----------------

There are some useful tags that you can use to maintain your Ubuntu host.

- host-preparation-apt-cacher-ng
- host-preparation-configure-hostname
- host-preparation-configure-ntp
- host-preparation-install-base-packages
- host-preparation-update-packages
- host-preparation-tuning
- host-preparation-configure-authorized_keys
- host-preparation-configure-user
- host-preparation-reboot (this need to configure ```host_preparation_need_reboot``` variable to true)

You can specific tag by using ```--tag``` for example if you only want to configure authorized_keys and limit to only production and database server group. You can run with command

```bash
ansible-playbook -i inventories/host_preparation --limit production:database \
--tag host-preparation-configure-authorized_keys host-preparation.yml
```

License
-------

MIT

Author Information
------------------

You can see my works at https://github.com/winggundamth
