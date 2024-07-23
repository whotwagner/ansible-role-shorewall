# Ansible Role: Shorewall

This role will assume the setup of [shorewall](http://shorewall.net/). It also supports IPv6

## Requirements

- Ansible 1.9.2+ ( might work with prior versions too)
- Debian-based linux-distribution
- shorewall (tested with 4.6)

## Dependencies

None.

## Installation

```
git clone https://github.com/whotwagner/ansible-role-shorewall.git
```

### Configuration example


#### Shorewall configuration with masquerading and dnat

```
    shorewall_startup: 1
    shorewall_ipforward: "On"
    shorewall_configs_dir: /etc/shorewall

    shorewall_configs:
      zones:
        - { name: inet, 
            type: ipv4,
            interface: { name: $INETIF, broadcast: detect, options: "routeback,nosmurfs" }
          }
        - { 
            name: lan, 
            type: ipv4,
            interface: { name: $LANIF, broadcast: detect, options: "routeback,bridge,nosmurfs" }
          }
      policy:
        - { source: fw,    dest: all,   policy: ACCEPT }
        - { source: lan,   dest: inet,  policy: ACCEPT }
        - { source: inet,  dest: inet,  policy: DROP }
        - { source: inet,  dest: fw,    policy: DROP }
        - { source: inet,  dest: lan,   policy: DROP }
        - THIS POLICY HAS TO BE THE LAST
        - { source: all,   dest: all,   policy: REJECT, log: info } 
      rules:
        - { action: DNAT, source: inet, dest: "lan:192.168.213.10:22", proto: tcp, dest_port: 10022 }
        - Permit access to SSH
        - { action: SSH/ACCEPT,  source: inet, dest: fw }
        - { action: ACCEPT,      source: inet, dest: fw, proto: tcp, dest_port: "443,8006" }
        - PING Rules
        - { action: Ping/ACCEPT, source: all, dest: all }
      snat:
        - { dest: $INETIF, source: 192.168.213.0/24 }
      params:
        - Interfaces
        - { name: INETIF, value: eth0 }
        - { name: LANIF, value: br0 }

```

#### Shorewall configuration with openvpn-tunnel

```
    shorewall_configs:
      zones:
        - This entry will is in the interfaces file for using the hosts-file
        - { name: "-", interface: { name: $OVPNIF } }
        - This entry will be added to the zones file
        - { name: vpn, type: ipv4 }
        - { name: inet, 
            type: ipv4,
            interface: { name: $INETIF, broadcast: detect, options: "routeback,nosmurfs" }
          }
        - { 
            name: lan, 
            type: ipv4,
            interface: { name: $LANIF, broadcast: detect, options: "routeback,bridge,nosmurfs" }
          }
      policy:
        - { source: fw,    dest: all,   policy: ACCEPT }
        - { source: lan,   dest: inet,  policy: ACCEPT }
        - { source: vpn,   dest: lan,   policy: ACCEPT }
        - { source: inet,  dest: inet,  policy: DROP }
        - { source: inet,  dest: fw,    policy: DROP }
        - { source: inet,  dest: lan,   policy: DROP }
        - THIS POLICY HAS TO BE THE LAST
        - { source: all,   dest: all,   policy: REJECT, log: info } 
      rules:
        - { action: DNAT, source: inet, dest: "lan:192.168.213.10:22", proto: tcp, dest_port: 10022 }
        - Permit access to SSH
        - { action: SSH/ACCEPT,  source: inet, dest: fw }
        - PING Rules
        - { action: Ping/ACCEPT, source: all, dest: all }
      snat:
        - { dest: $INETIF, source: 192.168.213.0/24 }
      params:
        - Interfaces
        - { name: INETIF, value: eth0 }
        - { name: LANIF, value: br0 }
        - { name: OVPNIF, value: "tun+" }
        - Nets
        - { name: VPNNET, value: "10.10.111.0/24" } 
      hosts:
        - Test
        - { zone: vpn, host: $OVPNIF:$VPNNET, options: "" }
      tunnels:
        - OpenVPN-Tunnel:
        - { type: "openvpn:1194", zone: ovpn, gateway: "0.0.0.0/0" }

```

#### Shorewall(IPv)6 configuration with 2 nets and smtp access
```
shorewall6_configs:
      rules:
        - { action: ACCEPT, source: "inet", dest: "lan:$MAIL", proto: tcp, dest_port: 25 }
      zones:
        - { name: inet, 
            type: ipv6,
            interface: { name: $INETIF, broadcast: detect, options: "routeback,nosmurfs" }
          }
        - { 
            name: lan, 
            type: ipv6,
            interface: { name: $LANIF, broadcast: detect, options: "routeback,bridge,nosmurfs" }
          }
      policy:
        - { source: fw,    dest: all,   policy: ACCEPT }
        - { source: lan,   dest: inet,  policy: ACCEPT }
        - { source: inet,  dest: inet,  policy: DROP }
        - { source: inet,  dest: fw,    policy: DROP }
        - { source: inet,  dest: lan,   policy: DROP }
        - THIS POLICY HAS TO BE THE LAST
        - { source: all,   dest: all,   policy: REJECT, log: info } 
      params:
        - Interfaces
        - { name: INETIF, value: eth0 }
        - { name: LANIF, value: br0 }
        - Hosts
        - { name: MAIL, value: "2607:f8b0:400c:c05::240" }
```
Of course it is possible to mix ipv4 and ipv6-configurations.

## Example playbook

    - hosts: servers
      roles:
         - { role: shorewall }

# Licence

MIT

# Author information

Wolfgang Hotwagner

Author of the forked Project: ELAO [**(http://www.elao.com/)**](http://www.elao.com)
