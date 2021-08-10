# K8s Quick Reference

## Kubeadm basics

Initialise a cluster's master node with e.g.

```
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Join nodes to the cluster with e.g.

```
kubeadm join 192.168.0.33:6443 --token v17biz.i4zap5ddzl2amqqo --discovery-token-ca-cert-hash sha256:cc7b3217b25c337ad98b7018fc29dc4ad47cf39b6fd5ee79bca010f12f41b346
```

---

## Kubectl basics

List nodes in the k8s cluster.

```
$ kubectl get nodes
NAME    STATUS   ROLES                  AGE     VERSION
node1   Ready    control-plane,master   21m     v1.20.1
node2   Ready    <none>                 9m1s    v1.20.1
node3   Ready    <none>                 8m46s   v1.20.1
```

Add a deployment to the cluster based on a yaml definition.

```
kubectl apply -f example.yml
```

List all deployments on the cluster.

```
$ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
pingtest   3/3     3            3           28s
```

List all pods on the cluster.

```
$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP         NODE    NOMINATED NODE   READINESS GATES
pingtest-84646bf5d4-72l9d   1/1     Running   0          6m    10.5.1.3   node2   <none>           <none>
pingtest-84646bf5d4-sdzrn   1/1     Running   0          6m    10.5.1.2   node2   <none>           <none>
pingtest-84646bf5d4-twzld   1/1     Running   0          6m    10.5.1.4   node2   <none>           <none>
```

Execute commands on a pod, or use *-it* and *bash* to pop a shell.

```
$ kubectl exec -it pingtest-84646bf5d4-8rk97 -- bash
root@pingtest-84646bf5d4-8rk97:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1
    link/ipip 0.0.0.0 brd 0.0.0.0
4: eth0@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1480 qdisc noqueue state UP group default
    link/ether 06:88:19:94:23:84 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.5.1.3/24 brd 10.5.1.255 scope global eth0
       valid_lft forever preferred_lft forever
```

Delete the deployment using the same file.

```
kubectl delete -f example.yml
```

---

## Networking

Use Kube Router to deploy a [pod network](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy) on the cluster with e.g.

```
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```