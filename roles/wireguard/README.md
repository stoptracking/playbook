# Wireguard
Configures Wireguard on the target machine and provisions arbitrary number of clients. This role uses "server" and "client" terminology which is while absent in Wireguard's dictionary (everyone is a "peer"), does make sense in many-to-one type of setup.

Role was made for and tested only on ipv4 setup. ipv6 is unsupported.

## Logic

Some notes to keep in mind when configuring it:

1. Neither `wg_clients` nor `wg_endpoints` are required.

   Former provisions peers that are connecting _to the target host_.

   Latter is for peers target host will attempt to connect _with_.

1. Role can be run:

  1. With both defined. Target host will accept connections from `wg_clients` and attempt to connect with `wg_endpoints`.

  1. With only `wg_clients`, target host will act as a VPN-server and route packets from clients within `AllowedIPs` in each `[Peer]` section.

  1. With only `wg_endpoints`, target host will connect to each server defined in all `[Peer]` sections.

  1. If neither is defined, target host can still be provisioned and Wireguard interface can be brought-up, however it won't accept any connections until valid peers are added to its config.

## Requirements

- Ansible ≥2.9
- Python ≥3.7
- Debian 11 (might work on its derivatives, however this is not guaranteed)
- [findif](https://github.com/savchenko/ansible-findif) module.
- `python3-netifaces`, version shipped with Debian is preferred.


## Role Variables
| Variable         | Description                                          | Default |
|------------------|------------------------------------------------------|---------|
| wg_autostart     | Autostart WG service?                                | True    |
| wg_clients       | List of clients that will connect to the target host | {}      |
| wg_endpoints     | List of endpoints target host will connect to        | {}      |
| wg_forward       | Forward between clients?                             | False   |
| wg_iface         | Interface name                                       | wg0     |
| wg_overwrite     | Overwrite existing config?                           | False   |
| wg_peerkey       | Server's private key, optional                       | -       |
| wg_port          | Server port                                          | 51820   |
| wg_server_ip     | Server IP address                                    | -       |
| wg_server_subnet | Server's subnet                                      | -       |
| wg_start         | Start WG at the end of play?                         | True    |
| wg_systemd       | Enable systemd unit on startup?                      | True    |


### wg_clients
Format is as following, `keepalive` is disabled if set to zero.
```yaml
wg_clients:
  - { name: 'MyClient', pubkey: '...xxxxxxx=', psk: '...xxxxxxx=', ip: '1.1.1.1', keepalive: '25' }
```


### wg_endpoints
Similar to the `wg_clients`:
```yaml
wg_endpoints:
  - { name: 'MyEndpoint', pubkey: '...xxxxxxx=', psk: '...xxxxxxx=', endpoint: '10.0.0.1:1234', keepalive: '25', allowedips: '10.0.0.0/24' }
```


### wg_server_subnet
Should be in CIDR notation, e.g. `192.168.1.0/24`.


## Example playbook
```yaml
---
- hosts: myhost
  tasks:

  - name: Execute `wireguard` role
    include_role:
      name: wireguard
      apply:
        tags:
          - role_wireguard
    tags:
      - always
    vars:
      wg_autostart: true
      wg_clients:
        - { name: 'ClientA', pubkey: 'WC5a0cSWATZwLsQH1REjGcQsomZyEtRTaAqyRK20emY=', psk: 'yKsg+u7zY9N5rjgR3YE6cSu90aFBpxEPA3uKzK9vJc0=', ip: '192.168.1.20', keepalive: '0' }
        - { name: 'ClientB', pubkey: 'qOwMjZuaVjfmdCSoOQbTp8MPADN+yB5gjRAmddA7jUM=', psk: 'Fo1UMMLxbyPiYwrJ/vE7tqPa66gblskjIc5xyQPsXgg=', ip: '192.168.1.30', keepalive: '33' }
      wg_forward: true
      wg_iface: wg5
      wg_overwrite: true
      wg_peerkey: wIZPnbsM9lto1MSD2J.22.gbIsSHmp1LFu96IGX2310=
      wg_port: 12344
      wg_server_ip: 192.168.1.1
      wg_server_subnet: '192.168.1.0/24'
      wg_start: true
      wg_systemd: true
```


## Dependencies
Tested on machine that is already provisioned with the [base role](https://github.com/savchenko/debian/blob/bullseye/roles/base/README.md).

Likely to complain about missing packages if executed against untouched minimal installation.


## License
MIT


## Author Information
Andrew Savchenko

https://savchenko.net
