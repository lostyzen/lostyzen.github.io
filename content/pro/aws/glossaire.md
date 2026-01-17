---
title: "Glossaire AWS & Kubernetes"
date: 2026-01-17
lastmod: 2026-01-17
draft: false
showToc: false
TocOpen: false
searchHidden: true
---

{{< rawhtml >}}
<style>
/* Tokyo Night Theme */
:root {
  --tn-bg: #1a1b26;
  --tn-bg-dark: #16161e;
  --tn-bg-highlight: #292e42;
  --tn-fg: #c0caf5;
  --tn-fg-dark: #a9b1d6;
  --tn-comment: #565f89;
  --tn-cyan: #7dcfff;
  --tn-blue: #7aa2f7;
  --tn-purple: #bb9af7;
  --tn-magenta: #ff007c;
  --tn-orange: #ff9e64;
  --tn-yellow: #e0af68;
  --tn-green: #9ece6a;
  --tn-teal: #73daca;
  --tn-red: #f7768e;
}

.glossary-container {
  background: var(--tn-bg);
  border-radius: 12px;
  padding: 2rem;
  margin: 1rem 0;
  font-family: 'JetBrains Mono', 'Fira Code', monospace;
}

.glossary-header {
  text-align: center;
  margin-bottom: 2rem;
}

.glossary-header h1 {
  color: var(--tn-purple);
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
  text-shadow: 0 0 20px rgba(187, 154, 247, 0.3);
}

.glossary-header p {
  color: var(--tn-comment);
  font-size: 1rem;
}

/* Search Box */
.search-container {
  position: relative;
  max-width: 600px;
  margin: 0 auto 2rem auto;
}

.search-input {
  width: 100%;
  padding: 1rem 1rem 1rem 3rem;
  font-size: 1.1rem;
  background: var(--tn-bg-dark);
  border: 2px solid var(--tn-bg-highlight);
  border-radius: 8px;
  color: var(--tn-fg);
  font-family: inherit;
  transition: all 0.3s ease;
}

.search-input:focus {
  outline: none;
  border-color: var(--tn-blue);
  box-shadow: 0 0 15px rgba(122, 162, 247, 0.3);
}

.search-input::placeholder {
  color: var(--tn-comment);
}

.search-icon {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--tn-comment);
  font-size: 1.2rem;
}

.search-clear {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: var(--tn-comment);
  cursor: pointer;
  font-size: 1.2rem;
  display: none;
}

.search-clear:hover {
  color: var(--tn-red);
}

/* Results count */
.results-count {
  text-align: center;
  color: var(--tn-comment);
  margin-bottom: 1.5rem;
  font-size: 0.9rem;
}

.results-count span {
  color: var(--tn-cyan);
  font-weight: bold;
}

/* Letter sections */
.letter-section {
  margin-bottom: 2rem;
}

.letter-header {
  display: flex;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid var(--tn-bg-highlight);
}

.letter {
  font-size: 2.5rem;
  font-weight: bold;
  color: var(--tn-magenta);
  width: 60px;
  text-shadow: 0 0 15px rgba(255, 0, 124, 0.4);
}

.letter-line {
  flex: 1;
  height: 2px;
  background: linear-gradient(90deg, var(--tn-bg-highlight), transparent);
}

/* Term cards */
.term-card {
  background: var(--tn-bg-dark);
  border-radius: 8px;
  padding: 1.25rem;
  margin-bottom: 1rem;
  border-left: 4px solid var(--tn-blue);
  transition: all 0.3s ease;
}

