Hetzner_debian11_ansible
=========

The purpose of this role is to create a server on the Hetzner Cloud infrastructure.

## how to define and use hetzner token.
* Put it in the vars.json (copy vars.example.json)
* install jq ([sed for json](https://stedolan.github.io/jq/)) on your server if you don't have it.
* initialize you token as a variable:
```
export HCLOUD_TOKEN=$(jq -r .hcloud_token vars.json)
```
* ansible_no_log: true|false
* 
## How to define all the hetzner information cloud in json files.
### Hetzner servers type informations
Hetzner servers types are all generated with the follwing command to create the json files:
```
hcloud server-type list -o noheader -o columns=name | while read SERVER; do  hcloud server-type describe $SERVER -o json >> vars/server-types-${SERVER}.json; done
```
### Hetzner location informations
Hetzner servers locations are all generated with the follwing command :
```
hcloud location list -o noheader -o columns=name| while read LOCATION; do hcloud location describe $LOCATION -o json > vars/location-${LOCATION}.json
```
### Hetzner iso images informations
Hetzner iso list is generated with the following command.
```
hcloud iso list -o noheader -o columns=id |while read ID; do hcloud iso describe $ID -o json; done > vars/hetzner-iso-list.json
```
Requirements
------------

none

Role Variables
--------------
```
---
hostname: "testserver"
 
# hcloud - Hetzner Cloud
h_server_location: "nbg1"
h_server_type: "cx11"
h_server_image: "ubuntu-18.04"
h_server_ssh_keys:
  - anna_beispiel_salbei_19_pub
  - peter_example_thymian_20_pub

h_volume_name: "testserver-data"
h_volume_size: 10   #in GB
h_volume_format: "ext4"
```
Dependencies
------------

none

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```
---
- name: create hcloud server with attached disk volume
  hosts: localhost
  connection: local
  gather_facts: false
  user: root
  vars_files:
    - vars/hcloud_token.yml
    - "../../../host_vars/{{ cloudserver }}.yml"   # there the host var file lives

  tasks:
  - name: list server
    hcloud_server_info:
      api_token: "{{ hcloud_token }}"
      name: "{{ cloudserver }}"

  - name: create server
    hcloud_server:
      api_token: "{{ hcloud_token }}"
      name: "{{ hostname }}"
      server_type: "{{ h_server_type }}"
      image: "{{ h_server_image }}"
      location: "{{ h_server_location }}"
      ssh_keys: "{{ h_server_ssh_keys }}"
      state: present
    register: server

  - name: attach disk volume
    hcloud_volume:
      api_token: "{{ hcloud_token }}"
      name: "{{ h_volume_name }}"
      size: "{{ h_volume_size }}"
      format: "{{ h_volume_format }}"
      server: "{{ server.hcloud_server.name }}"
      automount: no   # does not mount to the correct place
      state: present
    when: h_volume_size is defined

  - name: list server
    hcloud_server_info:
      api_token: "{{ hcloud_token }}"
      name: "{{ cloudserver }}"

  - name: print  status
    debug:
      msg:
        - "finished creating {{ server.hcloud_server.name }}"
        - "id: {{ server.hcloud_server.id }}"
        - "type:        {{ server.hcloud_server.server_type }}"
        - "status:      {{ server.hcloud_server.status }}"
        - "datacenter:  {{ server.hcloud_server.datacenter }}"
        - "{{ cloudserver }}  IN A     {{ server.hcloud_server.ipv4_address}}"
        - "{{ cloudserver }}  IN AAAA  {{ server.hcloud_server.ipv6 }}"
```
License
-------

BSD

Author Information
------------------

Patrice Le Guyader
