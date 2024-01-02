# Ansible role [redmine-1](https://galaxy.ansible.com/ui/standalone/roles/buluma/redmine-1/documentation)

Install Redmine on CentOS

|GitHub|Version|Issues|Pull Requests|Downloads|
|------|-------|------|-------------|---------|
|[![github](https://github.com/buluma/ansible-role-redmine-1/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-redmine-1/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-redmine-1.svg)](https://github.com/buluma/ansible-role-redmine-1/releases/)|[![Issues](https://img.shields.io/github/issues/buluma/ansible-role-redmine-1.svg)](https://github.com/buluma/ansible-role-redmine-1/issues/)|[![PullRequests](https://img.shields.io/github/issues-pr-closed-raw/buluma/ansible-role-redmine-1.svg)](https://github.com/buluma/ansible-role-redmine-1/pulls/)|[![Ansible Role](https://img.shields.io/ansible/role/d/buluma/redmine-1)](https://galaxy.ansible.com/ui/standalone/roles/buluma/redmine-1/documentation)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/buluma/ansible-role-redmine-1/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  vars:
    - centos_base_enable_epel: true
    - centos_base_fail2ban_configuration: false
    - centos_base_install_selinux_packages: true
    - centos_base_basic_packages: true
    - centos_base_security_packages: true
    - centos_base_firewalld: true
    - ruby_version: 2.3
    - ruby_install_globally: true
  roles:
    # - buluma.centos_base
    - bngsudheer.ruby
    # - buluma.ruby

- name: Converge
  hosts: all
  vars:
    - redmine_sql_database_name: "redmine"
    - redmine_sql_database_host: "localhost"
    - redmine_sql_username: "redmine"
    - redmine_sql_password: "password"
    - redmine_unicorn_port: 5777
    - redmine_configure_selinux: false
    - redmine_plugins:
        - name: scrum
          base_name: scrum
          url: https://redmine.ociotec.com/attachments/download/481/scrum-v0.18.1.tar.gz
          create_base_directory: false
        - name: timesheet
          base_name: timesheet
          url: https://github.com/Contargo/redmine-timesheet-plugin/archive/master.zip
          create_base_directory: false
    - mysql_databases:
        - name: redmine
    - mysql_users:
        - name: redmine
          password: password
          priv: "redmine.*:ALL"
    - redmine_additional_configuration: true
    - redmine_enable_smtp_email: true
    - redmine_smtp_settings_address: localhost
    - redmine_smtp_settings_port: 25
    - redmine_smtp_settings_authentication: plain
    - redmine_smtp_settings_domain: redmine.example.com
    - redmine_smtp_settings_user_name: redmine@redmine.example.com
    - redmine_smtp_settings_password: password
    - redmine_smtp_settings_enable_starttls_auto: true
  roles:
    # - buluma.mysql
    - buluma.redmine
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/buluma/ansible-role-redmine-1/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: yes
  gather_facts: no
  vars:
    ruby_version: 2.5

  tasks:
    # prepare
    - name: cp -rfT /etc/skel /root
      ansible.builtin.raw: |
        set -eu
        cp -rfT /etc/skel /root
      changed_when: false
      failed_when: false

    - name: setenforce 0
      ansible.builtin.raw: |
        set -eu
        setenforce 0
        sed -i 's/^SELINUX=.*$/SELINUX=permissive/g' /etc/selinux/config
      changed_when: false
      failed_when: false

    - name: systemctl stop firewalld.service
      ansible.builtin.raw: |
        set -eu
        systemctl stop firewalld.service
        systemctl disable firewalld.service
      changed_when: false
      failed_when: false

    - name: systemctl stop ufw.service
      ansible.builtin.raw: |
        set -eu
        systemctl stop ufw.service
        systemctl disable ufw.service
      changed_when: false
      failed_when: false

    - name: debian | apt-get install *.deb
      ansible.builtin.raw: |
        set -eu
        DEBIAN_FRONTEND=noninteractive apt-get install -y bzip2 ca-certificates curl gcc gnupg gzip hostname iproute2 passwd procps python3 python3-apt python3-jmespath python3-lxml python3-pip python3-setuptools python3-venv python3-virtualenv python3-wheel rsync sudo tar unzip util-linux xz-utils zip
      when: ansible_os_family | lower == "debian"
      changed_when: false
      failed_when: false

    - name: fedora | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      when: ansible_distribution | lower == "fedora"
      changed_when: false
      failed_when: false

    - name: redhat-9 | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum-config-manager --enable crb || echo $?
        yum-config-manager --enable codeready-builder-beta-for-rhel-9-x86_64-rpms || echo $?
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "9"
      changed_when: false
      failed_when: false

    - name: redhat-8 | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-dnf-plugin-versionlock python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-utils zip
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "8"
      changed_when: false
      failed_when: false

    - name: redhat-7 | yum install *.rpm
      ansible.builtin.raw: |
        set -eu
        subscription-manager repos --enable=rhel-7-server-optional-rpms || echo $?
        yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
        yum install -y bzip2 ca-certificates curl gcc gnupg2 gzip hostname iproute procps-ng python3 python3-jmespath python3-libselinux python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow-utils sudo tar unzip util-linux xz yum-plugin-versionlock yum-utils zip
      when: ansible_os_family | lower == "redhat" and ansible_distribution_major_version | lower == "7"
      changed_when: false
      failed_when: false

    - name: suse | zypper -n install *.rpm
      ansible.builtin.raw: |
        set -eu
        zypper -n install -y bzip2 ca-certificates curl gcc gpg2 gzip hostname iproute2 procps python3 python3-jmespath python3-lxml python3-pip python3-setuptools python3-virtualenv python3-wheel rsync shadow sudo tar unzip util-linux xz zip
      when: ansible_os_family | lower == "suse"
      changed_when: false
      failed_when: false

  roles:
    - role: buluma.bootstrap
    - role: buluma.epel
    - role: buluma.centos_base
    - role: bngsudheer.ruby
    - role: buluma.nginx
    # - role: buluma.mysql
```

Also see a [full explanation and example](https://buluma.github.io/how-to-use-these-roles.html) on how to use these roles.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/buluma/ansible-role-redmine-1/blob/master/defaults/main.yml):

```yaml
---
# defaults file for ansible-role-redmine
redmine_version: "3.4.11"
# redmine_runtime_directory: "/home/redmine/redmine-{{ redmine_version }}"
redmine_runtime_directory: "/srv/redmine/redmine"

redmine_sql_driver: mysql2
redmine_sql_database_name: "redmine"
redmine_sql_database_host: "localhost"
redmine_sql_username: "redmine"
redmine_sql_password: "password"

redmine_unicorn_worker_processes: 2

redmine_domain_name: redmine.example.com

redmine_configure_nginx: true
redmine_nginx_config_template: plain
redmine_nginx_custom_config_path:
redmine_configure_unicorn: true
redmine_configure_firewalld: true

redmine_unicorn_port: 5000
redmine_nginx_bind_ip:
redmine_plugins: []

redmine_configure_selinux: false
redmine_bundler_version: 1.16.1

redmine_additional_configuration: false
redmine_enable_smtp_email: false
redmine_smtp_settings_address: localhost
redmine_smtp_settings_port: 25
redmine_smtp_settings_authentication: plain
redmine_smtp_settings_domain: redmine.example.com
redmine_smtp_settings_user_name:
redmine_smtp_settings_password:
redmine_smtp_settings_enable_starttls_auto: false

redmine_default_data: false
redmine_language: en
redmine_ssl_certificate_path: "/etc/letsencrypt/live/{{ redmine_domain_name }}/fullchain.pem"
redmine_ssl_certificate_key_path: "/etc/letsencrypt/live/{{ redmine_domain_name }}/privkey.pem"
redmine_nginx_allowlist: false
redmine_nginx_allowlist_path: ''
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/buluma/ansible-role-redmine-1/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | Version |
|-------------|--------|--------|
|[buluma.bootstrap](https://galaxy.ansible.com/buluma/bootstrap)|[![Ansible Molecule](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-bootstrap/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-bootstrap.svg)](https://github.com/shadowwalker/ansible-role-bootstrap)|
|[buluma.epel](https://galaxy.ansible.com/buluma/epel)|[![Ansible Molecule](https://github.com/buluma/ansible-role-epel/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-epel/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-epel.svg)](https://github.com/shadowwalker/ansible-role-epel)|
|[buluma.centos_base](https://galaxy.ansible.com/buluma/centos_base)|[![Ansible Molecule](https://github.com/buluma/ansible-role-centos_base/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-centos_base/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-centos_base.svg)](https://github.com/shadowwalker/ansible-role-centos_base)|
|[bngsudheer.ruby](https://galaxy.ansible.com/buluma/bngsudheer.ruby)|[![Ansible Molecule](https://github.com/buluma/bngsudheer.ruby/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/bngsudheer.ruby/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/bngsudheer.ruby.svg)](https://github.com/shadowwalker/bngsudheer.ruby)|
|[buluma.mysql](https://galaxy.ansible.com/buluma/mysql)|[![Ansible Molecule](https://github.com/buluma/ansible-role-mysql/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-mysql/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-mysql.svg)](https://github.com/shadowwalker/ansible-role-mysql)|
|[buluma.nginx](https://galaxy.ansible.com/buluma/nginx)|[![Ansible Molecule](https://github.com/buluma/ansible-role-nginx/actions/workflows/molecule.yml/badge.svg)](https://github.com/buluma/ansible-role-nginx/actions/workflows/molecule.yml)|[![Version](https://img.shields.io/github/release/buluma/ansible-role-nginx.svg)](https://github.com/shadowwalker/ansible-role-nginx)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://buluma.github.io/) for further information.

Here is an overview of related roles:

![dependencies](https://raw.githubusercontent.com/buluma/ansible-role-redmine-1/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/buluma):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/repository/docker/buluma/enterpriselinux/general)|7|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/buluma/ansible-role-redmine-1/issues)

## [Changelog](#changelog)

[Role History](https://github.com/buluma/ansible-role-redmine-1/blob/master/CHANGELOG.md)

## [License](#license)

[license (Apache-2.0)](https://github.com/buluma/ansible-role-redmine-1/blob/master/LICENSE)

## [Author Information](#author-information)

[Shadow Walker](https://buluma.github.io/)

