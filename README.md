# TechNext4CNAM - Codelab GitOps

:rocket: Bievenue dans ce codelab GitOps ! :rocket:

Aujourd'hui, nous allons explorer la manière de mettre en place le GitOps avec ArgoCD !

L'approche GitOps repose sur l'utilisation de référentiels Git comme unique source de vérité pour distribuer l'infrastructure en tant que code. Ainsi, l'approche GitOps apporte les avantages suivants : 

* <b>Audit et Traçabilité :</b> Chaque changement est versionné et traçable grâce à l'historique Git, ce qui facilite l'audit des modifications et l'identification de la source des problèmes.

* <b>Automatisation et Cohérence :</b> En automatisant les déploiements à partir des dépôts Git, GitOps réduit les erreurs humaines et assure une cohérence entre votre environnement de développement, de test et de production.

* <b>Temps de Récupération Réduit :</b> En cas de problème, vous pouvez rapidement restaurer un état précédent de l'application ou de l'infrastructure simplement en revenant à une version antérieure du dépôt Git.

* <b>Collaboration Facilitée :</b> Les développeurs et opérateurs peuvent collaborer plus efficacement via des pull requests et des revues de code, facilitant l'adoption de meilleures pratiques et la gestion des modifications.

* <b>Rapidité de Déploiement :</b> Les changements peuvent être déployés rapidement et fréquemment grâce à un processus automatisé, favorisant l'amélioration continue et l'innovation.

* <b>Alignement avec les Principes DevOps :</b> GitOps s’aligne parfaitement avec les principes DevOps en intégrant le contrôle de version, l’automatisation et la collaboration dans le processus de gestion de l’infrastructure.

## Objectifs du codelab

Passons aux choses sérieuses ! Dans ce codelab, vous allez :
* Créez votre première applications sous ArgoCD, et découvrir toutes les subtilités de configurations applicables sur une application ArgoCD
* Construire un déploiement pour déployer une application backend
* Construire un déploiement pour déployer une application frontend
* Jouer avec les capacités de self-healing d'ArgoCD
* Explorer comment le GitOps facilite les opérations de décomissionnement
* Explorer comment le GitOps peut nous aider en cas de mise en production qui se passe mal

Allez, c'est parti ! 🙂

## Présentation de l'environnement

