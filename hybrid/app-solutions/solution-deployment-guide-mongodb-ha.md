---
title: Déployer une solution MongoDB hautement disponible sur Azure et Azure Stack Hub
description: Découvrez comment déployer une solution MongoDB hautement disponible sur Azure et Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f6064aaa1087a3c0cfc26e09371e81752c777edb
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477267"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="42c77-103">Déployer une solution MongoDB hautement disponible sur Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="42c77-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="42c77-104">Cet article vous guide dans le processus de déploiement automatisé d’un cluster MongoDB hautement disponible de base avec un site de récupération d’urgence dans deux environnements Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42c77-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="42c77-105">Pour en savoir plus sur MongoDB et la haute disponibilité, voir [Membres d’un jeu de réplicas](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="42c77-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="42c77-106">Dans cette solution, vous allez créer un exemple d’environnement pour :</span><span class="sxs-lookup"><span data-stu-id="42c77-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="42c77-107">Orchestrer un déploiement dans deux environnements Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42c77-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="42c77-108">Utiliser Docker pour minimiser les problèmes de dépendance avec des profils d’API Azure.</span><span class="sxs-lookup"><span data-stu-id="42c77-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="42c77-109">Déployer un cluster MongoDB hautement disponible de base avec un site de reprise d’activité.</span><span class="sxs-lookup"><span data-stu-id="42c77-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="42c77-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="42c77-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="42c77-111">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="42c77-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="42c77-112">Azure Stack Hub apporte l’agilité et l’innovation du cloud computing à votre environnement local et active le seul cloud hybride qui vous permet de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="42c77-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="42c77-113">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) en matière de conception, de déploiement et d’exploitation des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="42c77-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="42c77-114">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="42c77-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="42c77-115">Architecture pour MongoDB avec Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="42c77-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Architecture MongoDB hautement disponible dans Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="42c77-117">Conditions préalables pour MongoDB avec Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="42c77-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="42c77-118">Deux systèmes intégrés Azure Stack Hub connectés (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="42c77-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="42c77-119">Ce déploiement ne fonctionne pas sur le kit de développement Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="42c77-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="42c77-120">Pour en savoir plus sur Azure Stack Hub, consultez [Qu’est-ce qu’Azure Stack Hub ?](https://azure.microsoft.com/products/azure-stack/hub/).</span><span class="sxs-lookup"><span data-stu-id="42c77-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="42c77-121">Un abonnement de locataire sur chaque infrastructure Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42c77-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="42c77-122">**Prenez note de chaque ID d’abonnement et du point de terminaison Azure Resource Manager de chaque infrastructure Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="42c77-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="42c77-123">Un principal de service Azure Active Directory (Azure AD) disposant d’autorisations pour l’abonnement du locataire sur chaque infrastructure Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42c77-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="42c77-124">Vous devrez peut-être créer deux principaux de service si les infrastructures Azure Stack Hub sont déployées auprès de différents locataires Azure AD.</span><span class="sxs-lookup"><span data-stu-id="42c77-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="42c77-125">Pour apprendre à créer un principal de service pour Azure Stack Hub, consultez [Utiliser une identité d’application pour accéder aux ressources Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="42c77-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="42c77-126">**Prenez note de l’ID d’application, de la clé secrète client et du nom de locataire de chaque principal de service (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="42c77-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="42c77-127">Ubuntu 16.04 syndiqué sur la Place de marché de chaque Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42c77-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="42c77-128">Pour en savoir plus sur la syndication de la Place de marché, consultez [Télécharger des éléments de la Place de marché vers Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="42c77-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="42c77-129">[Docker pour Windows](https://docs.docker.com/docker-for-windows/) installé sur votre ordinateur local.</span><span class="sxs-lookup"><span data-stu-id="42c77-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="42c77-130">Obtenir l’image Docker</span><span class="sxs-lookup"><span data-stu-id="42c77-130">Get the Docker image</span></span>

<span data-ttu-id="42c77-131">Des images Docker pour chaque déploiement éliminent les problèmes de dépendance entre les différentes versions d’Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="42c77-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="42c77-132">Assurez-vous que Docker pour Windows utilise des conteneurs Windows.</span><span class="sxs-lookup"><span data-stu-id="42c77-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="42c77-133">Exécutez la commande suivante dans une invite de commandes avec élévation de privilèges pour obtenir le conteneur Docker avec les scripts de déploiement.</span><span class="sxs-lookup"><span data-stu-id="42c77-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="42c77-134">Déployer les clusters</span><span class="sxs-lookup"><span data-stu-id="42c77-134">Deploy the clusters</span></span>

1. <span data-ttu-id="42c77-135">Une fois l’image conteneur correctement extraite, démarrez l’image.</span><span class="sxs-lookup"><span data-stu-id="42c77-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="42c77-136">Une fois le conteneur démarré, vous recevrez un terminal PowerShell avec élévation de privilèges dans le conteneur.</span><span class="sxs-lookup"><span data-stu-id="42c77-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="42c77-137">Modifiez les répertoires pour obtenir le script de déploiement.</span><span class="sxs-lookup"><span data-stu-id="42c77-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="42c77-138">Exécutez le déploiement.</span><span class="sxs-lookup"><span data-stu-id="42c77-138">Run the deployment.</span></span> <span data-ttu-id="42c77-139">Fournissez les informations d’identification et les noms de ressources si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="42c77-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="42c77-140">HA fait référence à l’infrastructure Azure Stack Hub où le cluster hautement disponible sera déployé.</span><span class="sxs-lookup"><span data-stu-id="42c77-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="42c77-141">DR fait référence à l’infrastructure Azure Stack Hub où le cluster de reprise d’activité sera déployé.</span><span class="sxs-lookup"><span data-stu-id="42c77-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. <span data-ttu-id="42c77-142">Tapez `Y` pour autoriser l’installation du fournisseur NuGet qui déclenche l’installation des modules « 2018-03-01-hybrid » du profil d’API.</span><span class="sxs-lookup"><span data-stu-id="42c77-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="42c77-143">Les ressources hautement disponibles sont déployées en premier.</span><span class="sxs-lookup"><span data-stu-id="42c77-143">The HA resources will deploy first.</span></span> <span data-ttu-id="42c77-144">Supervisez le déploiement et attendez qu’il se termine.</span><span class="sxs-lookup"><span data-stu-id="42c77-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="42c77-145">Lorsque s’affiche le message indiquant que le déploiement haute disponibilité est terminé, vous pouvez consulter le portail de l’infrastructure Azure Stack Hub hautement disponible pour voir les ressources déployées.</span><span class="sxs-lookup"><span data-stu-id="42c77-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="42c77-146">Poursuivez avec le déploiement des ressources de récupération d’urgence, et décidez si vous souhaitez activer une machine virtuelle jumpbox sur l’infrastructure Azure Stack Hub de récupération d’urgence pour interagir avec le cluster.</span><span class="sxs-lookup"><span data-stu-id="42c77-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="42c77-147">Attendez la fin du déploiement des ressources de reprise d’activité.</span><span class="sxs-lookup"><span data-stu-id="42c77-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="42c77-148">Une fois le déploiement des ressources de reprise d’activité terminé, quittez le conteneur.</span><span class="sxs-lookup"><span data-stu-id="42c77-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="42c77-149">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="42c77-149">Next steps</span></span>

- <span data-ttu-id="42c77-150">Si vous avez activé la machine virtuelle jumpbox sur l’infrastructure Azure Stack Hub de récupération d’urgence, vous pouvez vous connecter via le protocole SSH et interagir avec le cluster MongoDB en installant l’interface de ligne de commande mongo.</span><span class="sxs-lookup"><span data-stu-id="42c77-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="42c77-151">Pour en savoir plus sur l’interaction avec MongoDB, voir [Interpréteur de commandes mongo](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="42c77-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="42c77-152">Pour en savoir plus sur les applications cloud hybrides, consultez [Solutions cloud hybrides](https://aka.ms/azsdevtutorials).</span><span class="sxs-lookup"><span data-stu-id="42c77-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="42c77-153">Modifiez le code pour cet exemple sur [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="42c77-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
