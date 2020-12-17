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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="6eb91-103">Déployer un cluster Kubernetes à haute disponibilité sur Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="6eb91-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="6eb91-104">Cet article vous montre comment créer un environnement de cluster Kubernetes hautement disponible, déployé sur plusieurs instances Azure Stack Hub, à des emplacements physiques différents.</span><span class="sxs-lookup"><span data-stu-id="6eb91-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="6eb91-105">Dans ce guide de déploiement de solution, vous allez apprendre à :</span><span class="sxs-lookup"><span data-stu-id="6eb91-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6eb91-106">Télécharger et préparer le moteur AKS</span><span class="sxs-lookup"><span data-stu-id="6eb91-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="6eb91-107">Se connecter à la machine virtuelle d’assistance du moteur AKS</span><span class="sxs-lookup"><span data-stu-id="6eb91-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="6eb91-108">Déployer un cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="6eb91-109">Se connecter au cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="6eb91-110">Connecter Azure Pipelines au cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="6eb91-111">Configuration de l’analyse</span><span class="sxs-lookup"><span data-stu-id="6eb91-111">Configure monitoring</span></span>
> - <span data-ttu-id="6eb91-112">Déployer l’application</span><span class="sxs-lookup"><span data-stu-id="6eb91-112">Deploy application</span></span>
> - <span data-ttu-id="6eb91-113">Mettre à l’échelle l’application automatiquement</span><span class="sxs-lookup"><span data-stu-id="6eb91-113">Autoscale application</span></span>
> - <span data-ttu-id="6eb91-114">Configuration de Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="6eb91-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="6eb91-115">Mettre à niveau Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="6eb91-116">Mettre à l’échelle Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="6eb91-117">![Fondements des applications hybrides](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="6eb91-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="6eb91-118">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="6eb91-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="6eb91-119">Azure Stack Hub offre à votre environnement local l’agilité et l’innovation du cloud computing grâce au seul cloud hybride qui vous permette de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="6eb91-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="6eb91-120">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) pour la conception, le déploiement et le fonctionnement des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="6eb91-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="6eb91-121">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="6eb91-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="6eb91-122">Prérequis</span><span class="sxs-lookup"><span data-stu-id="6eb91-122">Prerequisites</span></span>

