---
title: "Tests et validation de Karpenter"
date: 2026-01-16
lastmod: 2026-01-16
draft: false
showToc: true
TocOpen: true
searchHidden: true
---

# Tests et validation de Karpenter

Ce guide fournit des exemples concrets pour tester Karpenter et valider son bon fonctionnement sur votre cluster EKS.

> **Prérequis** : Avoir Karpenter installé et opérationnel. Voir [Installation de Karpenter](/pro/aws/karpenter-installation/).

---

## Vérification préalable

Avant de lancer les tests, vérifiez que Karpenter est opérationnel :

```bash
# Vérifier les pods Karpenter
kubectl get pods -n kube-system -l app.kubernetes.io/name=karpenter

# Vérifier le NodePool
kubectl get nodepool

# Vérifier l'EC2NodeClass
kubectl get ec2nodeclass
```

**Status attendu :**
- Pods : `2/2 Running`
- NodePool : `READY: True`
- EC2NodeClass : `READY: True`

### Exemple de sortie

```
NAME                         READY   STATUS    RESTARTS   AGE
karpenter-5f79bf99b8-5q7n2   1/1     Running   0          15h
karpenter-5f79bf99b8-wwfq2   1/1     Running   0          15h
```

```
NAME                            NODECLASS   NODES   READY   AGE
nodepool.karpenter.sh/default   default     0       True    15h

NAME                                     READY   AGE
ec2nodeclass.karpenter.k8s.aws/default   True    15h
```

> **Note** : `NODES: 0` est normal - Karpenter ne crée des nodes que quand des pods sont en attente.

---

## Test 1 : Provisioning automatique de nodes

### Objectif
Vérifier que Karpenter crée automatiquement des nodes quand des pods ne peuvent pas être schedulés.

### Étape 1 : Création du deployment de test

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inflate
spec:
  replicas: 5
  selector:
    matchLabels:
      app: inflate
  template:
    metadata:
      labels:
        app: inflate
    spec:
      containers:
        - name: inflate
          image: public.ecr.aws/eks-distro/kubernetes/pause:3.7
          resources:
            requests:
              cpu: "1"
              memory: "1.5Gi"
EOF
```

> **Explication** : Chaque pod demande 1 CPU et 1.5 Gi de mémoire. Avec 5 replicas, les nodes existants (t3.medium avec ~2 vCPU) ne peuvent pas tous les héberger.

### Étape 2 : Observer en temps réel

Ouvrez plusieurs terminaux pour observer le comportement :

```bash
# Terminal 1 : Observer les nodes
kubectl get nodes -w

# Terminal 2 : Observer les pods
kubectl get pods -l app=inflate -w

