---
title: Déployer une application qui effectue une mise à l’échelle multicloud dans Azure et Azure Stack Hub
description: Découvrez comment déployer une application qui effectue une mise à l’échelle multicloud dans Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910637"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="81051-103">Déployer une application qui effectue une mise à l’échelle multicloud à l’aide d’Azure et d’Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="81051-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="81051-104">Découvrez comment créer une solution multicloud pour fournir un processus déclenché manuellement permettant de passer d’une application web hébergée sur Azure Stack Hub à une application web hébergée sur Azure avec mise à l’échelle automatique via Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="81051-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="81051-105">Ce processus garantit la disponibilité d’un utilitaire cloud flexible et évolutif sous charge.</span><span class="sxs-lookup"><span data-stu-id="81051-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="81051-106">Avec ce modèle, il se peut que votre locataire ne soit pas prêt à exécuter votre application dans le cloud public.</span><span class="sxs-lookup"><span data-stu-id="81051-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="81051-107">Toutefois, ceci peut ne pas être économiquement faisable pour que l’entreprise puisse maintenir la capacité nécessaire dans son environnement local afin de gérer les pics de demande de l’application.</span><span class="sxs-lookup"><span data-stu-id="81051-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="81051-108">Votre locataire peut faire usage de l’élasticité du cloud public avec sa solution locale.</span><span class="sxs-lookup"><span data-stu-id="81051-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="81051-109">Dans cette solution, vous allez générer un exemple d’environnement pour :</span><span class="sxs-lookup"><span data-stu-id="81051-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="81051-110">Créer une application web à plusieurs nœuds.</span><span class="sxs-lookup"><span data-stu-id="81051-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="81051-111">Configurer et gérer le processus de déploiement continu (CD).</span><span class="sxs-lookup"><span data-stu-id="81051-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="81051-112">Publier l’application web dans Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="81051-113">Créer une mise en production.</span><span class="sxs-lookup"><span data-stu-id="81051-113">Create a release.</span></span>
> - <span data-ttu-id="81051-114">Apprenez à surveiller et à suivre vos déploiements.</span><span class="sxs-lookup"><span data-stu-id="81051-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="81051-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="81051-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="81051-116">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="81051-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="81051-117">Azure Stack Hub apporte l’agilité et l’innovation du cloud computing à votre environnement local et active le seul cloud hybride qui vous permet de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="81051-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="81051-118">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) pour la conception, le déploiement et le fonctionnement des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="81051-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="81051-119">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="81051-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="81051-120">Prérequis</span><span class="sxs-lookup"><span data-stu-id="81051-120">Prerequisites</span></span>

