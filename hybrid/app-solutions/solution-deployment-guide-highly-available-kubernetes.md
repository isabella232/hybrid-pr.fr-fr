---
title: Déployer un cluster Kubernetes hautement disponible sur Azure Stack Hub
description: Découvrez comment déployer une solution de cluster Kubernetes pour la haute disponibilité en utilisant Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911920"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Déployer un cluster Kubernetes à haute disponibilité sur Azure Stack Hub

Cet article vous montre comment créer un environnement de cluster Kubernetes hautement disponible, déployé sur plusieurs instances Azure Stack Hub, à des emplacements physiques différents.

Dans ce guide de déploiement de solution, vous allez apprendre à :

> [!div class="checklist"]
> - Télécharger et préparer le moteur AKS
> - Se connecter à la machine virtuelle d’assistance du moteur AKS
> - Déployer un cluster Kubernetes
> - Se connecter au cluster Kubernetes
> - Connecter Azure Pipelines au cluster Kubernetes
> - Configuration de l’analyse
> - Déployer l’application
> - Mettre à l’échelle l’application automatiquement
> - Configuration de Traffic Manager
> - Mettre à niveau Kubernetes
> - Mettre à l’échelle Kubernetes

> [!Tip]  
> ![Fondements des applications hybrides](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub est une extension d’Azure. Azure Stack Hub offre à votre environnement local l’agilité et l’innovation du cloud computing grâce au seul cloud hybride qui vous permette de créer et de déployer des applications hybrides en tout lieu.  
> 
> L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) pour la conception, le déploiement et le fonctionnement des applications hybrides. Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.

## <a name="prerequisites"></a>Prérequis

Avant de commencer à utiliser ce guide de déploiement, vous devez :