<span data-ttu-id="6eb91-123">Avant de commencer à utiliser ce guide de déploiement, vous devez :</span><span class="sxs-lookup"><span data-stu-id="6eb91-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="6eb91-124">Consulter l’article [Modèle de cluster Kubernetes à haute disponibilité](pattern-highly-available-kubernetes.md).</span><span class="sxs-lookup"><span data-stu-id="6eb91-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="6eb91-125">Passer en revue le [dépôt d’accompagnement GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), qui contient des ressources supplémentaires référencées dans cet article.</span><span class="sxs-lookup"><span data-stu-id="6eb91-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="6eb91-126">Disposer d’un compte qui peut accéder au [portail de l’utilisateur Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), avec au moins des [autorisations « contributeur »](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="6eb91-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="6eb91-127">Télécharger et préparer le moteur AKS</span><span class="sxs-lookup"><span data-stu-id="6eb91-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="6eb91-128">Le moteur AKS est un fichier binaire qui peut être utilisé à partir de n’importe quel hôte Windows ou Linux pouvant atteindre les points de terminaison Azure Stack Hub Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="6eb91-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="6eb91-129">Ce guide décrit le déploiement d’une nouvelle machine virtuelle Linux (ou Windows) sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="6eb91-130">Elle servira ultérieurement lors du déploiement des clusters Kubernetes par le moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="6eb91-131">Vous pouvez également utiliser une machine virtuelle Windows ou Linux existante pour déployer un cluster Kubernetes sur Azure Stack Hub à l’aide du moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="6eb91-132">Le processus pas à pas et les exigences pour le moteur AKS sont décrits ici :</span><span class="sxs-lookup"><span data-stu-id="6eb91-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="6eb91-133">[Installer le moteur AKS sur Linux dans Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (ou à l’aide de [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="6eb91-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="6eb91-134">Le moteur AKS est un outil d’assistance qui permet de déployer et d’utiliser des clusters Kubernetes (non managés) (dans Azure et Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="6eb91-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="6eb91-135">Les détails et les différences du moteur AKS sur Azure Stack Hub sont décrits ici :</span><span class="sxs-lookup"><span data-stu-id="6eb91-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="6eb91-136">Qu’est-ce que le moteur AKS sur Azure Stack Hub ?</span><span class="sxs-lookup"><span data-stu-id="6eb91-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="6eb91-137">[Moteur AKS sur Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (sur GitHub)</span><span class="sxs-lookup"><span data-stu-id="6eb91-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="6eb91-138">L’exemple d’environnement utilise Terraform pour automatiser le déploiement de la machine virtuelle du moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="6eb91-139">Vous trouverez [les détails et le code dans le dépôt d’accompagnement GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="6eb91-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="6eb91-140">Le résultat de cette étape est un nouveau groupe de ressources sur Azure Stack Hub qui contient la machine virtuelle d’assistance du moteur AKS et les ressources associées :</span><span class="sxs-lookup"><span data-stu-id="6eb91-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Ressources de machine virtuelle du moteur AKS dans Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="6eb91-142">Si vous devez déployer le moteur AKS dans un environnement hermétique déconnecté, passez en revue [Instances Azure Stack Hub déconnectées](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) pour en savoir plus.</span><span class="sxs-lookup"><span data-stu-id="6eb91-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="6eb91-143">À l’étape suivante, nous utiliserons la machine virtuelle du moteur AKS qui vient d’être déployée pour déployer un cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="6eb91-144">Se connecter à la machine virtuelle d’assistance du moteur AKS</span><span class="sxs-lookup"><span data-stu-id="6eb91-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="6eb91-145">Tout d’abord, vous devez vous connecter à la machine virtuelle d’assistance du moteur AKS créée.</span><span class="sxs-lookup"><span data-stu-id="6eb91-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="6eb91-146">La machine virtuelle doit avoir une adresse IP publique et être accessible via SSH (port 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="6eb91-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Page de vue d’ensemble de la machine virtuelle du moteur AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="6eb91-148">Vous pouvez utiliser un outil de votre choix comme MobaXterm, puTTY ou PowerShell dans Windows 10 pour vous connecter à une machine virtuelle Linux à l’aide de SSH.</span><span class="sxs-lookup"><span data-stu-id="6eb91-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="6eb91-149">Une fois connecté, exécutez la commande `aks-engine`.</span><span class="sxs-lookup"><span data-stu-id="6eb91-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="6eb91-150">Consultez [Versions du moteur AKS prises en charge](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) pour en découvrir plus sur les versions du moteur AKS et de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![Exemple de ligne de commande aks-engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="6eb91-152">Déployer un cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="6eb91-153">La machine virtuelle d’assistance du moteur AKS elle-même n’a pas encore créé de cluster Kubernetes sur notre instance Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="6eb91-154">La création du cluster est la première action à effectuer dans la machine virtuelle d’assistance du moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="6eb91-155">Le processus pas à pas est documenté ici :</span><span class="sxs-lookup"><span data-stu-id="6eb91-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="6eb91-156">Déployer un cluster Kubernetes avec le moteur AKS sur Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="6eb91-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="6eb91-157">Le résultat final de la commande `aks-engine deploy` et des préparatifs des étapes précédentes est un cluster Kubernetes complet déployé dans l’espace de locataire de la première instance Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="6eb91-158">Le cluster lui-même comprend des composants IaaS Azure, tels que des machines virtuelles, des équilibreurs de charge, des réseaux virtuels, des disques, etc.</span><span class="sxs-lookup"><span data-stu-id="6eb91-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Composants IaaS du cluster dans le portail Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="6eb91-160">Équilibreur de charge Azure (point de terminaison d’API k8s)</span><span class="sxs-lookup"><span data-stu-id="6eb91-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="6eb91-161">Nœuds worker (pool d’agents)</span><span class="sxs-lookup"><span data-stu-id="6eb91-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="6eb91-162">Nœuds master</span><span class="sxs-lookup"><span data-stu-id="6eb91-162">Master Nodes</span></span>

<span data-ttu-id="6eb91-163">Le cluster étant opérationnel, nous nous y connecterons à l’étape suivante.</span><span class="sxs-lookup"><span data-stu-id="6eb91-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="6eb91-164">Se connecter au cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="6eb91-165">Vous pouvez maintenant vous connecter au cluster Kubernetes créé, soit via SSH (à l’aide de la clé SSH spécifiée dans le cadre du déploiement), soit via `kubectl` (recommandé).</span><span class="sxs-lookup"><span data-stu-id="6eb91-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="6eb91-166">L’outil en ligne de commande Kubernetes `kubectl` est disponible pour Windows, Linux et macOS [ici](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="6eb91-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="6eb91-167">Il est déjà préinstallé et configuré sur les nœuds master de notre cluster.</span><span class="sxs-lookup"><span data-stu-id="6eb91-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Exécuter kubectl sur le nœud master](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="6eb91-169">Il n’est pas recommandé d’utiliser le nœud master comme serveurs de rebond (jumpbox) pour les tâches d’administration.</span><span class="sxs-lookup"><span data-stu-id="6eb91-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="6eb91-170">La configuration `kubectl` est stockée dans `.kube/config` sur le ou les nœuds master ainsi que sur la machine virtuelle du moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="6eb91-171">Vous pouvez copier la configuration sur une machine d’administration avec une connexion au cluster Kubernetes et utiliser la commande `kubectl` ici.</span><span class="sxs-lookup"><span data-stu-id="6eb91-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="6eb91-172">Le fichier `.kube/config` est également utilisé ultérieurement pour configurer une connexion de service dans Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="6eb91-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6eb91-173">Sécurisez ces fichiers, car ils contiennent les informations d’identification de votre cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="6eb91-174">Un attaquant ayant accès au fichier dispose d’informations suffisantes pour y accéder en tant qu’administrateur.</span><span class="sxs-lookup"><span data-stu-id="6eb91-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="6eb91-175">Toutes les actions effectuées à l’aide du fichier `.kube/config` initial sont réalisées à l’aide d’un compte d’administrateur de cluster.</span><span class="sxs-lookup"><span data-stu-id="6eb91-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="6eb91-176">Vous pouvez maintenant essayer diverses commandes à l’aide de `kubectl` pour vérifier l’état de votre cluster.</span><span class="sxs-lookup"><span data-stu-id="6eb91-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

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
> <span data-ttu-id="6eb91-177">Kubernetes a son propre modèle de _ *contrôle d’accès en fonction du rôle (RBAC)* \*, qui vous permet de créer des définitions de rôles et des liaisons de rôles affinées.</span><span class="sxs-lookup"><span data-stu-id="6eb91-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="6eb91-178">Il s’agit de la méthode préférable pour contrôler l’accès au cluster au lieu de distribuer les autorisations d’administrateur de cluster.</span><span class="sxs-lookup"><span data-stu-id="6eb91-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="6eb91-179">Connecter Azure Pipelines aux clusters Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="6eb91-180">Pour connecter Azure Pipelines au cluster Kubernetes récemment déployé, nous avons besoin de son fichier de configuration Kube (`.kube/config`), comme expliqué à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="6eb91-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="6eb91-181">Connectez-vous à l’un des nœuds master de votre cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="6eb91-182">Copiez le contenu du fichier `.kube/config`.</span><span class="sxs-lookup"><span data-stu-id="6eb91-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="6eb91-183">Accédez à Azure DevOps > Paramètres du projet > Connexions de service pour créer une connexion de service « Kubernetes » (utilisez KubeConfig comme méthode d’authentification).</span><span class="sxs-lookup"><span data-stu-id="6eb91-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6eb91-184">Azure Pipelines (ou ses agents de build) doit avoir accès à l’API Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="6eb91-185">S’il existe une connexion Internet entre Azure Pipelines et le cluster Kubernetes Azure Stack Hub, vous devez déployer un agent de build Azure Pipelines auto-hébergé.</span><span class="sxs-lookup"><span data-stu-id="6eb91-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="6eb91-186">Vous pouvez déployer des agents auto-hébergés pour Azure Pipelines sur Azure Stack Hub ou sur une machine disposant d’une connectivité réseau à tous les points de terminaison de gestion requis.</span><span class="sxs-lookup"><span data-stu-id="6eb91-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="6eb91-187">Consultez les informations détaillées ici :</span><span class="sxs-lookup"><span data-stu-id="6eb91-187">See the details here:</span></span>

* <span data-ttu-id="6eb91-188">[Agents Azure Pipelines](/azure/devops/pipelines/agents/agents) sur [Windows](/azure/devops/pipelines/agents/v2-windows) ou [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="6eb91-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="6eb91-189">La section [Considérations sur le déploiement (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) du modèle contient un flux de décision qui vous aide à comprendre s’il faut utiliser des agents hébergés par Microsoft ou des agents auto-hébergés :</span><span class="sxs-lookup"><span data-stu-id="6eb91-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="6eb91-190">[![Agents auto-hébergés - Flux de décision](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6eb91-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="6eb91-191">Dans cet exemple de solution, la topologie comprend un agent de build auto-hébergé sur chaque instance Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="6eb91-192">L’agent peut accéder aux points de terminaison de gestion Azure Stack Hub et aux points de terminaison d’API du cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="6eb91-193">[![Trafic sortant uniquement](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6eb91-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="6eb91-194">Cette conception respecte une exigence réglementaire courante, qui consiste à avoir uniquement des connexions sortantes à partir de la solution d’application.</span><span class="sxs-lookup"><span data-stu-id="6eb91-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="6eb91-195">Configuration de l’analyse</span><span class="sxs-lookup"><span data-stu-id="6eb91-195">Configure monitoring</span></span>

<span data-ttu-id="6eb91-196">Vous pouvez utiliser [Azure Monitor](/azure/azure-monitor/) pour conteneurs afin de superviser les conteneurs dans la solution.</span><span class="sxs-lookup"><span data-stu-id="6eb91-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="6eb91-197">Azure Monitor est alors pointé sur le cluster Kubernetes déployé par le moteur AKS sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="6eb91-198">Il existe deux méthodes pour activer Azure Monitor sur votre cluster.</span><span class="sxs-lookup"><span data-stu-id="6eb91-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="6eb91-199">Ces deux méthodes vous obligent à configurer un espace de travail Log Analytics dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6eb91-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="6eb91-200">La [méthode 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) utilise un graphique Helm</span><span class="sxs-lookup"><span data-stu-id="6eb91-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="6eb91-201">La [méthode 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) intervient dans le cadre de la spécification du cluster du moteur AKS</span><span class="sxs-lookup"><span data-stu-id="6eb91-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="6eb91-202">Dans l’exemple de topologie, la « méthode 1 » est utilisée, ce qui permet d’automatiser le processus et facilite l’installation des mises à jour.</span><span class="sxs-lookup"><span data-stu-id="6eb91-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="6eb91-203">Pour l’étape suivante, vous avez besoin d’un espace de travail Azure Log Analytics (ID et clé), de `Helm` (version 3) et de `kubectl` sur votre machine.</span><span class="sxs-lookup"><span data-stu-id="6eb91-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="6eb91-204">Helm est un gestionnaire de package Kubernetes, disponible sous la forme d’un fichier binaire qui s’exécute sur macOS, Windows et Linux.</span><span class="sxs-lookup"><span data-stu-id="6eb91-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="6eb91-205">Vous pouvez le télécharger ici : [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm s’appuie sur le fichier de configuration Kubernetes utilisé pour la commande `kubectl`.</span><span class="sxs-lookup"><span data-stu-id="6eb91-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="6eb91-206">Cette commande installe l’agent Azure Monitor sur votre cluster Kubernetes :</span><span class="sxs-lookup"><span data-stu-id="6eb91-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="6eb91-207">L’agent OMS (Operations Management Suite) sur votre cluster Kubernetes envoie des données de supervision à votre espace de travail Azure Log Analytics (à l’aide du protocole HTTPS sortant).</span><span class="sxs-lookup"><span data-stu-id="6eb91-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="6eb91-208">Vous pouvez désormais utiliser Azure Monitor pour obtenir des insights plus approfondis à propos de vos clusters Kubernetes sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="6eb91-209">Cette conception est un moyen puissant de démontrer la puissance de l’analytique qui peut être déployée automatiquement avec les clusters de votre application.</span><span class="sxs-lookup"><span data-stu-id="6eb91-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="6eb91-210">[![Clusters Azure Stack Hub dans Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6eb91-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="6eb91-211">[![Détails d’un cluster Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6eb91-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6eb91-212">Si Azure Monitor n’affiche aucune donnée Azure Stack Hub, vérifiez que vous avez attentivement suivi les instructions sur [la façon d’ajouter la solution AzureMonitor-Containers à un espace de travail Azure Log Analytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).</span><span class="sxs-lookup"><span data-stu-id="6eb91-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="6eb91-213">Déployer l’application</span><span class="sxs-lookup"><span data-stu-id="6eb91-213">Deploy the application</span></span>

<span data-ttu-id="6eb91-214">Avant d’installer notre exemple d’application, nous devons suivre une autre étape, qui consiste à configurer le contrôleur d’entrée Nginx sur notre cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="6eb91-215">Le contrôleur d’entrée est utilisé comme équilibreur de charge de couche 7 pour router le trafic dans notre cluster en fonction de l’hôte, du chemin ou du protocole.</span><span class="sxs-lookup"><span data-stu-id="6eb91-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="6eb91-216">L’entrée Nginx est disponible sous la forme d’un graphique Helm.</span><span class="sxs-lookup"><span data-stu-id="6eb91-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="6eb91-217">Pour obtenir des instructions détaillées, reportez-vous au [dépôt GitHub sur les graphiques Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="6eb91-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="6eb91-218">Notre exemple d’application est également empaqueté en tant que graphique Helm, comme l’[agent de supervision Azure](#configure-monitoring) à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="6eb91-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="6eb91-219">Ainsi, il est facile de déployer l’application sur notre cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="6eb91-220">Vous trouverez [les fichiers du graphique Helm dans le dépôt d’accompagnement GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm).</span><span class="sxs-lookup"><span data-stu-id="6eb91-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="6eb91-221">L’exemple d’application est une application à trois niveaux, déployée sur un cluster Kubernetes sur chacune des deux instances Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="6eb91-222">L’application utilise une base de données MongoDB.</span><span class="sxs-lookup"><span data-stu-id="6eb91-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="6eb91-223">Vous pouvez en découvrir plus sur la réplication des données sur plusieurs instances dans la section [Considérations relatives au stockage et aux données](pattern-highly-available-kubernetes.md#data-and-storage-considerations) du modèle.</span><span class="sxs-lookup"><span data-stu-id="6eb91-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="6eb91-224">Après avoir déployé le graphique Helm pour l’application, vous verrez les trois couches de votre application représentées en tant que déploiements et StatefulSets (pour la base de données) avec un seul pod :</span><span class="sxs-lookup"><span data-stu-id="6eb91-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

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

<span data-ttu-id="6eb91-225">Du côté des services, vous trouverez le contrôleur d’entrée Nginx et son adresse IP publique :</span><span class="sxs-lookup"><span data-stu-id="6eb91-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

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

<span data-ttu-id="6eb91-226">L’adresse IP externe est notre « point de terminaison d’application ».</span><span class="sxs-lookup"><span data-stu-id="6eb91-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="6eb91-227">Il détermine la manière dont les utilisateurs se connectent pour ouvrir l’application et est également utilisé comme point de terminaison pour l’étape suivante [Configuration de Traffic Manager](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="6eb91-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="6eb91-228">Mettre à l’échelle l’application automatiquement</span><span class="sxs-lookup"><span data-stu-id="6eb91-228">Autoscale the application</span></span>
<span data-ttu-id="6eb91-229">Vous pouvez éventuellement configurer l’[autoscaler de pods élastique](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) (HPA, Horizontal Pod Autoscaler) pour effectuer un scale-up ou un scale-down en fonction de certaines métriques telles que l’utilisation du processeur.</span><span class="sxs-lookup"><span data-stu-id="6eb91-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="6eb91-230">La commande suivante crée un autoscaler de pods élastique qui gère 1 à 10 réplicas des pods contrôlés par le déploiement ratings-web.</span><span class="sxs-lookup"><span data-stu-id="6eb91-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="6eb91-231">HPA augmente et diminue le nombre de réplicas (par le biais du déploiement) pour maintenir une utilisation moyenne du processeur sur tous les pods de 80 %.</span><span class="sxs-lookup"><span data-stu-id="6eb91-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="6eb91-232">Vous pouvez vérifier l’état actuel de l’autoscaler en exécutant la commande suivante :</span><span class="sxs-lookup"><span data-stu-id="6eb91-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="6eb91-233">Configuration de Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="6eb91-233">Configure Traffic Manager</span></span>

<span data-ttu-id="6eb91-234">Pour répartir le trafic entre deux (ou plus) déploiements de l’application, nous allons utiliser [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="6eb91-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="6eb91-235">Azure Traffic Manager est un équilibreur de charge de trafic DNS dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6eb91-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="6eb91-236">Traffic Manager utilise le système DNS pour diriger les requêtes des clients vers le point de terminaison de service le plus approprié, en fonction de la méthode de routage du trafic et de l’intégrité des points de terminaison.</span><span class="sxs-lookup"><span data-stu-id="6eb91-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="6eb91-237">Au lieu d’utiliser Azure Traffic Manager, vous pouvez également utiliser d’autres solutions d’équilibrage de charge mondiales hébergées en local.</span><span class="sxs-lookup"><span data-stu-id="6eb91-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="6eb91-238">Dans l’exemple de scénario, nous allons utiliser Azure Traffic Manager pour répartir le trafic entre deux instances de notre application.</span><span class="sxs-lookup"><span data-stu-id="6eb91-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="6eb91-239">Elles peuvent s’exécuter sur des instances Azure Stack Hub à des endroits identiques ou différents :</span><span class="sxs-lookup"><span data-stu-id="6eb91-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![Traffic Manager en local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="6eb91-241">Dans Azure, nous configurons Traffic Manager pour qu’il pointe vers les deux instances différentes de notre application :</span><span class="sxs-lookup"><span data-stu-id="6eb91-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="6eb91-242">[![Profil de point de terminaison TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="6eb91-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="6eb91-243">Comme vous pouvez le voir, les deux points de terminaison pointent vers les deux instances de l’application déployée à partir de la [section précédente](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="6eb91-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="6eb91-244">À ce stade :</span><span class="sxs-lookup"><span data-stu-id="6eb91-244">At this point:</span></span>
- <span data-ttu-id="6eb91-245">L’infrastructure Kubernetes a été créée, y compris un contrôleur d’entrée.</span><span class="sxs-lookup"><span data-stu-id="6eb91-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="6eb91-246">Les clusters ont été déployés sur deux instances Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="6eb91-247">La supervision a été configurée.</span><span class="sxs-lookup"><span data-stu-id="6eb91-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="6eb91-248">Azure Traffic Manager équilibre le trafic entre les deux instances Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="6eb91-249">Sur cette infrastructure, l’exemple d’application à trois niveaux a été déployé de manière automatisée à l’aide de graphiques Helm.</span><span class="sxs-lookup"><span data-stu-id="6eb91-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="6eb91-250">La solution doit maintenant être accessible aux utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="6eb91-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="6eb91-251">Il existe également des considérations opérationnelles de post-déploiement, qui sont abordées dans les deux sections suivantes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="6eb91-252">Mettre à niveau Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="6eb91-253">Consultez les rubriques suivantes lors de la mise à niveau du cluster Kubernetes :</span><span class="sxs-lookup"><span data-stu-id="6eb91-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="6eb91-254">La mise à niveau d’un cluster Kubernetes est une opération de 2 jours complexe qui peut être effectuée à l’aide du moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="6eb91-255">Pour plus d’informations, consultez [Mettre à niveau un cluster Kubernetes sur Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="6eb91-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="6eb91-256">Le moteur AKS vous permet de mettre à niveau des clusters vers des versions plus récentes de Kubernetes et d’images de système d’exploitation de base.</span><span class="sxs-lookup"><span data-stu-id="6eb91-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="6eb91-257">Pour plus d’informations, consultez [Procédure de mise à niveau vers une version plus récente de Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="6eb91-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="6eb91-258">En outre, vous pouvez mettre à niveau uniquement les nœuds sous-jacents vers des versions d’images de système d’exploitation de base plus récentes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="6eb91-259">Pour plus d’informations, consultez [Procédure de mise à niveau de l’image du système d’exploitation uniquement](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="6eb91-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="6eb91-260">Les images de système d’exploitation de base plus récentes contiennent des mises à jour de noyau et de sécurité.</span><span class="sxs-lookup"><span data-stu-id="6eb91-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="6eb91-261">Il incombe à l’opérateur de cluster de superviser la disponibilité des versions Kubernetes et des images de système d’exploitation plus récentes.</span><span class="sxs-lookup"><span data-stu-id="6eb91-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="6eb91-262">L’opérateur doit planifier et exécuter ces mises à niveau à l’aide du moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="6eb91-263">Les images de système d’exploitation de base doivent être téléchargées à partir de la Place de marché Azure Stack Hub par l’opérateur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="6eb91-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="6eb91-264">Mettre à l’échelle Kubernetes</span><span class="sxs-lookup"><span data-stu-id="6eb91-264">Scale Kubernetes</span></span>

<span data-ttu-id="6eb91-265">La mise à l’échelle est une autre opération de 2 jours qui peut être orchestrée à l’aide du moteur AKS.</span><span class="sxs-lookup"><span data-stu-id="6eb91-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="6eb91-266">La commande scale réutilise votre fichier de configuration de cluster (apimodel.json) dans le répertoire de sortie, en tant qu’entrée pour un nouveau déploiement Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="6eb91-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="6eb91-267">Le moteur AKS exécute l’opération de mise à l’échelle sur un pool d’agents spécifique.</span><span class="sxs-lookup"><span data-stu-id="6eb91-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="6eb91-268">Une fois l’opération de mise à l’échelle terminée, le moteur AKS met à jour la définition du cluster dans le même fichier apimodel.json.</span><span class="sxs-lookup"><span data-stu-id="6eb91-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="6eb91-269">La définition du cluster reflète le nouveau nombre de nœuds et, par là même, la configuration de cluster actuelle mise à jour.</span><span class="sxs-lookup"><span data-stu-id="6eb91-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="6eb91-270">Mettre à l’échelle un cluster Kubernetes sur Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="6eb91-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="6eb91-271">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="6eb91-271">Next steps</span></span>

- <span data-ttu-id="6eb91-272">En savoir plus sur les [considérations relatives à la conception d’applications hybrides](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="6eb91-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="6eb91-273">Passez en revue et proposez des améliorations pour [le code de cet exemple sur GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="6eb91-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>