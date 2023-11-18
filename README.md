[![ansible-lint](https://github.com/sscheib/ansible-role-service_management/actions/workflows/ansible-lint.yml/badge.svg?branch=main)](https://github.com/sscheib/ansible-role-service_management/actions/workflows/ansible-lint.yml) [![Publish latest release to Ansible Galaxy](https://github.com/sscheib/ansible-role-service_management/actions/workflows/ansible-galaxy.yml/badge.svg)](https://github.com/sscheib/ansible-role-service_management/actions/workflows/ansible-galaxy.yml)

service_management
=========
Simple role to manage services of different service managers without losing the flexibility of using each individual module specialized on the respective service
manager (systemd, sysvinit, etc.). This allows using a central role which takes care of calling the correct service manager.

Currently supported service managers with a specific module:
- `systemd`
- `sysvinit` (not tested, but *should* work)
- `openwrt_init`

The following service managers are supported with the the `ansible.builtin.service` module:
- `BSD init`
- `OpenRC`
- `Solaris SMF`
- `upstart`

The detection of the used service manager on a system is done via `host_vars[inventory_hostname].ansible_facts.service_mgr`, so the user does not need to specify the
service manager.

All parameters are checked (`ansible.builtin.assert`) to ensure that they are set properly. The checks have been implemented based on the documentation of the respective
module.

I know that this seems counterintuitive, because it basically is not the way Python is used ('Ask for forgiveness and not for permission'), and respectively Ansible,
but in my humble opinion, this way ensures that the role is kept up-to-date and *works* as intended. Of course, this has the backdraw that it needs to be updated whenever a
new module option is implemented or the way of passing a certain argument has changed. I rather go by 'better safe than sorry' :slightly_smiling_face: 

Role Variables
--------------
| variable                                     | default                      | required | description                                                                    |
| :---------------------------------           | :--------------------------- | :------- | :----------------------------------------------------------------------------- |
| `svm_services`                               | unset                        | true     | List of services to manage. See below example for the exact definition.        |
| `svm_quiet_assert`                           | `false`                      | false    | Whether to quiet the assert statements                                         |

## Variable `svm_services`

The definition of the variable `svm_services` depends on the service manager that is going to get used. In any case at least `name` and one of `state` or `enabled` is
**required**.

### [`systemd`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_service_module.html#ansible-collections-ansible-builtin-systemd-service-module)
```
svm_services:
  - name: 'qemu-guest-agent'
    state: 'started'
    enabled: true
    daemon_reexec: false
    daemon_reload: false
    force: false
    masked: false
    no_block: false
    scope: 'system'
```

### [`sysvinit`](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/sysvinit_module.html#ansible-collections-ansible-builtin-sysvinit-module)
```
svm_services:
  - name: 'qemu-guest-agent'
    enabled: true
    state: 'started'
    arguments: '--additional-argument add-this-to-the-commandline'
    daemonize: true
    pattern: 'look for this string'
    runlevels:  # this is supposed to be a list of strings, but might also work as integers (not tested!)
      - '1'
      - '7'
      - '9'
    sleep: 1337
```

### [`openwrt_init`](https://docs.ansible.com/ansible/latest/collections/community/general/openwrt_init_module.html)
```
svm_services:
  - name: 'zabbix_agentd'
    state: 'started'
    enabled: true
    pattern: 'look for this string'
```

### [other service managers](https://docs.ansible.com/ansible/latest/collections/community/general/openwrt_init_module.html)
```
svm_services:
  - name: 'zabbix_agentd'
    state: 'started'
    enabled: true
    arguments: '--additional-argument add-this-to-the-commandline'
    pattern: 'look for this string'
    runlevel: 1
    sleep: 1337
```

Dependencies
------------

If you want to use `openwrt_init`, the [`community.general`](https://github.com/ansible-collections/community.general) collection is required.

Other than that: None.

Example Playbook
----------------

```
---
- name: 'Manage services'
  hosts: 'all'
  gather_facts: false
  roles:
    - role: 'service_management'
      vars:
        svm_services:
          - name: 'qemu-guest-agent'
            state: 'started'
            enabled: true
            daemon_reexec: false
            daemon_reload: false
            force: false
            masked: false
            no_block: false
            scope: 'system'
...
```

License
-------

GPL-2.0-or-later
