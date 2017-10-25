# prometheus_alertmanager

Master: [![Build Status](https://travis-ci.org/sansible/prometheus_alertmanager.svg?branch=master)](https://travis-ci.org/sansible/prometheus_alertmanager)  
Develop: [![Build Status](https://travis-ci.org/sansible/prometheus_alertmanager.svg?branch=develop)](https://travis-ci.org/sansible/prometheus_alertmanager)

* [ansible.cfg](#ansible-cfg)
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
* [Examples](#examples)

This roles installs Prometheus Alert Manager.

Prometheus Alert Manager sends alerts on behalf of Prometheus server.

For more information about Prometheus Alert Manager please visit:
[https://github.com/prometheus/alertmanager](https://github.com/prometheus/alertmanager).
[https://prometheus.io/docs/alerting/alertmanager/](https://prometheus.io/docs/alerting/alertmanager/).

For more information about Prometheus please visit:
[https://prometheus.io/docs/](https://prometheus.io/docs/).

Note: The service script uses gosu to daemonise as the start/stop scripts in Ubuntu caused permission issues when trying to access the "data" directory.

## ansible.cfg

This role is designed to work with merge "hash_behaviour". Make sure your
ansible.cfg contains these settings

```INI
[defaults]
hash_behaviour = merge
```

## Installation and Dependencies

This role will install `sansible.users_and_groups` for managing `prometheus_alertmanager` user.

To install run `ansible-galaxy install sansible.prometheus_alertmanager` or add this to your `roles.yml`

```YAML
- name: sansible.prometheus_alertmanager
  version: v1.0
```

and run `ansible-galaxy install -p ./roles -r roles.yml`

## Tags

This role uses two tags: **build** and **configure**

* `build` - Installs Prometheus Server and all it's dependencies.
* `configure` - Configure and ensures that the Prometheus Server service is running.


## Examples

For more detailed configuration information please refer to [https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml](alertmanager)

Defaults:

smtp host: localhost:25
smtp from: ansible@prometheus
prometheus alert manager port: 9093

Prometheus Alert Manager configured to send emails to the "default-email" receivers for all services that trigger a critical alert.

```YAML
- name: Install and configure Prometheus Alert Manager
  hosts: "{{ host }}"

  roles:
    - role: sansible.prometheus_alertmanager
      sansible_prometheus_alertmanager:
        config:
          route:
            receiver: default-email
            group_wait: 30s
            group_interval: 5m
            repeat_interval: 20m
            group_by:
              - alertname
              - role
            routes:
              - match_re:
                  service: ^.*$
                receiver: default-email
                routes:
                  - match:
                      severity: critical
                    receiver: default-email
            receivers:
              - name: default-email
                email_configs:
                  - to: "support@example.com"
```
         
          
Prometheus Alert Manager configured to use an the mail server "mail.example" on port 465 and receiver "web-email-alert".
An alert is defined for the service "webapp", which will send an email to receiver group "web-email-alert" if a critical alert is triggered and will not send a "warning" message if the webapp service is already in a critical alert state.

```YAML
- name: Install and configure Prometheus Alert Manager
  hosts: "{{ host }}"

  roles:
    - role: sansible.prometheus_alertmanager
      sansible_prometheus_alertmanager:
        config:
          global: 
            smtp_smarthost: mail.example:465
            smtp_from: username@example.com
          routes:
            - match:
                service: webapp
              receiver: web-email-alert
              routes:
                - match:
                    severity: critical
                  receiver: web-email-alert
          inhibit_rules:
            - source_match:
                severity: critical
              target_match:
                severity: warning
              equal:
                - alertname
                - role
          receivers:
            - name: web-email-alert
              email_configs:
                - "support@example.com"
                - "devops@example.com"

```
