Hetzner_debian11_ansible
=========

The purpose of this role is to create a server on the Hetzner Cloud infrastructure.

## how to define and use hetzner token.
* Put it in the vars.json (copy vars.example.json)
* Encrypt it with ansible-vault to get more security:
```
ansible-vault encrypt vars.json
New Vault password: 
Confirm New Vault password: 
Encryption successful
```
* install jq ([sed for json](https://stedolan.github.io/jq/)) on your server if you don't have it.
* initialize you token as a variable:
```
export HCLOUD_TOKEN=$(ansible-vault view vars.json|jq -r .hcloud_token)
```
The password entered previously for encrypting the file, will be asked. You can check the variable with the folowing command:
```
echo $HCLOUD_TOKEN
```
* ansible_no_log: true|false
* ansible_debug: true|false

## How to define all the hetzner information cloud in json files.
Instead of using the following commands. The same informations are available true the hetzner.hcloud commands. A new module 'hcloud_isos_info' has been created.
This module has been develooped following the module used to query servers types. It gathers all the informations about the ISO images availables. tu use it for the moment, you have to make a link to this files.
For example if the hetzner collection is installed in the directory `/usr/local/lib/python3.9/dist-packages/ansible_collections/hetzner`
```
ln -s hetzner_servers_ansible/modules/hcloud_isos_info.py /usr/local/lib/python3.9/dist-packages/ansible_collections/hetzner/hcloud/plugins/modules/
```

### Hetzner servers type informations
Hetzner servers types are all generated with the following command to create the json files:
```
echo "[" > vars/hetzner-server-types.json;hcloud server-type list -o noheader -o columns=name | while read SERVER; do  hcloud server-type describe $SERVER -o json;echo ","; done >> vars/hetzner-server-types.json;echo "]" >> vars/hetzner-server-types.json
```
### Hetzner location informations
Hetzner servers locations are all generated with the follwing command :
```
echo "[" > vars/hetzner-location-list.json;hcloud location list -o noheader -o columns=name| while read LOCATION; do hcloud location describe $LOCATION -o json;echo ",";done >> vars/hetzner-location-list.json;echo "]" >> vars/hetzner-location-list.json
```
### Hetzner iso images informations
Hetzner iso list is generated with the following command.
```
echo "[" > vars/hetzner-iso-list.json;hcloud iso list -o noheader -o columns=id |while read ID; do hcloud iso describe $ID -o json;echo ","; done >> vars/hetzner-iso-list.json;echo "]" >> vars/hetzner-iso-list.json
```
The last comma is not needed in each file, just before the last ]. Need to be deleted.

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
