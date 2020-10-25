# Kubernetes CKA Notes
###### tags: `kubernetes` `CKAD` `cheatsheet`
## Curriculum
### Cluster Architecture, Installation & Configuration - 25%
* Manage role based access control (RBAC)
* Use Kubeadm to install a basic cluster
* Manage a highly-available Kubernetes cluster
* Provision underlying infrastructure to deploy a Kubernetes cluster
* Perform a version upgrade on a Kubernetes cluster using Kubeadm
* Implement etcd backup and restore

### Workloads & Scheduling - 15%
* Understand deployments and how to perform rolling update and rollbacks
* Use ConﬁgMaps and Secrets to conﬁgure applications
* Know how to scale applications
* Understand the primitives used to create robust, self-healing, application deployments
* Understand how resource limits can affect Pod scheduling
* Awareness of manifest management and common templating tools

### Services & Networking - 20%
* Understand host networking conﬁguration on the cluster nodes
* Understand connectivity between Pods
* Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
* Know how to use Ingress controllers and Ingress resources
* Know how to conﬁgure and use CoreDNS
* Choose an appropriate container network interface plugin

### Storage - 10%
* Understand storage classes, persistent volumes
* Understand volume mode, access modes and reclaim policies for volumes
* Understand persistent volume claims primitive
* Know how to conﬁgure applications with persistent storage

### Troubleshooting - 30%
* Evaluate cluster and node logging
* Understand how to monitor applications
* Manage container stdout & stderr logs
* Troubleshoot application failure
* Troubleshoot cluster component failure
* Troubleshoot networking

## Materials
* [Udemy - Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)
* [Kodekloud - Certified Kubernetes Administrator (CKA) Practice Tests](https://kodekloud.com/courses/certified-kubernetes-administrator-with-practice-tests-labs)

## Notes & cheatsheet
### Kubernetes Architecture
Master (Manage, plan, Schedule, Monitor Nodes)
* kube-apiserver
* kube-scheduler
* etcd
* controller-manager

Worker Node (Host Application as Containers)
* kubelet

### Scheduling
#### Manual Schduling
Use nodeName in pod definition on spec node

#### Daemon Sets
Running the pod in all nodes and master like for example kube-proxy

```yaml
apiVersion:
kind:
metadata:
  name:
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

#### Replica Set
A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

#### Deployment
A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a *desired state* in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
### Tains et Tolerations

Assigner un label à un nœud
`kubectl label node <NODE NAME> <key>=<value>`

Supprimer un label d'un un nœud (rajouter le signe "-" à la fin)
`kubectl label node <NODE NAME> <key>-`

Afficher les labels d'un nœud 
`kubectl get nodes <NODE NAME> --show-labels`

Créer une Taint
```
kubectl taint nodes <NODE NAME> key=value:<taint-effect>
    Valeurs possibles de <taint-effect> : 
      - NoSchedule
      - PreferNoSchedule
      - NoExecute
```

Récupérer la liste des Taints disponibles dans notre cluster
`kubectl describe node [En option <NODE NAME>] | grep 'Name:\|Taints:'`

Supprimer une Taint à un nœud (rajouter le signe moins "-" à la fin)
`kubectl taint nodes <NODE NAME> key=value:<taint-effect>-`

### Monitor and Logging

### Rolling Updates & Rollbacks
Create
```shell=sh
kubectl create –f deployment-definition.yml
```
Get
```shell=sh
 kubectl get deployments
```
Update
```shell=sh
kubectl apply –f deployment-definition.yml
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
```
Status
```shell=sh
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```
Rollback
```shell=sh
kubectl rollout undo deployment/myapp-deployment
```

### ConfigMaps

```env
APP_COLOR: blue
APP_MODE: prod
```

Imperative
```shell=sh
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod
kubectl create configmap app-config --from-file=app_config.properties
```

Declarative
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

Get ConfigMaps
```shell=sh
kubectl get configmaps
kubectl describe configmaps
```

ConfigMaps in Pods
```yaml
envFrom:
  - configMapRef:
    name: app-config
```
```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

### Secrets

```env
DB_Host: mysql
DB_User: root
DB_Password: paswrd
```

```shell=sh
echo –n ‘bXlzcWw=’ | base64 --decode
echo –n ‘paswrd’ | base64
```
Imperative
```shell=sh
kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=paswrd
kubectl create secret generic app-secret --from-file=app_secret.properties
```

Declarative
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

Get Secrets
```sh
kubectl get secrets
kubectl describe secrets
```

Secrets in Pods
```yaml=
envFrom:
  - secretRef:
    name: app-config
```
```yaml
env:
  - name: DB_Password
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_Password
```
```yaml
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret
```

### Operating System Upgrade
```sh
kubectl drain node-1
kubectl cordon node-2
kubectl uncordon node-1
```

### Cluster Upgrade Process

Don't forget to drain node and start by the master

kubeadm
```shell=sh
kubectl upgrade plan
apt-get upgrade -y kubeadm=1.12.0-00
kubeadm upgrade apply v1.12.0
```

kubelet
```shell=sh
apt-get upgrade -y kubelet=1.12.0-00
systemctl restart kubelet
```

Verify
```shell=sh
kubectl get nodes
```

Uncordon the node
```shell=sh
kubectl uncordon <node-name>
```

### Backup and Restore
#### Resource Configuration Backup

```shell=sh
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

#### ETCD
##### Backup
```shell=sh
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

#### Restore

Verify the backup
```shell=sh
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

Check the actual configuration on the etcd.service to locate data-directory if needed
```shell=sh
# Example of etcd.service
ExecStart=/usr/local/bin/etcd \
--name ${ETCD_NAME} \
--cert-file=/etc/etcd/kubernetes.pem \
--key-file=/etc/etcd/kubernetes-key.pem \
--peer-cert-file=/etc/etcd/kubernetes.pem \
--peer-key-file=/etc/etcd/kubernetes-key.pem \
--trusted-ca-file=/etc/etcd/ca.pem \
--peer-trusted-ca-file=/etc/etcd/ca.pem \
--peer-client-cert-auth \
--client-cert-auth \
--initial-advertise-peer-urls https://${INTERNAL_IP}:2380
--listen-peer-urls https://${INTERNAL_IP}:2380 \
--listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379
--advertise-client-urls https://${INTERNAL_IP}:2379 \
--initial-cluster-token etcd-cluster-0 \
--initial-cluster controller-0=https://${CONTROLLER0_IP}:2380,controller
--initial-cluster-state new \
--data-dir=/var/lib/etcd
```

Execute the restore
```shell=sh
# Stop the kube-apiserver
service kube-apiserver stop
# Restore ETCD
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
--data-dir /var/lib/etcd-from-backup \
--initial-cluster master1=https://192.168.5.11:2380,master-2=https://192.168.5.12:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-advertise-peer-urls https://${INTERNAL_IP}:2380
# Update etcd.service data-dir and initial-cluster-token
# Reload the daemon
systemctl daemon-reload
# Restart etcd
service etcd restart
# Start the kube-apiserver
service kube-apiserver start
```

### Security
**TODO**
### Storage
**TODO**
### Networking
**TODO**
### Kubernetes Installation
**TODO**