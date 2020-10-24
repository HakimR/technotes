# Kubernetes CKA Notes
###### tags: `kubernetes` `CKAD` `cheatsheet`
## Tains et Tolerations

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