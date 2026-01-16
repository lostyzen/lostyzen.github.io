---
title: "Installation d'un cluster EKS avec un compte de formation AWS"
date: 2026-01-16
lastmod: 2026-01-16
draft: false
showToc: true
TocOpen: true
searchHidden: true
---

# Installation d'un cluster EKS avec un compte de formation AWS

Ce guide explique comment deployer un cluster **Amazon EKS** (Elastic Kubernetes Service) sur un compte de formation AWS qui possede des restrictions SCP (Service Control Policies).

---

## Architecture cible

```
                              Internet
                                  |
                                  v
                        +------------------+
                        | Internet Gateway |
                        +--------+---------+
                                 |
+--------------------------------+--------------------------------+
|                               VPC                                |
|                          10.0.0.0/16                            |
|                                |                                 |
|    +---------------------------+---------------------------+    |
|    |              Subnets Publics (3 AZs)                   |    |
|    |  +--------------+ +--------------+ +--------------+    |    |
|    |  | 10.0.48.0/24 | | 10.0.49.0/24 | | 10.0.50.0/24 |    |    |
|    |  |  eu-west-1a  | |  eu-west-1b  | |  eu-west-1c  |    |    |
|    |  +--------------+ +--------------+ +--------------+    |    |
|    |         |                                              |    |
|    |         |  +-------------------------------------+     |    |
|    |         |  |  Application Load Balancer (ALB)    |     |    |
|    |         |  |  (cree par AWS LB Controller)       |     |    |
|    |         |  +-------------------------------------+     |    |
|    +---------+----------------------------------------------+    |
|              |                                                   |
|              v                                                   |
|    +------------------+                                          |
|    |   NAT Gateway    | <-- Permet aux nodes prives              |
|    |  (Elastic IP)    |     d'acceder a internet                 |
|    +--------+---------+                                          |
|             |                                                    |
|    +--------+----------------------------------------------+    |
|    |        v        Subnets Prives (3 AZs)                |    |
|    |  +--------------+ +--------------+ +--------------+   |    |
|    |  | 10.0.0.0/20  | | 10.0.16.0/20 | | 10.0.32.0/20 |   |    |
|    |  |  eu-west-1a  | |  eu-west-1b  | |  eu-west-1c  |   |    |
|    |  |              | |              | |              |   |    |
|    |  | +---------+  | | +---------+  | |              |   |    |
|    |  | | Static  |  | | | Static  |  | |              |   |    |
|    |  | |t3.medium|  | | |t3.medium|  | |              |   |    |
|    |  | +---------+  | | +---------+  | |              |   |    |
|    |  |              | |              | | +---------+  |   |    |
|    |  |              | |              | | |Karpenter|  |   |    |
|    |  |              | |              | | |t3a.xlarge| |   |    |
|    |  |              | |              | | |(dynamic)|  |   |    |
|    |  |              | |              | | +---------+  |   |    |
|    |  +--------------+ +--------------+ +--------------+   |    |
|    |                                                        |    |
|    |                    ^         ^         ^               |    |
|    +--------------------+---------+---------+---------------+    |
|                         |         |         |                    |
|              +----------+---------+---------+----------+        |
|              |         EKS Control Plane               |        |
|              |      (Gere par AWS - invisible)         |        |
|              |                                         |        |
|              |  - API Server    - Controller Manager   |        |
|              |  - etcd          - Scheduler            |        |
|              |                                         |        |
|              |  Composants deployes :                  |        |
|              |  - Karpenter (autoscaling)              |        |
|              |  - AWS LB Controller                    |        |
|              |  - CoreDNS, kube-proxy, vpc-cni         |        |
|              +-----------------------------------------+        |
|                              ^                                   |
+------------------------------+-----------------------------------+
                               |
                         kubectl / API
                               |
                        +--------------+
                        | Ton poste    |
                        | (AWS CLI)    |
                        +--------------+
```

### Vue en diagramme Mermaid