- Consulter l’article [Modèle de cluster Kubernetes à haute disponibilité](pattern-highly-available-kubernetes.md).
- Passer en revue le [dépôt d’accompagnement GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), qui contient des ressources supplémentaires référencées dans cet article.
- Disposer d’un compte qui peut accéder au [portail de l’utilisateur Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), avec au moins des [autorisations « contributeur »](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>Télécharger et préparer le moteur AKS

Le moteur AKS est un fichier binaire qui peut être utilisé à partir de n’importe quel hôte Windows ou Linux pouvant atteindre les points de terminaison Azure Stack Hub Azure Resource Manager. Ce guide décrit le déploiement d’une nouvelle machine virtuelle Linux (ou Windows) sur Azure Stack Hub. Elle servira ultérieurement lors du déploiement des clusters Kubernetes par le moteur AKS.

> [!NOTE]
> Vous pouvez également utiliser une machine virtuelle Windows ou Linux existante pour déployer un cluster Kubernetes sur Azure Stack Hub à l’aide du moteur AKS.

Le processus pas à pas et les exigences pour le moteur AKS sont décrits ici :

* [Installer le moteur AKS sur Linux dans Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (ou à l’aide de [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

Le moteur AKS est un outil d’assistance qui permet de déployer et d’utiliser des clusters Kubernetes (non managés) (dans Azure et Azure Stack Hub).

Les détails et les différences du moteur AKS sur Azure Stack Hub sont décrits ici :

* [Qu’est-ce que le moteur AKS sur Azure Stack Hub ?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Moteur AKS sur Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (sur GitHub)

L’exemple d’environnement utilise Terraform pour automatiser le déploiement de la machine virtuelle du moteur AKS. Vous trouverez [les détails et le code dans le dépôt d’accompagnement GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

Le résultat de cette étape est un nouveau groupe de ressources sur Azure Stack Hub qui contient la machine virtuelle d’assistance du moteur AKS et les ressources associées :

![Ressources de machine virtuelle du moteur AKS dans Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Si vous devez déployer le moteur AKS dans un environnement hermétique déconnecté, passez en revue [Instances Azure Stack Hub déconnectées](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) pour en savoir plus.

À l’étape suivante, nous utiliserons la machine virtuelle du moteur AKS qui vient d’être déployée pour déployer un cluster Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Se connecter à la machine virtuelle d’assistance du moteur AKS

Tout d’abord, vous devez vous connecter à la machine virtuelle d’assistance du moteur AKS créée.

La machine virtuelle doit avoir une adresse IP publique et être accessible via SSH (port 22/TCP).

![Page de vue d’ensemble de la machine virtuelle du moteur AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Vous pouvez utiliser un outil de votre choix comme MobaXterm, puTTY ou PowerShell dans Windows 10 pour vous connecter à une machine virtuelle Linux à l’aide de SSH.

```console
ssh <username>@<ipaddress>
```

Une fois connecté, exécutez la commande `aks-engine`. Consultez [Versions du moteur AKS prises en charge](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) pour en découvrir plus sur les versions du moteur AKS et de Kubernetes.

![Exemple de ligne de commande aks-engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Déployer un cluster Kubernetes

La machine virtuelle d’assistance du moteur AKS elle-même n’a pas encore créé de cluster Kubernetes sur notre instance Azure Stack Hub. La création du cluster est la première action à effectuer dans la machine virtuelle d’assistance du moteur AKS.

Le processus pas à pas est documenté ici :

* [Déployer un cluster Kubernetes avec le moteur AKS sur Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Le résultat final de la commande `aks-engine deploy` et des préparatifs des étapes précédentes est un cluster Kubernetes complet déployé dans l’espace de locataire de la première instance Azure Stack Hub. Le cluster lui-même comprend des composants IaaS Azure, tels que des machines virtuelles, des équilibreurs de charge, des réseaux virtuels, des disques, etc.

![Composants IaaS du cluster dans le portail Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Équilibreur de charge Azure (point de terminaison d’API k8s)
2) Nœuds worker (pool d’agents)
3) Nœuds master

Le cluster étant opérationnel, nous nous y connecterons à l’étape suivante.

## <a name="connect-to-the-kubernetes-cluster"></a>Se connecter au cluster Kubernetes

Vous pouvez maintenant vous connecter au cluster Kubernetes créé, soit via SSH (à l’aide de la clé SSH spécifiée dans le cadre du déploiement), soit via `kubectl` (recommandé). L’outil en ligne de commande Kubernetes `kubectl` est disponible pour Windows, Linux et macOS [ici](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Il est déjà préinstallé et configuré sur les nœuds master de notre cluster.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Exécuter kubectl sur le nœud master](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Il n’est pas recommandé d’utiliser le nœud master comme serveurs de rebond (jumpbox) pour les tâches d’administration. La configuration `kubectl` est stockée dans `.kube/config` sur le ou les nœuds master ainsi que sur la machine virtuelle du moteur AKS. Vous pouvez copier la configuration sur une machine d’administration avec une connexion au cluster Kubernetes et utiliser la commande `kubectl` ici. Le fichier `.kube/config` est également utilisé ultérieurement pour configurer une connexion de service dans Azure Pipelines.

> [!IMPORTANT]
> Sécurisez ces fichiers, car ils contiennent les informations d’identification de votre cluster Kubernetes. Un attaquant ayant accès au fichier dispose d’informations suffisantes pour y accéder en tant qu’administrateur. Toutes les actions effectuées à l’aide du fichier `.kube/config` initial sont réalisées à l’aide d’un compte d’administrateur de cluster.

Vous pouvez maintenant essayer diverses commandes à l’aide de `kubectl` pour vérifier l’état de votre cluster.

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> Kubernetes a son propre modèle de _ *contrôle d’accès en fonction du rôle (RBAC)* *, qui vous permet de créer des définitions de rôles et des liaisons de rôles affinées. Il s’agit de la méthode préférable pour contrôler l’accès au cluster au lieu de distribuer les autorisations d’administrateur de cluster.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Connecter Azure Pipelines aux clusters Kubernetes

Pour connecter Azure Pipelines au cluster Kubernetes récemment déployé, nous avons besoin de son fichier de configuration Kube (`.kube/config`), comme expliqué à l’étape précédente.

* Connectez-vous à l’un des nœuds master de votre cluster Kubernetes.
* Copiez le contenu du fichier `.kube/config`.
* Accédez à Azure DevOps > Paramètres du projet > Connexions de service pour créer une connexion de service « Kubernetes » (utilisez KubeConfig comme méthode d’authentification).

> [!IMPORTANT]
> Azure Pipelines (ou ses agents de build) doit avoir accès à l’API Kubernetes. S’il existe une connexion Internet entre Azure Pipelines et le cluster Kubernetes Azure Stack Hub, vous devez déployer un agent de build Azure Pipelines auto-hébergé.

Vous pouvez déployer des agents auto-hébergés pour Azure Pipelines sur Azure Stack Hub ou sur une machine disposant d’une connectivité réseau à tous les points de terminaison de gestion requis. Consultez les informations détaillées ici :

* [Agents Azure Pipelines](/azure/devops/pipelines/agents/agents) sur [Windows](/azure/devops/pipelines/agents/v2-windows) ou [Linux](/azure/devops/pipelines/agents/v2-linux)

La section [Considérations sur le déploiement (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) du modèle contient un flux de décision qui vous aide à comprendre s’il faut utiliser des agents hébergés par Microsoft ou des agents auto-hébergés :

[![Agents auto-hébergés - Flux de décision](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

Dans cet exemple de solution, la topologie comprend un agent de build auto-hébergé sur chaque instance Azure Stack Hub. L’agent peut accéder aux points de terminaison de gestion Azure Stack Hub et aux points de terminaison d’API du cluster Kubernetes.

[![Trafic sortant uniquement](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Cette conception respecte une exigence réglementaire courante, qui consiste à avoir uniquement des connexions sortantes à partir de la solution d’application.

## <a name="configure-monitoring"></a>Configuration de l’analyse

Vous pouvez utiliser [Azure Monitor](/azure/azure-monitor/) pour conteneurs afin de superviser les conteneurs dans la solution. Azure Monitor est alors pointé sur le cluster Kubernetes déployé par le moteur AKS sur Azure Stack Hub.

Il existe deux méthodes pour activer Azure Monitor sur votre cluster. Ces deux méthodes vous obligent à configurer un espace de travail Log Analytics dans Azure.

* La [méthode 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) utilise un graphique Helm
* La [méthode 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) intervient dans le cadre de la spécification du cluster du moteur AKS

Dans l’exemple de topologie, la « méthode 1 » est utilisée, ce qui permet d’automatiser le processus et facilite l’installation des mises à jour.

Pour l’étape suivante, vous avez besoin d’un espace de travail Azure Log Analytics (ID et clé), de `Helm` (version 3) et de `kubectl` sur votre machine.

Helm est un gestionnaire de package Kubernetes, disponible sous la forme d’un fichier binaire qui s’exécute sur macOS, Windows et Linux. Vous pouvez le télécharger ici : [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm s’appuie sur le fichier de configuration Kubernetes utilisé pour la commande `kubectl`.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Cette commande installe l’agent Azure Monitor sur votre cluster Kubernetes :

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

L’agent OMS (Operations Management Suite) sur votre cluster Kubernetes envoie des données de supervision à votre espace de travail Azure Log Analytics (à l’aide du protocole HTTPS sortant). Vous pouvez désormais utiliser Azure Monitor pour obtenir des insights plus approfondis à propos de vos clusters Kubernetes sur Azure Stack Hub. Cette conception est un moyen puissant de démontrer la puissance de l’analytique qui peut être déployée automatiquement avec les clusters de votre application.

[![Clusters Azure Stack Hub dans Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Détails d’un cluster Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Si Azure Monitor n’affiche aucune donnée Azure Stack Hub, vérifiez que vous avez attentivement suivi les instructions sur [la façon d’ajouter la solution AzureMonitor-Containers à un espace de travail Azure Log Analytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).

## <a name="deploy-the-application"></a>Déployer l’application

Avant d’installer notre exemple d’application, nous devons suivre une autre étape, qui consiste à configurer le contrôleur d’entrée Nginx sur notre cluster Kubernetes. Le contrôleur d’entrée est utilisé comme équilibreur de charge de couche 7 pour router le trafic dans notre cluster en fonction de l’hôte, du chemin ou du protocole. L’entrée Nginx est disponible sous la forme d’un graphique Helm. Pour obtenir des instructions détaillées, reportez-vous au [dépôt GitHub sur les graphiques Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

Notre exemple d’application est également empaqueté en tant que graphique Helm, comme l’[agent de supervision Azure](#configure-monitoring) à l’étape précédente. Ainsi, il est facile de déployer l’application sur notre cluster Kubernetes. Vous trouverez [les fichiers du graphique Helm dans le dépôt d’accompagnement GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm).

L’exemple d’application est une application à trois niveaux, déployée sur un cluster Kubernetes sur chacune des deux instances Azure Stack Hub. L’application utilise une base de données MongoDB. Vous pouvez en découvrir plus sur la réplication des données sur plusieurs instances dans la section [Considérations relatives au stockage et aux données](pattern-highly-available-kubernetes.md#data-and-storage-considerations) du modèle.

Après avoir déployé le graphique Helm pour l’application, vous verrez les trois couches de votre application représentées en tant que déploiements et StatefulSets (pour la base de données) avec un seul pod :

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

Du côté des services, vous trouverez le contrôleur d’entrée Nginx et son adresse IP publique :

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

L’adresse IP externe est notre « point de terminaison d’application ». Il détermine la manière dont les utilisateurs se connectent pour ouvrir l’application et est également utilisé comme point de terminaison pour l’étape suivante [Configuration de Traffic Manager](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Mettre à l’échelle l’application automatiquement
Vous pouvez éventuellement configurer l’[autoscaler de pods élastique](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) (HPA, Horizontal Pod Autoscaler) pour effectuer un scale-up ou un scale-down en fonction de certaines métriques telles que l’utilisation du processeur. La commande suivante crée un autoscaler de pods élastique qui gère 1 à 10 réplicas des pods contrôlés par le déploiement ratings-web. HPA augmente et diminue le nombre de réplicas (par le biais du déploiement) pour maintenir une utilisation moyenne du processeur sur tous les pods de 80 %.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Vous pouvez vérifier l’état actuel de l’autoscaler en exécutant la commande suivante :

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Configuration de Traffic Manager

Pour répartir le trafic entre deux (ou plus) déploiements de l’application, nous allons utiliser [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview). Azure Traffic Manager est un équilibreur de charge de trafic DNS dans Azure.

> [!NOTE]
> Traffic Manager utilise le système DNS pour diriger les requêtes des clients vers le point de terminaison de service le plus approprié, en fonction de la méthode de routage du trafic et de l’intégrité des points de terminaison.

Au lieu d’utiliser Azure Traffic Manager, vous pouvez également utiliser d’autres solutions d’équilibrage de charge mondiales hébergées en local. Dans l’exemple de scénario, nous allons utiliser Azure Traffic Manager pour répartir le trafic entre deux instances de notre application. Elles peuvent s’exécuter sur des instances Azure Stack Hub à des endroits identiques ou différents :

![Traffic Manager en local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Dans Azure, nous configurons Traffic Manager pour qu’il pointe vers les deux instances différentes de notre application :

[![Profil de point de terminaison TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Comme vous pouvez le voir, les deux points de terminaison pointent vers les deux instances de l’application déployée à partir de la [section précédente](#deploy-the-application).

À ce stade :
- L’infrastructure Kubernetes a été créée, y compris un contrôleur d’entrée.
- Les clusters ont été déployés sur deux instances Azure Stack Hub.
- La supervision a été configurée.
- Azure Traffic Manager équilibre le trafic entre les deux instances Azure Stack Hub.
- Sur cette infrastructure, l’exemple d’application à trois niveaux a été déployé de manière automatisée à l’aide de graphiques Helm. 

La solution doit maintenant être accessible aux utilisateurs.

Il existe également des considérations opérationnelles de post-déploiement, qui sont abordées dans les deux sections suivantes.

## <a name="upgrade-kubernetes"></a>Mettre à niveau Kubernetes

Consultez les rubriques suivantes lors de la mise à niveau du cluster Kubernetes :

- La mise à niveau d’un cluster Kubernetes est une opération de 2 jours complexe qui peut être effectuée à l’aide du moteur AKS. Pour plus d’informations, consultez [Mettre à niveau un cluster Kubernetes sur Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Le moteur AKS vous permet de mettre à niveau des clusters vers des versions plus récentes de Kubernetes et d’images de système d’exploitation de base. Pour plus d’informations, consultez [Procédure de mise à niveau vers une version plus récente de Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- En outre, vous pouvez mettre à niveau uniquement les nœuds sous-jacents vers des versions d’images de système d’exploitation de base plus récentes. Pour plus d’informations, consultez [Procédure de mise à niveau de l’image du système d’exploitation uniquement](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Les images de système d’exploitation de base plus récentes contiennent des mises à jour de noyau et de sécurité. Il incombe à l’opérateur de cluster de superviser la disponibilité des versions Kubernetes et des images de système d’exploitation plus récentes. L’opérateur doit planifier et exécuter ces mises à niveau à l’aide du moteur AKS. Les images de système d’exploitation de base doivent être téléchargées à partir de la Place de marché Azure Stack Hub par l’opérateur Azure Stack Hub.

## <a name="scale-kubernetes"></a>Mettre à l’échelle Kubernetes

La mise à l’échelle est une autre opération de 2 jours qui peut être orchestrée à l’aide du moteur AKS.

La commande scale réutilise votre fichier de configuration de cluster (apimodel.json) dans le répertoire de sortie, en tant qu’entrée pour un nouveau déploiement Azure Resource Manager. Le moteur AKS exécute l’opération de mise à l’échelle sur un pool d’agents spécifique. Une fois l’opération de mise à l’échelle terminée, le moteur AKS met à jour la définition du cluster dans le même fichier apimodel.json. La définition du cluster reflète le nouveau nombre de nœuds et, par là même, la configuration de cluster actuelle mise à jour.

- [Mettre à l’échelle un cluster Kubernetes sur Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [considérations relatives à la conception d’applications hybrides](overview-app-design-considerations.md)
- Passez en revue et proposez des améliorations pour [le code de cet exemple sur GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).