---
title: "Installation de Karpenter sur un cluster EKS"
date: 2026-01-16
lastmod: 2026-01-16
draft: false
showToc: true
TocOpen: true
searchHidden: true
---

# Installation de Karpenter sur un cluster EKS

Ce guide explique comment installer et configurer **Karpenter** sur un cluster EKS, avec une attention particulière aux contraintes des comptes de formation AWS (SCP).

> **Prérequis** : Avoir un cluster EKS opérationnel. Voir [Installation d'un cluster EKS](/pro/aws/eks-installation/).

---

## Qu'est-ce que Karpenter ?

**Karpenter** est un autoscaler Kubernetes open-source développé par AWS qui provisionne automatiquement des nodes en fonction des besoins réels de vos workloads.

### Fonctionnalités principales

| Fonctionnalité | Description |
|----------------|-------------|
| **Autoscaling intelligent** | Provisionne automatiquement les nodes EC2 en fonction des pods en attente |
| **Optimisation des coûts** | Sélectionne les types d'instances les plus économiques selon les contraintes |
| **Rapidité** | Provisionne des nodes en ~40 secondes (vs 3-5 min avec Cluster Autoscaler) |
| **Consolidation** | Remplace automatiquement plusieurs petits nodes par un seul plus gros pour réduire les coûts |
| **Flexibilité** | Peut utiliser n'importe quel type d'instance EC2 (Spot, On-Demand, ARM, x86) |

### Karpenter vs Cluster Autoscaler

| Critère | Cluster Autoscaler | Karpenter |
|---------|-------------------|-----------|
| Vitesse de provisioning | 3-5 minutes | ~40 secondes |
| Configuration | Node Groups prédéfinis | Sélection dynamique d'instances |
| Optimisation des coûts | Limitée | Avancée (bin-packing, Spot) |
| Gestion du cycle de vie | Basique | Avancée (consolidation, expiration) |
| Complexité | Simple | Modérée |

---

## Architecture avec Karpenter

{{< mermaid >}}
flowchart LR
    subgraph EKS["EKS Cluster"]
        Pod["Pod<br/>(Pending)"]
        Scheduler["kube-scheduler"]
        Karpenter["Karpenter<br/>Controller"]
        NodePool["NodePool<br/>(CRD)"]
        EC2NodeClass["EC2NodeClass<br/>(CRD)"]
    end

    subgraph AWS["AWS"]
        EC2["EC2 API"]
        Instance["EC2 Instance<br/>(Spot/On-Demand)"]
    end

    Pod -->|"1. Cannot schedule"| Scheduler
    Scheduler -->|"2. Pod pending"| Karpenter
    Karpenter -->|"3. Read config"| NodePool
    Karpenter -->|"3. Read config"| EC2NodeClass
    Karpenter -->|"4. RunInstances"| EC2
    EC2 -->|"5. Create"| Instance
    Instance -->|"6. Join cluster"| Scheduler
    Scheduler -->|"7. Schedule pod"| Pod

    style Karpenter fill:#f7768e,stroke:#c53030,color:#fff
    style NodePool fill:#2E7D32,stroke:#1B5E20,color:#fff
    style EC2NodeClass fill:#2E7D32,stroke:#1B5E20,color:#fff
    style Instance fill:#ff9e64,stroke:#d67e3e,color:#1a1b26
    style Pod fill:#bb9af7,stroke:#7c4dff,color:#1a1b26
    style Scheduler fill:#7aa2f7,stroke:#3d59a1,color:#1a1b26
{{< /mermaid >}}

### Composants à installer

| Composant | Type | Description |
|-----------|------|-------------|
| **Module Karpenter** | IAM Resources | Rôles IAM, Instance Profile |
| **Module IRSA** | IAM Resources | Rôle IRSA pour le controller |
| **Helm Release** | Kubernetes | Deployment du controller Karpenter |
| **NodePool** | CRD Kubernetes | Configuration des contraintes de nodes |
| **EC2NodeClass** | CRD Kubernetes | Configuration EC2 (AMI, subnets, SG) |
| **Tags** | AWS EC2 | Tags sur les security groups |

---

## Configuration Terraform pour compte de formation (SCP)

### Restrictions SCP rencontrées

| Service | Action bloquée | Impact sur Karpenter |
|---------|---------------|---------------------|
| **SQS** | `sqs:CreateQueue` | Pas de gestion des interruptions Spot |
| **EventBridge** | `events:PutRule` | Pas de notifications |
| **Pod Identity** | `eks-auth:AssumeRoleForPodIdentity` | Doit utiliser IRSA |
| **SSM** | `ssm:GetParameter` | Pas de lookup AMI dynamique |
| **KMS** | `kms:CreateKey` | Pas de chiffrement EBS |
| **Spot SLR** | `iam:CreateServiceLinkedRole` | Pas d'instances Spot |

### Module Karpenter

```hcl
# Module Karpenter - Provisionne les ressources IAM pour Karpenter
# Note: SQS, EventBridge et Pod Identity désactivés - SCP du compte les bloque
module "karpenter" {
  source  = "terraform-aws-modules/eks/aws//modules/karpenter"
  version = "~> 20.0"

  cluster_name = module.eks.cluster_name

  # Utiliser IRSA (Pod Identity bloqué par SCP)
  enable_irsa                     = true
  irsa_oidc_provider_arn          = module.eks.oidc_provider_arn
  irsa_namespace_service_accounts = ["kube-system:karpenter"]

  # Désactiver Pod Identity (bloqué par SCP)
  enable_pod_identity             = false
  create_pod_identity_association = false

  # Créer l'instance profile pour les nodes Karpenter
  node_iam_role_use_name_prefix = false
  node_iam_role_name            = "${var.cluster_name}-karpenter-node"

  tags = local.tags
}
```

### Helm Release Karpenter

```hcl
# Helm release pour Karpenter
resource "helm_release" "karpenter" {
  namespace  = "kube-system"
  name       = "karpenter"
  repository = "oci://public.ecr.aws/karpenter"
  chart      = "karpenter"
  version    = "1.1.1"
  wait       = false

  set {
    name  = "settings.clusterName"
    value = module.eks.cluster_name
  }

  set {
    name  = "settings.clusterEndpoint"
    value = module.eks.cluster_endpoint
  }

  set {
    name  = "serviceAccount.annotations.eks\\.amazonaws\\.com/role-arn"
    value = module.karpenter.iam_role_arn
  }

  depends_on = [
    module.eks,
    module.karpenter
  ]
}
```

### NodePool

```hcl
resource "kubectl_manifest" "karpenter_node_pool" {
  yaml_body = <<-YAML
    apiVersion: karpenter.sh/v1
    kind: NodePool
    metadata:
      name: default
    spec:
      template:
        metadata:
          labels:
            karpenter.sh/nodepool: default
        spec:
          nodeClassRef:
            group: karpenter.k8s.aws
            kind: EC2NodeClass
            name: default
          requirements:
            - key: "karpenter.sh/capacity-type"
              operator: In
              values: ["on-demand"]  # Spot désactivé (SCP bloque le SLR)
            - key: "kubernetes.io/arch"
              operator: In
              values: ["amd64"]
            - key: "karpenter.k8s.aws/instance-category"
              operator: In
              values: ["t"]
            - key: "karpenter.k8s.aws/instance-family"
              operator: In
              values: ["t3", "t3a", "t2"]
            - key: "karpenter.k8s.aws/instance-size"
              operator: In
              values: ["medium", "large", "xlarge", "2xlarge"]
      limits:
        cpu: "100"
        memory: "200Gi"
      disruption:
        consolidationPolicy: WhenEmptyOrUnderutilized
        consolidateAfter: 1m
        budgets:
          - nodes: "10%"
  YAML

  depends_on = [helm_release.karpenter]
}
```

#### Explication des requirements

| Requirement | Valeurs | Signification |
|-------------|---------|---------------|
| `capacity-type` | on-demand | On-Demand uniquement (Spot bloqué par SCP) |
| `arch` | amd64 | Architecture x86_64 (pas ARM) |
| `instance-category` | t | Catégorie T (burstable) |
| `instance-family` | t3, t3a, t2 | Familles d'instances autorisées |
| `instance-size` | medium à 2xlarge | Tailles d'instances (2 à 8 vCPUs) |

#### Explication de la disruption

| Paramètre | Valeur | Signification |
|-----------|--------|---------------|
| `consolidationPolicy` | WhenEmptyOrUnderutilized | Consolide les nodes vides OU sous-utilisés |
| `consolidateAfter` | 1m | Attend 1 minute avant de consolider |
| `budgets` | 10% | Maximum 10% des nodes peuvent être disruptés simultanément |

### EC2NodeClass

```hcl
resource "kubectl_manifest" "karpenter_node_class" {
  yaml_body = <<-YAML
    apiVersion: karpenter.k8s.aws/v1
    kind: EC2NodeClass
    metadata:
      name: default
    spec:
      amiFamily: AL2023
      # AMI spécifiée en dur car ssm:GetParameter est bloqué par SCP
      amiSelectorTerms:
        - id: ami-05521d50f4e9c827d  # amazon-eks-node-al2023-x86_64-standard-1.33 eu-west-1
      # instanceProfile pré-créé par Terraform car les SCP du compte de formation bloquent la création d'instance profiles par Karpenter
      instanceProfile: ${module.karpenter.instance_profile_name}
      subnetSelectorTerms:
        - tags:
            karpenter.sh/discovery: ${var.cluster_name}
      securityGroupSelectorTerms:
        - tags:
            karpenter.sh/discovery: ${module.eks.cluster_name}
      tags:
        karpenter.sh/discovery: ${module.eks.cluster_name}
        Environment: ${var.environment}
        Project: "eks-sandbox"
        ManagedBy: "karpenter"
      blockDeviceMappings:
        - deviceName: /dev/xvda
          ebs:
            volumeSize: 20Gi
            volumeType: gp3
            encrypted: false  # KMS bloqué par SCP (en production, activer le chiffrement EBS)
            deleteOnTermination: true
  YAML

  depends_on = [helm_release.karpenter]
}
```

#### Points clés de la configuration

| Paramètre | Valeur | Raison |
|-----------|--------|--------|
| `amiSelectorTerms.id` | AMI fixe | SSM bloqué par SCP |
| `instanceProfile` | Pré-créé | SCP bloque la création par Karpenter |
| `encrypted: false` | Pas de chiffrement | KMS bloqué par SCP (activer en production) |

### Tags sur les Security Groups

```hcl
# Ajouter le tag karpenter.sh/discovery aux security groups
resource "aws_ec2_tag" "cluster_security_group_tag" {
  for_each    = toset([module.eks.cluster_security_group_id])
  resource_id = each.value
  key         = "karpenter.sh/discovery"
  value       = module.eks.cluster_name
}

resource "aws_ec2_tag" "node_security_group_tag" {
  for_each    = toset([module.eks.node_security_group_id])
  resource_id = each.value
  key         = "karpenter.sh/discovery"
  value       = module.eks.cluster_name
}
```

---

## Guide d'installation

### Étape 1 : Initialiser les providers

```bash
cd terraform
terraform init -upgrade
```

### Étape 2 : Vérifier les changements

```bash
terraform plan
```

Vous devriez voir :
- `module.karpenter` (IAM resources)
- `helm_release.karpenter`
- `kubectl_manifest.karpenter_node_pool`
- `kubectl_manifest.karpenter_node_class`
- `aws_ec2_tag.cluster_security_group_tag`
- `aws_ec2_tag.node_security_group_tag`

### Étape 3 : Déployer

```bash
terraform apply
```

**Durée estimée : 3-5 minutes**

---

## Vérification de l'installation

### 1. Vérifier les pods Karpenter

```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=karpenter
```

**Sortie attendue :**
```
NAME                         READY   STATUS    RESTARTS   AGE
karpenter-5d7f8c9b4d-xxxxx   1/1     Running   0          2m
karpenter-5d7f8c9b4d-yyyyy   1/1     Running   0          2m
```

### 2. Vérifier le NodePool

```bash
kubectl get nodepool
```

**Sortie attendue :**
```
NAME      NODECLASS   NODES   READY   AGE
default   default     0       True    5m
```

### 3. Vérifier l'EC2NodeClass

```bash
kubectl get ec2nodeclass
```

**Sortie attendue :**
```
NAME      READY   AGE
default   True    5m
```

### 4. Vérifier les logs

```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter --tail=20
```

---

## Commandes utiles

### Karpenter

| Commande | Description |
|----------|-------------|
| `kubectl get nodepools` | Liste les NodePools |
| `kubectl get ec2nodeclasses` | Liste les EC2NodeClasses |
| `kubectl get nodeclaims` | Liste les NodeClaims (nodes en cours/créés) |
| `kubectl describe nodepool default` | Détails du NodePool |
| `kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter -f` | Logs Karpenter en temps réel |

### Gestion des nodes

```bash
# Voir les nodes créés par Karpenter
kubectl get nodes -l karpenter.sh/nodepool=default

# Voir les NodeClaims
kubectl get nodeclaims

# Détails du NodeClaim
kubectl describe nodeclaim
```

### Administration

```bash
# Forcer la consolidation
kubectl annotate nodepool default karpenter.sh/do-not-disrupt-

# Désactiver temporairement Karpenter
kubectl scale deployment karpenter -n kube-system --replicas=0

# Réactiver Karpenter
kubectl scale deployment karpenter -n kube-system --replicas=2
```

---

## Troubleshooting

### Problème : Pods Karpenter en CrashLoopBackOff

**Cause probable :** Pod Identity bloqué par SCP

**Solution :** Utiliser IRSA au lieu de Pod Identity

```bash
# Vérifier les logs
kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter

# Si erreur "eks-auth:AssumeRoleForPodIdentity"
# -> Configurer IRSA comme dans la section compte formation
```

### Problème : EC2NodeClass READY: False

**Cause probable :** AMI lookup échoue (SSM bloqué)

**Solution :** Spécifier l'AMI ID en dur

```yaml
amiSelectorTerms:
  - id: ami-05521d50f4e9c827d  # Remplacer par l'AMI de votre région
```

**Trouver l'AMI (via EC2 describe-images) :**
```bash
# Lister les AMI EKS disponibles pour K8s 1.33 en eu-west-1
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amazon-eks-node-al2023-x86_64-standard-1.33*" \
  --query "Images[*].[ImageId,Description]" \
  --output table \
  --region eu-west-1

# Récupérer uniquement la plus récente
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amazon-eks-node-al2023-x86_64-standard-1.33*" \
  --query "Images | sort_by(@, &CreationDate) | [-1].[ImageId,Name]" \
  --output text \
  --region eu-west-1
```

### Problème : Karpenter ne peut pas créer de nodes

**Vérifier les erreurs IAM :**
```bash
kubectl logs -n kube-system -l app.kubernetes.io/name=karpenter | grep -i error
```

**Causes possibles :**
- Instance Profile manquant → Utiliser `instanceProfile:` au lieu de `role:`
- Permissions EC2 manquantes → Vérifier la policy IRSA
- Security groups non trouvés → Ajouter les tags `karpenter.sh/discovery`

### Problème : "SecurityGroupSelector did not match any SecurityGroups"

**Solution :** Vérifier et ajouter les tags sur les security groups

```bash
# Vérifier les tags existants
aws ec2 describe-security-groups --filters "Name=tag:Name,Values=*sandbox-eks-node*" \
  --query "SecurityGroups[*].Tags"

# Si le tag manque, réappliquer avec Terraform
terraform apply -target=aws_ec2_tag.node_security_group_tag
```

### Warnings non bloquants

Avec un compte formation, vous verrez ces warnings dans les logs Karpenter :

```
ERROR pricing.GetProducts, failed to get products ... AccessDeniedException
```

Ce warning n'empêche pas Karpenter de fonctionner. Karpenter utilisera des valeurs par défaut pour le pricing.

---

## Bonnes pratiques

1. **Toujours définir des limites** : Évitez les coûts imprévus
2. **Diversifiez les types d'instances** : Augmentez la disponibilité
3. **Activez la consolidation** : Optimisez l'utilisation
4. **Monitorez activement** : Suivez les métriques et les coûts
5. **Testez en sandbox** : Validez la configuration avant production

---

## Ressources

- [Documentation officielle Karpenter](https://karpenter.sh/)
- [Guide AWS pour Karpenter](https://aws.github.io/aws-eks-best-practices/karpenter/)
- [Module Terraform Karpenter](https://github.com/terraform-aws-modules/terraform-aws-eks/tree/master/modules/karpenter)
- [Exemples de NodePools](https://karpenter.sh/docs/concepts/nodepools/)

---

## Prochaine étape

Une fois Karpenter installé et opérationnel, vous pouvez passer aux [Tests de Karpenter](/pro/aws/karpenter-tests/).