```mermaid
flowchart TB
    subgraph Internet["Internet"]
        User["Utilisateur / kubectl"]
        DockerHub["Docker Hub / ECR"]
    end

    subgraph AWS["AWS Cloud (eu-west-1)"]
        subgraph IAM["IAM (Global Service)"]
            IRSA_ALBC["IRSA Role<br/>LB Controller"]
            IRSA_Karp["IRSA Role<br/>Karpenter"]
            InstProfile["Instance Profile<br/>Karpenter Nodes"]
        end

        IGW["Internet Gateway"]

        subgraph VPC["VPC: sandbox-eks-vpc (10.0.0.0/16)"]
            subgraph PublicSubnets["Public Subnets"]
                NAT["NAT Gateway"]
                ALB["Application<br/>Load Balancer"]
            end

            subgraph PrivateSubnets["Private Subnets (3 AZs)"]
                Node1["Static Node 1<br/>t3.medium"]
                Node2["Static Node 2<br/>t3.medium"]
                KarpNode1["Karpenter Node<br/>On-Demand/Dynamic"]
            end

            subgraph EKS["EKS Cluster: sandbox-eks"]
                CP["Control Plane K8s 1.33"]

                subgraph Workloads["Workloads"]
                    ALBC["AWS LB Controller"]
                    Karpenter["Karpenter"]
                    Addons["Add-ons:<br/>CoreDNS, kube-proxy,<br/>vpc-cni, pod-identity"]
                end
            end
        end
    end

    User -->|"kubectl"| IGW
    IGW -->|"API"| CP

    CP -.->|"schedule"| Node1
    CP -.->|"schedule"| Node2
    CP -.->|"schedule"| KarpNode1

    Karpenter -->|"EC2 API"| KarpNode1

    ALBC -->|"create"| ALB

    Node1 --> NAT
    Node2 --> NAT
    NAT --> IGW
    IGW -->|"pull"| DockerHub

    ALBC -.->|"assume"| IRSA_ALBC
    Karpenter -.->|"assume"| IRSA_Karp
    InstProfile -.->|"attach"| KarpNode1

    style KarpNode1 fill:#2E7D32,stroke:#1B5E20,color:#FFFFFF
```

---

## Composants deployes

### 1. VPC (Virtual Private Cloud)
- **CIDR** : `10.0.0.0/16` (65,536 adresses IP)
- **Region** : `eu-west-1` (Irlande)
- **Isolation** : Reseau prive virtuel dedie au cluster

### 2. Subnets

| Type | Quantite | Usage |
|------|----------|-------|
| Publics | 3 | Load Balancers, NAT Gateway |
| Prives | 3 | Worker nodes EKS (statiques et Karpenter) |

**Pourquoi 3 de chaque ?** Ce setup permet d'experimenter des use cases de haute disponibilite sur 3 zones de disponibilite (AZs). Meme s'il s'agit d'un cluster destine a la formation, l'experimentation ou des POC, cette configuration permet de tester des scenarios realistes de distribution multi-AZ.

### 3. NAT Gateway
- Permet aux instances dans les subnets prives d'acceder a internet
- Necessaire pour que les nodes puissent telecharger les images Docker
- **Mode single** : 1 seul NAT Gateway pour economiser (~45$/mois par NAT)

### 4. EKS Cluster
- **Control Plane** : Gere par AWS (tu ne le vois pas, tu ne paies que le service)
- **Version Kubernetes** : 1.33
- **Endpoint public** : Accessible depuis ton poste via internet

### 5. Node Group (Statique)
- **Type d'instances** : `t3.medium` (2 vCPU, 4 GB RAM)
- **Scaling** : min=1, max=3, desired=2
- **Placement** : Dans les subnets prives (securite)
- **Usage** : Workloads critiques (Karpenter, CoreDNS, etc.)

### 6. Karpenter (Autoscaling dynamique)
- **Role** : Cree automatiquement des nodes quand des pods sont en attente (Pending)
- **Types d'instances** : t3, t3a, t2 (medium a 2xlarge)
- **Capacity type** : On-Demand uniquement (Spot desactive a cause des SCP)
- **Consolidation** : Supprime les nodes vides apres 1 minute
- **Temps de provisioning** : ~30-40 secondes

### 7. AWS Load Balancer Controller
- **Role** : Cree automatiquement des ALB/NLB quand tu deploies un Ingress ou Service LoadBalancer
- **Authentification** : Via IRSA (IAM Roles for Service Accounts)

### 8. Add-ons EKS

| Add-on | Description |
|--------|-------------|
| CoreDNS | Resolution DNS interne au cluster |
| kube-proxy | Gestion des regles reseau (iptables) |
| vpc-cni | Plugin reseau AWS (IP natives du VPC) |
| pod-identity-agent | Authentification IAM pour les pods |

---

## Contraintes SCP (Service Control Policies)

Ce compte AWS a des restrictions organisationnelles qui impactent la configuration :

| Contrainte | Impact | Workaround |
|------------|--------|------------|
| KMS bloque | Pas de chiffrement EBS natif | `encrypted: false` dans EC2NodeClass |
| SSM bloque | Pas de lookup automatique d'AMI | AMI ID defini en dur |
| Spot SLR bloque | Pas d'instances Spot | On-Demand uniquement |
| DescribeAZs bloque | Pas de decouverte dynamique | AZs definies en dur |

