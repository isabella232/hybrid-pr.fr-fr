---
title: Modèle Kubernetes à haute disponibilité avec Azure et Azure Stack Hub
description: Découvrez comment une solution de cluster Kubernetes fournit une haute disponibilité en utilisant Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 454cc0a0531882b7a8ec050a461420ce13eebcfe
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96912000"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Modèle de cluster Kubernetes à haute disponibilité

Cet article explique comment architecturer et utiliser une infrastructure Kubernetes à haute disponibilité en utilisant le moteur Azure Kubernetes Service (AKS) sur Azure Stack Hub. Ce scénario est courant pour les organisations ayant des charges de travail critiques dans des environnements hautement confidentiels et réglementés. Organisations dans des domaines tels que la finance, la défense et l’administration.

## <a name="context-and-problem"></a>Contexte et problème

De nombreuses organisations développent des solutions cloud natives qui tirent parti de services et de technologies de pointe, comme Kubernetes. Bien qu’Azure fournisse des centres de donnés dans la plupart des régions du monde, il y a parfois des cas d’utilisation et des scénarios particuliers où des applications critiques doivent s’exécuter à un endroit particulier. Éléments à prendre en compte :

- Sensibilité de la localisation
- Latence entre l’application et les systèmes locaux
- Conservation de la bande passante
- Connectivité
- Obligations réglementaires ou statutaires

Azure, en combinaison avec Azure Stack Hub, répond à la plupart de ces préoccupations. Vous trouverez ci-dessous un large éventail d’options, de décisions et de considérations à prendre en compte pour une implémentation réussie de Kubernetes s’exécutant sur Azure Stack Hub.

## <a name="solution"></a>Solution

Ce modèle part du principe que nous devons respecter un ensemble de contraintes strictes. L’application doit s’exécuter en local et aucunes données personnelles ne doivent atteindre les services cloud publics. Les données de supervision et les données autres que les informations d'identification personnelle peuvent être envoyées à Azure et traitées ici. Des services externes, comme un registre de conteneurs, sont accessibles mais peuvent être filtrés via un pare-feu ou un serveur proxy.

L’exemple d’application présenté ici (basé sur [Atelier Azure Kubernetes Service](/learn/modules/aks-workshop/)) est conçu pour utiliser des solutions natives Kubernetes chaque fois que c’est possible. Cette conception évite les situations de monopole du fournisseur, au lieu d’utiliser les services natifs de la plateforme. Par exemple, l’application utilise un back-end de base de données MongoDB auto-hébergé au lieu d’un service PaaS ou d’un service de base de données externe.

[![Modèle d’application - Hybride](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Le diagramme ci-dessus illustre l’architecture d’application de l’exemple d’application s’exécutant sur Kubernetes, sur Azure Stack Hub. L’application est constituée de plusieurs composants :

 1) Un cluster Kubernetes basé sur le moteur AKS sur Azure Stack Hub.
 2) [cert-manager](https://www.jetstack.io/cert-manager/), qui fournit une suite d’outils pour la gestion des certificats dans Kubernetes, utilisée pour demander automatiquement des certificats auprès de Let's Encrypt.
 3) Un espace de noms Kubernetes qui contient les composants de l’application pour le front-end (ratings-web), l’API (ratings-api) et la base de données (ratings-mongodb).
 4) Le contrôleur d’entrée qui route le trafic HTTP/HTTPS vers les points de terminaison au sein du cluster Kubernetes.

L’exemple d’application est utilisé pour illustrer l’architecture de l’application. Tous les composants sont des exemples. L’architecture contient un seul déploiement d’application. Pour obtenir une haute disponibilité, nous allons effectuer le déploiement au moins deux fois sur deux instances Azure Stack Hub différentes, qui peuvent s’exécuter au même emplacement ou dans deux sites (ou plus) différents :

![Architecture de l'infrastructure](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Les services, comme Container Registry, Azure Monitor et d’autres, sont hébergés en dehors d’Azure Stack Hub dans Azure ou localement. Cette conception hybride protège la solution contre les pannes d’une instance Azure Stack Hub.

## <a name="components"></a>Components

L’architecture globale est constituée des composants suivants :

**Azure Stack Hub** est une extension d’Azure qui peut exécuter des charges de travail dans un environnement local en fournissant des services Azure dans votre centre de données. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Stack Hub](/azure-stack/operator/azure-stack-overview).

**Moteur Azure Kubernetes Service (Moteur AKS)** est le moteur qui se trouve derrière l’offre du service Kubernetes managé, Azure Kubernetes Service (AKS), disponible aujourd’hui dans Azure. Pour Azure Stack Hub, le moteur AKS nous permet de déployer, de mettre à l’échelle et de mettre à niveau des clusters Kubernetes entièrement fonctionnels et automanagés en utilisant les fonctionnalités IaaS d’Azure Stack Hub. Pour plus d’informations, consultez [Vue d’ensemble du moteur AKS](https://github.com/Azure/aks-engine).

Pour plus d’informations sur les différences entre le moteur AKS sur Azure et le moteur AKS sur Azure Stack Hub, consultez [Problèmes connus et limitations](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations).

**Le réseau virtuel Azure (VNet)** est utilisé pour fournir l’infrastructure réseau sur chaque Azure Stack Hub pour les machines virtuelles hébergeant l’infrastructure de cluster Kubernetes.

**Azure Load Balancer** est utilisé pour le point de terminaison de l’API Kubernetes et le contrôleur d’entrée Nginx. L’équilibreur de charge route le trafic externe (par exemple Internet) vers les nœuds et les machines virtuelles offrant un service spécifique.

**Azure Container Registry (ACR)** est utilisé pour stocker les images Docker privées et les charts Helm qui sont déployés sur le cluster. Le moteur AKS peut s’authentifier auprès de Container Registry en utilisant une identité Azure AD. Kubernetes ne nécessite pas ACR. Vous pouvez utiliser d’autres registres de conteneurs, tels que Docker Hub.

**Azure Repos** est un ensemble d’outils de gestion de versions que vous pouvez utiliser pour gérer votre code. Vous pouvez également utiliser GitHub ou d’autres dépôts Git. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Repos](/azure/devops/repos/get-started/what-is-repos).

