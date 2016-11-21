# cicd-deploy-consul
##deploy consul cluster

This simple playbook sets up a consul cluster based on consul docker image. Playbook deploys the consul docker containers in server mode and web ui in client mode. These docker containers use the docker host network for communication. Consul cluster can be accessed externall with the http://<web ui external IP>:8500 or via curl. 

**Master**
```
docker run -d --name consul --net=host \
    -p $PRIVATE_IP:8300:8300 -p $PRIVATE_IP:8301:8301 -p $PRIVATE_IP:8301:8301/udp \
    -p $PRIVATE_IP:8302:8302 -p $PRIVATE_IP:8302:8302/udp -p $PRIVATE_IP:8400:8400 \
    -p $PRIVATE_IP:8500:8500 -p $BRIDGE_IP:53:53/udp \
    consul agent -server -advertise=$PRIVATE_IP -bind=$PRIVATE_IP -bootstrap-expect=$CLUSTER_SIZE
```

**Member servers**
```
docker run -d --name consul --net=host  \
    -p $PRIVATE_IP:8300:8300 -p $PRIVATE_IP:8301:8301 -p $PRIVATE_IP:8301:8301/udp \
    -p $PRIVATE_IP:8302:8302 -p $PRIVATE_IP:8302:8302/udp -p $PRIVATE_IP:8400:8400 \
    -p $PRIVATE_IP:8500:8500 -p $BRIDGE_IP:53:53/udp \
    consul agent -server -advertise=$PRIVATE_IP -bind=$PRIVATE_IP -bootstrap-expect=$CLUSTER_SIZE -retry-join={{ master_ip.stdout }}
```

**Web UI Client**
```
docker run -d --name consul --net=host  \
    -p $PRIVATE_IP:8300:8300 -p $PRIVATE_IP:8301:8301 -p $PRIVATE_IP:8301:8301/udp \
    -p $PRIVATE_IP:8302:8302 -p $PRIVATE_IP:8302:8302/udp -p $PRIVATE_IP:8400:8400 \
    -p $PRIVATE_IP:8500:8500 -p $BRIDGE_IP:53:53/udp \
    consul agent -ui -bind=$PRIVATE_IP -retry-join={{ master_ip.stdout }} -client=$PRIVATE_IP
```

The cluster master is brought up and bootstrap. Then the other members are brought up and join the master to form the cluster. A consul client is then brought up with --client option, join the cluster and offer the web ui.


Playbook needs master's and web ui client's FQDN, size of the cluster in group_vars/all and a valid inventory file.

To execute the create cluster playbook, 
ansible-playbook create-consul-cluster.yml -i inventory.yml -u root -k

To delete consul cluster use, 
ansible-playbook destroy-consul-cluster.yml -i inventory.yml -u root -k


## Examples
- adding kv to consul
```
curl -X PUT -d 'wasantha gamage' http://1.2.3.4:8500/v1/kv/wasantha/name
```
- lookup kv values in consul
```
wasantha@wasantha-ThinkPad-P50 ~/code/deploy-consul/ansible $ curl 1.2.3.4:8500/v1/kv/wasantha/name
[{"LockIndex":0,"Key":"wasantha/name","Flags":0,"Value":"d2FzYW50aGEgZ2FtYWdl","CreateIndex":610,"ModifyIndex":610}]
wasantha@wasantha-ThinkPad-P50 ~/code/deploy-consul/ansible $ 
wasantha@wasantha-ThinkPad-P50 ~/code/deploy-consul/ansible $ echo "d2FzYW50aGEgZ2FtYWdl" | base64 --decode
wasantha gamage
wasantha@wasantha-ThinkPad-P50 ~/code/deploy-consul/ansible $
```
- lookup the state of the consul cluster
```
wasantha@wasantha-ThinkPad-P50 ~/code/deploy-consul/ansible $ curl 1.2.3.4:8500/v1/catalog/nodes
```
