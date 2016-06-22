# Openstack all-in-one environment built and do spice-support development
## 1. Building Openstack all-in-one environment
### Start from devstack
* git clone https://git.openstack.org/openstack-dev/devstack

#### create a sudo group user (cloudy for me)
* adduser cloudy
* echo "cloudy ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

#### edit configure file `localrc` of ./stack.sh
`cd devstack/ && touch localrc`		
```c
# localhost
HOST_IP=192.168.110.151
DEST=/opt/stack
LOGFILE=/opt/stack/logs/stack.sh.log

ADMIN_PASSWORD=root
MYSQL_PASSWORD=root
RABBIT_PASSWORD=root
SERVICE_PASSWORD=root
SERVICE_TOKEN=root

# core compute (glance / keystone / nova (+ nova-network))
ENABLED_SERVICES=g-api,g-reg,key,n-api,n-crt,n-obj,n-cpu,n-net,n-cond,n-sch,n-spice,n-cauth

# neutron
#ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta

# cinder
#ENABLED_SERVICES+=,c-sch,c-api,c-vol

# dashboard
ENABLED_SERVICES+=,horizon

# additional services
ENABLED_SERVICES+=,rabbit,tempest,mysql

```