---

## Couts estimes (sandbox)

| Ressource | Cout estime/mois |
|-----------|------------------|
| EKS Control Plane | ~$73 |
| 2x t3.medium (static nodes) | ~$60 |
| NAT Gateway + data | ~$45 |
| Karpenter nodes (variable) | ~$0-50 (selon usage) |
| **Total** | **~$180-230/mois** |

> **Important** : Pense a detruire l'infrastructure quand tu ne l'utilises pas avec `terraform destroy`. Les nodes Karpenter sont automatiquement supprimes quand ils sont vides.

---

## Prerequis

- [x] AWS CLI installe et configure
- [x] Terraform installe (`terraform version`)
- [x] kubectl installe

### Verifier la configuration AWS CLI

```bash
# Verifie que la CLI est bien configuree avec ta cle d'API
aws sts get-caller-identity
```

**Resultat attendu :**
```json
{
    "UserId": "AIDAXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/terraform-admin"
}
```

### Installation de kubectl (si necessaire)

```powershell
# Via scoop (recommande sur Windows)
scoop install kubectl

# Verification
kubectl version --client
```

---

## Configuration Terraform

### Structure des fichiers

```
terraform/
├── versions.tf    # Contraintes de versions
├── providers.tf   # Configuration des providers (aws, kubernetes, helm, kubectl)
├── variables.tf   # Variables d'entree
├── main.tf        # Ressources principales (VPC, EKS, Karpenter, LB Controller)
└── outputs.tf     # Valeurs de sortie
```

### versions.tf - Contraintes de versions

```hcl
terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.12"
    }
    kubectl = {
      source  = "gavinbunney/kubectl"
      version = "~> 1.14"
    }
  }
}
```

### providers.tf - Configuration des providers

```hcl
provider "aws" {
  region = var.aws_region
}

provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)

  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
  }
}

provider "helm" {
  kubernetes {
    host                   = module.eks.cluster_endpoint
    cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
    exec {
      api_version = "client.authentication.k8s.io/v1beta1"
      command     = "aws"
      args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
    }
  }
}

provider "kubectl" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
  }
}
```

### variables.tf - Variables d'entree

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "eu-west-1"
}

variable "cluster_name" {
  description = "EKS cluster name"
  type        = string
  default     = "sandbox-eks"
}

variable "cluster_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.33"
}

variable "environment" {
  description = "Environment tag"
  type        = string
  default     = "sandbox"
}
```

### main.tf - Ressources principales

#### Locals - Variables calculees

```hcl
locals {
  vpc_cidr = "10.0.0.0/16"

  # AZs definies en dur car la SCP bloque DescribeAvailabilityZones
  azs = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]

  tags = {
    Environment = var.environment
    Project     = "eks-sandbox"
    ManagedBy   = "terraform"
  }
}
```

#### Module VPC

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "${var.cluster_name}-vpc"
  cidr = local.vpc_cidr

  azs             = local.azs
  private_subnets = [for k, v in local.azs : cidrsubnet(local.vpc_cidr, 4, k)]
  public_subnets  = [for k, v in local.azs : cidrsubnet(local.vpc_cidr, 8, k + 48)]

  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = 1
    "karpenter.sh/discovery"          = var.cluster_name
  }

  tags = local.tags
}
```

**Calcul des subnets :**
```
VPC: 10.0.0.0/16

Private subnets (cidrsubnet avec /4 = /20):
  - 10.0.0.0/20   (AZ-a)
  - 10.0.16.0/20  (AZ-b)
  - 10.0.32.0/20  (AZ-c)

Public subnets (cidrsubnet avec /8 = /24, offset 48):
  - 10.0.48.0/24  (AZ-a)
  - 10.0.49.0/24  (AZ-b)
  - 10.0.50.0/24  (AZ-c)
```

#### Module EKS

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  cluster_endpoint_public_access = true

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  # Workarounds SCP
  create_kms_key              = false
  create_cloudwatch_log_group = false
  cluster_encryption_config   = {}

  eks_managed_node_groups = {
    default = {
      name           = "default-node-group"
      instance_types = ["t3.medium"]
      min_size       = 1
      max_size       = 3
      desired_size   = 2
    }
  }

  enable_cluster_creator_admin_permissions = true

  tags = local.tags
}
```

### outputs.tf - Valeurs de sortie

```hcl
output "cluster_name" {
  description = "EKS cluster name"
  value       = module.eks.cluster_name
}

