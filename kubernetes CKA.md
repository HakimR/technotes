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