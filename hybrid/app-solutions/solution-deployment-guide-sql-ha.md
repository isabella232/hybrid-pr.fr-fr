---
title: Déployer un groupe de disponibilité SQL Server 2016 sur Azure et Azure Stack Hub
description: Apprenez à déployer un groupe de disponibilité SQL Server 2016 sur Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852471"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="728dd-103">Déployer un groupe de disponibilité SQL Server 2016 sur Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="728dd-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="728dd-104">Cet article vous guidera tout au long du processus de déploiement automatisé d'un cluster de base SQL Server 2016 Entreprise haute disponibilité (HA) avec un site de récupération d'urgence asynchrone dans deux environnements Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="728dd-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="728dd-105">Pour en savoir plus sur SQL Server 2016 et la haute disponibilité, voir [Groupes de disponibilité Always On : une solution de haute disponibilité et de récupération d’urgence](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="728dd-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="728dd-106">Dans cette solution, vous allez générer un exemple d’environnement pour :</span><span class="sxs-lookup"><span data-stu-id="728dd-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="728dd-107">Orchestrer un déploiement dans deux environnements Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="728dd-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="728dd-108">Utiliser Docker pour minimiser les problèmes de dépendance avec des profils d’API Azure.</span><span class="sxs-lookup"><span data-stu-id="728dd-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="728dd-109">Déployer un cluster SQL Server 2016 Entreprise hautement disponible de base avec un site de reprise d’activité.</span><span class="sxs-lookup"><span data-stu-id="728dd-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="728dd-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="728dd-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="728dd-111">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="728dd-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="728dd-112">Azure Stack Hub apporte l’agilité et l’innovation du cloud computing à votre environnement local et active le seul cloud hybride qui vous permet de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="728dd-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="728dd-113">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) en matière de conception, de déploiement et d’exploitation des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="728dd-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="728dd-114">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="728dd-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="728dd-115">Architecture pour SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="728dd-115">Architecture for SQL Server 2016</span></span>

![Azure Stack Hub haute disponibilité pour SQL Server 2016](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="728dd-117">Conditions préalables pour SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="728dd-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="728dd-118">Deux systèmes intégrés Azure Stack Hub connectés (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="728dd-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="728dd-119">Ce déploiement ne fonctionne pas sur le kit de développement Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="728dd-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="728dd-120">Pour en savoir plus sur Azure Stack Hub, consultez [Vue d’ensemble d’Azure Stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="728dd-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="728dd-121">Un abonnement de locataire sur chaque infrastructure Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="728dd-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="728dd-122">**Prenez note de chaque ID d’abonnement et du point de terminaison Azure Resource Manager de chaque infrastructure Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="728dd-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="728dd-123">Un principal de service Azure Active Directory (Azure AD) disposant d’autorisations pour l’abonnement du locataire sur chaque infrastructure Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="728dd-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="728dd-124">Vous devrez peut-être créer deux principaux de service si les infrastructures Azure Stack Hub sont déployées auprès de différents locataires Azure AD.</span><span class="sxs-lookup"><span data-stu-id="728dd-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="728dd-125">Pour apprendre à créer un principal de service pour Azure Stack Hub, consultez [Créer des principaux de service pour permettre à des applications d’accéder à des ressources Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="728dd-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="728dd-126">**Prenez note de l’ID d’application, de la clé secrète client et du nom de locataire de chaque principal de service (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="728dd-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="728dd-127">SQL Server 2016 Entreprise syndiqué sur la Place de marché de chaque infrastructure Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="728dd-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="728dd-128">Pour en savoir plus sur la syndication de la Place de marché, consultez [Télécharger des éléments de la Place de marché vers Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="728dd-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="728dd-129">**Assurez-vous que votre organisation dispose des licences SQL appropriées.**</span><span class="sxs-lookup"><span data-stu-id="728dd-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="728dd-130">[Docker pour Windows](https://docs.docker.com/docker-for-windows/) installé sur votre ordinateur local.</span><span class="sxs-lookup"><span data-stu-id="728dd-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="728dd-131">Obtenir l’image Docker</span><span class="sxs-lookup"><span data-stu-id="728dd-131">Get the Docker image</span></span>

<span data-ttu-id="728dd-132">Des images Docker pour chaque déploiement éliminent les problèmes de dépendance entre les différentes versions d’Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="728dd-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="728dd-133">Assurez-vous que Docker pour Windows utilise des conteneurs Windows.</span><span class="sxs-lookup"><span data-stu-id="728dd-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="728dd-134">Exécutez le script suivant dans une invite de commandes avec élévation de privilèges pour obtenir le conteneur Docker avec les scripts de déploiement.</span><span class="sxs-lookup"><span data-stu-id="728dd-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="728dd-135">Déployer le groupe de disponibilité</span><span class="sxs-lookup"><span data-stu-id="728dd-135">Deploy the availability group</span></span>

1. <span data-ttu-id="728dd-136">Une fois l’image conteneur correctement extraite, démarrez l’image.</span><span class="sxs-lookup"><span data-stu-id="728dd-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="728dd-137">Une fois le conteneur démarré, vous recevrez un terminal PowerShell avec élévation de privilèges dans le conteneur.</span><span class="sxs-lookup"><span data-stu-id="728dd-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="728dd-138">Modifiez les répertoires pour obtenir le script de déploiement.</span><span class="sxs-lookup"><span data-stu-id="728dd-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="728dd-139">Exécutez le déploiement.</span><span class="sxs-lookup"><span data-stu-id="728dd-139">Run the deployment.</span></span> <span data-ttu-id="728dd-140">Fournissez les informations d’identification et les noms de ressources si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="728dd-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="728dd-141">HA fait référence à l’infrastructure Azure Stack Hub où le cluster hautement disponible sera déployé.</span><span class="sxs-lookup"><span data-stu-id="728dd-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="728dd-142">DR fait référence à l’infrastructure Azure Stack Hub où le cluster de reprise d’activité sera déployé.</span><span class="sxs-lookup"><span data-stu-id="728dd-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="728dd-143">Tapez `Y` pour autoriser l’installation du fournisseur NuGet qui déclenche l’installation des modules « 2018-03-01-hybrid » du profil d’API.</span><span class="sxs-lookup"><span data-stu-id="728dd-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="728dd-144">Attendez la fin du déploiement de la ressource.</span><span class="sxs-lookup"><span data-stu-id="728dd-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="728dd-145">Une fois le déploiement de la ressource de récupération d’urgence terminé, quittez le conteneur.</span><span class="sxs-lookup"><span data-stu-id="728dd-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="728dd-146">Inspectez le déploiement en examinant les ressources sur le portail de chaque infrastructure Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="728dd-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="728dd-147">Connectez-vous à l’une des instances SQL dans l’environnement hautement disponible et inspectez le groupe de disponibilité via SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="728dd-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![haute disponibilité SQL dans SQL Server 2016](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="728dd-149">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="728dd-149">Next steps</span></span>

- <span data-ttu-id="728dd-150">Utilisez SQL Server Management Studio pour basculer manuellement le cluster.</span><span class="sxs-lookup"><span data-stu-id="728dd-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="728dd-151">Consultez [Effectuer un basculement manuel forcé d’un groupe de disponibilité Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="728dd-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="728dd-152">Découvrez plus en détail les applications cloud hybrides.</span><span class="sxs-lookup"><span data-stu-id="728dd-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="728dd-153">Consultez [Solutions cloud hybrides](/azure-stack/user/).</span><span class="sxs-lookup"><span data-stu-id="728dd-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="728dd-154">Utilisez vos propres données ou modifiez le code pour cet exemple sur [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="728dd-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>