# Terminal 3 : Logs Karpenter
kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter -f
```

### Étape 3 : Observer les pods Pending

```bash
kubectl get pods -l app=inflate -o wide
```

**Résultat immédiat :**
```
NAME                       READY   STATUS    NODE
inflate-5cf78d58f6-9lfgr   0/1     Pending   <none>
inflate-5cf78d58f6-ghv7s   0/1     Pending   <none>
inflate-5cf78d58f6-jh7g4   1/1     Running   ip-10-0-2-228.eu-west-1.compute.internal
inflate-5cf78d58f6-nvtsp   1/1     Running   ip-10-0-36-183.eu-west-1.compute.internal
inflate-5cf78d58f6-wzkvg   0/1     Pending   <none>
```

> **Observation** : 2 pods Running sur les nodes existants, 3 pods Pending (pas assez de ressources).

### Étape 4 : Karpenter détecte et crée un NodeClaim

```bash
kubectl get nodeclaims
```

**Résultat :**
```
NAME            TYPE         CAPACITY    ZONE         NODE   READY     AGE
default-4tw4v   t3a.xlarge   on-demand   eu-west-1b          Unknown   16s
```

> **Observation** : Karpenter a automatiquement créé un NodeClaim pour un t3a.xlarge (4 vCPU, 16 Gi RAM).

### Étape 5 : Analyser les logs Karpenter

```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter --tail=20 | grep -E "(found|computed|created|launched|registered)"
```

**Logs clés :**
```json
{"message":"found provisionable pod(s)","Pods":"default/inflate-xxx, default/inflate-yyy, default/inflate-zzz"}
{"message":"computed new nodeclaim(s) to fit pod(s)","nodeclaims":1,"pods":3}
{"message":"created nodeclaim","NodePool":{"name":"default"},"NodeClaim":{"name":"default-4tw4v"}}
{"message":"launched nodeclaim","instance-type":"t3a.xlarge","zone":"eu-west-1b","capacity-type":"on-demand"}
{"message":"registered nodeclaim","Node":{"name":"ip-10-0-25-184.eu-west-1.compute.internal"}}
```

**Explication du cycle :**
1. **found provisionable pod(s)** : Karpenter détecte 3 pods en attente
2. **computed new nodeclaim** : Calcule qu'1 node suffit pour 3 pods
3. **created nodeclaim** : Crée la demande avec les besoins
4. **launched nodeclaim** : Lance l'instance EC2
5. **registered nodeclaim** : Le node rejoint le cluster Kubernetes

### Étape 6 : Vérification finale

```bash
kubectl get nodes
```

**Résultat :**
```
NAME                                        STATUS   ROLES    AGE   VERSION
ip-10-0-2-228.eu-west-1.compute.internal    Ready    <none>   20h   v1.33.5-eks-ecaa3a6
ip-10-0-25-184.eu-west-1.compute.internal   Ready    <none>   27s   v1.33.5-eks-ecaa3a6  <-- NOUVEAU
ip-10-0-36-183.eu-west-1.compute.internal   Ready    <none>   20h   v1.33.5-eks-ecaa3a6
```

```bash
kubectl get pods -l app=inflate -o wide
```

**Résultat :**
```
NAME                       READY   STATUS    NODE
inflate-5cf78d58f6-9lfgr   1/1     Running   ip-10-0-25-184.eu-west-1.compute.internal  <-- Sur node Karpenter
inflate-5cf78d58f6-ghv7s   1/1     Running   ip-10-0-25-184.eu-west-1.compute.internal  <-- Sur node Karpenter
inflate-5cf78d58f6-jh7g4   1/1     Running   ip-10-0-2-228.eu-west-1.compute.internal
inflate-5cf78d58f6-nvtsp   1/1     Running   ip-10-0-36-183.eu-west-1.compute.internal
inflate-5cf78d58f6-wzkvg   1/1     Running   ip-10-0-25-184.eu-west-1.compute.internal  <-- Sur node Karpenter
```

### Résultat du test 1

| Métrique | Valeur |
|----------|--------|
| Temps de détection | < 1 seconde |
| Temps de provisioning total | ~25-40 secondes |
| Type d'instance choisi | t3a.xlarge |
| Capacity type | on-demand |

---

## Test 2 : Consolidation automatique

### Objectif
Vérifier que Karpenter supprime automatiquement les nodes vides après le délai configuré (`consolidateAfter: 1m`).

### Étape 1 : Suppression de la charge de travail

```bash
kubectl scale deployment inflate --replicas=0
```

### Étape 2 : Vérification que le node est vide

```bash
kubectl get pods -l app=inflate
```

**Résultat :**
```
No resources found in default namespace.
```

### Étape 3 : Observer la consolidation

```bash
# Attendre ~1 minute (consolidateAfter: 1m)
# Observer les logs
kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter --tail=50 | grep -E "(disrupting|tainted|deleted)"
```

**Logs clés :**
```json
{"message":"disrupting nodeclaim(s) via delete, terminating 1 nodes (0 pods)","reason":"empty"}
{"message":"tainted node","Node":{"name":"ip-10-0-25-184.eu-west-1.compute.internal"},"taint.Key":"karpenter.sh/disrupted"}
{"message":"deleted node","Node":{"name":"ip-10-0-25-184.eu-west-1.compute.internal"}}
{"message":"deleted nodeclaim","NodeClaim":{"name":"default-4tw4v"}}
```

**Explication du cycle de consolidation :**
1. **disrupting** : Karpenter identifie le node comme vide ("empty")
2. **tainted** : Applique un taint pour éviter de nouveaux pods
3. **deleted node** : Supprime le node du cluster Kubernetes
4. **deleted nodeclaim** : Supprime le NodeClaim et termine l'instance EC2

### Étape 4 : Vérification finale

```bash
kubectl get nodes
```

**Résultat :**
```
NAME                                        STATUS   ROLES    AGE   VERSION
ip-10-0-2-228.eu-west-1.compute.internal    Ready    <none>   20h   v1.33.5-eks-ecaa3a6
ip-10-0-36-183.eu-west-1.compute.internal   Ready    <none>   20h   v1.33.5-eks-ecaa3a6
```

```bash
kubectl get nodeclaims
```

**Résultat :**
```
No resources found
```

### Résultat du test 2

| Métrique | Valeur |
|----------|--------|
| Délai avant consolidation | ~1 minute (configuré) |
| Raison de suppression | "empty" |

---

## Test 3 : Diversification d'instances

### Objectif
Vérifier que Karpenter choisit différents types d'instances selon les besoins.

### Commandes

```bash
# Workload CPU-intensive
kubectl create deployment cpu-test --image=nginx --replicas=3
kubectl set resources deployment cpu-test --requests=cpu=2,memory=512Mi