Nous vous avons mis un disposition un cluster Kubernetes déployé sur le cloud public Azure ([Azure Kubernetes Services](https://azure.microsoft.com/fr-fr/products/kubernetes-service)).

Un <b>identifiant</b> vous est attribué à tous. Ce dernier est systématiquement composé de la manière suivante : <b>[premiere_lettre_de_votre_prenom][votre_nom]</b>. Voici deux exemples : 
* Rémi KAEFFER -> rkaeffer
* Jason BOURLARD -> jbourlard

Par défaut, ce cluster vous propose les services suivants : 
* Une instance commune d'ArgoCD, qui va vous permettre de déployer vos applications, et disponible à l'URL suivante : https://argo-cd.codelab.cloud-sp.eu/
    * Username : admin
    * Password : A REMPLIR LE JOUR DU LAB - ET A EFFACER APRES
* Un environnement de développement pour chacun d'entre vous, composés de deux namespaces : 
    * Un namespace, nommé kcl-[identifiant]-wk, portant un OpenVScode et tout plein d'outils (kubectl, git, ...), et accessible à l'url [[identifiant]-kcl.codelab.cloud-sp.eu](tochange-kcl.codelab.sp.eu)
    * Un namespace, nommé kcl-[identifiant], où devra être déployé vos applications

## Pré-requis avant de démarrer

Pour pouvoir effectuer ce codelab, quatres pré-requis sont nécessaires (Pas de panique, rien à installer 😁) : 
* Etre connecté à notre réseau Wifi SSG-Guest (Normalement, on vous a autorisé ce matin, si ce n'est pas le cas, faites le nous savoir) !
* Disposer d'un compte Github, et avoir généré un [personal token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic) d'accès pour pouvoir push sur vos repositories
* Vérifier que vous avez bien accès à [ArgoCD](https://argo-cd.codelab.cloud-sp.eu/)
* Vérifier que vous avez bien accès à votre environnement de travail : [[identifiant]-kcl.codelab.cloud-sp.eu](tochange-kcl.codelab.sp.eu)

Cette fois-ci, vous êtes prêt !

## Instructions du codelab

### Etape 0 - Préparation et exploration

Pour démarrer ce codelab, vous devez commencer par <b>forker dans votre espace personnel github</b> le repository https://github.com/rkaeffer/codelab-gitops-argocd. 

![Fork github](docs/Fork.PNG "Fork github")

Ce repository contient les sources qui seront à compléter durant ce lab.

Il est composé comme suit : 

```
codelab-gitops-argocd
│   README.md ==> Les instructions du lab
│   argocd-application.yaml ==> Descripteur application argocd 
│
└───helm-chart ==> Contient un chart Helm que nous allons déployer 
│   │   Chart.yaml ==> Descripteur du chart Helm
│   │   values.yaml ==> Fichier de values du chart Helm
│   │
│   └───templates ==> Contient les manifests K8S que l'on va vouloir déployer
│       │   ...
│   
└───docs ==> Contient les images du README (A ignorer)
    │   ...
```

Une fois que vous avez forké votre le repository dans votre espace personnel github, vous devez le cloner dans votre environnement de travail.

Pour cela, ouvrez un terminal dans votre espace de travail OpenVScode ([[identifiant]-kcl.codelab.cloud-sp.eu](tochange-kcl.codelab.sp.eu)) :
*  Terminal -> New Terminal (Ou Ctrl + Shift + ù)

Par défaut, le terminal est positionné dans un dossier "workspace" dans lequel vous allez travailler. C'est également ce dossier qui est positionné dans l'explotateur de fichier d'OpenVSCode.

```bash
git clone https://github.com/[username]/codelab-gitops-argocd
```

Votre environnement est configuré pour pouvoir manipuler le cluster Kubernetes sans configurer spécifique.
Lancer la commande suivante et observer le résultat.

```bash
kubectl get ns
```

Vous pouvez observer que tout un ensemble de namespaces sont présents :
* Les namespaces portant vos environnements de développement
* Un namespace argo-cd, portant le déploiement d'ArgoCD
* D'autres namespaces "techniques" nécessaires au bon fonctionnement de ce codelab

Allons faire un tour sur [ArgoCD](https://argo-cd.codelab.cloud-sp.eu/) :

![Présentation ArgoCD](docs/argo_initial.PNG "Présentation d'ArgoCD")

N'hésitez pas à faire un tour des différents onglets pour explorer ce qu'ils contiennent !

### Etape 1 - Créer une application dans ArgoCD

Comme expliqué en introduction, l'approche GitOps repose sur l'<b>utilisation de référentiels Git comme unique source de vérité</b> pour distribuer l'infrastructure en tant que code. ArgoCD nous permet de mettre en oeuvre ce principe en déployant sur un ou plusieurs clusters des descripteurs de déploiement stocké dans Git.

ArgoCD va ainsi nous permettre de définir des <b>applications</b>, décrites par un <b>ensemble de paramètre, notamment un lien vers un repository Git</b> qui contient les descriteurs que nous voulons déployer.

Vous allez devoir créer votre première aplication dans ArgoCD en complétant la partie haute du fichier argocd-application !

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: #TODO - Nom de votre application - Libre :)
  namespace: argo-cd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  labels:
    name: #TODO - Nom de votre application - Libre :)
spec:
  project: default
  # Source of the application manifests
  source:
    repoURL: #TODO - Lien vers votre repository GitHub
    path: #TODO - Chemin dans votre repository git 
    targetRevision: #TODO - Branche à cibler de votre repository Git 
    helm:
      releaseName: #TODO - Nom à donner à votre release helm - Libre :)
  # Destination cluster and namespace to deploy the application
  destination:
    name: in-cluster
    namespace: #TODO - Namespace de déploiement de votre application
```

A vous de jouer 😉

Observez ensuite la partie basse du fichier : 

```yaml
 # Sync policy
  syncPolicy:
    automated: # automated sync by default retries failed attempts 5 times with following delays between attempts ( 5s, 10s, 20s, 40s, 80s ); retry controlled using `retry` field.
      prune: true # Specifies if resources should be pruned during auto-syncing ( false by default ).
      selfHeal: false # Specifies if partial app sync should be executed when resources are changed only in target Kubernetes cluster and no git change detected ( false by default ).
    syncOptions:     # Sync options which modifies sync behavior
    - Validate=true # disables resource validation (equivalent to 'kubectl apply --validate=false') ( true by default ).
    - CreateNamespace=true # Namespace Auto-Creation ensures that namespace specified as the application destination exists in the destination cluster.
    - PrunePropagationPolicy=foreground # Supported policies are background, foreground and orphan.
    - PruneLast=false # Allow the ability for resource pruning to happen as a final, implicit wave of a sync operation
    managedNamespaceMetadata: # Sets the metadata for the application namespace. Only valid if CreateNamespace=true (see above), otherwise it's a no-op.
      labels: # The labels to set on the application namespace
      annotations: # The annotations to set on the application namespace

    retry:
      limit: 5 # number of failed sync attempt retries; unlimited number of attempts if less than 0
      backoff:
        duration: 5s # the amount to back off. Default unit is seconds, but could also be a duration (e.g. "2m", "1h")
        factor: 2 # a factor to multiply the base duration after each failed retry
        maxDuration: 3m # the maximum amount of time allowed for the backoff strategy

  # RevisionHistoryLimit limits the number of items kept in the application's revision history, which is used for
  # informational purposes as well as for rollbacks to previous versions. This should only be changed in exceptional
  # circumstances. Setting to zero will store no history. This will reduce storage used. Increasing will increase the
  # space used to store the history, so we do not recommend increasing it.
  revisionHistoryLimit: 50
```

Ici, nous avons configurer tout ce qui est nécessaire pour le déploiement de notre application. Chaque paramètre est détaillé en commentaire.

Il est temps de tester (Sans douter, bien entendu) ! Déployer votre application dans le namespace argo-cd :
```bash
cd codelab-gitops-argocd
kubectl apply -f argocd-application.yaml -n argo-cd
```

Par défaut, argo-cd ne reconnait que les CRD (Custom Ressource Definition) argoproj.io/v1alpha1/application positionné dans son namespace de déploiement : argo-cd. Il est possible de le faire reconnaitre ces CRD dans d'autres namespaces en le configurant spécifiquement, mais ce n'est pas le sujet du jour !

Si tout s'est bien passé, vous devriez obtenir sur ArgoCD le résultat suivant : 



NB : Pour aller plus loin, et pour passer à un cran au dessus dans l'approche GitOps, une utilisation courante dans l'industrie est de déployer un ArgoCD "applicatif" avec les configurations des applications qu'il doit déployer à l'aide d'un ArgoCD "infrastructure", pour que les équipes implémentant les applicatifs adopte une approche full GitOps (Pas de commande d'apply à faire sur le cluster Kubernetes). En somme : "Un ArgoCD pour les gouverner tous, un ArgoCD pour les déployer, un ArgoCD pour les superviser et dans le cloud les lier !"

### Etape 2 - Deploiement du backend

Avoir une application ArgoCD correctement configuré c'est bien, que cette application ArgoCD déploie un applicatif, c'est mieux ! 

Nous allons commencer par déployer un simple microservice Java. Nous avons préparé en avance une image de conteneur prête à l'emploi.

Voici un petit schéma qui rapelle les composants à mettre en oeuvre sur Kubernetes pour déployer et exposer une application : 



### Etape 3 - Deploiement du frontend

### Etape 4 - Jouons avec ArgoCD

### Etape 5 - Décomissionement du frontend

### Etape 6 - Rollback du décomissionement

### Etape 7 - Pour aller plus loin

## Tips pour le lab