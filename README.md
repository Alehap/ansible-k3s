# ansible-k3s
Ansible for K3s cluster HA

# Notes
This cluster setup with ETCD-HA, so you can add/remove etcd node from cluster easily by following this steps:

Get etcd member ID

```
export ETCDCTL_API=3
HOST_1=MASTER.IP.1
HOST_2=MASTER.IP.2
HOST_3=MASTER.IP.3
```

```
# etcdctl --endpoints=${HOST_1}:2380,${HOST_2}:2380,${HOST_3}:2380 --write-out=table member list

+------------------+---------+----------+----------------------------+----------------------------+------------+
|        ID        | STATUS  |   NAME   |         PEER ADDRS         |        CLIENT ADDRS        | IS LEARNER |
+------------------+---------+----------+----------------------------+----------------------------+------------+
| 340f7aa39axxxx7e | started |   node-1 |      http://1.21.33.1:2380 |      http://1.21.33.1:2379 |      false |
| b17375682xxxxa07 | started |   node-2 |    http://1.108.205.2:2380 |    http://1.108.205.2:2379 |      false |
| f94ff79e08xxxx02 | started |   node-3 |     http://1.109.59.3:2380 |     http://1.109.59.3:2379 |      false |
+------------------+---------+----------+----------------------------+----------------------------+------------+
```

Remove ETCD node from cluster: (For example, I need to remove node-1)
```
# MEMBER_ID=340f7aa39axxxx7e
# etcdctl --endpoints=${HOST_1}:2380,${HOST_2}:2380,${HOST_3}:2380 member remove ${MEMBER_ID}
Member 340f7aa39axxxx7e removed from cluster  6ab75xxxx4fafed
```

Add new ETCD node to cluster:
```
NEW_NODE_NAME="node-4"
NEW_NODE_IP="1.2.3.4"
etcdctl --endpoints=${HOST_2}:2379,${HOST_3}:2379 member add ${NEW_NODE_NAME} --peer-urls=http://${NEW_NODE_IP}:2380

```

One the new node, you need to update the etcd config as below:

```
root@node-4 ~ # cat /etc/etcd/etcd.conf.yaml
data-dir: /var/lib/etcd/node-4.etcd
name: node-4
initial-advertise-peer-urls: http://1.2.3.4:2380
listen-peer-urls: http://1.2.3.4:2380,http://127.0.0.1:2380
advertise-client-urls: http://1.2.3.4:2379
listen-client-urls: http://1.2.3.4:2379,http://127.0.0.1:2379
initial-cluster-state: existing
initial-cluster: node-2=http://1.108.205.2:2380,node-3=http://1.21.33.3:2380,node-4=http://1.2.3.4:2380
```

And restart the etcd service.

FAQ etcd service: https://etcd.io/docs/v3.4/faq/