# Workload Memory-intensive
kubectl create deployment mem-test --image=nginx --replicas=2
kubectl set resources deployment mem-test --requests=cpu=500m,memory=4Gi
```

### Vérifier les types d'instances

```bash
# Voir les types d'instances choisis
kubectl get nodeclaims -o wide

# Ou
kubectl get nodes -l karpenter.sh/nodepool=default \
  -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.node\.kubernetes\.io/instance-type}{"\n"}{end}'
```

### Nettoyer

```bash
kubectl delete deployment cpu-test mem-test
```

---

## Test 4 : Limites du NodePool

### Objectif
Vérifier que Karpenter respecte les limites définies.

### Configuration actuelle

```yaml
limits:
  cpu: "100"
  memory: "200Gi"
```

### Test

```bash
# Créer un deployment qui dépasse les limites
kubectl create deployment big-test --image=nginx --replicas=200
kubectl set resources deployment big-test --requests=cpu=1,memory=1Gi
```

### Vérifier

```bash
# Voir l'utilisation vs limites
kubectl describe nodepool default | grep -A 5 "Status:"

# Certains pods resteront Pending si les limites sont atteintes
kubectl get pods | grep Pending | wc -l
```

### Nettoyer

```bash
kubectl delete deployment big-test
```

---

## Test 5 : Script de démo complet

```bash
#!/bin/bash

echo "=== Test Karpenter - Démo complète ==="

echo ""
echo "1. État initial"
kubectl get nodes
kubectl get nodepool

echo ""
echo "2. Création du deployment inflate"
kubectl create deployment inflate \
  --image=public.ecr.aws/eks-distro/kubernetes/pause:3.7 \
  --replicas=0
kubectl set resources deployment inflate --requests=cpu=1,memory=1.5Gi

echo ""
echo "3. Scaling à 5 replicas - Karpenter va créer un node"
kubectl scale deployment inflate --replicas=5

echo ""
echo "Attente du provisioning (60 secondes)..."
sleep 60

echo ""
echo "4. Vérification des nodes et pods"
kubectl get nodes
kubectl get pods -l app=inflate
kubectl get nodeclaims

echo ""
echo "5. Scale down - Karpenter va consolider"
kubectl scale deployment inflate --replicas=0

echo ""
echo "Attente de la consolidation (90 secondes)..."
sleep 90

echo ""
echo "6. Vérification finale"
kubectl get nodes
kubectl get nodeclaims

