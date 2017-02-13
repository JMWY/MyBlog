# Openstack all-in-one environment built and do spice-support development
## 1. Building Openstack all-in-one environment
### Start from devstack
* git clone https://git.openstack.org/openstack-dev/devstack

#### create a sudo group user (cloudy for me)
* sudo -i
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


#### [if want to extend compute node(s), use configure file(local.conf) as below]

```c
[[local|localrc]]
HOST_IP=192.168.110.154
DEST=/opt/stack
LOGFILE=/opt/stack/logs/stack.sh.log

MULTI_HOST=1

ADMIN_PASSWORD=root
MYSQL_PASSWORD=root
RABBIT_PASSWORD=root
SERVICE_PASSWORD=root
SERVICE_TOKEN=root

# Service information
SERVICE_HOST=192.168.110.151
MYSQL_HOST=$SERVICE_HOST
RABBIT_HOST=$SERVICE_HOST
GLANCE_HOSTPORT=$SERVICE_HOST:9292
Q_HOST=$SERVICE_HOST
KEYSTONE_AUTH_HOST=$SERVICE_HOST
KEYSTONE_SERVICE_HOST=$SERVICE_HOST

disable_all_services

# core compute (glance / keystone / nova (+ nova-network))
ENABLED_SERVICES=n-cpu,n-net

```

