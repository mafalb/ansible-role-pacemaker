# Pacemaker role for Ansible

## Requirements

This role has been written for and tested on Scientific Linux 7, so it should be applicable to Fedora 19 and later. It might also work in other distros, please share your experience.

## Role variables

## Required

#### `pacemaker_cluster_name`

Name of the cluster.

#### `pacemaker_password`

The plaintext password for the cluster user. It will be hashed with per-host salt to maintain idempotency.

## Optional

#### `pacemaker_hosts`

The list of cluster members.

Default: `ansible_play_batch`

#### `pacemaker_user`

The system user to authenticate PCS nodes with. PCS will authenticate all nodes with each other.

Default: hacluster

#### `pacemaker_properties`

The keys of this dict/hash with underscores correspond to pacemaker properties with hyphens.

**Be sure to quote cluster properties!** By default, YAML parser will guess variable types, so the string "false" will be converted to Boolean False and then to string "False". Pacemaker properties are case-sensitive, e.g. "stonith-enabled=False" will be accepted, but STONITH will still be on.

Correct example:

    pacemaker_properties:
      stonith_enabled: "false"

#### `pacemaker_resources`

An array of resource definitions. Each definition is a dict of two mandatory members, *id* (resource name) and *type* (standard:provider:type string, see output of *pcs resource providers*).

They can also have optional members like *options* dict, *op* list with operation actions and their options.

Additionally, there might be mutually exclusive members: Boolean *clone*, or dicts *masterslave* or *group* with their respective options. 

Finally, the values *disabled* and *wait* might be present.

For the detailed descriptions check out the resources below.

## Example playbooks

### Active-active chrooted BIND DNS server

    ---
    - name: Configure DNS cluster
      hosts: dns-servers
      tasks:
        - name: Setup cluster
          include_role:
            name: devgateway.pacemaker
          vars:
            pacemaker_password: hunter2
            pacemaker_cluster_name: named
            pacemaker_properties:
              stonith_enabled: "false"
            pacemaker_resources:
              - id: dns-ip
                type: "ocf:heartbeat:IPaddr2"
                options:
                  ip: 10.0.0.1
                  cidr_netmask: 8
                op:
                  - action: monitor
                    options:
                      interval: 5s
              - id: dns-srv
                type: "systemd:named-chroot"
                op:
                  - action: monitor
                    options:
                      interval: 5s
                clone: true
            pacemaker_constraints:
              - order dns-ip then dns-srv-clone

### Active-active Squid proxy

    ---
    - name: Configure Squid cluster
      hosts: proxy-servers
      tasks:
        - name: Setup cluster
          include_role:
            name: devgateway.pacemaker
          vars:
            pacemaker_password: hunter2
            pacemaker_cluster_name: squid
            pacemaker_properties:
              stonith_enabled: "false"
            pacemaker_resources:
              - id: squid-ip
                type: "ocf:heartbeat:IPaddr2"
                options:
                  ip: 192.168.0.200
                  cidr_netmask: 24
                op:
                  - action: monitor
                    options:
                      interval: 5s
              - id: squid-srv
                type: "systemd:squid"
                op:
                  - action: monitor
                    options:
                      interval: 5s
                clone: true
            pacemaker_constraints:
              - order squid-ip then squid-srv-clone

## See also

- [The official Pacemaker documentation](http://clusterlabs.org/doc/)
- *man pcs(8)*

## Copyright

Copyright 2015-2018, Development Gateway. Licensed under GPL v3+.
