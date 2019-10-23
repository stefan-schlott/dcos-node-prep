# STEP 1: Configure YUM correctly on all nodes: (all) 

variables are

host_group (mandatory)
yum_mirror_base_url (non mandatory, default value is `http://{{ hostvars[groups['bootstraps'][0]]['ansible_default_ipv4']['address'] }}:8085`)

`ansible-playbook <PLAYBOOK_PATH>/configure-local-repo-mirror.yaml -e 'host_group=all yum_mirror_base_url="http://172.16.12.19:8080"'`

validate with

`ansible all -m shell -a 'sudo cat /etc/yum.repos.d/local-repos.repo'`

`ansible all -m shell -a 'sudo yum repolist -v'`

# STEP 2: Clean Up all old/unused Docker images on all nodes

`ansible all -m shell -a 'sudo docker image prune --all --force'` 

Potentially with minimized parallelism (forks) `-f <number>`

# STEP 3: Update / Replace Docker 

* updates in node carefully selected node waves
* always do a pre-flight check (e.g. including `docker ps`)  on the running docker containers and check there state after the upgrades (e.g. if they are successfully re-scheduled onto another node) 

## Checks

* `docker ps` on the nodes - e.g. `ansible agents[0] -m shell -a 'sudo docker ps'`
* using the DC/OS UI and the Marathon defintions `apps.json` - especially looking for Containers running as type `DOCKER`
* identify Docker containers that are not launched by DC/OS and figure out how they could be relaunched after the Docker upgrade


## STEP 3a: Private Agents

Pre-Checks

`ansible-playbook <PLAYBOOK_PATH>/install_correct_docker_version.yaml -e 'host_group=wave1'`

Post-Checks

Pre-Checks

`ansible-playbook <PLAYBOOK_PATH>/install_correct_docker_version.yaml -e 'host_group=wave2'`

Post-Checks

... 

## STEP 3b: Public Agents

* do the public agents one a piece

`ansible-playbook <PLAYBOOK_PATH>/install_correct_docker_version.yaml -e 'host_group=54.191.190.239'`

## STEP 3c: Masters

* do one master at a time
* do the non-leading masters first

`ansible-playbook <PLAYBOOK_PATH>/install_correct_docker_version.yaml -e 'host_group=54.191.190.239'` -v

# STEP 4: validate success and configuration

* systemd running?

`ansible all -m shell -a 'sudo systemctl status docker | grep Active'`

* check necessary Docker parameters (version, storage backend overlay2, live restore disabled)

`ansible all -m shell -a 'sudo docker info | grep -i version'`
`ansible all -m shell -a 'sudo docker info | grep -i storage'`
`ansible all -m shell -a 'sudo docker info | grep -i live'`

* check responsiveness of Docker
* check running images
* check docker stats

`ansible all -m shell -a 'sudo docker ps'`
`ansible all -m shell -a 'sudo docker stats --no-stream'` 


