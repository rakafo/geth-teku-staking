# Geth and Teku staking playbook

Made for hardware running Almalinux 8, i.e. [hetzner](https://www.hetzner.com/dedicated-rootserver/ex52-nvme).

Goals are readability, lower complexity due to limited customization, idempotency.

Meant for people who know ansible and want to automate their setup yet don't want to use complex/extensive implementations.

## Features

- prepare OS baseline:
  - enable firewalld, SEL;
  - configure SSH to only accept public keys and only 1 user;
- prepare teku keys -
  - create relevant dir structures;
  - validate files based on names;
  - set minimal viable permissions after files have been manually uploaded;
- prepare geth, teku:
  - install and configure;
  - designed to run multiple networks in parallel as all per-instance ports, dirs are set explicitly;
  - beacon and validator are separated for added isolation;
- prepare monitoring:
  - install and configure prometheus, node_exporter, alert_manager, grafana;
  - modify prometheus.yml to gather relevant scrape locations;
  - configure grafana to be accessible publicly via https (using nginx and letsencrypt);
  - alerts to email enabled based on [node_exporter quickstart](<(https://grafana.com/oss/prometheus/exporters/node-exporter/?tab=alerting-rules)>) - covers all major areas;
  - running as separate users for added isolation;
- all playbooks are idempotent;
- all variables are in vars/all.yml;

## Requirements

- ansible can login to host, use sudo;
- ssh is key-based;
- free infura account for quick teku initial synchronization;
- domain for grafana (created automatically in hetzner);
- email for alertmanager alerts;

# Using

- prepare vars and rename `vars/all example.yml` to `vars/all.yml`;

- prepare OS baseline - `ansible-playbook prepare_host.yml`;
- prepare geth - `ansible-playbook geth.yml`;
- prepare teku keys - `ansible-playbook teku-keys.yml`;
- prepare teku - `ansible-playbook teku.yml`;
- prepare monitorng - `ansible-playbook monitoring.yml`;

# Q/A

## what is manual and what is automated?

- [] create mnemonic phrase using https://github.com/ethereum/staking-deposit-cli/releases;
- [] upload deposit data using https://launchpad.ethereum.org/en/upload-deposit-data;
- [x] create restricted keystore structure for teku;
- [] upload keystore-X.json, keystore-X.txt to node;
- [x] prepare geth;
- [x] prepare teku;
- [x] prepare prometheus;
- [x] prepare grafana;
- [x] prepare nginx to serve grafana via https;
- [] customize grafana with dashboards;

## why almalinux and not X?

Meant to replace centos 8.

## staking

### geth or teku released a new version. What needs changing?

1. update relevant release dictionary to new values;
2. re-run relevant playbook;

If you need to revert, repeat the previous steps, just change the values in step 2 to old ones. Systemd services will be restarted.

### why is teku beacon and validator running separately?

Security benefits over running together:

- different user;
- different binary instance;
- validator communication restricted only to localhost;

## monitoring

### what are the recommended grafana dashboards?

- teku: https://grafana.com/grafana/dashboards/13457
- os brief: https://grafana.com/grafana/dashboards/13978
- os extensive: https://grafana.com/grafana/dashboards/1860

## security

### how to securely transfer keys without exposing them to copy/paste buffer?

one of the options - Ansible itself.

1. `dnf install ansible` - on node and environment where you're creating keys;
2. `ansible-vault encrypt keystore keystore-X.json` - password doesn't need to be guarded as it's used only temporarly;
3. copy/paste that value to node;
4. `ansible-vault decrypt keystore keystore-X.json`;

### what can be done to further increase security?

- setup server with full disk encryption: https://community.hetzner.com/tutorials/install-ubuntu-2004-with-full-disk-encryption;

- update periodically;