output "configure_kubectl" {
  description = "Command to configure kubectl"
  value       = "aws eks update-kubeconfig --region ${var.aws_region} --name ${module.eks.cluster_name}"
}
```

---

## Guide de deploiement

### Etape 1 : Initialisation Terraform

```bash
cd terraform
terraform init
```

**Ce que ca fait :**
- Telecharge le provider AWS (~100 MB)
- Telecharge les providers Kubernetes, Helm, Kubectl
- Telecharge les modules VPC, EKS et Karpenter depuis le registry Terraform
- Cree le dossier `.terraform/` avec les plugins
- Cree `.terraform.lock.hcl` pour verrouiller les versions

### Etape 2 : Planification

```bash
terraform plan
```

**Resultat attendu :**
```
Plan: 70+ to add, 0 to change, 0 to destroy.
```

### Etape 3 : Deploiement

```bash
terraform apply
```

**Duree estimee : 15-20 minutes**

Le plus long est la creation du cluster EKS lui-meme (~10 min).

**Resultat attendu :**
```
Apply complete! Resources: 70+ added, 0 changed, 0 destroyed.

Outputs:

cluster_name = "sandbox-eks"
cluster_endpoint = "https://XXXXX.yl4.eu-west-1.eks.amazonaws.com"
configure_kubectl = "aws eks update-kubeconfig --region eu-west-1 --name sandbox-eks"
```

### Etape 4 : Configuration de kubectl

Copie-colle la commande affichee dans les outputs :

```bash
aws eks update-kubeconfig --region eu-west-1 --name sandbox-eks
```

**Verification :**
```bash
# Verifie la connexion
kubectl cluster-info

# Liste les nodes
kubectl get nodes

# Liste les pods systeme
kubectl get pods -A
```

### Etape 5 : Verification de Karpenter

```bash
# Pods Karpenter (2 replicas)
kubectl get pods -n kube-system -l app.kubernetes.io/name=karpenter

# Ressources Karpenter
kubectl get nodepools,ec2nodeclasses

# Details du NodePool
kubectl describe nodepool default
```

**Resultat attendu :**
```
NAME                            NODECLASS   NODES   READY   AGE
nodepool.karpenter.sh/default   default     0       True    5m

NAME                                     READY   AGE
ec2nodeclass.karpenter.k8s.aws/default   True    5m
```

> **Note** : `NODES: 0` est normal - Karpenter ne cree des nodes que quand des pods sont en attente.

---

## Test du cluster

### Deployer une application de test

```bash
# Cree un deploiement nginx
kubectl create deployment nginx --image=nginx

# Expose le deploiement
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Attends que le LoadBalancer soit provisionne (2-3 min)
kubectl get svc nginx -w
```

Quand l'EXTERNAL-IP apparait, ouvre-la dans ton navigateur !

### Nettoyage du test

```bash
kubectl delete deployment nginx
kubectl delete svc nginx
```

---

## Destruction de l'infrastructure

**Important** : Pour eviter les couts, detruis l'infrastructure quand tu ne l'utilises pas.

```bash
terraform destroy
```

**Duree estimee : 10-15 minutes**

---

## Commandes utiles

### Terraform

| Commande | Description |
|----------|-------------|
| `terraform init` | Initialise le projet |
| `terraform plan` | Previsualise les changements |
| `terraform apply` | Applique les changements |
| `terraform destroy` | Supprime tout |
| `terraform output` | Affiche les outputs |

### kubectl

| Commande | Description |
|----------|-------------|
| `kubectl get nodes` | Liste les nodes |
| `kubectl get pods -A` | Liste tous les pods |
| `kubectl get svc -A` | Liste tous les services |
| `kubectl describe node <name>` | Details d'un node |
| `kubectl logs <pod> -n <ns>` | Logs d'un pod |

---

## Troubleshooting

### Erreurs generales

**"Error: No valid credential sources found"**
→ Verifie que `aws configure` a bien ete fait

**"Error: creating EKS Cluster: AccessDeniedException"**
→ L'utilisateur IAM n'a pas les permissions suffisantes

**"kubectl: Unable to connect to the server"**
→ Lance `aws eks update-kubeconfig --region eu-west-1 --name sandbox-eks`

### Erreurs SCP (specifiques a ce compte)

**"Error: creating KMS Key: AccessDeniedException"**
→ Le module EKS doit avoir `create_kms_key = false`

**"Error: creating CloudWatch Log Group: AccessDeniedException"**
→ Le module EKS doit avoir `create_cloudwatch_log_group = false`

---

## Prochaine etape

Une fois le cluster EKS operationnel, vous pouvez passer a l'[Installation de Karpenter](/pro/aws/karpenter-installation/).
