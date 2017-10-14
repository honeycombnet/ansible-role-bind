# Ansible role: Bind
[![Build Status](https://travis-ci.org/honeycombnet/ansible-role-bind.svg?branch=master)](https://travis-ci.org/honeycombnet/ansible-role-bind)

BIND with DNSSEC signed zones.

## Requirements

None

## Role variables

* `bind_role` - the role `master` or `slave`, don't generate dnssec key on `slave`
* `bind_slave_host_group` - name of the host group that make up the DNS slaves
* `bind_master_host_group` - name of the host group that make up the DNS masters
* `bind_zones` - the dns zones

## How to use

Inventory:

```
all:
  children:
    dns:
      children:
        dns-master:
          hosts:
            ns1.example.com
        dns-slave:
          hosts:
            ns2.example.com
```

`group_vars/dns-master.yml`:

```
bind_role: master
bind_slave_host_group: dns-slave
bind_zones:
  test.local:
    ns_primary: ns1.test.local
    mail: root@test.local
    serial: 2017092202
    dnssec: yes
    entries:
      - { name: '@', type: ns, value: localhost. }
      - { name: hello, type: a, value: 1.2.3.4 }
  hello.local:
    ns_primary: ns1.hello.local
    mail: root@hello.local
    serial: 2017092201
    dnssec: no
    entries:
      - { name: '@', type: ns, value: localhost. }
      - { name: hello, type: a, value: 4.3.2.1 }
```

`group_vars/dns-slave.yml`:

```
bind_role: slave
bind_slave_host_group: dns-master
bind_zones:
  test.local:
    ns_primary: ns1.test.local
  hello.local:
    ns_primary: ns1.hello.local
```

Playbook:

```
- hosts: dns
  roles:
    - bind 
```

### DS records

When a new zone is created using this role, it will generate new private keys to sign the zone.

You must then add the newly-generated DS records to the parent DNS zone.
(Usually this means giving it to your domain registrar.)

These DS records are located on the master server at `/etc/bind/keys/dsset-ZONE`

## Development
### Tests with docker

  * install [docker](https://docs.docker.com/engine/installation/)
  * install dependencies `bundle install`
  * run the tests `rake`