**Azure Pipelines** fait partie d'Azure DevOps Services, et effectue des builds, des tests et des déploiements automatisés. Vous pouvez également utiliser des solutions CI/CD tierces telles que Jenkins. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines).

**Azure Monitor** recueille et stocke les métriques et journaux d’activité, notamment les métriques de plateforme pour les services Azure dans les données de télémétrie d’application et de solution. Utilisez ces données pour superviser l’application, définir des alertes et des tableaux de bord et effectuer une analyse de la cause racine des échecs. Azure Monitor s’intègre à Kubernetes pour collecter des métriques auprès des contrôleurs, des nœuds et des conteneurs ainsi que les journaux d’activité des conteneurs et les journaux d’activité des nœuds master. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Monitor](/azure/azure-monitor/overview).

**Azure Traffic Manager** est un équilibreur de charge du trafic DNS qui vous permet de distribuer le trafic de façon optimale aux services entre différentes régions Azure ou entre différents déploiements Azure Stack Hub. Traffic Manager offre également une haute disponibilité et une haute réactivité. Les points de terminaison d’application doivent être accessibles à partir de l’extérieur. D’autres solutions locales sont également disponibles.

Le **contrôleur d’entrée Kubernetes** expose les routes HTTP(S) aux services dans un cluster Kubernetes. Nginx ou n’importe quel contrôleur d’entrée approprié peut être utilisé à cette fin.

**Helm** est un gestionnaire de package pour le déploiement de Kubernetes, qui fournit un moyen de regrouper différents objets Kubernetes, comme les déploiements, les services et les secrets, dans un même « chart ». Vous pouvez publier, déployer, contrôler la gestion de versions et mettre à jour un objet du chart. Azure Container Registry peut être utilisé comme référentiel pour stocker les charts Helm empaquetés.

## <a name="design-considerations"></a>Remarques relatives à la conception

Ce modèle suit quelques considérations générales expliquées plus en détails dans les sections suivantes de cet article :

- L’application utilise des solutions natives Kubernetes afin d’éviter les situations de monopole des fournisseurs.
- L’application utilise une architecture de microservices.
- Azure Stack Hub n’a pas besoin de trafic entrant, mais il autorise la connectivité Internet sortante.

Ces pratiques recommandées s’appliquent également aux charges de travail et aux scénarios réels.

## <a name="scalability-considerations"></a>Considérations relatives à l’extensibilité

La scalabilité est importante pour fournir aux utilisateurs un accès cohérent, fiable et performant à l’application.

L’exemple de scénario couvre la scalabilité sur plusieurs couches de la pile d’applications. Voici une vue d’ensemble des différentes couches :

| Niveau architecture | Éléments affectés | Comment faire ? |
| --- | --- | ---
| Application | Application | Mise à l’échelle horizontale en fonction du nombre de Pods/Réplicas/Instances de conteneur* |
| Cluster | Cluster Kubernetes | Le nombre de nœuds (entre 1 et 50), les tailles des références SKU de machine virtuelle et les pools de nœuds (le moteur AKS sur Azure Stack Hub prend actuellement en charge un seul pool de nœuds) ; utilisation de la commande de mise à l’échelle du moteur AKS (manuelle) |
| Infrastructure | Azure Stack Hub | Nombre de nœuds, capacité et unités d’échelle au sein d’un déploiement Azure Stack Hub |

\* Utilisation du HPA (Horizontal Pod Autoscaler) de Kubernetes ; mise à l’échelle automatique basée sur des métriques ou mise à l’échelle verticale en redimensionnant les instances de conteneur (processeur/mémoire).

**Azure Stack Hub (niveau infrastructure)**

L’infrastructure Azure Stack Hub est la base de cette implémentation, car Azure Stack Hub s’exécute sur du matériel physique dans un centre de données. Quand vous sélectionnez le matériel de votre hub, vous devez faire des choix pour le processeur, la densité de la mémoire, la configuration du stockage et le nombre de serveurs. Pour plus d’informations sur la scalabilité d’Azure Stack Hub, consultez les ressources suivantes :