echo ""
echo "7. Nettoyage"
kubectl delete deployment inflate

echo ""
echo "=== Démo terminée ==="
```

---

## Métriques et Monitoring

### Voir les événements Karpenter

```bash
kubectl get events -n kube-system --field-selector source=karpenter --sort-by='.lastTimestamp'
```

### Métriques Prometheus

```bash
# Port-forward vers le service Karpenter
kubectl port-forward -n kube-system svc/karpenter 8000:8000

# Puis accéder à http://localhost:8000/metrics
```

**Métriques importantes :**
- `karpenter_nodes_created_total` : Nombre total de nodes créés
- `karpenter_nodes_terminated_total` : Nombre total de nodes terminés
- `karpenter_pods_startup_duration_seconds` : Temps de démarrage des pods
- `karpenter_nodepools_limit` : Limites des NodePools
- `karpenter_nodepools_usage` : Utilisation actuelle

---

## Nettoyage complet

Après les tests, nettoyez les ressources :

```bash
# Supprimer tous les deployments de test
kubectl delete deployment inflate cpu-test mem-test big-test --ignore-not-found

# Vérifier qu'il n'y a plus de NodeClaims
kubectl get nodeclaims

# Si des NodeClaims persistent, les supprimer
kubectl delete nodeclaim --all
```

---

## Troubleshooting des tests

### Pods restent en Pending

```bash
# Vérifier les événements du pod
kubectl describe pod <pod-name> | grep -A 10 Events

# Vérifier les logs Karpenter
kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter --tail=100 | grep -i error
```

### Node créé mais ne rejoint pas le cluster

```bash
# Vérifier le status du node dans AWS
aws ec2 describe-instances \
  --filters "Name=tag:karpenter.sh/nodepool,Values=default" \
  --query "Reservations[].Instances[].{ID:InstanceId,State:State.Name}"

# Vérifier les logs de l'instance (si accessible)
```

### Karpenter ne crée pas de node

```bash
# Vérifier le NodePool
kubectl describe nodepool default

# Vérifier l'EC2NodeClass
kubectl describe ec2nodeclass default

# Vérifier les limites
kubectl get nodepool default -o jsonpath='{.status}'
```

---

## Résumé des tests

| Test | Description | Résultat attendu |
|------|-------------|------------------|
| Provisioning | Création automatique de nodes pour pods Pending | Node créé en ~40 secondes |
| Consolidation | Suppression automatique des nodes vides | Node supprimé après ~1 minute |
| Diversification | Choix de types d'instances variés | Instances adaptées aux besoins |
| Limites | Respect des limites CPU/mémoire | Pods Pending si limites atteintes |

---

## Commandes utiles

### Karpenter

| Commande | Description |
|----------|-------------|
| `kubectl get nodepools` | Liste les NodePools |
| `kubectl get ec2nodeclasses` | Liste les EC2NodeClasses |
| `kubectl get nodeclaims` | Liste les NodeClaims |
| `kubectl describe nodepool default` | Détails du NodePool |
| `kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter -f` | Logs en temps réel |

### Nodes Karpenter

```bash
# Voir les nodes créés par Karpenter
kubectl get nodes -l karpenter.sh/nodepool=default

# Détails d'un NodeClaim
kubectl describe nodeclaim <name>

# Forcer la consolidation
kubectl annotate nodepool default karpenter.sh/do-not-disrupt-
```

---

## Ressources

- [Documentation officielle Karpenter](https://karpenter.sh/)
- [Guide AWS pour Karpenter](https://aws.github.io/aws-eks-best-practices/karpenter/)
- [Exemples de tests Karpenter](https://karpenter.sh/docs/getting-started/getting-started-with-karpenter/)

---

## Navigation

- [Retour à l'installation EKS](/pro/aws/eks-installation/)
- [Retour à l'installation Karpenter](/pro/aws/karpenter-installation/)
- [Index AWS](/pro/aws/)
