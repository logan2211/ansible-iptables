## ansible-iptables

An ansible role to install and configure persistent iptables.

![Build Status](https://github.com/bbc/ansible-iptables/actions/workflows/main.yml/badge.svg?branch=master)

#### Requirements

##### Supported Operating Systems
* Ubuntu 16.04
* RHEL/CentOS 7
* Debian 9

#### Rule Definitions

iptables rule definitions can be configured several ways:
* vars file list loaded from `iptables_vars_files_extra`
* A list of rule definitions can be added to `iptables_rule_definitions`
* Lists of rule definitions are loaded from vars prefixed with `firewall_` (ex. `firewall_webserver`)
* A list of variables containing rule definition lists can be passed in `iptables_rule_vars`

Example rule definitions used in the default firewall can be seen in
`vars/firewall_base.yml`. An example rule definition may look like:

```yaml
firewall_webserver:
  - table: filter # filter, nat, raw, etc. (default: filter)
    protocol: ipv4 # Either ipv4 or ipv6 (default: ipv4)
    order: 99 # Lower order receives higher precedence in the firewall
    chains:
      - name: INPUT
        policy: DROP # Policy can only be defined on chains which support it.
      - name: WEBSERVER
    rules:
      - '-A INPUT -j WEBSERVER'
      - '-A WEBSERVER -p tcp -m multiport --dports 80,443 -j ACCEPT'
```

A different variable could then build on top of previous rules by using
the ordering attribute.
```yaml
firewall_webserver_https:
  - order: 100
    rules:
      - '-A WEBSERVER -p tcp --dport 8080 -j ACCEPT -m comment --comment Proxy'
```

Note: All of the attributes on rule definitions are _optional_.

#### Example Playbooks

##### Deploy a basic firewall blocking incoming traffic by default and only allowing ping and UDP traceroute.

```yaml
- hosts: all
  roles:
      - iptables
```

##### Deploy basic firewall with allow rules for http/https and smtp
```yaml
- hosts: webservers
  vars:
    firewall_webserver:
      - chains:
          - name: WEBSERVER
        rules:
          - '-A INPUT -j WEBSERVER'
          - '-A WEBSERVER -p tcp -m multiport --dports 80,443 -j ACCEPT'
    firewall_email:
      - rules:
          - '-A INPUT -p tcp --dport 25 -m comment --comment SMTP'
  roles:
    - iptables
```

##### Ordering rules allows you to provide higher precedence overrides, for example we will override the default firewall in `vars/firewall_base.yml` to change the INPUT chain policy to "ACCEPT" instead of "DENY".
```yaml
# The default order of '50' will override the firewall_base order value of '9999'.
- hosts: all
  vars:
    iptables_rule_definitions:
      - chains:
          - name: INPUT
            policy: ACCEPT
  roles:
    - iptables
```

#### License

Apache2

#### Author Information

Logan V.
