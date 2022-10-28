# homelab-k8s-ha-cluster_components-kubeapi-server-loadbalancers

Playbook that deploys a highly available set of load balancers to sit in front of the master nodes of the Kubernetes cluster.

Developed for use with Ubuntu 22. Modify accordingly for other operating systems.

# Deployment

    Name your hosts in the hosts file with the same names you used in your Terraform files.
    Run:

```
ansible-playbook -i hosts playbook.yaml -k
```

# Sanity Check

The following commands are to be ran to check for sanity.

## Normal operation, both load-balancers online
On both load balancers, ensure that the keepalived service is up and  run the following command

```
ip addr show eth0
```

The output should look similar to this:

```
root@set1-kube-lb-01:~# ip addr show eth0 
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether ca:f8:5c:72:ec:14 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    altname ens18
    inet 10.98.68.11/8 brd 10.255.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 10.250.200.134/8 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::c8f8:5cff:fe72:ec14/64 scope link 
       valid_lft forever preferred_lft forever
```

```
root@set2-kube-lb-01:~# ip addr show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether b2:40:c3:a4:04:e3 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    altname ens18
    inet 10.98.68.21/8 brd 10.255.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b040:c3ff:fea4:4e3/64 scope link 
       valid_lft forever preferred_lft forever
```
## High Availability Triggered
On LB1, to mirror load balancer failure, execute

```
systemctl stop keepalived
```

On both load balancers, now the following command

```
ip addr show eth0
```

The output should look similar to this:

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether ca:f8:5c:72:ec:14 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    altname ens18
    inet 10.98.68.11/8 brd 10.255.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::c8f8:5cff:fe72:ec14/64 scope link 
       valid_lft forever preferred_lft forever
```

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether b2:40:c3:a4:04:e3 brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    altname ens18
    inet 10.98.68.21/8 brd 10.255.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 10.250.200.134/8 scope global secondary eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::b040:c3ff:fea4:4e3/64 scope link 
       valid_lft forever preferred_lft forever
```

As you can see, the Virtual IP has moved to the secondary Load Balancer.