- [Vue d’ensemble de la planification de la capacité pour Azure Stack Hub](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Ajouter des nœuds d’unité d’échelle dans Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Cluster Kubernetes (niveau cluster)**

Le cluster Kubernetes lui-même est constitué de composants IaaS Azure (Stack) incluant des ressources de calcul, de stockage et réseau. Les solutions Kubernetes impliquent des nœuds master et worker, qui sont déployés en tant que machines virtuelles dans Azure (et Azure Stack Hub).

- Les [nœuds de plan de contrôle](/azure/aks/concepts-clusters-workloads#control-plane) (master) fournissent les services Kubernetes de base et l’orchestration des charges de travail d’application.
- Les [nœuds worker](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (worker) exécutent vos charges de travail d’application.

Lors de la sélection des tailles de machines virtuelles pour le déploiement initial, plusieurs éléments sont à prendre en compte :  

- **Coût** : lors de la planification de vos nœuds worker, gardez à l’esprit le coût global par machine virtuelle que vous allez supporter. Par exemple, si vos charges de travail d’application nécessitent des ressources limitées, vous devez prévoir de déployer des machines virtuelles d’une taille inférieure. Azure Stack Hub, comme Azure, est normalement facturé en fonction de la consommation : un dimensionnement approprié des machines virtuelles pour les rôles Kubernetes est donc crucial pour optimiser les coûts de la consommation. 

- **Scalabilité** : la scalabilité du cluster est obtenue en effectuant un scale-in et un scale-out du nombre de nœuds master et worker, ou en ajoutant des pools de nœuds supplémentaires (non disponibles à ce jour sur Azure Stack Hub). La mise à l’échelle du cluster peut être effectuée en fonction des données de performances, collectées avec Container Insights (Azure Monitor + Log Analytics). 

    Si votre application a besoin de plus (ou de moins) de ressources, vous pouvez effectuer un scale-out (ou un scale-in) horizontal de vos nœuds actuels (entre 1 et 50 nœuds). Si vous avez besoin de plus de 50 nœuds, vous pouvez créer un cluster supplémentaire dans un abonnement distinct. Vous ne pouvez effectuer un scale-up vertical des machines virtuelles réelles vers une autre taille de machine virtuelle sans redéployer le cluster.

    La mise à l’échelle se fait manuellement en utilisant la machine virtuelle d’assistance du moteur AKS qui a été utilisée pour le déploiement initial du cluster Kubernetes. Pour plus d’informations, consultez [Mise à l’échelle des clusters Kubernetes](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md).

- **Quotas** : tenez compte des [quotas](/azure-stack/operator/azure-stack-quota-types) que vous avez configurés lors de la planification d’un déploiement AKS sur votre Azure Stack Hub. Vérifiez que chaque [abonnement](/azure-stack/operator/service-plan-offer-subscription-overview) dispose des plans appropriés et des quotas configurés. L’abonnement va devoir prendre en charge la quantité de calcul, de stockage et d’autres services nécessaires à vos clusters au fur et à mesure de leur scale-out.

- **Charges de travail d’application** : reportez-vous à la section [Concepts des clusters et des charges de travail](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) dans le document des concepts de base de Kubernetes pour Azure Kubernetes Service. Cet article vous aidera à déterminer la taille de machine virtuelle appropriée en fonction des besoins en calcul et en mémoire de votre application.  

**Application (niveau application)**

Sur la couche application, nous utilisons Kubernetes [HPA (Horizontal Pod Autoscaler)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). HPA peut augmenter ou diminuer le nombre de réplicas (Pod/Instances de conteneur) dans notre déploiement en fonction de différentes métriques (comme l’utilisation du processeur).

Une autre option est de mettre à l’échelle verticalement les instances de conteneur. Pour cela, vous pouvez modifier la quantité de processeurs et de mémoire demandée et disponible pour un déploiement spécifique. Pour plus d’informations, consultez [Gestion des ressources pour les conteneurs](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) sur kubernetes.io.

## <a name="networking-and-connectivity-considerations"></a>Considérations sur le réseau et la connectivité

Le réseau et la connectivité affectent également les trois couches mentionnées précédemment pour Kubernetes sur Azure Stack Hub. Le tableau suivant montre les couches et les services qu’elles contiennent :

| Couche | Éléments affectés | Quoi ? |
| --- | --- | ---
| Application | Application | Comment l’application est-elle accessible ? Sera-t-elle exposée à Internet ? |
| Cluster | Cluster Kubernetes | API Kubernetes, machine virtuelle du moteur AKS, extraction d’images de conteneur (sortie), envoi des données de supervision et télémétrie (sortie) |
| Infrastructure | Azure Stack Hub | Accessibilité des points de terminaison de gestion Azure Stack Hub, comme le portail et les points de terminaison Azure Resource Manager. |

**Application**

Pour la couche application, la considération la plus importante est de savoir si l’application est exposée et accessible à partir d’Internet. Du point de vue de Kubernetes, l’accessibilité Internet signifie l’exposition d’un déploiement ou d’un pod en utilisant un service Kubernetes ou un contrôleur d’entrée.

> [!NOTE]
> Nous vous recommandons d’utiliser des contrôleurs d’entrée pour exposer les services Kubernetes, car le nombre d’adresses IP publiques de front-end sur Azure Stack Hub est limité à 5. Cette conception limite également le nombre de services Kubernetes (avec le type LoadBalancer) à 5, ce qui est trop petit pour beaucoup de déploiements. Pour plus d’informations, consultez la [documentation du moteur AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips).

L’exposition d’une application avec une adresse IP publique via un équilibreur de charge ou un contrôleur d’entrée ne signifie pas que l’application est maintenant accessible via Internet. Azure Stack Hub peut avoir une adresse IP publique qui n’est visible que sur l’intranet local : toutes les adresses IP publiques ne sont pas réellement accessibles sur Internet.

Le bloc précédent considère le trafic entrant vers l’application. Pour la réussite d’un déploiement Kubernetes, il faut également considérer le trafic sortant. Voici quelques cas d’utilisation qui nécessitent un trafic sortant :

- Extraction d’images de conteneur stockées sur DockerHub ou Azure Container Registry
- Récupération de charts Helm
- Émission de données Application Insights (ou d’autres données de supervision)

Certains environnements d’entreprise peuvent nécessiter l’utilisation de serveurs proxy _transparents_ ou _non transparents_. Ces serveurs nécessitent une configuration spécifique sur les différents composants de notre cluster. La documentation du moteur AKS contient différentes informations détaillées sur la prise en charge des proxys réseau. Pour plus d’informations, consultez [Moteur AKS et serveurs proxy](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Enfin, un trafic entre les clusters doit circuler entre les instances Azure Stack Hub. L’exemple de déploiement consiste en clusters Kubernetes individuels exécutés sur des instances Azure Stack Hub individuelles. Le trafic entre ces instances, par exemple le trafic de réplication entre deux bases de données, est du « trafic externe ». Le trafic externe doit être routé à travers un VPN de site à site ou des adresses IP publiques Azure Stack Hub pour connecter Kubernetes sur deux instances Azure Stack Hub :

![trafic inter-cluster et intra-cluster](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

Le cluster Kubernetes n’a pas nécessairement besoin d’être accessible via Internet. La partie pertinente est l’API Kubernetes utilisée pour faire fonctionner un cluster, par exemple en utilisant `kubectl`. Le point de terminaison de l’API Kubernetes doit être accessible à tous les utilisateurs qui font fonctionner le cluster, ou qui déploient des applications et des services sur celui-ci. Ce sujet est traité plus en détails du point de vue de DevOps dans la section [Considérations sur le déploiement (CI/CD)](#deployment-cicd-considerations) ci-dessous.

Au niveau du cluster, quelques éléments sont également à prendre en compte concernant le trafic sortant :

- Mises à jour des nœuds (pour Ubuntu)
- Données de supervision (envoyées à Azure Log Analytics)
- Autres agents nécessitant un trafic sortant (spécifique à l’environnement de chaque utilisateur effectuant un déploiement)

Avant de déployer votre cluster Kubernetes en utilisant le moteur AKS, planifiez la conception de finale du réseau. Au lieu de créer un réseau virtuel dédié, il peut être plus efficace de déployer un cluster dans un réseau existant. Par exemple, vous pouvez tirer parti d’une connexion VPN de site à site existante déjà configurée dans votre environnement Azure Stack Hub.

**Infrastructure**  

L’infrastructure fait référence à l’accès aux points de terminaison de gestion d’Azure Stack Hub. Les points de terminaison incluent les portails de locataire et d’administration ainsi que les points de terminaison d’administration et de locataire d’Azure Resource Manager. Ces points de terminaison sont nécessaires pour utiliser Azure Stack Hub et ses services de base.

## <a name="data-and-storage-considerations"></a>Considérations relatives au stockage et aux données

Deux instances de notre application seront déployées sur deux clusters Kubernetes individuels, sur deux instances Azure Stack Hub. Cette conception nous oblige à réfléchir à la façon de répliquer et de synchroniser les données entre elles.

Avec Azure, nous avons la possibilité de répliquer le stockage entre plusieurs régions et zones au sein du cloud. Avec Azure Stack Hub, il n’y a actuellement pas de moyen natif de répliquer le stockage entre deux instances Azure Stack Hub différentes : elles forment deux clouds indépendants sans aucun moyen de les gérer comme un tout. La planification de la résilience des applications exécutées sur Azure Stack Hub vous oblige à prendre en compte cette indépendance dans la conception et le déploiement de votre application.

Dans la plupart des cas, la réplication du stockage n’est pas nécessaire pour une application résiliente et à haute disponibilité déployée sur AKS. Vous devez cependant envisager un stockage indépendant par instance Azure Stack Hub dans la conception de votre application. Si cette conception est un problème ou un frein pour le déploiement de la solution sur Azure Stack Hub, des solutions possibles existent chez des partenaires de Microsoft, qui fournissent des attachements de stockage. Les attachements de stockage fournissent une solution de réplication du stockage entre plusieurs instances Azure Stack Hub et Azure. Pour plus d’informations, consultez les [Solutions de partenaires](#partner-solutions).

Dans notre architecture, ces couches ont été considérées :

**Configuration**

La configuration comprend la configuration d’Azure Stack Hub, du moteur AKS et du cluster Kubernetes lui-même. La configuration doit être automatisée autant que possible et stockée comme « Infrastructure en tant que code » dans un système de gestion de versions basé sur Git, comme Azure DevOps ou GitHub. Ces paramètres ne peuvent pas être synchronisés facilement entre plusieurs déploiements. Par conséquent, nous vous recommandons de stocker et d’appliquer la configuration depuis l’extérieur et d’utiliser le pipeline DevOps.

**Application**

L’application doit être stockée dans un dépôt Git. Chaque fois qu’il y a un nouveau déploiement, des modifications apportées à l’application ou une reprise d’activité, elle peut ainsi être déployée facilement en utilisant Azure Pipelines.

**Données**

La question des données est ce qu’il y a de plus important à considérer dans la plupart des conceptions d’application. Les données d’application doivent rester synchronisées entre les différentes instances de l’application. Les données ont également besoin d’une stratégie de sauvegarde et de reprise d’activité en cas de panne.

Réussir cette conception dépend fortement des choix technologiques. Voici quelques exemples de solutions permettant d’implémenter une base de données avec une haute disponibilité sur Azure Stack Hub :

- [Déployer un groupe de disponibilité SQL Server 2016 sur Azure et Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Déployer une solution MongoDB hautement disponible sur Azure et Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Les considérations à prendre en compte lors de l’utilisation de données se trouvant à plusieurs endroits sont encore plus complexes pour une solution à haute disponibilité et à haute résilience. Considérez les aspects suivants :

- Latence et connectivité réseau entre les instances Azure Stack Hub.
- Disponibilité des identités pour les services et les autorisations. Chaque instance Azure Stack Hub s’intègre à un annuaire externe. Lors du déploiement, vous choisissez d’utiliser Azure Active Directory (Azure AD) ou les services de fédération Active Directory (AD FS). Par conséquent, il est possible d’utiliser une seule identité qui peut interagir avec plusieurs instances Azure Stack Hub indépendantes.

## <a name="business-continuity-and-disaster-recovery"></a>Continuité d’activité et reprise d’activité

La continuité d’activité et la reprise d’activité (BCDR, Business Continuity and Disaster Recovery) sont un sujet important à la fois dans Azure Stack Hub et dans Azure. La principale différence est que dans Azure Stack Hub, l’opérateur doit gérer l’ensemble du processus BCDR. Dans Azure, les composants de BCDR sont gérés automatiquement par Microsoft.

BCDR affecte les mêmes éléments mentionnés dans la section précédente [Considérations relatives au stockage et aux données](#data-and-storage-considerations) :

- Infrastructure / Configuration
- Disponibilité de l’application
- Données d'application

Comme mentionné dans la section précédente, ces éléments sont de la responsabilité de l’opérateur Azure Stack Hub et peuvent varier d’une organisation à l’autre. Planifiez le BCDR en fonction de vos outils et processus disponibles.

**Infrastructure et configuration**

Cette section traite de l’infrastructure physique et logique, et de la configuration d’Azure Stack Hub. Elle couvre les actions dans les espaces du locataire et de l’administrateur.

L’opérateur (ou l’administrateur) Azure Stack Hub est responsable de la maintenance des instances Azure Stack Hub, y compris les composants comme le réseau, le stockage, l’identité et d’autres éléments qui n’entrent pas dans le cadre de cet article. Pour plus d’informations sur les spécificités des opérations Azure Stack Hub, consultez les ressources suivantes :

- [Récupérer des données dans Azure Stack Hub avec le service Infrastructure Backup](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Activer la sauvegarde d’Azure Stack Hub à partir du portail administrateur](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Récupérer des données suite à une perte catastrophique](/azure-stack/operator/azure-stack-backup-recover-data)
- [Meilleures pratiques concernant le service de sauvegarde de l’infrastructure](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub est la plateforme et la structure fabric sur lesquelles les applications Kubernetes seront déployées. Le propriétaire de l’application Kubernetes sera un utilisateur d’Azure Stack Hub, avec un accès accordé pour déployer l’infrastructure d’application nécessaire pour la solution. Dans ce cas, l’infrastructure d’application correspond au cluster Kubernetes, déployé en utilisant le moteur AKS, et aux services environnants. Ces composants seront déployés dans Azure Stack Hub, contraints par une offre Azure Stack Hub. Vérifiez que l’offre acceptée par le propriétaire de l’application Kubernetes a une capacité suffisante (exprimée en quotas Azure Stack Hub) pour déployer l’ensemble de la solution. Comme nous l’avons recommandé dans la section précédente, le déploiement de l’application doit être automatisé en utilisant « Infrastructure en tant que code » et des pipelines de déploiement comme Azure Pipelines d’Azure DevOps.

Pour plus d’informations sur les offres et les quotas Azure Stack Hub, consultez [Vue d’ensemble des services, plans, offres et abonnements Azure Stack Hub](/azure-stack/operator/service-plan-offer-subscription-overview).

Il est important d’enregistrer et de stocker de façon sécurisée la configuration du moteur AKS, y compris ses sorties. Ces fichiers contiennent des informations confidentielles utilisées pour accéder au cluster Kubernetes : ils doivent donc être protégés d’une exposition à des non-administrateurs.

**Disponibilité de l’application**

L’application ne doit pas s’appuyer sur les sauvegardes d’une instance déployée. Au titre de pratique standard, redéployez complètement l’application en suivant les modèles « Infrastructure en tant que Code ». Par exemple, redéployez en utilisant Azure Pipelines d’Azure DevOps. La procédure BCDR doit impliquer le redéploiement de l’application sur le même ou sur un autre cluster Kubernetes.

**Données d’application**

Les données d’application sont la partie critique de la réduction des pertes de données. Dans la section précédente, les techniques de réplication et de synchronisation des données entre deux ou plusieurs instances de l’application ont été décrites. En fonction de l’infrastructure de base de données (MySQL, MongoDB, MSSQL ou autres) utilisée pour stocker les données, vous pouvez choisir parmi différentes techniques de sauvegarde et de disponibilité de la base de données.

Pour garantir l’intégrité, il est recommandé d’utiliser un des moyens suivants :
- Une solution de sauvegarde native pour la base de données spécifique.
- Une solution de sauvegarde qui prend officiellement en charge la sauvegarde et la récupération du type de base de données utilisé par votre application.

> [!IMPORTANT]
> Ne stockez pas vos données de sauvegarde sur la l’instance Azure Stack Hub où se trouvent vos données d’application. Une panne totale de l’instance Azure Stack Hub compromettrait également vos sauvegardes.

## <a name="availability-considerations"></a>Considérations relatives à la disponibilité

Kubernetes sur Azure Stack Hub déployé via le moteur AKS n’est pas un service managé. C’est un déploiement et une configuration automatisés d’un cluster Kubernetes utilisant Azure IaaS (Infrastructure-as-a-Service). À ce titre, il offre la même disponibilité que l’infrastructure sous-jacente.

L’infrastructure Azure Stack Hub est déjà résiliente aux défaillances et fournit des fonctionnalités comme des groupes à haute disponibilité pour distribuer des composants sur plusieurs [domaines d’erreur et de mise à jour](/azure-stack/user/azure-stack-vm-considerations#high-availability). La technologie sous-jacente (clustering de basculement) implique néanmoins toujours un temps d’arrêt pour les machines virtuelles qui se trouvent sur un serveur physique impacté en cas de défaillance matérielle.

C’est une bonne pratique que de déployer votre cluster Kubernetes de production ainsi que la charge de travail sur deux clusters (ou plus). Ces clusters doivent être hébergés dans des endroits ou des centres de données différents, et utiliser des technologies comme Azure Traffic Manager pour router les utilisateurs en fonction du temps de réponse du cluster ou de la zone géographique.

![Utilisation de Traffic Manager pour contrôler les flux de trafic](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Les clients qui ont un seul cluster Kubernetes se connectent généralement à l’adresse IP du service ou au nom DNS d’une application donnée. Dans un déploiement impliquant plusieurs clusters, les clients doivent se connecter à un nom DNS de Traffic Manager qui pointe vers les services/entrées sur chaque cluster Kubernetes.

![Utilisation de Traffic Manager pour router vers un cluster local](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Ce modèle est également une [bonne pratique pour les clusters AKS (managés) dans Azure](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

Le cluster Kubernetes lui-même, déployé via le moteur AKS, doit comprendre au moins trois nœuds master et deux nœuds worker.

## <a name="identity-and-security-considerations"></a>Considérations relatives à l’identité et à la sécurité

L’identité et la sécurité sont des sujets importants, en particulier quand la solution comprend des instances Azure Stack Hub indépendantes. Kubernetes et Azure (y compris Azure Stack Hub) ont tous deux des mécanismes distincts pour le contrôle d’accès en fonction du rôle (RBAC) :

- RBAC Azure contrôle l’accès aux ressources dans Azure (et dans Azure Stack Hub), y compris la possibilité de créer des ressources Azure. Ces autorisations peuvent être assignées aux utilisateurs, groupes ou principaux de service. (Un principal de service est une identité de sécurité utilisée par les applications.)
- Kubernetes RBAC contrôle les autorisations sur l’API Kubernetes. Par exemple, la création de pods et le listage de pods sont des actions qui peuvent être accordées (ou refusées) à un utilisateur au moyen de RBAC. Pour affecter des autorisations Kubernetes à des utilisateurs, vous créez des rôles et des liaisons de rôle.

**Identité Azure Stack Hub et RBAC**

Azure Stack Hub offre deux choix de fournisseur d’identité. Le fournisseur que vous utilisez dépend de l’environnement et de l’exécution dans un environnement connecté ou déconnecté :

- Azure AD : ne peut être utilisé que dans un environnement connecté.
- ADFS vers une forêt Active Directory traditionnelle : peut être utilisé dans un environnement connecté ou déconnecté.

Le fournisseur d’identité gère les utilisateurs et les groupes, y compris l’authentification et l’autorisation pour accéder aux ressources. L’accès peut être accordé à des ressources Azure Stack Hub, comme des abonnements, des groupes de ressources et des ressources individuelles, par exemple des machines virtuelles ou des équilibreurs de charge. Pour avoir un modèle d’accès cohérent, vous devez envisager d’utiliser les mêmes groupes (directs ou imbriqués) pour toutes les instances Azure Stack Hub. Voici un exemple de configuration :

![Groupes AAD imbriqués avec Azure Stack Hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

L’exemple contient un groupe dédié (utilisant AAD ou ADFS) à des fins spécifiques. Par exemple, pour fournir des autorisations de contributeur pour le groupe de ressources qui contient notre infrastructure de clusters Kubernetes sur une instance Azure Stack Hub spécifique (ici, « Seattle K8s Cluster Contributor »). Ces groupes sont ensuite imbriqués dans un groupe global qui contient les « sous-groupes » pour chaque Azure Stack Hub.

Notre exemple d’utilisateur dispose désormais des autorisations « Contributeur » pour les deux groupes de ressources qui contiennent l’ensemble des ressources de l’infrastructure Kubernetes. L’utilisateur aura accès aux ressources sur les deux instances Azure Stack Hub, car les instances partagent le même fournisseur d’identité.

> [!IMPORTANT]
> Ces autorisations affectent seulement Azure Stack Hub et certaines des ressources qui y sont déployées. Un utilisateur disposant de ce niveau d’accès peut faire beaucoup de dégâts, mais il ne peut pas accéder aux machines virtuelles IaaS Kubernetes ni à l’API Kubernetes sans accès supplémentaire au déploiement Kubernetes.

**Identité Kubernetes et RBAC**

Par défaut, un cluster Kubernetes n’utilise pas le même fournisseur d’identité que l’instance Azure Stack Hub sous-jacente. Les machines virtuelles hébergeant le cluster Kubernetes et les nœuds master et worker utilisent la clé SSH spécifiée lors du déploiement du cluster. Cette clé SSH est nécessaire pour se connecter à ces nœuds en utilisant SSH.

L’API Kubernetes (par exemple accessible via `kubectl`) est également protégée par les comptes de service, y compris un compte de service « administrateur de cluster » par défaut. Les informations d’identification pour ce compte de service sont initialement stockées dans le fichier `.kube/config` sur vos nœuds master Kubernetes.

**Gestion des secrets et informations d’identification d’application**

Pour stocker des secrets, comme des chaînes de connexion ou des informations d’identification de base de données, plusieurs choix sont possibles, notamment :

- Azure Key Vault
- Secrets Kubernetes
- Solutions de tiers, comme HashiCorp Vault (qui s’exécute sur Kubernetes)

Ne stockez pas les secrets ou les informations d’identification en texte clair dans vos fichiers de configuration, dans votre code d’application ou dans des scripts. Ne les stockez pas non plus dans un système de gestion de versions. Au lieu de cela, l’automatisation du déploiement doit récupérer les secrets en fonction des besoins.

## <a name="patch-and-update"></a>Correctifs et mises à jour

Le processus **Corriger et mettre à jour (PNU, Patch and Update)** dans Azure Kubernetes Service est partiellement automatisé. Les mises à niveau des versions de Kubernetes sont déclenchées manuellement, tandis que les mises à jour de sécurité sont appliquées automatiquement. Ces mises à jour incluent des correctifs de sécurité ou des mises à jour du noyau du système d’exploitation. AKS ne redémarre pas automatiquement ces nœuds Linux pour terminer le processus de mise à jour. 

Le processus PNU pour un cluster Kubernetes déployé en utilisant le moteur AKS sur Azure Stack Hub n’est pas géré et est de la responsabilité de l’opérateur du cluster. 

Le moteur AKS facilite les deux tâches les plus importantes :

- [Mettre à niveau vers une version plus récente de Kubernetes et de l’image du système d’exploitation de base](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Mettre à niveau l’image du système d’exploitation de base uniquement](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Les images de système d’exploitation de base plus récentes contiennent les derniers correctifs de sécurité et les dernières mises à jour du noyau du système d’exploitation. 

Le mécanisme de [mise à niveau sans assistance](https://wiki.debian.org/UnattendedUpgrades) installe automatiquement les mises à jour de sécurité publiées avant qu’une nouvelle version de l’image du système d’exploitation de base ne soit disponible dans la place de marché Azure Stack Hub. La mise à niveau sans assistance est activée par défaut et installe automatiquement les mises à jour de sécurité, mais elle ne redémarre pas les nœuds du cluster Kubernetes. Le redémarrage des nœuds peut être automatisé en utilisant le [**K** Ubernetes **RE** boot **D** aemon (kured))](/azure/aks/node-updates-kured) open source. Kured surveille les nœuds Linux qui nécessitent un redémarrage, puis gère automatiquement la replanification des pods en cours d’exécution et le processus de redémarrage des nœuds.

## <a name="deployment-cicd-considerations"></a>Points à prendre en considération pour le déploiement (CI/CD)

Azure et Azure Stack Hub exposent les mêmes API REST Azure Resource Manager. Vous faites appel à ces API comme pour tout autre cloud (Azure, Azure China 21Vianet, Azure Government). Il peut y avoir des différences dans les versions des API entre les clouds, et Azure Stack Hub fournit seulement un sous-ensemble de services. L’URI du point de terminaison de gestion est également différent pour chaque cloud et pour chaque instance Azure Stack Hub.

Hormis les subtiles différences mentionnées, les API REST Azure Resource Manager offrent un moyen cohérent pour interagir avec Azure et Azure Stack Hub. Le même ensemble d’outils peut être utilisé ici comme il serait utilisé avec n’importe quel autre cloud Azure. Vous pouvez utiliser Azure DevOps, des outils comme Jenkins ou PowerShell pour déployer et orchestrer des services sur Azure Stack Hub.

**Considérations**

Une des principales différences concernant les déploiements d’Azure Stack Hub est la question de l’accessibilité à Internet. L’accessibilité à Internet détermine s’il faut sélectionner un agent de build hébergé par Microsoft ou auto-hébergé pour vos travaux CI/CD.

Un agent auto-hébergé peut s’exécuter sur Azure Stack Hub (en tant que machine virtuelle IaaS) ou dans le sous-réseau d’un réseau qui peut accéder à Azure Stack Hub. Pour plus d’informations sur les différences, consultez [Agents Azure Pipelines](/azure/devops/pipelines/agents/agents).

L’image suivante vous aide à déterminer si vous avez besoin d’un agent de build auto-hébergé ou hébergé par Microsoft :

![Agents de build auto-hébergés : Oui ou Non](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Les points de terminaison de gestion d’Azure Stack Hub sont-ils accessibles via Internet ?
  - Oui : Nous pouvons utiliser Azure Pipelines avec des agents hébergés par Microsoft pour se connecter à Azure Stack Hub.
  - Non : Nous avons besoin d’agents auto-hébergés qui peuvent se connecter aux points de terminaison de gestion d’Azure Stack Hub.
- Notre cluster Kubernetes est-il accessible via Internet ?
  - Oui : Nous pouvons utiliser Azure Pipelines avec des agents hébergés par Microsoft pour interagir directement avec le point de terminaison de l’API Kubernetes.
  - Non : Nous avons besoin d’agents auto-hébergés qui peuvent se connecter au point de terminaison de l’API du cluster Kubernetes.

Dans les scénarios où les points de terminaison de gestion d’Azure Stack Hub et l’API Kubernetes sont accessibles via Internet, le déploiement peut utiliser un agent hébergé par Microsoft. Ce déploiement va se traduire par une architecture d’application comme celle-ci :

[![Vue d’ensemble de l’architecture publique](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Si les points de terminaison Azure Resource Manager, l’API Kubernetes, ou les deux, ne sont pas directement accessibles via Internet, nous pouvons tirer parti d’un agent de build auto-hébergé pour exécuter les étapes du pipeline. Cette conception nécessite moins de connectivité et peut être déployée avec seulement une connectivité réseau locale aux points de terminaison Azure Resource Manager et à l’API Kubernetes :

[![Vue d'ensemble de l'architecture locale](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Qu’en est-il des scénarios déconnectés ?** Dans les scénarios où Azure Stack Hub ou Kubernetes, ou les deux, n’ont pas de points de terminaison de gestion accessibles sur Internet, il est toujours possible d’utiliser Azure DevOps pour vos déploiements. Vous pouvez utiliser un pool d’agents auto-hébergés (qui est un agent DevOps s’exécutant localement ou sur Azure Stack Hub lui-même) ou un serveur Azure DevOps Server local entièrement auto-hébergé. L’agent auto-hébergé a seulement besoin d’une connectivité Internet HTTPS (TCP/443) sortante.

Le modèle peut utiliser un cluster Kubernetes (déployé et orchestrée avec le moteur AKS) sur chaque instance Azure Stack Hub. Il inclut une application composée d’un front-end, d’un niveau intermédiaire, de services back-ends (par exemple MongoDB) et d’un contrôleur d’entrée basé sur Nginx. Au lieu d’utiliser une base de données hébergée sur le cluster Kubernetes, vous pouvez tirer parti de « magasins de données externes ». Les options de base de données incluent MySQL, SQL Server ou tout type de base de données hébergé en dehors d’Azure Stack Hub ou dans IaaS. Les configurations comme celle-ci ne sont pas abordées ici.

## <a name="partner-solutions"></a>Solutions de partenaires

Il existe des solutions de partenaires Microsoft qui peuvent étendre les fonctionnalités d’Azure Stack Hub. Ces solutions se sont avérées utiles pour les déploiements d’applications s’exécutant sur des clusters Kubernetes.  

## <a name="storage-and-data-solutions"></a>Solutions de stockage et de données

Comme décrit dans [Considérations relatives au stockage et aux données](#data-and-storage-considerations), Azure Stack Hub n’a actuellement pas de solution native pour répliquer le stockage sur plusieurs instances. Contrairement à Azure, la fonctionnalité de réplication du stockage sur plusieurs régions n’existe pas. Dans Azure Stack Hub, chaque instance est son propre cloud distinct. Cependant, des solutions sont disponibles auprès de partenaires Microsoft qui permettent la réplication du stockage sur les instances Azure Stack Hub et Azure. 

**SCALITY**

[Scality](https://www.scality.com/) offre un stockage à l’échelle du web qui est utilisé par des entreprises du numérique depuis 2009. Scality RING, notre stockage défini par logiciel, transforme des serveurs x86 en un pool de stockage illimité pour n’importe quel type de données (fichier et objet) à l’échelle du pétaoctets.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) simplifie le stockage d’entreprise avec un stockage évolutif sans limite qui regroupe des ensembles de données massifs en un environnement unique et facile à gérer.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les sujets abordés dans cet article :

- [Mise à l’échelle entre clouds](pattern-cross-cloud-scale.md) et [Modèles d’applications géodistribuées](pattern-geo-distributed.md) dans Azure Stack Hub.
- [Architecture des microservices sur AKS (Azure Kubernetes Service)](/azure/architecture/reference-architectures/microservices/aks).

Quand vous êtes prêt à tester l’exemple de solution, poursuivez avec le [Guide de déploiement du cluster Kubernetes à haute disponibilité](solution-deployment-guide-highly-available-kubernetes.md). Ce guide de déploiement fournit des instructions pas à pas sur le déploiement et sur le test de ses composants.