.term-card:hover {
  border-left-color: var(--tn-cyan);
  transform: translateX(5px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.term-card.aws {
  border-left-color: var(--tn-orange);
}

.term-card.aws:hover {
  border-left-color: var(--tn-yellow);
}

.term-card.k8s {
  border-left-color: var(--tn-blue);
}

.term-card.k8s:hover {
  border-left-color: var(--tn-cyan);
}

.term-card.network {
  border-left-color: var(--tn-green);
}

.term-card.network:hover {
  border-left-color: var(--tn-teal);
}

.term-card.security {
  border-left-color: var(--tn-red);
}

.term-card.security:hover {
  border-left-color: var(--tn-magenta);
}

.term-card.general {
  border-left-color: var(--tn-purple);
}

.term-name {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 0.5rem;
}

.term-name h3 {
  color: var(--tn-fg);
  font-size: 1.2rem;
  margin: 0;
}

.term-acronym {
  color: var(--tn-cyan);
  font-size: 0.85rem;
  background: var(--tn-bg-highlight);
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
}

.term-category {
  font-size: 0.75rem;
  padding: 0.15rem 0.4rem;
  border-radius: 3px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.term-category.aws { background: rgba(255, 158, 100, 0.2); color: var(--tn-orange); }
.term-category.k8s { background: rgba(122, 162, 247, 0.2); color: var(--tn-blue); }
.term-category.network { background: rgba(158, 206, 106, 0.2); color: var(--tn-green); }
.term-category.security { background: rgba(247, 118, 142, 0.2); color: var(--tn-red); }
.term-category.general { background: rgba(187, 154, 247, 0.2); color: var(--tn-purple); }

.term-description {
  color: var(--tn-fg-dark);
  line-height: 1.7;
  font-size: 0.95rem;
}

.term-description code {
  background: var(--tn-bg-highlight);
  color: var(--tn-teal);
  padding: 0.1rem 0.4rem;
  border-radius: 3px;
  font-size: 0.85rem;
}

.term-description strong {
  color: var(--tn-yellow);
}

.term-example {
  margin-top: 0.75rem;
  padding: 0.75rem;
  background: var(--tn-bg);
  border-radius: 4px;
  font-size: 0.85rem;
  color: var(--tn-comment);
}

.term-example::before {
  content: "💡 ";
}

.term-see-also {
  margin-top: 0.5rem;
  font-size: 0.85rem;
  color: var(--tn-comment);
}

.term-see-also a {
  color: var(--tn-cyan);
  text-decoration: none;
}

.term-see-also a:hover {
  text-decoration: underline;
}

/* No results */
.no-results {
  text-align: center;
  padding: 3rem;
  color: var(--tn-comment);
}

.no-results-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

/* Highlight matching text */
.highlight {
  background: rgba(224, 175, 104, 0.3);
  color: var(--tn-yellow);
  padding: 0 2px;
  border-radius: 2px;
}

/* Quick nav */
.quick-nav {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.5rem;
  margin-bottom: 2rem;
  padding: 1rem;
  background: var(--tn-bg-dark);
  border-radius: 8px;
}

.quick-nav a {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--tn-bg-highlight);
  color: var(--tn-fg);
  text-decoration: none;
  border-radius: 4px;
  font-weight: bold;
  transition: all 0.2s ease;
}

.quick-nav a:hover {
  background: var(--tn-blue);
  color: var(--tn-bg);
  transform: scale(1.1);
}

.quick-nav a.disabled {
  opacity: 0.3;
  pointer-events: none;
}

/* Mobile responsive */
@media (max-width: 768px) {
  .glossary-container {
    padding: 1rem;
  }

  .glossary-header h1 {
    font-size: 1.8rem;
  }

  .letter {
    font-size: 2rem;
    width: 45px;
  }

  .term-name {
    flex-wrap: wrap;
  }
}
</style>

<div class="glossary-container">
  <div class="glossary-header">
    <h1>📚 Glossaire AWS & Kubernetes</h1>
    <p>Tous les termes et acronymes essentiels pour débuter</p>
  </div>

  <div class="search-container">
    <span class="search-icon">🔍</span>
    <input type="text" class="search-input" id="glossarySearch" placeholder="Rechercher un terme, acronyme ou mot-clé..." autocomplete="off">
    <button class="search-clear" id="clearSearch">✕</button>
  </div>

  <div class="results-count" id="resultsCount"></div>

  <nav class="quick-nav" id="quickNav">
    <a href="#letter-A">A</a>
    <a href="#letter-B">B</a>
    <a href="#letter-C">C</a>
    <a href="#letter-D">D</a>
    <a href="#letter-E">E</a>
    <a href="#letter-F">F</a>
    <a href="#letter-G">G</a>
    <a href="#letter-H">H</a>
    <a href="#letter-I">I</a>
    <a href="#letter-J">J</a>
    <a href="#letter-K">K</a>
    <a href="#letter-L">L</a>
    <a href="#letter-M">M</a>
    <a href="#letter-N">N</a>
    <a href="#letter-O">O</a>
    <a href="#letter-P">P</a>
    <a href="#letter-Q" class="disabled">Q</a>
    <a href="#letter-R">R</a>
    <a href="#letter-S">S</a>
    <a href="#letter-T">T</a>
    <a href="#letter-U" class="disabled">U</a>
    <a href="#letter-V">V</a>
    <a href="#letter-W">W</a>
    <a href="#letter-X" class="disabled">X</a>
    <a href="#letter-Y">Y</a>
    <a href="#letter-Z" class="disabled">Z</a>
  </nav>

  <div id="glossaryContent">
    <!-- A -->
    <div class="letter-section" id="letter-A">
      <div class="letter-header">
        <span class="letter">A</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card aws" data-term="ALB" data-search="alb application load balancer équilibreur charge layer 7 http https routing ingress">
        <div class="term-name">
          <h3>ALB</h3>
          <span class="term-acronym">Application Load Balancer</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Équilibreur de charge de niveau 7 (couche application) d'AWS. Il distribue le trafic HTTP/HTTPS vers plusieurs cibles (instances EC2, conteneurs, IPs) en fonction des règles de routage basées sur le contenu (URL, headers, etc.). C'est le type de Load Balancer recommandé pour les applications web et les APIs REST.
          <div class="term-example">Dans EKS, l'AWS Load Balancer Controller crée automatiquement un ALB lorsque vous déployez un Ingress Kubernetes.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="AMI" data-search="ami amazon machine image image système os ec2 instance">
        <div class="term-name">
          <h3>AMI</h3>
          <span class="term-acronym">Amazon Machine Image</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Image pré-configurée contenant le système d'exploitation, les applications et la configuration nécessaires pour lancer une instance EC2. AWS fournit des AMI optimisées pour EKS qui incluent le runtime containerd, kubelet, et tous les outils nécessaires pour qu'un node rejoigne un cluster Kubernetes.
          <div class="term-example">L'AMI EKS-optimized pour Kubernetes 1.33 est <code>amazon-eks-node-al2023-x86_64-standard-1.33</code>.</div>
        </div>
      </div>

      <div class="term-card general" data-term="API" data-search="api application programming interface interface programmation rest http endpoint">
        <div class="term-name">
          <h3>API</h3>
          <span class="term-acronym">Application Programming Interface</span>
          <span class="term-category general">Général</span>
        </div>
        <div class="term-description">
          Interface permettant à des applications de communiquer entre elles. Dans le contexte Kubernetes, l'<strong>API Server</strong> est le composant central qui expose l'API REST du cluster. Toutes les interactions avec le cluster (via kubectl, les controllers, etc.) passent par cette API.
          <div class="term-example"><code>kubectl get pods</code> envoie une requête GET à l'API Server sur l'endpoint <code>/api/v1/namespaces/default/pods</code>.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="ARN" data-search="arn amazon resource name identifiant unique ressource aws">
        <div class="term-name">
          <h3>ARN</h3>
          <span class="term-acronym">Amazon Resource Name</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Identifiant unique pour toute ressource AWS. Il suit le format <code>arn:aws:service:region:account-id:resource</code>. Les ARN sont utilisés dans les policies IAM pour spécifier précisément quelles ressources sont concernées par une autorisation.
          <div class="term-example"><code>arn:aws:eks:eu-west-1:123456789012:cluster/my-cluster</code> identifie un cluster EKS spécifique.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Autoscaling" data-search="autoscaling mise échelle automatique hpa vpa cluster autoscaler karpenter scaling">
        <div class="term-name">
          <h3>Autoscaling</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Mécanisme permettant d'ajuster automatiquement les ressources en fonction de la charge. Dans Kubernetes, il existe plusieurs niveaux : <strong>HPA</strong> (Horizontal Pod Autoscaler) pour les pods, <strong>VPA</strong> (Vertical Pod Autoscaler) pour les ressources des pods, et <strong>Cluster Autoscaler</strong> ou <strong>Karpenter</strong> pour les nodes.
          <div class="term-example">Si le CPU de vos pods dépasse 80%, le HPA peut automatiquement créer des pods supplémentaires.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="Availability Zone" data-search="az availability zone zone disponibilité datacenter région haute disponibilité ha">
        <div class="term-name">
          <h3>Availability Zone (AZ)</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Un ou plusieurs datacenters isolés au sein d'une région AWS. Chaque AZ dispose de sa propre alimentation électrique, réseau et connectivité, ce qui permet de créer des architectures hautement disponibles. Les ressources réparties sur plusieurs AZ survivent à la panne d'un datacenter entier.
          <div class="term-example">La région <code>eu-west-1</code> (Irlande) contient les AZ : <code>eu-west-1a</code>, <code>eu-west-1b</code>, <code>eu-west-1c</code>.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="AWS CLI" data-search="aws cli command line interface ligne commande terminal shell">
        <div class="term-name">
          <h3>AWS CLI</h3>
          <span class="term-acronym">AWS Command Line Interface</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Outil en ligne de commande permettant d'interagir avec les services AWS depuis un terminal. Il permet de gérer toutes les ressources AWS (EC2, S3, EKS, etc.) via des commandes. Essentiel pour l'automatisation et les scripts.
          <div class="term-example"><code>aws eks update-kubeconfig --name my-cluster</code> configure kubectl pour accéder à votre cluster EKS.</div>
        </div>
      </div>
    </div>

    <!-- B -->
    <div class="letter-section" id="letter-B">
      <div class="letter-header">
        <span class="letter">B</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card general" data-term="Bin Packing" data-search="bin packing optimisation ressources placement pods nodes efficacité">
        <div class="term-name">
          <h3>Bin Packing</h3>
          <span class="term-category general">Général</span>
        </div>
        <div class="term-description">
          Algorithme d'optimisation visant à placer le maximum de pods sur le minimum de nodes, comme ranger des objets dans des boîtes. Karpenter utilise le bin packing pour sélectionner le type d'instance EC2 optimal qui peut accueillir tous les pods en attente, réduisant ainsi les coûts et le gaspillage de ressources.
          <div class="term-example">Au lieu de créer 3 nodes t3.medium, Karpenter peut choisir 1 seul t3.xlarge pour héberger tous les pods.</div>
        </div>
      </div>
    </div>

    <!-- C -->
    <div class="letter-section" id="letter-C">
      <div class="letter-header">
        <span class="letter">C</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card network" data-term="CIDR" data-search="cidr classless inter-domain routing notation ip réseau subnet plage adresses">
        <div class="term-name">
          <h3>CIDR</h3>
          <span class="term-acronym">Classless Inter-Domain Routing</span>
          <span class="term-category network">Réseau</span>
        </div>
        <div class="term-description">
          Notation permettant de définir une plage d'adresses IP. Le format est <code>IP/nombre</code> où le nombre indique combien de bits sont fixes. Plus le nombre est petit, plus la plage est grande. <code>/16</code> = 65,536 IPs, <code>/20</code> = 4,096 IPs, <code>/24</code> = 256 IPs.
          <div class="term-example"><code>10.0.0.0/16</code> couvre toutes les IPs de <code>10.0.0.0</code> à <code>10.0.255.255</code>.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Cluster" data-search="cluster groupe serveurs nodes master worker ensemble kubernetes">
        <div class="term-name">
          <h3>Cluster</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Ensemble de machines (nodes) qui exécutent des applications conteneurisées gérées par Kubernetes. Un cluster comprend un <strong>Control Plane</strong> (qui prend les décisions globales) et des <strong>Worker Nodes</strong> (qui exécutent les applications). Dans EKS, le Control Plane est géré par AWS.
          <div class="term-example">Un cluster EKS typique peut avoir 2-3 worker nodes pour la production, avec autoscaling pour gérer les pics de charge.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="ConfigMap" data-search="configmap configuration données clé valeur environnement variables">
        <div class="term-name">
          <h3>ConfigMap</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Objet Kubernetes permettant de stocker des données de configuration non sensibles sous forme de paires clé-valeur. Les ConfigMaps permettent de séparer la configuration du code, facilitant ainsi le déploiement de la même application dans différents environnements.
          <div class="term-example">Stocker l'URL de votre API backend dans un ConfigMap permet de la modifier sans reconstruire l'image Docker.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Container" data-search="container conteneur docker image processus isolation application">
        <div class="term-name">
          <h3>Container (Conteneur)</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Unité d'exécution légère et portable qui encapsule une application avec toutes ses dépendances. Contrairement aux machines virtuelles, les conteneurs partagent le noyau du système hôte, ce qui les rend plus légers et rapides à démarrer. Docker et containerd sont les runtimes les plus courants.
          <div class="term-example">Un conteneur nginx démarre en quelques secondes et utilise seulement quelques Mo de RAM.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="containerd" data-search="containerd container runtime cri docker kubernetes execution">
        <div class="term-name">
          <h3>containerd</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Runtime de conteneur industriel, léger et performant. C'est le composant qui gère le cycle de vie des conteneurs (téléchargement des images, création, démarrage, arrêt). Depuis Kubernetes 1.24, containerd a remplacé Docker comme runtime par défaut dans la plupart des distributions Kubernetes, dont EKS.
          <div class="term-example">Les AMI EKS-optimized utilisent containerd 2.x comme runtime de conteneur.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Control Plane" data-search="control plane plan contrôle master api server etcd scheduler controller manager cerveau">
        <div class="term-name">
          <h3>Control Plane</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Le "cerveau" du cluster Kubernetes. Il comprend : l'<strong>API Server</strong> (point d'entrée), <strong>etcd</strong> (base de données), le <strong>Scheduler</strong> (placement des pods), et le <strong>Controller Manager</strong> (boucles de contrôle). Dans EKS, le Control Plane est entièrement géré par AWS et facturé ~$73/mois.
          <div class="term-example">Vous n'avez jamais accès direct aux machines du Control Plane EKS - c'est un service managé.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="CoreDNS" data-search="coredns dns résolution nom service discovery découverte services kubernetes">
        <div class="term-name">
          <h3>CoreDNS</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Serveur DNS flexible déployé dans chaque cluster Kubernetes. Il permet aux pods de résoudre les noms des Services Kubernetes (ex: <code>my-service.default.svc.cluster.local</code>) en adresses IP internes. C'est un composant critique pour la communication inter-services.
          <div class="term-example">Un pod peut appeler <code>http://backend-api</code> et CoreDNS résout automatiquement vers le bon Service.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="CRD" data-search="crd custom resource definition ressource personnalisée extension api kubernetes">
        <div class="term-name">
          <h3>CRD</h3>
          <span class="term-acronym">Custom Resource Definition</span>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Mécanisme permettant d'étendre l'API Kubernetes avec vos propres types de ressources. Les CRDs permettent de définir des objets personnalisés (comme <code>NodePool</code> pour Karpenter) qui sont ensuite gérés par des controllers dédiés, exactement comme les ressources natives.
          <div class="term-example">Karpenter définit les CRDs <code>NodePool</code> et <code>EC2NodeClass</code> pour configurer l'autoscaling.</div>
        </div>
      </div>
    </div>

    <!-- D -->
    <div class="letter-section" id="letter-D">
      <div class="letter-header">
        <span class="letter">D</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="DaemonSet" data-search="daemonset daemon pod chaque node monitoring agent logs">
        <div class="term-name">
          <h3>DaemonSet</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Objet Kubernetes garantissant qu'un pod s'exécute sur chaque node du cluster (ou un sous-ensemble). Idéal pour les agents de monitoring, collecteurs de logs, ou composants réseau qui doivent être présents partout. Quand un nouveau node rejoint le cluster, le DaemonSet y déploie automatiquement un pod.
          <div class="term-example">Les composants système comme <code>kube-proxy</code> et <code>aws-node</code> (VPC CNI) sont déployés via DaemonSet.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Deployment" data-search="deployment déploiement pods replicas mise à jour rolling update application">
        <div class="term-name">
          <h3>Deployment</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Objet Kubernetes déclaratif pour gérer des applications stateless. Il définit le nombre souhaité de réplicas (pods identiques), l'image à utiliser, et la stratégie de mise à jour. Le Deployment Controller s'assure que l'état actuel correspond toujours à l'état désiré.
          <div class="term-example"><code>kubectl create deployment nginx --image=nginx --replicas=3</code> crée 3 pods nginx identiques.</div>
        </div>
      </div>

      <div class="term-card general" data-term="Docker" data-search="docker conteneur image container registry build">
        <div class="term-name">
          <h3>Docker</h3>
          <span class="term-category general">Général</span>
        </div>
        <div class="term-description">
          Plateforme de conteneurisation qui a popularisé les conteneurs. Docker permet de construire (<code>docker build</code>), partager (<code>docker push</code>) et exécuter (<code>docker run</code>) des conteneurs. Bien que Kubernetes n'utilise plus Docker comme runtime, le format d'image Docker (OCI) reste le standard.
          <div class="term-example">Les images Docker sont stockées dans des registres comme Docker Hub, ECR (AWS), ou GitHub Container Registry.</div>
        </div>
      </div>
    </div>

    <!-- E -->
    <div class="letter-section" id="letter-E">
      <div class="letter-header">
        <span class="letter">E</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card aws" data-term="EBS" data-search="ebs elastic block store stockage persistant volume disque ssd">
        <div class="term-name">
          <h3>EBS</h3>
          <span class="term-acronym">Elastic Block Store</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Service de stockage bloc persistant pour EC2. Les volumes EBS sont comme des disques durs virtuels qui persistent indépendamment du cycle de vie des instances. Dans Kubernetes, le CSI Driver EBS permet d'utiliser des volumes EBS comme PersistentVolumes pour les données qui doivent survivre aux redémarrages de pods.
          <div class="term-example">Une base de données PostgreSQL dans Kubernetes utilise un PersistentVolume EBS de type <code>gp3</code> pour stocker ses données.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="EC2" data-search="ec2 elastic compute cloud instance serveur virtuel machine vm compute">
        <div class="term-name">
          <h3>EC2</h3>
          <span class="term-acronym">Elastic Compute Cloud</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Service de machines virtuelles d'AWS. EC2 fournit une capacité de calcul redimensionnable dans le cloud. Dans le contexte EKS, les worker nodes sont des instances EC2 qui exécutent vos pods. Différents types d'instances (t3, m5, c5, etc.) offrent différents ratios CPU/mémoire.
          <div class="term-example">Une instance <code>t3.medium</code> offre 2 vCPUs et 4 Go de RAM, idéale pour des workloads légers.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="ECR" data-search="ecr elastic container registry docker images registry conteneur stockage">
        <div class="term-name">
          <h3>ECR</h3>
          <span class="term-acronym">Elastic Container Registry</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Registre Docker privé entièrement géré par AWS. ECR stocke, gère et déploie vos images de conteneurs. Il s'intègre nativement avec EKS et IAM pour le contrôle d'accès. Les images sont automatiquement scannées pour détecter les vulnérabilités.
          <div class="term-example">Format d'une image ECR : <code>123456789012.dkr.ecr.eu-west-1.amazonaws.com/my-app:v1.0</code></div>
        </div>
      </div>

      <div class="term-card aws" data-term="EKS" data-search="eks elastic kubernetes service managed cluster aws conteneur orchestration">
        <div class="term-name">
          <h3>EKS</h3>
          <span class="term-acronym">Elastic Kubernetes Service</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Service Kubernetes managé d'AWS. EKS gère le Control Plane (API Server, etcd, Scheduler, Controller Manager) pour vous, garantissant haute disponibilité et mises à jour automatiques. Vous gérez uniquement les worker nodes et vos applications. Alternative à l'auto-gestion d'un cluster Kubernetes.
          <div class="term-example">EKS supporte les versions Kubernetes de n-2 à n (ex: si la dernière est 1.33, EKS supporte 1.31, 1.32, 1.33).</div>
        </div>
      </div>

      <div class="term-card aws" data-term="Elastic IP" data-search="elastic ip eip adresse ip statique publique fixe nat gateway">
        <div class="term-name">
          <h3>Elastic IP</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Adresse IPv4 publique statique allouée à votre compte AWS. Contrairement aux IPs publiques normales qui changent au redémarrage, une Elastic IP reste fixe. Elle est utilisée pour les NAT Gateways et les ressources qui nécessitent une IP stable et prévisible.
          <div class="term-example">Le NAT Gateway de votre VPC utilise une Elastic IP pour que vos nodes privés accèdent à internet avec une IP constante.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="etcd" data-search="etcd base données stockage état cluster key value distributed">
        <div class="term-name">
          <h3>etcd</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Base de données clé-valeur distribuée qui stocke tout l'état du cluster Kubernetes. Chaque objet créé (Pods, Services, ConfigMaps, etc.) est persisté dans etcd. C'est le composant le plus critique du Control Plane - si etcd est perdu, le cluster perd toute sa configuration.
          <div class="term-example">Dans EKS, AWS gère etcd avec réplication multi-AZ et sauvegardes automatiques - vous n'y avez pas accès directement.</div>
        </div>
      </div>
    </div>

    <!-- F -->
    <div class="letter-section" id="letter-F">
      <div class="letter-header">
        <span class="letter">F</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Finalizer" data-search="finalizer suppression deletion protection nettoyage ressource">
        <div class="term-name">
          <h3>Finalizer</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Mécanisme permettant à un controller d'effectuer des actions de nettoyage avant la suppression définitive d'une ressource. Quand vous supprimez un objet avec finalizer, il passe en état "Terminating" jusqu'à ce que le controller retire le finalizer après avoir terminé son nettoyage.
          <div class="term-example">Karpenter utilise des finalizers pour s'assurer que les instances EC2 sont terminées avant de supprimer les NodeClaims.</div>
        </div>
      </div>
    </div>

    <!-- G -->
    <div class="letter-section" id="letter-G">
      <div class="letter-header">
        <span class="letter">G</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card network" data-term="Gateway" data-search="gateway passerelle internet nat réseau routing trafic">
        <div class="term-name">
          <h3>Gateway</h3>
          <span class="term-category network">Réseau</span>
        </div>
        <div class="term-description">
          Point d'entrée ou de sortie du réseau. Dans AWS VPC, l'<strong>Internet Gateway</strong> permet la communication avec internet, tandis que le <strong>NAT Gateway</strong> permet aux ressources privées d'accéder à internet sans être directement exposées. Un gateway route le trafic entre différents réseaux.
          <div class="term-example">Le NAT Gateway permet à vos worker nodes en subnet privé de télécharger des images Docker depuis internet.</div>
        </div>
      </div>
    </div>

    <!-- H -->
    <div class="letter-section" id="letter-H">
      <div class="letter-header">
        <span class="letter">H</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card general" data-term="Helm" data-search="helm package manager charts kubernetes déploiement templates">
        <div class="term-name">
          <h3>Helm</h3>
          <span class="term-category general">Général</span>
        </div>
        <div class="term-description">
          Gestionnaire de packages pour Kubernetes, souvent appelé "le apt/yum de Kubernetes". Helm utilise des <strong>Charts</strong> (packages) qui contiennent des templates Kubernetes avec des valeurs configurables. Il simplifie l'installation et la mise à jour d'applications complexes.
          <div class="term-example"><code>helm install karpenter oci://public.ecr.aws/karpenter/karpenter --version 1.1.1</code></div>
        </div>
      </div>

      <div class="term-card k8s" data-term="HPA" data-search="hpa horizontal pod autoscaler autoscaling pods réplicas cpu mémoire métriques">
        <div class="term-name">
          <h3>HPA</h3>
          <span class="term-acronym">Horizontal Pod Autoscaler</span>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Controller Kubernetes qui ajuste automatiquement le nombre de réplicas d'un Deployment en fonction de métriques (CPU, mémoire, ou métriques custom). Quand la charge augmente, le HPA crée plus de pods ; quand elle diminue, il en supprime.
          <div class="term-example"><code>kubectl autoscale deployment my-app --cpu-percent=80 --min=2 --max=10</code></div>
        </div>
      </div>
    </div>

    <!-- I -->
    <div class="letter-section" id="letter-I">
      <div class="letter-header">
        <span class="letter">I</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card security" data-term="IAM" data-search="iam identity access management identité accès permissions rôles policies sécurité">
        <div class="term-name">
          <h3>IAM</h3>
          <span class="term-acronym">Identity and Access Management</span>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Service AWS de gestion des identités et des accès. IAM permet de créer des utilisateurs, groupes et rôles avec des permissions granulaires définies par des <strong>policies</strong>. C'est le système d'autorisation central d'AWS - toute action sur une ressource AWS passe par IAM.
          <div class="term-example">Un rôle IAM avec la policy <code>AmazonEKSClusterPolicy</code> permet au Control Plane EKS de gérer les ressources AWS.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Ingress" data-search="ingress entrée http https routing load balancer external accès externe tls ssl">
        <div class="term-name">
          <h3>Ingress</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Objet Kubernetes qui gère l'accès externe aux Services du cluster, généralement HTTP/HTTPS. Un Ingress définit des règles de routage (par host ou path) et peut gérer la terminaison TLS. Il nécessite un <strong>Ingress Controller</strong> (comme AWS Load Balancer Controller) pour être fonctionnel.
          <div class="term-example">Un Ingress peut router <code>api.example.com</code> vers le service API et <code>app.example.com</code> vers le frontend.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="Instance Profile" data-search="instance profile rôle ec2 iam credentials métadonnées">
        <div class="term-name">
          <h3>Instance Profile</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Conteneur IAM qui attache un rôle à une instance EC2. Il permet à l'instance d'assumer le rôle et d'obtenir des credentials temporaires pour appeler les APIs AWS. Les worker nodes EKS utilisent un Instance Profile pour accéder à ECR, créer des ENI, etc.
          <div class="term-example">Karpenter nécessite un Instance Profile pré-créé pour les nodes qu'il provisionne.</div>
        </div>
      </div>

      <div class="term-card security" data-term="IRSA" data-search="irsa iam roles service accounts pods sécurité authentification oidc">
        <div class="term-name">
          <h3>IRSA</h3>
          <span class="term-acronym">IAM Roles for Service Accounts</span>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Mécanisme permettant aux pods Kubernetes d'assumer des rôles IAM de manière sécurisée. Au lieu de donner des permissions à tout le node, IRSA permet d'assigner des permissions IAM granulaires à des ServiceAccounts spécifiques. Utilise OIDC pour l'authentification.
          <div class="term-example">Le controller Karpenter utilise IRSA pour avoir uniquement les permissions EC2 nécessaires, pas tout le node.</div>
        </div>
      </div>
    </div>

    <!-- J -->
    <div class="letter-section" id="letter-J">
      <div class="letter-header">
        <span class="letter">J</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Job" data-search="job tâche batch exécution unique cron scheduled">
        <div class="term-name">
          <h3>Job</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Objet Kubernetes pour exécuter des tâches ponctuelles jusqu'à complétion (contrairement aux Deployments qui tournent indéfiniment). Un Job crée un ou plusieurs pods et s'assure qu'ils terminent avec succès. Pour les tâches récurrentes, utilisez <strong>CronJob</strong>.
          <div class="term-example">Un Job de migration de base de données s'exécute une fois au déploiement puis se termine.</div>
        </div>
      </div>
    </div>

    <!-- K -->
    <div class="letter-section" id="letter-K">
      <div class="letter-header">
        <span class="letter">K</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Karpenter" data-search="karpenter autoscaler nodes provisioning scaling dynamique instances ec2">
        <div class="term-name">
          <h3>Karpenter</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Autoscaler de nodes open-source développé par AWS. Contrairement au Cluster Autoscaler qui utilise des Node Groups prédéfinis, Karpenter sélectionne dynamiquement le type d'instance optimal pour les pods en attente. Il provisionne en ~40 secondes (vs 3-5 min) et consolide automatiquement les nodes sous-utilisés.
          <div class="term-example">Quand 3 pods demandent 1 CPU chacun, Karpenter choisit un t3a.xlarge (4 CPU) plutôt que 3 t3.medium.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="kubectl" data-search="kubectl cli command line kubernetes client outil commande">
        <div class="term-name">
          <h3>kubectl</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Outil en ligne de commande officiel pour interagir avec les clusters Kubernetes. kubectl envoie des requêtes à l'API Server et affiche les résultats. C'est l'outil principal pour gérer les ressources Kubernetes (pods, services, deployments, etc.).
          <div class="term-example"><code>kubectl get pods -A</code> liste tous les pods dans tous les namespaces.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="kubelet" data-search="kubelet agent node pod container runtime gestion worker">
        <div class="term-name">
          <h3>kubelet</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Agent qui tourne sur chaque worker node. Le kubelet reçoit les spécifications de pods depuis l'API Server et s'assure que les conteneurs correspondants sont en cours d'exécution et en bonne santé. Il communique avec le container runtime (containerd) pour gérer le cycle de vie des conteneurs.
          <div class="term-example">Si un conteneur crash, le kubelet le redémarre automatiquement selon la restartPolicy du pod.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="kube-proxy" data-search="kube-proxy réseau service iptables networking load balancing interne">
        <div class="term-name">
          <h3>kube-proxy</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Composant réseau qui tourne sur chaque node et maintient les règles réseau permettant la communication vers les Services Kubernetes. Il utilise iptables (ou IPVS) pour rediriger le trafic destiné à une ClusterIP vers les pods backend appropriés.
          <div class="term-example">Quand vous appelez un Service, kube-proxy route la requête vers l'un des pods backend de manière équilibrée.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Kubernetes" data-search="kubernetes k8s orchestration conteneur container open source google">
        <div class="term-name">
          <h3>Kubernetes (K8s)</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Plateforme open-source d'orchestration de conteneurs, initialement développée par Google. Kubernetes automatise le déploiement, la mise à l'échelle et la gestion des applications conteneurisées. "K8s" est l'abréviation (K + 8 lettres + s). C'est devenu le standard de facto pour l'orchestration de conteneurs.
          <div class="term-example">Kubernetes gère automatiquement le redémarrage des pods défaillants, le load balancing, et les mises à jour sans downtime.</div>
        </div>
      </div>
    </div>

    <!-- L -->
    <div class="letter-section" id="letter-L">
      <div class="letter-header">
        <span class="letter">L</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Label" data-search="label étiquette sélection metadata pods selector organisation">
        <div class="term-name">
          <h3>Label</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Paire clé-valeur attachée aux objets Kubernetes pour les identifier et les organiser. Les labels permettent de sélectionner des groupes d'objets via des <strong>selectors</strong>. Contrairement aux annotations, les labels sont utilisés pour le filtrage et la sélection.
          <div class="term-example"><code>kubectl get pods -l app=nginx,env=prod</code> sélectionne les pods avec ces deux labels.</div>
        </div>
      </div>

      <div class="term-card network" data-term="Load Balancer" data-search="load balancer équilibreur charge distribution trafic alb nlb elb haute disponibilité">
        <div class="term-name">
          <h3>Load Balancer</h3>
          <span class="term-category network">Réseau</span>
        </div>
        <div class="term-description">
          Composant qui distribue le trafic réseau entre plusieurs serveurs pour assurer haute disponibilité et performance. AWS propose l'ALB (Layer 7/HTTP), le NLB (Layer 4/TCP), et le CLB (Classic, déprécié). Dans Kubernetes, un Service de type <code>LoadBalancer</code> provisionne automatiquement un LB cloud.
          <div class="term-example">L'AWS Load Balancer Controller crée un ALB quand vous déployez un Ingress avec l'annotation appropriée.</div>
        </div>
      </div>
    </div>

    <!-- M -->
    <div class="letter-section" id="letter-M">
      <div class="letter-header">
        <span class="letter">M</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Manifest" data-search="manifest yaml fichier configuration déclaratif ressource kubernetes spec">
        <div class="term-name">
          <h3>Manifest</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Fichier YAML (ou JSON) décrivant une ressource Kubernetes. Un manifest contient l'<code>apiVersion</code>, le <code>kind</code>, les <code>metadata</code> et la <code>spec</code> de l'objet souhaité. Kubernetes utilise ces fichiers pour créer et maintenir l'état désiré du cluster.
          <div class="term-example"><code>kubectl apply -f deployment.yaml</code> applique le manifest et crée/met à jour la ressource.</div>
        </div>
      </div>
    </div>

    <!-- N -->
    <div class="letter-section" id="letter-N">
      <div class="letter-header">
        <span class="letter">N</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Namespace" data-search="namespace espace nom isolation multi-tenant séparation ressources">
        <div class="term-name">
          <h3>Namespace</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Mécanisme de partitionnement logique d'un cluster Kubernetes. Les namespaces permettent d'isoler les ressources entre équipes ou environnements (dev, staging, prod). Les noms de ressources doivent être uniques dans un namespace, mais peuvent être dupliqués entre namespaces différents.
          <div class="term-example">Namespaces par défaut : <code>default</code>, <code>kube-system</code> (composants système), <code>kube-public</code>.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="NAT Gateway" data-search="nat network address translation gateway sortie internet privé subnet">
        <div class="term-name">
          <h3>NAT Gateway</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Service AWS permettant aux ressources dans un subnet privé d'accéder à internet (pour télécharger des mises à jour, images Docker, etc.) sans être directement accessibles depuis internet. Le NAT Gateway traduit les adresses IP privées en son adresse IP publique (Elastic IP).
          <div class="term-example">Coût : ~$45/mois + frais de transfert de données. Utilisez <code>single_nat_gateway = true</code> pour économiser en sandbox.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Node" data-search="node nœud machine serveur worker kubelet pods exécution">
        <div class="term-name">
          <h3>Node</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Machine (virtuelle ou physique) qui exécute des pods dans un cluster Kubernetes. Chaque node exécute le kubelet, un container runtime (containerd), et kube-proxy. Dans EKS, les nodes sont des instances EC2 qui rejoignent le cluster via le kubelet.
          <div class="term-example"><code>kubectl get nodes</code> liste tous les nodes du cluster avec leur status et version Kubernetes.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="NodeClaim" data-search="nodeclaim karpenter demande node provisioning instance ec2">
        <div class="term-name">
          <h3>NodeClaim</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Ressource Karpenter représentant une demande de node. Quand Karpenter détecte des pods non schedulables, il crée un NodeClaim qui décrit les besoins (CPU, mémoire, etc.). Le NodeClaim passe par les états : Created → Launched → Registered → Ready, puis devient un Node Kubernetes.
          <div class="term-example"><code>kubectl get nodeclaims</code> montre les nodes en cours de provisionnement par Karpenter.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="Node Group" data-search="node group groupe nœuds managed autoscaling asg instances ec2">
        <div class="term-name">
          <h3>Node Group</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Groupe d'instances EC2 homogènes servant de worker nodes EKS. Un <strong>Managed Node Group</strong> est géré par AWS (mises à jour automatiques, drain), tandis qu'un <strong>Self-Managed Node Group</strong> vous donne plus de contrôle. Les Node Groups utilisent des Auto Scaling Groups (ASG) sous le capot.
          <div class="term-example">Un Node Group avec <code>min=1, max=5, desired=2</code> maintient 2 nodes et peut scaler jusqu'à 5.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="NodePool" data-search="nodepool karpenter configuration contraintes instances types">
        <div class="term-name">
          <h3>NodePool</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          CRD Karpenter définissant les contraintes et limites pour le provisionnement de nodes. Un NodePool spécifie les types d'instances autorisés, les AZs, les limites CPU/mémoire, et la politique de consolidation. Plusieurs NodePools peuvent coexister pour différents workloads.
          <div class="term-example">Un NodePool peut autoriser uniquement les instances Spot pour les workloads non critiques.</div>
        </div>
      </div>
    </div>

    <!-- O -->
    <div class="letter-section" id="letter-O">
      <div class="letter-header">
        <span class="letter">O</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card security" data-term="OIDC" data-search="oidc openid connect authentification identity token jwt sso">
        <div class="term-name">
          <h3>OIDC</h3>
          <span class="term-acronym">OpenID Connect</span>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Protocole d'authentification basé sur OAuth 2.0. Dans EKS, chaque cluster a un <strong>OIDC Provider</strong> qui permet à AWS IAM de faire confiance aux tokens émis par Kubernetes. C'est la base d'IRSA pour donner des permissions IAM aux pods.
          <div class="term-example">L'URL OIDC d'un cluster EKS : <code>https://oidc.eks.eu-west-1.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE</code></div>
        </div>
      </div>

      <div class="term-card aws" data-term="On-Demand" data-search="on-demand instance ec2 prix fixe disponibilité garanti pas interruption">
        <div class="term-name">
          <h3>On-Demand</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Mode de tarification EC2 où vous payez à l'heure ou à la seconde sans engagement. Les instances On-Demand offrent une disponibilité garantie mais sont plus chères que les Spot Instances. C'est le choix par défaut pour les workloads critiques qui ne peuvent pas tolérer d'interruptions.
          <div class="term-example">Une instance <code>t3.medium</code> On-Demand coûte environ $0.042/heure en eu-west-1.</div>
        </div>
      </div>
    </div>

    <!-- P -->
    <div class="letter-section" id="letter-P">
      <div class="letter-header">
        <span class="letter">P</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Pod" data-search="pod conteneur container unité base kubernetes plus petit déployable">
        <div class="term-name">
          <h3>Pod</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Plus petite unité déployable dans Kubernetes. Un pod encapsule un ou plusieurs conteneurs qui partagent le même réseau (localhost) et stockage. La plupart des pods ne contiennent qu'un seul conteneur. Les pods sont éphémères - ils peuvent être détruits et recréés à tout moment.
          <div class="term-example">Un pod nginx avec un sidecar pour les logs contient 2 conteneurs qui communiquent via localhost.</div>
        </div>
      </div>

      <div class="term-card security" data-term="Policy IAM" data-search="policy iam politique permissions autorisation json document aws">
        <div class="term-name">
          <h3>Policy (IAM)</h3>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Document JSON définissant les permissions dans AWS IAM. Une policy spécifie quelles <strong>Actions</strong> sont autorisées ou refusées sur quelles <strong>Resources</strong>. Les policies peuvent être attachées à des utilisateurs, groupes ou rôles. Le principe du moindre privilège recommande de n'accorder que les permissions strictement nécessaires.
          <div class="term-example"><code>{"Effect": "Allow", "Action": "ec2:RunInstances", "Resource": "*"}</code></div>
        </div>
      </div>

      <div class="term-card k8s" data-term="PersistentVolume" data-search="pv persistent volume stockage persistant données disque ebs">
        <div class="term-name">
          <h3>PersistentVolume (PV)</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Ressource de stockage dans le cluster, provisionné par un administrateur ou dynamiquement via une StorageClass. Un PV a un cycle de vie indépendant des pods qui l'utilisent. Les données persistent même après la suppression du pod.
          <div class="term-example">Un PV EBS de 100 Go peut être attaché à un pod PostgreSQL pour stocker la base de données.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="PersistentVolumeClaim" data-search="pvc persistent volume claim demande stockage liaison pod">
        <div class="term-name">
          <h3>PersistentVolumeClaim (PVC)</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Demande de stockage par un utilisateur. Un PVC spécifie la taille et le mode d'accès souhaités. Kubernetes lie automatiquement le PVC à un PV correspondant. C'est l'abstraction qui permet aux développeurs de demander du stockage sans connaître les détails de l'infrastructure.
          <div class="term-example"><code>kubectl get pvc</code> montre les demandes de stockage et leur état (Bound, Pending).</div>
        </div>
      </div>
    </div>

    <!-- R -->
    <div class="letter-section" id="letter-R">
      <div class="letter-header">
        <span class="letter">R</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card security" data-term="RBAC" data-search="rbac role based access control contrôle accès rôle permissions kubernetes">
        <div class="term-name">
          <h3>RBAC</h3>
          <span class="term-acronym">Role-Based Access Control</span>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Méthode de régulation de l'accès aux ressources Kubernetes basée sur les rôles. RBAC utilise des <strong>Roles</strong> (permissions dans un namespace) ou <strong>ClusterRoles</strong> (cluster-wide), liés à des utilisateurs via <strong>RoleBindings</strong> ou <strong>ClusterRoleBindings</strong>.
          <div class="term-example">Un développeur peut avoir un Role permettant de lire les pods dans le namespace "dev" mais pas de les supprimer.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="Region" data-search="region région géographique datacenter localisation aws">
        <div class="term-name">
          <h3>Region</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Zone géographique AWS contenant plusieurs Availability Zones. Chaque région est complètement isolée des autres pour une meilleure tolérance aux pannes. Le choix de la région impacte la latence, la conformité aux réglementations (RGPD), et les coûts.
          <div class="term-example">Principales régions européennes : <code>eu-west-1</code> (Irlande), <code>eu-west-3</code> (Paris), <code>eu-central-1</code> (Francfort).</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="ReplicaSet" data-search="replicaset replicas pods nombre copies identiques garantie">
        <div class="term-name">
          <h3>ReplicaSet</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Controller garantissant qu'un nombre spécifié de réplicas de pods sont en cours d'exécution. Si un pod meurt, le ReplicaSet en crée un nouveau. En pratique, on utilise rarement les ReplicaSets directement - on passe par les Deployments qui les gèrent automatiquement.
          <div class="term-example">Un Deployment crée un ReplicaSet, qui lui-même crée et gère les pods.</div>
        </div>
      </div>

      <div class="term-card security" data-term="Role IAM" data-search="role iam rôle identité assumable permissions temporaire aws">
        <div class="term-name">
          <h3>Role (IAM)</h3>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Identité IAM avec des permissions, conçue pour être assumée plutôt qu'associée directement à une personne. Un rôle fournit des credentials temporaires. Les instances EC2, les fonctions Lambda, et les pods EKS (via IRSA) peuvent assumer des rôles pour accéder aux services AWS.
          <div class="term-example">Le rôle <code>KarpenterControllerRole</code> permet au pod Karpenter de créer et terminer des instances EC2.</div>
        </div>
      </div>
    </div>

    <!-- S -->
    <div class="letter-section" id="letter-S">
      <div class="letter-header">
        <span class="letter">S</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card aws" data-term="S3" data-search="s3 simple storage service stockage objet bucket fichiers">
        <div class="term-name">
          <h3>S3</h3>
          <span class="term-acronym">Simple Storage Service</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Service de stockage d'objets AWS, infiniment scalable. S3 stocke des données sous forme d'objets dans des <strong>buckets</strong>. Idéal pour les fichiers statiques, backups, logs, et data lakes. Intégration native avec de nombreux services AWS.
          <div class="term-example">Terraform peut stocker son state dans un bucket S3 pour le partager entre membres d'une équipe.</div>
        </div>
      </div>

      <div class="term-card security" data-term="SCP" data-search="scp service control policy organisation restrictions limites aws">
        <div class="term-name">
          <h3>SCP</h3>
          <span class="term-acronym">Service Control Policy</span>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Politique au niveau de l'organisation AWS qui définit les permissions maximales pour les comptes membres. Les SCPs sont des "guardrails" - elles ne donnent pas de permissions mais limitent ce qui est possible. Même un administrateur ne peut pas outrepasser une SCP.
          <div class="term-example">Les comptes de formation AWS ont des SCPs qui bloquent KMS, certaines API SSM, et les Spot Instances.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Scheduler" data-search="scheduler planificateur placement pods nodes affectation">
        <div class="term-name">
          <h3>Scheduler</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Composant du Control Plane qui décide sur quel node placer les nouveaux pods. Le Scheduler prend en compte les ressources disponibles, les contraintes (nodeSelector, affinity, taints/tolerations), et tente d'équilibrer la charge. Un pod "Pending" attend généralement une décision du Scheduler.
          <div class="term-example">Si aucun node n'a assez de ressources, le pod reste Pending et Karpenter peut créer un nouveau node.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Secret" data-search="secret données sensibles mot de passe credentials base64 sécurité">
        <div class="term-name">
          <h3>Secret</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Objet Kubernetes pour stocker des données sensibles (mots de passe, tokens, clés). Les Secrets sont encodés en base64 (pas chiffrés par défaut !). Ils peuvent être montés comme fichiers ou exposés comme variables d'environnement dans les pods.
          <div class="term-example"><code>kubectl create secret generic db-pass --from-literal=password=mysecret</code></div>
        </div>
      </div>

      <div class="term-card network" data-term="Security Group" data-search="security group sg firewall pare-feu règles trafic réseau aws">
        <div class="term-name">
          <h3>Security Group</h3>
          <span class="term-category network">Réseau</span>
        </div>
        <div class="term-description">
          Pare-feu virtuel AWS contrôlant le trafic entrant et sortant des ressources. Les Security Groups sont stateful (si vous autorisez le trafic entrant, la réponse est automatiquement autorisée). Ils s'appliquent au niveau de l'instance/ENI, pas du subnet.
          <div class="term-example">Un SG pour les worker nodes autorise le trafic depuis le Control Plane (port 443, 10250) et entre nodes.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Service" data-search="service exposition réseau endpoint clusterip nodeport loadbalancer découverte">
        <div class="term-name">
          <h3>Service</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Abstraction qui définit un ensemble logique de pods et une politique d'accès réseau. Un Service fournit une IP stable et un nom DNS pour accéder à des pods qui peuvent changer. Types : <strong>ClusterIP</strong> (interne), <strong>NodePort</strong> (port sur chaque node), <strong>LoadBalancer</strong> (LB cloud).
          <div class="term-example"><code>kubectl expose deployment nginx --port=80 --type=LoadBalancer</code></div>
        </div>
      </div>

      <div class="term-card k8s" data-term="ServiceAccount" data-search="serviceaccount sa identité pod authentification api rbac">
        <div class="term-name">
          <h3>ServiceAccount</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Identité utilisée par les processus dans les pods pour s'authentifier auprès de l'API Server. Chaque namespace a un ServiceAccount <code>default</code>. Les ServiceAccounts peuvent avoir des permissions RBAC et, avec IRSA, des permissions IAM AWS.
          <div class="term-example">Le ServiceAccount <code>karpenter</code> dans <code>kube-system</code> est annoté avec un rôle IAM pour IRSA.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="Spot Instance" data-search="spot instance ec2 économie réduction prix interruption spare capacity">
        <div class="term-name">
          <h3>Spot Instance</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Instance EC2 utilisant la capacité excédentaire AWS à prix réduit (jusqu'à 90% moins cher). En contrepartie, AWS peut interrompre l'instance avec un préavis de 2 minutes quand il a besoin de la capacité. Idéal pour les workloads tolérants aux interruptions (batch, CI/CD, stateless).
          <div class="term-example">Karpenter peut mixer Spot et On-Demand : Spot pour les workloads non critiques, On-Demand pour le reste.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="Subnet" data-search="subnet sous-réseau ip plage vpc public privé réseau">
        <div class="term-name">
          <h3>Subnet</h3>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Sous-division d'un VPC avec sa propre plage d'adresses IP. Un subnet existe dans une seule AZ. Les <strong>subnets publics</strong> ont une route vers l'Internet Gateway (ressources accessibles depuis internet). Les <strong>subnets privés</strong> n'ont pas d'accès direct à internet.
          <div class="term-example">Architecture typique : worker nodes en subnets privés, load balancers en subnets publics.</div>
        </div>
      </div>
    </div>

    <!-- T -->
    <div class="letter-section" id="letter-T">
      <div class="letter-header">
        <span class="letter">T</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Taint" data-search="taint toleration répulsion node pods scheduling exclusion">
        <div class="term-name">
          <h3>Taint & Toleration</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Mécanisme pour empêcher les pods d'être schedulés sur certains nodes. Un <strong>Taint</strong> sur un node repousse les pods, sauf ceux qui ont une <strong>Toleration</strong> correspondante. Utilisé pour dédier des nodes à certains workloads ou éviter de scheduler pendant la maintenance.
          <div class="term-example">Karpenter applique le taint <code>karpenter.sh/disrupted</code> avant de supprimer un node pour éviter de nouveaux pods.</div>
        </div>
      </div>

      <div class="term-card general" data-term="Terraform" data-search="terraform iac infrastructure as code hashicorp hcl provisioning automation">
        <div class="term-name">
          <h3>Terraform</h3>
          <span class="term-category general">Général</span>
        </div>
        <div class="term-description">
          Outil d'Infrastructure as Code (IaC) open-source de HashiCorp. Terraform permet de définir et provisionner des ressources cloud (AWS, GCP, Azure) via des fichiers de configuration HCL. Il maintient un <strong>state</strong> pour suivre les ressources créées et appliquer les changements de manière idempotente.
          <div class="term-example"><code>terraform apply</code> crée/modifie les ressources pour atteindre l'état défini dans les fichiers .tf.</div>
        </div>
      </div>

      <div class="term-card security" data-term="TLS" data-search="tls ssl https certificat chiffrement sécurité transport encryption">
        <div class="term-name">
          <h3>TLS</h3>
          <span class="term-acronym">Transport Layer Security</span>
          <span class="term-category security">Sécurité</span>
        </div>
        <div class="term-description">
          Protocole cryptographique assurant la confidentialité et l'intégrité des communications réseau. TLS (successeur de SSL) est utilisé pour HTTPS. Dans Kubernetes, la communication avec l'API Server est toujours chiffrée via TLS. Les Ingress peuvent gérer la terminaison TLS pour vos applications.
          <div class="term-example">Un Ingress avec l'annotation TLS provisionne automatiquement un certificat via cert-manager ou ACM.</div>
        </div>
      </div>
    </div>

    <!-- V -->
    <div class="letter-section" id="letter-V">
      <div class="letter-header">
        <span class="letter">V</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card network" data-term="VPC" data-search="vpc virtual private cloud réseau virtuel isolation aws">
        <div class="term-name">
          <h3>VPC</h3>
          <span class="term-acronym">Virtual Private Cloud</span>
          <span class="term-category network">Réseau</span>
        </div>
        <div class="term-description">
          Réseau virtuel isolé dans AWS où vous lancez vos ressources. Vous contrôlez la plage d'IP (CIDR), les subnets, les tables de routage et les gateways. Un VPC est régional et peut s'étendre sur toutes les AZs de la région. C'est la fondation réseau de toute architecture AWS.
          <div class="term-example">Un VPC <code>10.0.0.0/16</code> peut contenir jusqu'à 65,536 adresses IP pour vos ressources.</div>
        </div>
      </div>

      <div class="term-card aws" data-term="VPC CNI" data-search="vpc cni container network interface plugin réseau pods ip aws">
        <div class="term-name">
          <h3>VPC CNI</h3>
          <span class="term-acronym">VPC Container Network Interface</span>
          <span class="term-category aws">AWS</span>
        </div>
        <div class="term-description">
          Plugin réseau AWS pour Kubernetes qui attribue des adresses IP VPC natives aux pods. Chaque pod obtient une IP du subnet, permettant une communication directe avec les autres ressources VPC. Le CNI gère les <strong>ENIs</strong> (Elastic Network Interfaces) sur les nodes.
          <div class="term-example">Avec VPC CNI, un pod peut communiquer directement avec une base RDS sans passer par un Service.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Volume" data-search="volume stockage données mount persistant éphémère conteneur">
        <div class="term-name">
          <h3>Volume</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Répertoire accessible aux conteneurs d'un pod. Les volumes permettent de partager des données entre conteneurs et de persister des données au-delà du cycle de vie d'un conteneur. Types courants : <code>emptyDir</code> (éphémère), <code>configMap</code>, <code>secret</code>, <code>persistentVolumeClaim</code>.
          <div class="term-example">Un volume <code>emptyDir</code> permet à deux conteneurs du même pod de partager des fichiers temporaires.</div>
        </div>
      </div>
    </div>

    <!-- W -->
    <div class="letter-section" id="letter-W">
      <div class="letter-header">
        <span class="letter">W</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card k8s" data-term="Worker Node" data-search="worker node nœud travail pods exécution kubelet compute">
        <div class="term-name">
          <h3>Worker Node</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Node du cluster qui exécute les applications (pods). Les worker nodes sont gérés par le Control Plane mais exécutent les workloads utilisateur. Dans EKS, les worker nodes sont des instances EC2 dans vos subnets privés, exécutant kubelet et containerd.
          <div class="term-example">Un cluster EKS typique a 2-3 worker nodes en Managed Node Group, plus des nodes Karpenter dynamiques.</div>
        </div>
      </div>

      <div class="term-card k8s" data-term="Workload" data-search="workload charge travail application pods deployment statefulset">
        <div class="term-name">
          <h3>Workload</h3>
          <span class="term-category k8s">Kubernetes</span>
        </div>
        <div class="term-description">
          Terme générique désignant une application ou un ensemble de pods qui effectuent un travail. Les types de workloads Kubernetes incluent : <strong>Deployment</strong> (stateless), <strong>StatefulSet</strong> (stateful), <strong>DaemonSet</strong> (sur chaque node), <strong>Job</strong> (batch), <strong>CronJob</strong> (planifié).
          <div class="term-example">Déployer un workload signifie créer les ressources Kubernetes (Deployment, Service, etc.) pour faire tourner votre application.</div>
        </div>
      </div>
    </div>

    <!-- Y -->
    <div class="letter-section" id="letter-Y">
      <div class="letter-header">
        <span class="letter">Y</span>
        <div class="letter-line"></div>
      </div>

      <div class="term-card general" data-term="YAML" data-search="yaml ain't markup language format fichier configuration kubernetes manifest">
        <div class="term-name">
          <h3>YAML</h3>
          <span class="term-acronym">YAML Ain't Markup Language</span>
          <span class="term-category general">Général</span>
        </div>
        <div class="term-description">
          Format de sérialisation de données lisible par l'humain, utilisé pour les fichiers de configuration Kubernetes. YAML utilise l'indentation (espaces, pas tabulations !) pour structurer les données. C'est le format standard pour les manifests Kubernetes, Helm values, et docker-compose.
          <div class="term-example">Attention : une mauvaise indentation dans un fichier YAML provoque des erreurs de parsing difficiles à débuguer.</div>
        </div>
      </div>
    </div>

  </div>

  <div class="no-results" id="noResults" style="display: none;">
    <div class="no-results-icon">🔍</div>
    <p>Aucun terme trouvé pour "<span id="searchTerm"></span>"</p>
    <p>Essayez avec d'autres mots-clés</p>
  </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const searchInput = document.getElementById('glossarySearch');
  const clearButton = document.getElementById('clearSearch');
  const glossaryContent = document.getElementById('glossaryContent');
  const noResults = document.getElementById('noResults');
  const searchTermSpan = document.getElementById('searchTerm');
  const resultsCount = document.getElementById('resultsCount');
  const quickNav = document.getElementById('quickNav');

  const allTerms = document.querySelectorAll('.term-card');
  const allSections = document.querySelectorAll('.letter-section');

  function normalizeText(text) {
    return text.toLowerCase()
      .normalize('NFD')
      .replace(/[\u0300-\u036f]/g, '')
      .replace(/[^a-z0-9\s]/g, ' ');
  }

  function highlightText(element, searchTerms) {
    const walker = document.createTreeWalker(element, NodeFilter.SHOW_TEXT, null, false);
    const textNodes = [];

    while (walker.nextNode()) {
      if (walker.currentNode.parentNode.tagName !== 'SCRIPT' &&
          walker.currentNode.parentNode.tagName !== 'STYLE' &&
          !walker.currentNode.parentNode.classList.contains('highlight')) {
        textNodes.push(walker.currentNode);
      }
    }

    textNodes.forEach(node => {
      let text = node.textContent;
      let newHtml = text;

      searchTerms.forEach(term => {
        if (term.length > 1) {
          const regex = new RegExp(`(${term})`, 'gi');
          newHtml = newHtml.replace(regex, '<span class="highlight">$1</span>');
        }
      });

      if (newHtml !== text) {
        const span = document.createElement('span');
        span.innerHTML = newHtml;
        node.parentNode.replaceChild(span, node);
      }
    });
  }

  function removeHighlights() {
    document.querySelectorAll('.highlight').forEach(el => {
      el.outerHTML = el.innerHTML;
    });
  }

  function search(query) {
    removeHighlights();

    if (!query.trim()) {
      allTerms.forEach(term => term.style.display = '');
      allSections.forEach(section => section.style.display = '');
      noResults.style.display = 'none';
      glossaryContent.style.display = '';
      quickNav.style.display = '';
      resultsCount.innerHTML = '';
      clearButton.style.display = 'none';
      return;
    }

    clearButton.style.display = 'block';
    quickNav.style.display = 'none';

    const searchTerms = normalizeText(query).split(/\s+/).filter(t => t.length > 0);
    let matchCount = 0;

    allTerms.forEach(term => {
      const searchData = normalizeText(term.dataset.search || '');
      const termName = normalizeText(term.dataset.term || '');
      const allSearchable = searchData + ' ' + termName;

      const matches = searchTerms.every(st => allSearchable.includes(st));

      if (matches) {
        term.style.display = '';
        matchCount++;
        highlightText(term, searchTerms);
      } else {
        term.style.display = 'none';
      }
    });

    // Hide empty sections
    allSections.forEach(section => {
      const visibleTerms = section.querySelectorAll('.term-card[style=""], .term-card:not([style])');
      const hasVisible = Array.from(section.querySelectorAll('.term-card')).some(t => t.style.display !== 'none');
      section.style.display = hasVisible ? '' : 'none';
    });

    if (matchCount === 0) {
      glossaryContent.style.display = 'none';
      noResults.style.display = '';
      searchTermSpan.textContent = query;
      resultsCount.innerHTML = '';
    } else {
      glossaryContent.style.display = '';
      noResults.style.display = 'none';
      resultsCount.innerHTML = `<span>${matchCount}</span> terme${matchCount > 1 ? 's' : ''} trouvé${matchCount > 1 ? 's' : ''}`;
    }
  }

  let debounceTimer;
  searchInput.addEventListener('input', function() {
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(() => search(this.value), 150);
  });

  clearButton.addEventListener('click', function() {
    searchInput.value = '';
    search('');
    searchInput.focus();
  });

  // Keyboard shortcut: Escape to clear
  searchInput.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') {
      this.value = '';
      search('');
    }
  });
});
</script>
{{< /rawhtml >}}

---

## Navigation

- [Retour à l'installation EKS](/pro/aws/eks-installation/)
- [Retour à l'installation Karpenter](/pro/aws/karpenter-installation/)
- [Tests de Karpenter](/pro/aws/karpenter-tests/)
- [Index AWS](/pro/aws/)