- <span data-ttu-id="81051-121">Abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="81051-121">Azure subscription.</span></span> <span data-ttu-id="81051-122">Si nécessaire, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.</span><span class="sxs-lookup"><span data-stu-id="81051-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="81051-123">Système intégré Azure Stack Hub ou déploiement du kit de développement Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="81051-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="81051-124">Pour obtenir des instructions concernant l’installation d’Azure Stack Hub, consultez [Installer le kit ASDK](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="81051-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="81051-125">Pour un script d’automatisation post-déploiement ASDK, accédez à : [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="81051-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="81051-126">Cette installation peut prendre quelques heures.</span><span class="sxs-lookup"><span data-stu-id="81051-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="81051-127">Déployez les services PaaS [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="81051-128">[Créez des plans/offres](/azure-stack/operator/service-plan-offer-subscription-overview.md) dans l’environnement Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="81051-129">[Créez un abonnement de locataire](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) dans l'environnement Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="81051-130">Créez une application web dans l’abonnement du locataire.</span><span class="sxs-lookup"><span data-stu-id="81051-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="81051-131">Notez l’URL de la nouvelle application web. Vous en aurez besoin plus tard.</span><span class="sxs-lookup"><span data-stu-id="81051-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="81051-132">Déployez la machine virtuelle Azure Pipelines au sein de l’abonnement du locataire.</span><span class="sxs-lookup"><span data-stu-id="81051-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="81051-133">Une machine virtuelle Windows Server 2016 avec .NET 3.5 est nécessaire.</span><span class="sxs-lookup"><span data-stu-id="81051-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="81051-134">Cette machine virtuelle est créée dans l’abonnement du locataire sur Azure Stack Hub en tant qu’agent de build privé.</span><span class="sxs-lookup"><span data-stu-id="81051-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="81051-135">[Windows Server 2016 avec l’image de machine virtuelle SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) est disponible dans la Place de marché Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="81051-136">Si cette image n’est pas disponible, utilisez un opérateur Azure Stack Hub pour vous assurer qu’elle est ajoutée à l’environnement.</span><span class="sxs-lookup"><span data-stu-id="81051-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="81051-137">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="81051-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="81051-138">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="81051-138">Scalability</span></span>

<span data-ttu-id="81051-139">Le principal composant de la mise à l’échelle inter-cloud est la capacité à fournir une mise à l’échelle à la demande immédiate entre l’infrastructure cloud publique et locale, garantissant un service fiable et cohérent.</span><span class="sxs-lookup"><span data-stu-id="81051-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="81051-140">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="81051-140">Availability</span></span>

<span data-ttu-id="81051-141">Vérifiez que les applications déployées en local sont configurées pour la haute disponibilité via la configuration matérielle locale et le déploiement de logiciels.</span><span class="sxs-lookup"><span data-stu-id="81051-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="81051-142">Simplicité de gestion</span><span class="sxs-lookup"><span data-stu-id="81051-142">Manageability</span></span>

<span data-ttu-id="81051-143">La solution dans le cloud garantit une gestion transparente et une interface familière entre les environnements.</span><span class="sxs-lookup"><span data-stu-id="81051-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="81051-144">PowerShell est recommandé pour une gestion multiplateforme.</span><span class="sxs-lookup"><span data-stu-id="81051-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="81051-145">Mise à l’échelle multicloud</span><span class="sxs-lookup"><span data-stu-id="81051-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="81051-146">Obtenir un domaine personnalisé et configurer DNS</span><span class="sxs-lookup"><span data-stu-id="81051-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="81051-147">Mettez à jour le fichier de zone DNS pour le domaine.</span><span class="sxs-lookup"><span data-stu-id="81051-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="81051-148">Azure AD vérifie la propriété du nom de domaine personnalisé.</span><span class="sxs-lookup"><span data-stu-id="81051-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="81051-149">Utilisez [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) pour les enregistrements DNS Azure/Office 365/externes dans Azure, ou ajoutez l’entrée DNS à [un autre bureau d’enregistrement DNS](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span><span class="sxs-lookup"><span data-stu-id="81051-149">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="81051-150">Inscrivez un domaine personnalisé auprès d’un bureau d’enregistrement public.</span><span class="sxs-lookup"><span data-stu-id="81051-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="81051-151">Connectez-vous au Bureau d’enregistrement des noms de domaine pour le domaine.</span><span class="sxs-lookup"><span data-stu-id="81051-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="81051-152">Il se peut qu’un administrateur approuvé doive effectuer les mises à jour DNS.</span><span class="sxs-lookup"><span data-stu-id="81051-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="81051-153">Mettez à jour le fichier de zone DNS du domaine en ajoutant l’entrée DNS fournie par Azure AD.</span><span class="sxs-lookup"><span data-stu-id="81051-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="81051-154">(L’entrée DNS n’affecte pas le routage des e-mails ni les comportements d’hébergement web.)</span><span class="sxs-lookup"><span data-stu-id="81051-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="81051-155">Créer une application web à plusieurs nœuds par défaut dans Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="81051-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="81051-156">Configurez l’intégration continue/le déploiement continu (CI/CD) hybride pour déployer des applications web sur Azure et Azure Stack Hub, et envoyer automatiquement les modifications aux deux clouds.</span><span class="sxs-lookup"><span data-stu-id="81051-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="81051-157">Vous avez besoin d’Azure Stack Hub avec les images appropriées syndiquées pour s’exécuter (Windows Server et SQL), et App Service doit être déployé.</span><span class="sxs-lookup"><span data-stu-id="81051-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="81051-158">Pour plus d’informations, consultez la documentation App Service [Prérequis pour le déploiement d’App Service sur Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="81051-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="81051-159">Ajouter du code à Azure Repos</span><span class="sxs-lookup"><span data-stu-id="81051-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="81051-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="81051-160">Azure Repos</span></span>

1. <span data-ttu-id="81051-161">Connectez-vous à Azure Repos avec un compte ayant les droits de création de projet sur Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="81051-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="81051-162">La CI/CD hybride peut s’appliquer tant au code d’application qu’au code d’infrastructure.</span><span class="sxs-lookup"><span data-stu-id="81051-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="81051-163">Utilisez les [modèles Azure Resource Manager](https://azure.microsoft.com/resources/templates/) pour les développements cloud privé et hébergé.</span><span class="sxs-lookup"><span data-stu-id="81051-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Se connecter à un projet sur Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="81051-165">**Clonez le référentiel** en créant et en ouvrant l’application web par défaut.</span><span class="sxs-lookup"><span data-stu-id="81051-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Cloner un référentiel dans une application web Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="81051-167">Créer un déploiement d’applications web autonomes pour App Services dans les deux clouds</span><span class="sxs-lookup"><span data-stu-id="81051-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="81051-168">Modifiez le fichier **WebApplication.csproj**.</span><span class="sxs-lookup"><span data-stu-id="81051-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="81051-169">Sélectionnez `Runtimeidentifier`, puis ajoutez `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="81051-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="81051-170">(Consultez la documentation sur le [déploiement autonome](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf).)</span><span class="sxs-lookup"><span data-stu-id="81051-170">(See [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Modifier un fichier de projet d’application web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="81051-172">Archivez le code dans Azure Repos avec Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="81051-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="81051-173">Vérifiez que le code d’application a été archivé dans Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="81051-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="81051-174">Créer la définition de build</span><span class="sxs-lookup"><span data-stu-id="81051-174">Create the build definition</span></span>

1. <span data-ttu-id="81051-175">Connectez-vous à Azure Pipelines pour vérifier la possibilité de créer des définitions de build.</span><span class="sxs-lookup"><span data-stu-id="81051-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="81051-176">Ajoutez le code **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="81051-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="81051-177">Cet ajout est nécessaire pour déclencher un déploiement autonome avec .NET Core.</span><span class="sxs-lookup"><span data-stu-id="81051-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Ajouter du code à l’application web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="81051-179">Exécutez la build.</span><span class="sxs-lookup"><span data-stu-id="81051-179">Run the build.</span></span> <span data-ttu-id="81051-180">Le processus de [build de déploiement autonome](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) publie des artefacts qui s’exécutent sur Azure et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-180">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="81051-181">Utiliser un agent hébergé sur Azure</span><span class="sxs-lookup"><span data-stu-id="81051-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="81051-182">L’utilisation d’un agent de build hébergé dans Azure Pipelines est une option pratique pour créer et déployer des applications web.</span><span class="sxs-lookup"><span data-stu-id="81051-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="81051-183">La maintenance et les mises à niveau sont effectuées automatiquement par Microsoft Azure, ce qui permet un cycle de développement continu et sans interruption.</span><span class="sxs-lookup"><span data-stu-id="81051-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="81051-184">Gérer et configurer le processus CD</span><span class="sxs-lookup"><span data-stu-id="81051-184">Manage and configure the CD process</span></span>

<span data-ttu-id="81051-185">Azure Pipelines et Azure DevOps Services fournissent un pipeline hautement configurable et gérable pour des mises en production sur plusieurs environnements, comme des environnements de développement, de préproduction, d’assurance qualité et de production, avec des demandes d’approbation à des étapes spécifiques.</span><span class="sxs-lookup"><span data-stu-id="81051-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="81051-186">Créer une définition de mise en production</span><span class="sxs-lookup"><span data-stu-id="81051-186">Create release definition</span></span>

1. <span data-ttu-id="81051-187">Sélectionnez le bouton **plus** pour ajouter une nouvelle mise en production sous l’onglet **Mises en production** dans la section **Build et mise en production** d’Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="81051-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Création d’une définition de version](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="81051-189">Appliquez le modèle de déploiement Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="81051-189">Apply the Azure App Service Deployment template.</span></span>

   ![Appliquer un modèle de déploiement Azure App Service](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="81051-191">Sous **Ajouter un artefact**, ajoutez l’artefact pour l’application de build Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="81051-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Ajouter un artefact à la build Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="81051-193">Sous l’onglet Pipeline, sélectionnez le lien **Phase, Tâche** de l’environnement, et définissez les valeurs de l’environnement cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="81051-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Définir des valeurs d’environnement cloud Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="81051-195">Définissez le **nom d’environnement** et sélectionnez l’**abonnement Azure** pour le point de terminaison du Cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="81051-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Sélectionner un abonnement Azure pour le point de terminaison du Cloud Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="81051-197">Sous **Nom de l’App Service**, définissez le nom de service de l’application Azure.</span><span class="sxs-lookup"><span data-stu-id="81051-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Définir un nom de service d’application Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="81051-199">Entrez VS2017 hébergé sous **File d’attente de l’agent** pour l’environnement hébergé dans le cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="81051-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Définir une file d’attente pour l’environnement hébergé dans le cloud Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="81051-201">Dans le menu Déployer Azure App Service, sélectionnez **le package ou le dossier** valide pour l’environnement.</span><span class="sxs-lookup"><span data-stu-id="81051-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="81051-202">Sélectionnez **OK** pour **l’emplacement du dossier**.</span><span class="sxs-lookup"><span data-stu-id="81051-202">Select **OK** to **folder location**.</span></span>
  
      ![Sélectionner un package ou un dossier pour l’environnement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Sélectionner un package ou un dossier pour l’environnement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="81051-205">Enregistrez toutes les modifications et revenez au **pipeline de mises en production**.</span><span class="sxs-lookup"><span data-stu-id="81051-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Enregistrer les modifications dans le pipeline de mise en production](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="81051-207">Ajoutez un nouvel artefact en sélectionnant la build pour l’application Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Ajouter un nouvel artefact pour un application Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="81051-209">Ajoutez un environnement supplémentaire en appliquant le déploiement Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="81051-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Ajouter un environnement à un déploiement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="81051-211">Nommez le nouvel environnement « Azure Stack ».</span><span class="sxs-lookup"><span data-stu-id="81051-211">Name the new environment "Azure Stack".</span></span>

    ![Nommer un environnement dans un déploiement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="81051-213">Recherchez l’environnement Azure Stack sous l’onglet **Tâche**.</span><span class="sxs-lookup"><span data-stu-id="81051-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Environnement Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="81051-215">Sélectionnez l’abonnement pour le point de terminaison Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="81051-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Sélectionner l’abonnement pour le point de terminaison Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="81051-217">Définissez le nom de l’application web Azure Stack comme nom de l’App Service.</span><span class="sxs-lookup"><span data-stu-id="81051-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="81051-218">![Définir un nom d’application web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="81051-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="81051-219">Sélectionnez l’agent Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="81051-219">Select the Azure Stack agent.</span></span>

    ![Sélectionner l’agent Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="81051-221">Sous la section Déployer Azure App Service, sélectionnez **le package ou le dossier** valides pour l’environnement.</span><span class="sxs-lookup"><span data-stu-id="81051-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="81051-222">Sélectionnez **OK** pour l’emplacement du dossier.</span><span class="sxs-lookup"><span data-stu-id="81051-222">Select **OK** to folder location.</span></span>

    ![Sélectionner un dossier pour un déploiement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Sélectionner un dossier pour un déploiement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="81051-225">Sous l’onglet Variable, ajoutez une variable nommée `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, définissez sa valeur sur **true** et sa portée sur Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="81051-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Ajouter une variable à un déploiement Azure App](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="81051-227">Sélectionnez l’icône du déclencheur de déploiement **Continu** dans les deux artefacts et activez le déclencheur de déploiement **Continu**.</span><span class="sxs-lookup"><span data-stu-id="81051-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Sélectionner un déclencheur de déploiement continu](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="81051-229">Sélectionnez l’icône des conditions **Prédéploiement** dans l’environnement Azure Stack et définissez le déclencheur sur **Après la mise en production**.</span><span class="sxs-lookup"><span data-stu-id="81051-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Sélectionner des conditions préalables au déploiement](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="81051-231">Enregistrez toutes les modifications.</span><span class="sxs-lookup"><span data-stu-id="81051-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="81051-232">Certains paramètres des tâches peuvent avoir été automatiquement définis en tant que [variables d’environnement](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) lors de la création d’une définition de mise en production à partir d’un modèle.</span><span class="sxs-lookup"><span data-stu-id="81051-232">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="81051-233">Ces paramètres ne peuvent pas être modifiés dans les paramètres de la tâche. Au lieu de cela, l’élément d’environnement parent doit être sélectionné pour modifier ces paramètres.</span><span class="sxs-lookup"><span data-stu-id="81051-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="81051-234">Publier sur Azure Stack Hub via Visual Studio</span><span class="sxs-lookup"><span data-stu-id="81051-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="81051-235">En créant des points de terminaison, une build Azure DevOps Services peut déployer des applications Azure App Service sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="81051-236">Azure Pipelines se connecte à l’agent de build, qui se connecte à Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="81051-237">Connectez-vous à Azure DevOps Services et accédez à la page Paramètres de l’application.</span><span class="sxs-lookup"><span data-stu-id="81051-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="81051-238">Dans **Paramètres**, sélectionnez **Sécurité**.</span><span class="sxs-lookup"><span data-stu-id="81051-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="81051-239">Dans **Groupes VSTS**, sélectionnez **Créateurs de points de terminaison**.</span><span class="sxs-lookup"><span data-stu-id="81051-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="81051-240">Sous l’onglet **Membres**, sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="81051-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="81051-241">Dans **Ajouter des utilisateurs et groupes**, entrez un nom d’utilisateur et sélectionnez cet utilisateur dans la liste des utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="81051-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="81051-242">Sélectionnez **Enregistrer les modifications**.</span><span class="sxs-lookup"><span data-stu-id="81051-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="81051-243">Dans la liste **Groupes VSTS**, sélectionnez **Administrateurs du point de terminaison**.</span><span class="sxs-lookup"><span data-stu-id="81051-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="81051-244">Sous l’onglet **Membres**, sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="81051-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="81051-245">Dans **Ajouter des utilisateurs et groupes**, entrez un nom d’utilisateur et sélectionnez cet utilisateur dans la liste des utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="81051-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="81051-246">Sélectionnez **Enregistrer les modifications**.</span><span class="sxs-lookup"><span data-stu-id="81051-246">Select **Save changes**.</span></span>

<span data-ttu-id="81051-247">Maintenant que les informations du point de terminaison ont été ajoutées, la connexion entre Azure Pipelines et Azure Stack Hub est prête à être utilisée.</span><span class="sxs-lookup"><span data-stu-id="81051-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="81051-248">L’agent de build dans Azure Stack Hub reçoit des instructions d’Azure Pipelines, puis transmet les informations du point de terminaison pour la communication avec Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="81051-249">Développer la build de l’application</span><span class="sxs-lookup"><span data-stu-id="81051-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="81051-250">Vous avez besoin d’Azure Stack Hub avec les images appropriées syndiquées pour s’exécuter (Windows Server et SQL), et App Service doit être déployé.</span><span class="sxs-lookup"><span data-stu-id="81051-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="81051-251">Pour plus d’informations, consultez [Conditions préalables au déploiement d’App Service sur Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="81051-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="81051-252">Utilisez des [modèles Azure Resource Manager](https://azure.microsoft.com/resources/templates/) comme du code d’application web d’Azure Repos pour le déploiement dans les deux clouds.</span><span class="sxs-lookup"><span data-stu-id="81051-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="81051-253">Ajouter du code à un projet Azure Repos</span><span class="sxs-lookup"><span data-stu-id="81051-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="81051-254">Connectez-vous à Azure Repos avec un compte ayant les droits de création de projet sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="81051-255">**Clonez le référentiel** en créant et en ouvrant l’application web par défaut.</span><span class="sxs-lookup"><span data-stu-id="81051-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="81051-256">Créer un déploiement d’applications web autonomes pour App Services dans les deux clouds</span><span class="sxs-lookup"><span data-stu-id="81051-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="81051-257">Modifiez le fichier **WebApplication.csproj** : Sélectionnez `Runtimeidentifier`, puis ajoutez `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="81051-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="81051-258">Pour plus d’informations, consultez la documentation sur le [déploiement autonome](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="81051-258">For more information, see [Self-contained deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="81051-259">Utilisez Team Explorer pour archiver le code dans Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="81051-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="81051-260">Vérifiez que le code d’application a été archivé dans Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="81051-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="81051-261">Créer la définition de build</span><span class="sxs-lookup"><span data-stu-id="81051-261">Create the build definition</span></span>

1. <span data-ttu-id="81051-262">Connectez-vous à Azure Pipelines avec un compte permettant de créer une définition de build.</span><span class="sxs-lookup"><span data-stu-id="81051-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="81051-263">Accédez à la page **Build Web Application** (Créer une application web) du projet.</span><span class="sxs-lookup"><span data-stu-id="81051-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="81051-264">Dans **Arguments**, ajoutez le code **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="81051-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="81051-265">Cet ajout est obligatoire pour déclencher un déploiement autonome avec .NET Core.</span><span class="sxs-lookup"><span data-stu-id="81051-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="81051-266">Exécutez la build.</span><span class="sxs-lookup"><span data-stu-id="81051-266">Run the build.</span></span> <span data-ttu-id="81051-267">Le processus de [build de déploiement autonome](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) publie des artefacts qui peuvent s’exécuter sur Azure et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-267">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="81051-268">Utiliser un agent de build hébergé Azure</span><span class="sxs-lookup"><span data-stu-id="81051-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="81051-269">L’utilisation d’un agent de build hébergé dans Azure Pipelines est une option pratique pour créer et déployer des applications web.</span><span class="sxs-lookup"><span data-stu-id="81051-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="81051-270">La maintenance et les mises à niveau sont effectuées automatiquement par Microsoft Azure, ce qui permet un cycle de développement continu et sans interruption.</span><span class="sxs-lookup"><span data-stu-id="81051-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="81051-271">Configurer le processus de déploiement continu (CD)</span><span class="sxs-lookup"><span data-stu-id="81051-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="81051-272">Azure Pipelines et Azure DevOps Services fournissent un pipeline hautement configurable et gérable pour des mises en production sur plusieurs environnements, comme des environnements de développement, de préproduction, d’assurance qualité et de production.</span><span class="sxs-lookup"><span data-stu-id="81051-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="81051-273">Ce processus peut inclure des demandes d’approbation à des étapes spécifiques du cycle de vie de l’application.</span><span class="sxs-lookup"><span data-stu-id="81051-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="81051-274">Créer une définition de mise en production</span><span class="sxs-lookup"><span data-stu-id="81051-274">Create release definition</span></span>

<span data-ttu-id="81051-275">La création d’une définition de mise en production est la dernière étape du processus de création d’application.</span><span class="sxs-lookup"><span data-stu-id="81051-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="81051-276">Cette définition de mise en production est utilisée pour créer une mise en production et déployer une build.</span><span class="sxs-lookup"><span data-stu-id="81051-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="81051-277">Connectez-vous à Azure Pipelines et accédez à **Build et mise en production** pour le projet.</span><span class="sxs-lookup"><span data-stu-id="81051-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="81051-278">Sous l’onglet **Versions**, sélectionnez **[ + ]** , puis choisissez **Créer une définition de mise en production**.</span><span class="sxs-lookup"><span data-stu-id="81051-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="81051-279">Sous **Sélectionner un modèle**, choisissez **Déploiement d'Azure App Service**, puis sélectionnez **Appliquer**.</span><span class="sxs-lookup"><span data-stu-id="81051-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="81051-280">Sous **Ajouter un artefact**, dans **Source (définition de build)** , sélectionnez l’application de build Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="81051-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="81051-281">Sous l’onglet **Pipeline**, sélectionnez le lien **1 Phase**, **1 Tâche** vers **Afficher les tâches d’environnement**.</span><span class="sxs-lookup"><span data-stu-id="81051-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="81051-282">Sous l’onglet **Tâches**, entrez Azure comme **nom d’environnement** et sélectionnez AzureCloud Traders-Web EP dans la liste **Abonnement Azure**.</span><span class="sxs-lookup"><span data-stu-id="81051-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="81051-283">Entrez le **nom de l’Azure App Service**, qui est `northwindtraders` dans la capture d’écran suivante.</span><span class="sxs-lookup"><span data-stu-id="81051-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="81051-284">Pour la phase d’agent, sélectionnez **VS2017 hébergé** dans la liste **File d’attente d’agents**.</span><span class="sxs-lookup"><span data-stu-id="81051-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="81051-285">Dans **Déployer Azure App Service**, sélectionnez le **package ou dossier** valide pour l’environnement.</span><span class="sxs-lookup"><span data-stu-id="81051-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="81051-286">Dans **Sélectionner un fichier ou un dossier**, sélectionnez **OK** pour **Emplacement**.</span><span class="sxs-lookup"><span data-stu-id="81051-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="81051-287">Enregistrez toutes les modifications et revenez à **Pipeline**.</span><span class="sxs-lookup"><span data-stu-id="81051-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="81051-288">Sous l’onglet **Pipeline**, sélectionnez **Ajouter un artefact** et choisissez **NorthwindCloud Traders-Vessel** dans la liste **Source (définition de build)** .</span><span class="sxs-lookup"><span data-stu-id="81051-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="81051-289">Sous **Sélectionner un modèle**, ajoutez un autre environnement.</span><span class="sxs-lookup"><span data-stu-id="81051-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="81051-290">Choisissez **Déploiement d'Azure App Service**, puis sélectionnez **Appliquer**.</span><span class="sxs-lookup"><span data-stu-id="81051-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="81051-291">Entrez `Azure Stack Hub` comme **nom d’environnement**.</span><span class="sxs-lookup"><span data-stu-id="81051-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="81051-292">Sous l’onglet **Tâches**, recherchez et sélectionnez Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="81051-293">Dans la liste **Abonnement Azure**, sélectionnez **AzureStack Traders-Vessel EP** pour le point de terminaison Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="81051-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="81051-294">Entrez le nom de l’application web Azure Stack Hub comme **nom de l’App Service**.</span><span class="sxs-lookup"><span data-stu-id="81051-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="81051-295">Sous **Sélection de l’Agent**, choisissez **AzureStack -b Douglas Fir** dans la liste **File d’attente d’agents**.</span><span class="sxs-lookup"><span data-stu-id="81051-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="81051-296">Pour **Déployer Azure App Service**, sélectionnez le **package ou dossier** valide pour l’environnement.</span><span class="sxs-lookup"><span data-stu-id="81051-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="81051-297">Sous **Sélectionner un fichier ou un dossier**, sélectionnez **OK** pour le dossier **Emplacement**.</span><span class="sxs-lookup"><span data-stu-id="81051-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="81051-298">Sous l’onglet **Variable**, recherchez la variable nommée `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span><span class="sxs-lookup"><span data-stu-id="81051-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="81051-299">Définissez la valeur de la variable sur **true** et définissez sa portée sur **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="81051-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="81051-300">Sous l’onglet **Pipeline**, sélectionnez l’icône **Déclencheur de déploiement continu** pour l’artefact NorthwindCloud Traders-Web et définissez le **Déclencheur de déploiement continu** sur **Activé**.</span><span class="sxs-lookup"><span data-stu-id="81051-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="81051-301">Procédez de la même manière pour l’artefact **NorthwindCloud Traders-Vessel**.</span><span class="sxs-lookup"><span data-stu-id="81051-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="81051-302">Pour l’environnement Azure Stack Hub, sélectionnez l’icône **Conditions préalables au déploiement** et définissez le déclencheur sur **Après la mise en production**.</span><span class="sxs-lookup"><span data-stu-id="81051-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="81051-303">Enregistrez toutes les modifications.</span><span class="sxs-lookup"><span data-stu-id="81051-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="81051-304">Certains paramètres des tâches de mise en production sont automatiquement définis en tant que [variables d’environnement](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) lors de la création d’une définition de mise en production à partir d’un modèle.</span><span class="sxs-lookup"><span data-stu-id="81051-304">Some settings for release tasks are automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="81051-305">Ces paramètres ne peuvent pas être modifiés dans les paramètres de tâche, mais peuvent l’être dans les éléments d’environnement parent.</span><span class="sxs-lookup"><span data-stu-id="81051-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="81051-306">Créer une mise en production</span><span class="sxs-lookup"><span data-stu-id="81051-306">Create a release</span></span>

1. <span data-ttu-id="81051-307">Sous l’onglet **Pipeline**, ouvrez la liste **Version finale**, et choisissez **Créer une mise en production**.</span><span class="sxs-lookup"><span data-stu-id="81051-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="81051-308">Entrez une description de la mise en production, vérifiez que les artefacts corrects sont sélectionnés, puis sélectionnez **Créer**.</span><span class="sxs-lookup"><span data-stu-id="81051-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="81051-309">Après quelques instants, une bannière s’affiche, indiquant que la nouvelle mise en production a été créée, et le nom de la mise en production est affichée sous forme de lien.</span><span class="sxs-lookup"><span data-stu-id="81051-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="81051-310">Sélectionnez le lien pour consulter la page récapitulative de la mise en production.</span><span class="sxs-lookup"><span data-stu-id="81051-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="81051-311">La page récapitulative de mise en production fournit des détails sur la mise en production.</span><span class="sxs-lookup"><span data-stu-id="81051-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="81051-312">Dans la capture d’écran suivante pour « Release-2 », la section **Environnements** indique que l’**état du déploiement** d’Azure est « IN PROGRESS » (En cours) et que l’état d’Azure Stack Hub est « SUCCEEDED » (Réussi).</span><span class="sxs-lookup"><span data-stu-id="81051-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="81051-313">Lorsque l’état du déploiement de l’environnement Azure passe à « SUCCEEDED » (Réussi), une bannière indiquant que la mise en production est prête pour l’approbation s’affiche.</span><span class="sxs-lookup"><span data-stu-id="81051-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="81051-314">Lorsqu’un déploiement est en attente ou a échoué, une icône d’informations **(i)** bleue est affichée.</span><span class="sxs-lookup"><span data-stu-id="81051-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="81051-315">Pointez sur l’icône pour afficher une fenêtre contextuelle qui indique le motif du retard ou de l’échec.</span><span class="sxs-lookup"><span data-stu-id="81051-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="81051-316">D’autres vues, comme la liste des mises en production, affichent aussi une icône indiquant qu’une approbation est en attente.</span><span class="sxs-lookup"><span data-stu-id="81051-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="81051-317">La fenêtre contextuelle de cette icône indique le nom de l’environnement et plus de détails sur le déploiement.</span><span class="sxs-lookup"><span data-stu-id="81051-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="81051-318">Un administrateur peut facilement voir la progression globale des mises en production et savoir quelles mises en production sont en attente d’approbation.</span><span class="sxs-lookup"><span data-stu-id="81051-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="81051-319">Surveiller et suivre les déploiements</span><span class="sxs-lookup"><span data-stu-id="81051-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="81051-320">Sur la page récapitulative **Release-2**, sélectionnez **Journaux d’activité**.</span><span class="sxs-lookup"><span data-stu-id="81051-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="81051-321">Pendant un déploiement, cette page affiche le journal en direct de l’agent.</span><span class="sxs-lookup"><span data-stu-id="81051-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="81051-322">Le volet gauche indique l’état de chaque opération du déploiement pour chaque environnement.</span><span class="sxs-lookup"><span data-stu-id="81051-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="81051-323">Sélectionnez l’icône représentant une personne dans la colonne **Action** pour une approbation avant déploiement ou après déploiement afin de voir qui a approuvé (ou rejeté) le déploiement ainsi que le message que ces personnes ont fourni.</span><span class="sxs-lookup"><span data-stu-id="81051-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="81051-324">Lorsque le déploiement est terminé, l’intégralité du fichier journal s’affiche dans le volet droit.</span><span class="sxs-lookup"><span data-stu-id="81051-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="81051-325">Sélectionnez une **étape** dans le volet gauche pour voir le fichier journal d’une étape spécifique comme **Initialiser le travail**.</span><span class="sxs-lookup"><span data-stu-id="81051-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="81051-326">La possibilité de voir les journaux d’activité individuels facilite le suivi et le débogage de parties du déploiement global.</span><span class="sxs-lookup"><span data-stu-id="81051-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="81051-327">**Enregistrez** le fichier journal d’une étape ou **téléchargez tous les journaux au format zip**.</span><span class="sxs-lookup"><span data-stu-id="81051-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="81051-328">Ouvrez l’onglet **Résumé** pour afficher des informations générales sur la mise en production.</span><span class="sxs-lookup"><span data-stu-id="81051-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="81051-329">Cette vue montre les détails de la build, les environnements où elle a été déployée, l’état du déploiement et d’autres informations sur la mise en production.</span><span class="sxs-lookup"><span data-stu-id="81051-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="81051-330">Sélectionnez un lien d’environnement (**Azure** ou **Azure Stack Hub**) pour afficher des informations sur les déploiements existants et en attente dans un environnement spécifique.</span><span class="sxs-lookup"><span data-stu-id="81051-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="81051-331">Utilisez ces vues pour vérifier rapidement que la même build a été déployée sur les deux environnements.</span><span class="sxs-lookup"><span data-stu-id="81051-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="81051-332">Ouvrez l’**application de production déployée** dans un navigateur.</span><span class="sxs-lookup"><span data-stu-id="81051-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="81051-333">Par exemple, pour le site web Azure App Services, ouvrez l’URL `https://[your-app-name\].azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="81051-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="81051-334">L’intégration d’Azure et d’Azure Stack Hub fournit une solution multicloud scalable</span><span class="sxs-lookup"><span data-stu-id="81051-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="81051-335">Un service à plusieurs clouds flexible et fiable offre sécurité des données, sauvegarde et redondance, disponibilité rapide et cohérente, stockage et distribution évolutifs, et routage conforme géographiquement.</span><span class="sxs-lookup"><span data-stu-id="81051-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="81051-336">Ce processus déclenché manuellement garantit une commutation de charge fiable et efficace entre les applications web hébergées, ainsi qu’une disponibilité immédiate des données critiques.</span><span class="sxs-lookup"><span data-stu-id="81051-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="81051-337">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="81051-337">Next steps</span></span>

- <span data-ttu-id="81051-338">Pour plus d’informations sur les modèles Azure Cloud, consultez [Modèles de conception cloud](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="81051-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
