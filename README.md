# Ansible weareinteractive.sudo role

[![Build Status](https://img.shields.io/travis/weareinteractive/ansible-sudo.svg)](https://travis-ci.org/weareinteractive/ansible-sudo)
[![Galaxy](http://img.shields.io/badge/galaxy-weareinteractive.sudo-blue.svg)](https://galaxy.ansible.com/weareinteractive/users)
[![GitHub Tags](https://img.shields.io/github/tag/weareinteractive/ansible-sudo.svg)](https://github.com/weareinteractive/ansible-sudo)
[![GitHub Stars](https://img.shields.io/github/stars/weareinteractive/ansible-sudo.svg)](https://github.com/weareinteractive/ansible-sudo)

> `weareinteractive.sudo` is an [Ansible](http://www.ansible.com) role which:
>
> * installs sudo
> * configures sudo

**Note:**

> Since Ansible Galaxy supports [organization](https://www.ansible.com/blog/ansible-galaxy-2-release) now, this role has moved from `franklinkim.sudo` to `weareinteractive.sudo`!

## Installation

Using `ansible-galaxy`:

```shell
$ ansible-galaxy install weareinteractive.sudo
```

Using `requirements.yml`:

```yaml
- src: weareinteractive.sudo
```

Using `git`:

```shell
$ git clone https://github.com/weareinteractive/ansible-sudo.git weareinteractive.sudo
```

## Dependencies

* Ansible >= 2.5

## Variables

Here is a list of all the default variables for this role, which are also available in `defaults/main.yml`.

```yaml
---
# sudo_sudoers_user_aliases:
#  WEBADMINS:
#    - webadmin1
#    - webadmin2
# sudo_sudoers_cmnd_aliases:
#  WEBCOMMANDS:
#    - /bin/systemctl status nginx
#    - /bin/systemctl start nginx
#    - /bin/systemctl stop nginx
#    - /bin/systemctl restart nginx
#  PACKAGECOMMANDS: '/bin/apt, /bin/yum'
# sudo_defaults:
#  - defaults: env_reset
#  - name: user1
#    defaults: requiretty
# sudo_users:
#  - name: '%group1'
#  - name: 'bar'
#    nopasswd: yes
#  - name: '%group2'
#    commands: '/bin/ls'
#  - name: '%group3'
#    commands:
#      - /bin/ls
#      - /bin/df
#  - name: '%group4'
#    hosts: 127.0.0.1

# package name (version)
sudo_package: sudo
# list of username or %groupname
sudo_users: []
# list of username or %groupname and their defaults
sudo_defaults: []
# default sudoers file
sudo_sudoers_file: ansible
# list for additional sudoers files
sudo_sudoers_additional_files: []
# path of the sudoers.d directory
sudo_sudoers_d_path: /etc/sudoers.d
# delete other files in `sudo_sudoers_d_path`
purge_other_sudoers_files: no
# create logrotate for logfiles
sudo_logfiles: []
# they must be defined inside a default as well if you want sudo to use them
```
It is possible for sudo_logfiles to define a different rotate value, to define a new schedule or an mail_address. Look into the example below for that.
The default values are:
```
  rotate 24
  weekly
  compress
  missingok
  notifempty
```

## Usage

This is an example playbook:

```yaml
---

- hosts: all
  become: yes
  roles:
    - weareinteractive.sudo
  vars:
    sudo_sudoers_user_aliases:
      WEBADMINS:
        - webadmin1
        - webadmin2
    sudo_sudoers_runas_aliases:
      WEBUSERS: 'www-data, www'
    sudo_sudoers_cmnd_aliases:
      WEBCOMMANDS:
        - /bin/systemctl status nginx
        - /bin/systemctl start nginx
        - /bin/systemctl stop nginx
        - /bin/systemctl restart nginx
      PACKAGECOMMANDS: '/bin/apt, /bin/yum'
    sudo_defaults:
      - defaults: env_reset
      - defaults: secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
      - defaults: log_host, log_year, logfile="/var/log/sudo.log"
      - name: 'user1'
        defaults: 'requiretty'
        #type: user
      - name: '%group1'
        defaults: '!requiretty'
      - name: PAGER
        defaults: noexec
        type: cmnd
      - name: root
        defaults: '!set_logname'
        type: runas
    sudo_users:
      - name: 'user1'
      - name: 'user2'
        nopasswd: yes
      - name: '%group1'
        hosts: 127.0.0.1
      - name: '%group2'
        commands: '/bin/ls'
      - name: '%group3'
        commands:
          - '/usr/bin/ls'
          - '/usr/bin/df'
          - '/usr/bin/mailq'
      - name: '%group4'
        users: 'user1,user2'
        groups: 'group1,group2'
      - name: WEBADMINS
        commands: WEBCOMMANDS
        users: WEBUSERS
        groups: WEBUSERS
    purge_other_sudoers_files: yes

    sudo_logfiles:
      - path: /var/log/sudo.log
        rotate: 7
        rotate_schedule: weekly
        #mail_address:
    sudo_sudoers_additional_files:
      - web
    sudo_users_web:
      - name: 'webuser1'  

```
If you are going to make use of sudo_sudoers_additional_files then all the other variables are available like before, but you have to suffix them with the filename.
This is like in the upper example the name `web`.

## Testing

```shell
$ git clone https://github.com/weareinteractive/ansible-sudo.git
$ cd ansible-sudo
$ make test
```

## Contributing
In lieu of a formal style guide, take care to maintain the existing coding style. Add unit tests and examples for any new or changed functionality.

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

*Note: To update the `README.md` file please install and run `ansible-role`:*

```shell
$ gem install ansible-role
$ ansible-role docgen
```

## License
Copyright (c) We Are Interactive under the MIT license.
