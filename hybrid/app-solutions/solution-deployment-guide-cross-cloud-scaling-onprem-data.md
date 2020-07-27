---
title: Déployer une application hybride avec des données locales qui effectue une mise à l’échelle multicloud
description: Découvrez comment déployer une application qui utilise des données locales et effectue une mise à l’échelle multicloud à l’aide d’Azure et d’Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477318"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="fd7c5-103">Déployer une application hybride avec des données locales qui effectue une mise à l’échelle multicloud</span><span class="sxs-lookup"><span data-stu-id="fd7c5-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="fd7c5-104">Ce guide de solution vous explique comment déployer une application hybride qui s’étend sur Azure et Azure Stack Hub, et utilise une source de données locale unique.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="fd7c5-105">En utilisant une solution de cloud hybride, vous pouvez combiner les avantages de la conformité d’un cloud privé avec l’extensibilité du cloud public.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="fd7c5-106">Vos développeurs peuvent aussi tirer parti de l’écosystème de développement Microsoft et appliquer leurs compétences aux environnements cloud et locaux.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="fd7c5-107">Vue d’ensemble et hypothèses</span><span class="sxs-lookup"><span data-stu-id="fd7c5-107">Overview and assumptions</span></span>

<span data-ttu-id="fd7c5-108">Suivez ce didacticiel pour configurer un flux de travail qui permet aux développeurs de déployer une application web identique vers un cloud public et un cloud privé.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="fd7c5-109">Cette application peut accéder à un réseau routable non-Internet hébergé sur le cloud privé.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="fd7c5-110">Ces applications web sont surveillées : en cas de pic de trafic, un programme modifie les enregistrements DNS pour rediriger le trafic vers le cloud public.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="fd7c5-111">Lorsque le trafic redescend au niveau avant le pic, le trafic est routé vers le cloud privé.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="fd7c5-112">Ce tutoriel décrit les tâches suivantes :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="fd7c5-113">Déployer un serveur de base de données SQL Server à connexion hybride.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="fd7c5-114">Connecter une application web présente dans Azure global à un réseau hybride.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="fd7c5-115">Configurer DNS pour une mise à l’échelle dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="fd7c5-116">Configurer des certificats SSL pour une mise à l’échelle dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="fd7c5-117">Configurez et déployez l’application web.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="fd7c5-118">Créer un profil Traffic Manager et configurez-le pour une mise à l’échelle dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="fd7c5-119">Configurer la supervision et les alertes Application Insights en cas d’augmentation du trafic.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="fd7c5-120">Configurer la commutation automatique du trafic entre Azure global et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="fd7c5-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="fd7c5-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="fd7c5-122">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="fd7c5-123">Azure Stack Hub offre à votre environnement local l’agilité et l’innovation du cloud computing grâce au seul cloud hybride qui vous permette de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="fd7c5-124">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) pour la conception, le déploiement et le fonctionnement des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="fd7c5-125">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="fd7c5-126">Hypothèses</span><span class="sxs-lookup"><span data-stu-id="fd7c5-126">Assumptions</span></span>

<span data-ttu-id="fd7c5-127">Ce didacticiel suppose que vous disposez de connaissances de base sur Azure global et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="fd7c5-128">Si vous voulez en savoir plus avant de commencer le didacticiel, consultez les articles suivants :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="fd7c5-129">Présentation de Microsoft Azure</span><span class="sxs-lookup"><span data-stu-id="fd7c5-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="fd7c5-130">Concepts clés d'Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="fd7c5-131">Ce didacticiel part du principe que vous disposez d’un abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="fd7c5-132">Si vous n’avez pas d’abonnement, [créez un compte gratuit](https://azure.microsoft.com/free/) avant de commencer.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="fd7c5-133">Prérequis</span><span class="sxs-lookup"><span data-stu-id="fd7c5-133">Prerequisites</span></span>

<span data-ttu-id="fd7c5-134">Avant de commencer cette solution, vérifiez que les conditions suivantes sont réunies :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="fd7c5-135">Un Kit de développement Azure Stack (ASDK) ou un abonnement à un système intégré Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="fd7c5-136">Pour déployer le kit ASDK, suivez les instructions qui figurent dans [Déployer le kit ASDK à l’aide du programme d’installation](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="fd7c5-137">Votre installation Azure Stack Hub doit avoir installé les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="fd7c5-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-138">The Azure App Service.</span></span> <span data-ttu-id="fd7c5-139">Travaillez avec votre opérateur Azure Stack Hub pour déployer et configurer Azure App Service sur votre environnement.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="fd7c5-140">Ce didacticiel nécessite qu’App Service dispose d’au moins (1) rôle de travail dédié disponible.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="fd7c5-141">Une image Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="fd7c5-142">Un serveur Windows Server 2016 avec une image Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="fd7c5-143">Les plans et offres appropriés.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="fd7c5-144">Un nom de domaine pour votre application web.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-144">A domain name for your web app.</span></span> <span data-ttu-id="fd7c5-145">Si vous n’avez pas de nom de domaine, vous pouvez en acheter un auprès d’un fournisseur de domaine comme GoDaddy, Bluehost et InMotion.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="fd7c5-146">Un certificat SSL pour votre domaine reçu d’une autorité de certification de confiance comme LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="fd7c5-147">Une application web qui communique avec une base de données SQL Server et prend en charge Application Insights.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="fd7c5-148">Vous pouvez télécharger l’exemple d’application [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) à partir de GitHub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="fd7c5-149">Un réseau hybride entre un réseau virtuel Azure et un réseau virtuel Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="fd7c5-150">Pour obtenir des instructions détaillées, consultez [Configurer la connectivité de cloud hybride avec Azure et Azure Stack Hub](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="fd7c5-151">Un pipeline hybride d’intégration continue/déploiement continu (CI/CD) avec un agent de build privé sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="fd7c5-152">Pour obtenir des instructions détaillées, consultez [Configurer l’identité de cloud hybride avec Azure et Azure Stack Hub](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="fd7c5-153">Déployer un serveur de base de données SQL Server à connexion hybride</span><span class="sxs-lookup"><span data-stu-id="fd7c5-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="fd7c5-154">Connectez-vous au portail utilisateur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="fd7c5-155">Sur le **Tableau de bord**, sélectionnez **Place de marché**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Place de marché Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="fd7c5-157">Dans **Place de marché**, sélectionnez **Calcul**, puis choisissez **Plus**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="fd7c5-158">Sous **Plus**, sélectionnez l’image **Licence SQL Server gratuite : SQL Server 2017 Developer sur Windows Server**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Sélectionner une image de machine virtuelle dans le portail utilisateur Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="fd7c5-160">Dans **Licence SQL Server gratuite : SQL Server 2017 Developer sur Windows Server**, sélectionnez **Créer**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="fd7c5-161">Sur **Paramètres de base > Configurer les paramètres de base**, saisissez un **Nom** pour la machine virtuelle (VM), un **Nom d’utilisateur** pour l’association de sécurité de SQL Server et un **Mot de passe** pour l’association de sécurité.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="fd7c5-162">Dans la liste déroulante **Abonnement**, sélectionnez l’abonnement sur lequel vous effectuez le déploiement.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="fd7c5-163">Pour **Groupe de ressources**, utilisez **Choisir un élément déjà existant** et placez la machine virtuelle dans le même groupe de ressources que votre application web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Configurer les paramètres de base de la machine virtuelle dans le portail utilisateur Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="fd7c5-165">Sous **Taille**, choisissez une taille pour votre machine virtuelle.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="fd7c5-166">Pour ce didacticiel, nous vous recommandons A2_Standard ou un DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="fd7c5-167">Sous **Paramètres > Configurer les fonctionnalités facultatives**, configurez les paramètres suivants :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="fd7c5-168">**Compte de stockage** : Créez un nouveau compte si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="fd7c5-169">**Réseau virtuel** :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="fd7c5-170">Assurez-vous que votre machine virtuelle SQL Server est déployée sur le même réseau virtuel que les passerelles VPN.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="fd7c5-171">**Adresse IP publique** : Conservez les paramètres par défaut.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="fd7c5-172">**Groupe de sécurité réseau** : (NSG).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="fd7c5-173">Créer un NSG.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-173">Create a new NSG.</span></span>
   - <span data-ttu-id="fd7c5-174">**Extensions et supervision** : Conservez les paramètres par défaut.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="fd7c5-175">**Compte de stockage de diagnostics** : Créez un nouveau compte si nécessaire.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="fd7c5-176">Cliquez sur **OK** pour enregistrer votre configuration.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-176">Select **OK** to save your configuration.</span></span>

     ![Configurer des fonctionnalités de machine virtuelle facultatives dans le portail utilisateur Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="fd7c5-178">Sous **Paramètres SQL Server**, configurez les paramètres suivants :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="fd7c5-179">Pour **Connectivité SQL**, sélectionnez **Public (Internet)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="fd7c5-180">Pour **Port**, conservez la valeur par défaut, **1433**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="fd7c5-181">Pour **Authentification SQL**, sélectionnez **Activer**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="fd7c5-182">Lorsque vous activez l’authentification SQL, les informations de « SQLAdmin » que vous avez configurées dans **Paramètres de base** devraient se renseigner automatiquement.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="fd7c5-183">Conservez les valeurs par défaut pour les autres paramètres.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="fd7c5-184">Sélectionnez **OK**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-184">Select **OK**.</span></span>

     ![Configurer des paramètres SQL Server dans le portail utilisateur Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="fd7c5-186">Dans **Résumé**, examinez la configuration de la machine virtuelle, puis sélectionnez **OK** pour démarrer le déploiement.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Résumé de la configuration dans le portail utilisateur Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="fd7c5-188">La création de la machine virtuelle prend un certain temps.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="fd7c5-189">Vous pouvez afficher l’ÉTAT de vos machines virtuelles dans **Machines virtuelles**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![État des machines virtuelles dans le portail utilisateur Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="fd7c5-191">Créer des applications web dans Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd7c5-192">Le service Azure App Service simplifie l’exécution et la gestion d’une application web.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="fd7c5-193">Étant donné qu’Azure Stack Hub est cohérent avec Azure, le service App Service peut s’exécuter dans les deux environnements.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="fd7c5-194">Vous allez utiliser App Service pour héberger votre application.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="fd7c5-195">Créer des applications web</span><span class="sxs-lookup"><span data-stu-id="fd7c5-195">Create web apps</span></span>

1. <span data-ttu-id="fd7c5-196">Créez une application web dans Azure en suivant les instructions dans [Gérer un plan App Service dans Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="fd7c5-197">Veillez à placer l’application web dans le même abonnement et groupe de ressources que votre réseau hybride.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="fd7c5-198">Répétez l’étape précédente (1) dans Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="fd7c5-199">Ajouter une route pour Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="fd7c5-200">Le service App Service sur Azure Stack Hub doit être routable à partir de l’Internet public pour permettre aux utilisateurs d’accéder à votre application.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="fd7c5-201">Si votre Azure Stack Hub est accessible à partir d’Internet, notez l’adresse IP ou l’URL publiques de l’application web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="fd7c5-202">Si vous utilisez un ASDK, vous pouvez [configurer un mappage NAT statique](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) pour exposer App Service en dehors de l’environnement virtuel.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="fd7c5-203">Connecter une application web présente dans Azure à un réseau hybride</span><span class="sxs-lookup"><span data-stu-id="fd7c5-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="fd7c5-204">Pour fournir la connectivité entre le front-end web dans Azure et la base de données SQL Server dans Azure Stack Hub, l’application web doit être connectée au réseau hybride entre Azure et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="fd7c5-205">Pour activer la connectivité, vous devrez :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="fd7c5-206">Configurer la connectivité point à site.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="fd7c5-207">Configurer l’application web.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-207">Configure the web app.</span></span>
- <span data-ttu-id="fd7c5-208">Modifier la passerelle de réseau local dans Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="fd7c5-209">Configurer le réseau virtuel Azure pour une connectivité point à site</span><span class="sxs-lookup"><span data-stu-id="fd7c5-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="fd7c5-210">La passerelle de réseau virtuel du côté Azure du réseau hybride doit autoriser l’intégration des connexions point à site à Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="fd7c5-211">Dans Azure, accédez à la page de la passerelle de réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="fd7c5-212">Sous **Paramètres**, sélectionnez **Configuration de point à site**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Option point à site dans la passerelle de réseau virtuel Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="fd7c5-214">Sélectionnez **Configurer maintenant** pour configurer la connectivité point à site.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Démarrer la configuration point à site dans la passerelle de réseau virtuel Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="fd7c5-216">Sur la page de la configuration de **Point à site**, ajoutez la plage d’adresses IP privées que vous souhaitez utiliser dans la zone **Pool d’adresses**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="fd7c5-217">Assurez-vous que la plage que vous spécifiez ne chevauche aucune des plages d’adresses déjà utilisées par les sous-réseaux dans les composants Azure global ou Azure Stack Hub du réseau hybride.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="fd7c5-218">Sous **Type de Tunnel**, décochez **VPN IKEv2**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="fd7c5-219">Sélectionnez **Enregistrer** pour terminer la configuration de point-to-site.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Paramètres point à site dans la passerelle de réseau virtuel Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="fd7c5-221">Intégrer l’application Azure App Service avec le réseau hybride</span><span class="sxs-lookup"><span data-stu-id="fd7c5-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="fd7c5-222">Pour connecter l’application au réseau virtuel Azure, suivez les instructions fournies dans la section [Intégration de réseau virtuel requise par la passerelle](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="fd7c5-223">Accédez aux **Paramètres** du plan App Service hébergeant l’application web.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="fd7c5-224">Sous **Paramètres**, sélectionnez **Mise en réseau**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-224">In **Settings**, select **Networking**.</span></span>

    ![Configurer la mise en réseau pour le plan App Service](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="fd7c5-226">Dans **Intégration au réseau virtuel**, sélectionnez **Cliquer ici pour gérer**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Gérer l’intégration au réseau virtuel pour le plan App Service](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="fd7c5-228">Sélectionnez le réseau virtuel à configurer.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="fd7c5-229">Sous **ADRESSES IP ROUTÉES VERS LE RÉSEAU VIRTUEL**, entrez la plage d’adresses IP pour les espaces d’adressage du réseau virtuel Azure, du réseau virtuel Azure Stack Hub et du réseau de point à site.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="fd7c5-230">Cliquez sur **Enregistrer** pour valider et enregistrer ces paramètres.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-230">Select **Save** to validate and save these settings.</span></span>

    ![Plages d’adresses IP à router dans l’intégration au réseau virtuel](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="fd7c5-232">Pour en savoir plus sur comment App Service s’intègre aux réseaux virtuels Azure, consultez [Intégrer votre application à un réseau virtuel Azure](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="fd7c5-233">Configurer le réseau virtuel Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="fd7c5-234">La passerelle de réseau local dans le réseau virtuel Azure Stack Hub doit être configurée pour router le trafic à partir de la plage d’adresses de point-to-site App Service.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="fd7c5-235">Dans Azure Stack Hub, accédez à **Passerelle de réseau local**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="fd7c5-236">Sous **Paramètres**, sélectionnez **Configuration**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-236">Under **Settings**, select **Configuration**.</span></span>

    ![Option de configuration de la passerelle dans la passerelle de réseau local Azure Stack Hub](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="fd7c5-238">Dans **Espace d’adressage**, entrez la plage d’adresses point à site pour la passerelle de réseau virtuel dans Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Espace d’adressage point à site dans la passerelle de réseau local Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="fd7c5-240">Sélectionnez **Enregistrer** pour valider et enregistrer la configuration.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="fd7c5-241">Configurer DNS pour une mise à l’échelle dans le cloud</span><span class="sxs-lookup"><span data-stu-id="fd7c5-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="fd7c5-242">En configurant correctement DNS pour les applications inter-cloud, les utilisateurs peuvent accéder aux instances globales Azure et Azure Stack Hub de votre application web.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="fd7c5-243">La configuration DNS pour ce didacticiel permet également à Azure Traffic Manager de router le trafic lorsque la charge augmente ou diminue.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="fd7c5-244">Ce didacticiel utilise Azure DNS pour gérer le serveur DNS, car des domaines App Service ne fonctionneront pas.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="fd7c5-245">Créer des sous-domaines</span><span class="sxs-lookup"><span data-stu-id="fd7c5-245">Create subdomains</span></span>

<span data-ttu-id="fd7c5-246">Étant donné que Traffic Manager s’appuie sur les CNAME DNS, un sous-domaine est nécessaire pour router correctement le trafic vers les points de terminaison.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="fd7c5-247">Pour plus d’informations sur les enregistrements DNS et le mappage de domaine, voir [Mapper des domaines avec Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="fd7c5-248">Pour le point de terminaison Azure, vous allez créer un sous-domaine que les utilisateurs peuvent utiliser pour accéder à votre application web.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="fd7c5-249">Pour ce didacticiel, vous pouvez utiliser **app.northwind.com**, mais nous vous conseillons de personnaliser cette valeur en fonction de votre propre domaine.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="fd7c5-250">Vous devrez également créer un sous-domaine avec un enregistrement A pour le point de terminaison Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="fd7c5-251">Vous pouvez utiliser **azurestack.northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="fd7c5-252">Configurer un domaine personnalisé dans Azure</span><span class="sxs-lookup"><span data-stu-id="fd7c5-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="fd7c5-253">Ajoutez le nom d’hôte **app.northwind.com** à l’application web Azure en [mappant un CNAME vers Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="fd7c5-254">Configurer des domaines personnalisés dans Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="fd7c5-255">Ajoutez le nom d’hôte **azurestack.northwind.com** à l’application web Azure Stack Hub en [mappant un enregistrement A vers Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="fd7c5-256">Utilisez l’adresse IP routable sur Internet pour l’application App Service.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="fd7c5-257">Ajoutez le nom d’hôte **app.northwind.com** à l’application web Azure Stack Hub en [mappant un CNAME vers Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="fd7c5-258">Utilisez le nom d’hôte que vous avez configuré à l’étape précédente (1) comme cible pour le CNAME.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="fd7c5-259">Configurer des certificats SSL pour une mise à l’échelle dans le cloud</span><span class="sxs-lookup"><span data-stu-id="fd7c5-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="fd7c5-260">Il est important de vérifier que les données sensibles recueillies par votre application web sont sécurisées tant en transit vers la base de données SQL qu’au repos dans celle-ci.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="fd7c5-261">Vous allez configurer vos applications web Azure et Azure Stack Hub pour utiliser des certificats SSL pour tout le trafic entrant.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="fd7c5-262">Ajouter SSL à Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd7c5-263">Pour ajouter SSL à Azure :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="fd7c5-264">Assurez-vous que le certificat SSL que vous obtenez est valide pour le sous-domaine que vous avez créé.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="fd7c5-265">(Vous pouvez utiliser des certificats avec caractères génériques.)</span><span class="sxs-lookup"><span data-stu-id="fd7c5-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="fd7c5-266">Dans Azure, suivez les instructions fournies dans les sections **Préparer votre application web** et **Lier votre certificat SSL** de l’article [Lier un certificat SSL personnalisé existant à Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="fd7c5-267">Sélectionnez **SSL basé sur SNI** en tant que **Type de SSL**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="fd7c5-268">Rediriger tout le trafic vers le port HTTPS.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="fd7c5-269">Suivez les instructions dans la section **Appliquer le protocole HTTPS** de l’article [Lier un certificat SSL personnalisé existant à Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="fd7c5-270">Pour ajouter SSL à Azure Stack Hub :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="fd7c5-271">Répétez les étapes 1 à 3 effectuées pour Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="fd7c5-272">Configurer et déployer l’application web</span><span class="sxs-lookup"><span data-stu-id="fd7c5-272">Configure and deploy the web app</span></span>

<span data-ttu-id="fd7c5-273">Vous allez configurer le code d’application pour envoyer les données de télémétrie à l’instance Application Insights appropriée, puis configurer les applications web avec les chaînes de connexion adéquates.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="fd7c5-274">Pour en savoir plus sur Application Insights, consultez [Présentation d’Application Insights](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="fd7c5-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="fd7c5-275">Ajouter Application Insights</span><span class="sxs-lookup"><span data-stu-id="fd7c5-275">Add Application Insights</span></span>

1. <span data-ttu-id="fd7c5-276">Ouvrez votre application web dans Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="fd7c5-277">[Ajoutez Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) à votre projet pour transmettre les données de télémétrie qu’Application Insights utilise pour créer des alertes lorsque le trafic web augmente ou diminue.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="fd7c5-278">Configurer des chaînes de connexion dynamiques</span><span class="sxs-lookup"><span data-stu-id="fd7c5-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="fd7c5-279">Chaque instance de l’application web utilise une méthode différente pour se connecter à la base de données SQL.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="fd7c5-280">L’application dans Azure utilise l’adresse IP privée de la machine virtuelle SQL Server, et l’application dans Azure Stack Hub l’adresse IP publique de la machine virtuelle SQL Server.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="fd7c5-281">Sur un système intégré Azure Stack Hub, l’adresse IP publique ne doit pas être routable sur Internet.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="fd7c5-282">Sur un kit ASDK, l’adresse IP publique n’est pas routable en dehors du kit.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="fd7c5-283">Vous pouvez utiliser des variables d’environnement App Service pour transmettre une chaîne de connexion différente à chaque instance de l’application.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="fd7c5-284">Ouvrez l’application dans Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="fd7c5-285">Ouvrez Startup.cs et recherchez le bloc de code suivant :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="fd7c5-286">Remplacez le bloc de code précédent par le code suivant qui utilise une chaîne de connexion définie dans le fichier *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="fd7c5-287">Configurer les paramètres d’application App Service</span><span class="sxs-lookup"><span data-stu-id="fd7c5-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="fd7c5-288">Créez des chaînes de connexion pour Azure et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="fd7c5-289">Les chaînes doivent être identiques à l’exception des adresses IP utilisées.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="fd7c5-290">Dans Azure et Azure Stack Hub, ajoutez la chaîne de connexion appropriée [comme paramètre d’application](/azure/app-service/web-sites-configure) dans l’application web en utilisant `SQLCONNSTR\_` comme préfixe dans le nom.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="fd7c5-291">**Enregistrez** les paramètres de l’application web et redémarrez l’application.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="fd7c5-292">Activer la mise à l’échelle automatique dans Azure global</span><span class="sxs-lookup"><span data-stu-id="fd7c5-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="fd7c5-293">Lorsque vous créez votre application web dans un environnement App Service, l’application démarre avec une seule instance.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="fd7c5-294">Vous pouvez ensuite effectuer un scale-out automatiquement pour fournir des ressources de calcul supplémentaires à votre application.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="fd7c5-295">De même, vous pouvez automatiquement effectuer un scale-in et réduire le nombre d’instances dont votre application a besoin.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="fd7c5-296">Vous devez disposer d’un plan App Service pour configurer un scale-out et un scale-in.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="fd7c5-297">Si vous n’avez pas de plan, créez-le avant de commencer les étapes suivantes.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="fd7c5-298">Activer l’augmentation automatique de la taille des instances</span><span class="sxs-lookup"><span data-stu-id="fd7c5-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="fd7c5-299">Dans Azure, recherchez le plan App Service des sites pour lesquels vous souhaitez effectuer un scale-out, puis sélectionnez **Scale-out (plan App Service)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Effectuer un scale-out d’Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="fd7c5-301">Sélectionnez **Activer la mise à l’échelle automatique**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-301">Select **Enable autoscale**.</span></span>

    ![Activer la mise à l’échelle automatique dans Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="fd7c5-303">Fournissez un nom pour le **Nom du paramètre de mise à l’échelle automatique**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="fd7c5-304">Pour la règle de mise à l’échelle automatique **Par défaut**, sélectionnez **Mettre à l’échelle selon une mesure**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="fd7c5-305">Définissez les **Limites d’instance** sur **Minimum : 1**, **Maximum : 10** et **Par défaut : 1**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Configurer la mise à l’échelle automatique dans Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="fd7c5-307">Sélectionnez **+Ajouter une règle**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="fd7c5-308">Dans **Source de la mesure**, sélectionnez **Ressource actuelle**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="fd7c5-309">Utilisez les critères et les actions suivantes pour la règle.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="fd7c5-310">Critères</span><span class="sxs-lookup"><span data-stu-id="fd7c5-310">Criteria</span></span>

1. <span data-ttu-id="fd7c5-311">Sous **Agrégation du temps**, sélectionnez **Moyenne**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="fd7c5-312">Sous **Nom de mesure**, sélectionnez **Pourcentage UC**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="fd7c5-313">Sous **Opérateur**, sélectionnez **Supérieur à**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="fd7c5-314">Définissez le **Seuil** sur **50**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="fd7c5-315">Définir la **Durée** sur **10**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="fd7c5-316">Action</span><span class="sxs-lookup"><span data-stu-id="fd7c5-316">Action</span></span>

1. <span data-ttu-id="fd7c5-317">Sous **Opération**, sélectionnez **Augmenter le nombre de**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="fd7c5-318">Définissez le **Nombre d’instances** sur **2**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="fd7c5-319">Définissez le **Refroidissement** sur **5**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="fd7c5-320">Sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-320">Select **Add**.</span></span>

5. <span data-ttu-id="fd7c5-321">Sélectionnez **+Ajouter une règle**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="fd7c5-322">Dans **Source de la mesure**, sélectionnez **Ressource actuelle**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="fd7c5-323">La ressource actuelle contient le nom/GUID de votre plan App Service, et les listes déroulantes **Type de ressource** et **Ressource** ne sont pas disponibles.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="fd7c5-324">Activer le scale-in automatique</span><span class="sxs-lookup"><span data-stu-id="fd7c5-324">Enable automatic scale in</span></span>

<span data-ttu-id="fd7c5-325">Lorsque le trafic diminue, l’application web Azure peut diminuer automatiquement le nombre d’instances actives afin de réduire les coûts.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="fd7c5-326">Cette action est moins agressive que l’augmentation de la taille des instances et minimise l’impact sur les utilisateurs de l’application.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="fd7c5-327">Accédez à la condition de scale-out **Par défaut**, puis sélectionnez **+ Ajouter une règle**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="fd7c5-328">Utilisez les critères et les actions suivantes pour la règle.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="fd7c5-329">Critères</span><span class="sxs-lookup"><span data-stu-id="fd7c5-329">Criteria</span></span>

1. <span data-ttu-id="fd7c5-330">Sous **Agrégation du temps**, sélectionnez **Moyenne**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="fd7c5-331">Sous **Nom de mesure**, sélectionnez **Pourcentage UC**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="fd7c5-332">Sous **Opérateur**, sélectionnez **Inférieur à**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="fd7c5-333">Définissez le **Seuil** sur **30**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="fd7c5-334">Définir la **Durée** sur **10**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="fd7c5-335">Action</span><span class="sxs-lookup"><span data-stu-id="fd7c5-335">Action</span></span>

1. <span data-ttu-id="fd7c5-336">Sous **Opération**, sélectionnez **Diminuer le nombre de**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="fd7c5-337">Définissez le **Nombre d’instances** sur **1**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="fd7c5-338">Définissez le **Refroidissement** sur **5**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="fd7c5-339">Sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="fd7c5-340">Créer un profil Traffic Manager et le configurer pour une mise à l’échelle dans le cloud</span><span class="sxs-lookup"><span data-stu-id="fd7c5-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="fd7c5-341">Créez un profil Traffic Manager dans Azure, puis configurez les points de terminaison pour activer la mise à l’échelle inter-cloud.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="fd7c5-342">Créer un profil Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="fd7c5-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="fd7c5-343">Sélectionnez **Créer une ressource**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="fd7c5-344">Sélectionnez **Mise en réseau**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-344">Select **Networking**.</span></span>
3. <span data-ttu-id="fd7c5-345">Sélectionnez **Profil Traffic Manager**, puis configurez les paramètres suivants :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="fd7c5-346">Sous **Nom**, entrez un nom pour votre profil.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="fd7c5-347">Ce nom **doit** être unique dans la zone trafficmanager.net et il est utilisé pour créer un nom DNS (par exemple, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="fd7c5-348">Pour la **Méthode de routage**, sélectionnez la méthode **Pondérée**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="fd7c5-349">Sous **Abonnement**, sélectionnez l’abonnement dans lequel vous souhaitez créer ce profil.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="fd7c5-350">Sous **Groupe de ressources**, créez un groupe de ressources pour ce profil.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="fd7c5-351">Sous **Emplacement du groupe de ressources**, sélectionnez l’emplacement du groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="fd7c5-352">Ce paramètre fait référence à l’emplacement du groupe de ressources et n’a pas d’impact sur le profil Traffic Manager qui est déployé globalement.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="fd7c5-353">Sélectionnez **Create** (Créer).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-353">Select **Create**.</span></span>

    ![Créer un profil Traffic Manager](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="fd7c5-355">Lorsque le déploiement global de votre profil Traffic Manager est terminé, il apparait dans la liste des ressources pour le groupe de ressources sous lequel vous l’avez créé.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="fd7c5-356">Ajouter des points de terminaison Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="fd7c5-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="fd7c5-357">Recherchez le profil Traffic Manager que vous avez créé.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="fd7c5-358">Si vous avez accédé au groupe de ressources pour le profil, sélectionnez celui-ci.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="fd7c5-359">Dans **Profil Traffic Manager**, sous **PARAMÈTRES**, sélectionnez **Points de terminaison**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="fd7c5-360">Sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-360">Select **Add**.</span></span>

4. <span data-ttu-id="fd7c5-361">Dans **Ajouter un point de terminaison**, utilisez les paramètres suivants pour Azure Stack Hub :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="fd7c5-362">Pour **Type**, sélectionnez **Point de terminaison externe**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="fd7c5-363">Entrez un **Nom** pour le point de terminaison.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="fd7c5-364">Pour **Nom de domaine complet (FQDN) ou adresse IP**, entrez l’URL externe de votre application web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="fd7c5-365">Pour **Poids**, conservez la valeur par défaut **1**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="fd7c5-366">Ce poids a pour effet que tout le trafic est dirigé vers ce point de terminaison s’il est intègre.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="fd7c5-367">Laissez la case **Ajouter comme désactivé** décochée.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="fd7c5-368">Sélectionnez **OK** pour enregistrer le point de terminaison Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="fd7c5-369">Vous configurerez ensuite le point de terminaison Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="fd7c5-370">Dans **Profil Traffic Manager**, sélectionnez **Points de terminaison**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="fd7c5-371">Sélectionnez **+Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-371">Select **+Add**.</span></span>
3. <span data-ttu-id="fd7c5-372">Dans **Ajouter un point de terminaison**, utilisez les paramètres suivants pour Azure :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="fd7c5-373">Sous **Type**, sélectionnez **Point de terminaison Azure**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="fd7c5-374">Entrez un **Nom** pour le point de terminaison.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="fd7c5-375">Sous **Type de ressource cible**, sélectionnez **App Service**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="fd7c5-376">Sous **Ressource cible**, sélectionnez **Choisir un service d’application** pour afficher la liste des applications Web dans le même abonnement.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="fd7c5-377">Dans **Ressources**, choisissez le service d’application que vous souhaitez ajouter en tant que premier point de terminaison.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="fd7c5-378">Pour **Poids**, sélectionnez **2**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="fd7c5-379">Ainsi, tout le trafic est dirigé vers ce point de terminaison si le point de terminaison principal n’est pas intègre ou si vous disposez d’une règle/alerte qui redirige le trafic lorsqu’elle est déclenchée.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="fd7c5-380">Laissez la case **Ajouter comme désactivé** décochée.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="fd7c5-381">Sélectionnez **OK** pour enregistrer le point de terminaison Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="fd7c5-382">Une fois que les deux points de terminaison sont configurés, ils sont répertoriés dans **Profil Traffic Manager** lorsque vous sélectionnez **Points de terminaison**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="fd7c5-383">L’exemple sur la capture d’écran suivante montre deux points de terminaison, avec les informations d’état et de configuration pour chacun d’entre eux.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Points de terminaison dans le profil Traffic Manager](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="fd7c5-385">Configurer la supervision et les alertes Application Insights</span><span class="sxs-lookup"><span data-stu-id="fd7c5-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="fd7c5-386">Azure Application Insights vous permet de surveiller votre application et d’envoyer des alertes en fonction des conditions que vous configurez.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="fd7c5-387">Voici quelques exemples : l’application n’est pas disponible, rencontre des erreurs ou présente des problèmes de performances.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="fd7c5-388">Les mesures d’Application Insights vous permettront de créer des alertes.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="fd7c5-389">Lorsque ces alertes se déclenchent, l’instance de votre application web bascule automatiquement d’Azure Stack Hub vers Azure pour effectuer un scale-out avant de revenir à Azure Stack Hub pour effectuer un scale-in.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="fd7c5-390">Créer une alerte à partir de mesures</span><span class="sxs-lookup"><span data-stu-id="fd7c5-390">Create an alert from metrics</span></span>

<span data-ttu-id="fd7c5-391">Accédez au groupe de ressources de ce tutoriel, puis sélectionnez l’instance Application Insights pour ouvrir **Application Insights**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="fd7c5-393">Cette vue vous permettra de créer une alerte de scale-out et une alerte de scale-in.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="fd7c5-394">Créer l’alerte pour augmenter la taille des instances</span><span class="sxs-lookup"><span data-stu-id="fd7c5-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="fd7c5-395">Sous **CONFIGURER**, sélectionnez **Alertes (classique)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="fd7c5-396">Sélectionnez **Ajouter une alerte métrique (classique)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="fd7c5-397">Dans **Ajouter une règle**, configurez les paramètres suivants :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="fd7c5-398">Pour **Nom**, entrez **Augmenter la taille dans Azure Cloud**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="fd7c5-399">La **Description** est facultative.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="fd7c5-400">Sous **Source** > **Alerte pour**, sélectionnez **Métriques**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="fd7c5-401">Sous **Critères**, sélectionnez votre abonnement, le groupe de ressources pour votre profil Traffic Manager, et le nom du profil Traffic Manager pour la ressource.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="fd7c5-402">Pour **Mesure**, sélectionnez **Taux de requêtes**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="fd7c5-403">Pour **Condition**, sélectionnez **Supérieur à**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="fd7c5-404">Pour **Seuil**, entrez **2**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="fd7c5-405">Pour **Période**, sélectionnez **Au cours des 5 dernières minutes**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="fd7c5-406">Sous **Notifier via** :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="fd7c5-407">Cochez la case pour **Envoyer des e-mails aux propriétaires, contributeurs et lecteurs**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="fd7c5-408">Saisissez votre adresse e-mail pour **Adresse(s) e-mail administrateur supplémentaire(s)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="fd7c5-409">Dans la barre de menus, sélectionnez **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="fd7c5-410">Créer l’alerte de scale-in</span><span class="sxs-lookup"><span data-stu-id="fd7c5-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="fd7c5-411">Sous **CONFIGURER**, sélectionnez **Alertes (classique)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="fd7c5-412">Sélectionnez **Ajouter une alerte métrique (classique)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="fd7c5-413">Dans **Ajouter une règle**, configurez les paramètres suivants :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="fd7c5-414">Pour **Nom**, entrez **Diminuer la taille dans Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="fd7c5-415">La **Description** est facultative.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="fd7c5-416">Sous **Source** > **Alerte pour**, sélectionnez **Métriques**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="fd7c5-417">Sous **Critères**, sélectionnez votre abonnement, le groupe de ressources pour votre profil Traffic Manager, et le nom du profil Traffic Manager pour la ressource.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="fd7c5-418">Pour **Mesure**, sélectionnez **Taux de requêtes**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="fd7c5-419">Pour **Condition**, sélectionnez **Inférieur à**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="fd7c5-420">Pour **Seuil**, entrez **2**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="fd7c5-421">Pour **Période**, sélectionnez **Au cours des 5 dernières minutes**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="fd7c5-422">Sous **Notifier via** :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="fd7c5-423">Cochez la case pour **Envoyer des e-mails aux propriétaires, contributeurs et lecteurs**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="fd7c5-424">Saisissez votre adresse e-mail pour **Adresse(s) e-mail administrateur supplémentaire(s)** .</span><span class="sxs-lookup"><span data-stu-id="fd7c5-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="fd7c5-425">Dans la barre de menus, sélectionnez **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="fd7c5-426">La capture d’écran suivante illustre les alertes de scale-out et de scale-in.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Alertes Application Insights (classiques)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="fd7c5-428">Rediriger le trafic entre Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd7c5-429">Vous pouvez configurer le basculement manuel ou automatique du trafic de votre application web entre Azure et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="fd7c5-430">Configurer un basculement manuel entre Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd7c5-431">Lorsque votre site web atteint les seuils que vous avez configurés, vous recevez une alerte.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="fd7c5-432">Utilisez les étapes suivantes pour rediriger manuellement le trafic vers Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="fd7c5-433">Dans le portail Azure, sélectionnez votre profil Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Points de terminaison Traffic Manager dans le portail Azure](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="fd7c5-435">Sélectionnez **Points de terminaison**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="fd7c5-436">Sélectionnez le **point de terminaison Azure**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="fd7c5-437">Sous **État**, sélectionnez **Activé**, puis **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Activer un point de terminaison Azure dans le portail Azure](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="fd7c5-439">Sous **Points de terminaison**du profil Traffic Manager, sélectionnez **Point de terminaison externe**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="fd7c5-440">Sous **État**, sélectionnez **Désactivé**, puis **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Désactiver un point de terminaison Azure Stack Hub dans le portail Azure](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="fd7c5-442">Une fois les points de terminaison configurés, le trafic de l’application accède à votre application web augmentant la taille des instances Azure plutôt qu’à l’application web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Points de terminaison modifiés dans le trafic des applications web Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="fd7c5-444">Pour inverser le flux vers Azure Stack Hub, utilisez les étapes précédentes pour :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="fd7c5-445">Ajout du point de terminaison Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="fd7c5-446">désactiver le point de terminaison Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="fd7c5-447">Configurer un basculement automatique entre Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd7c5-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd7c5-448">Vous pouvez également utiliser la supervision d’Application Insights si votre application s’exécute dans un environnement [serverless](https://azure.microsoft.com/overview/serverless-computing/) fourni par Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="fd7c5-449">Dans ce scénario, vous pouvez configurer Application Insights pour utiliser un webhook qui appelle une application de fonction.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="fd7c5-450">Cette application active ou désactive automatiquement un point de terminaison en réponse à une alerte.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="fd7c5-451">Utilisez les étapes suivantes comme guide pour configurer le basculement automatique du trafic.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="fd7c5-452">Créez une application Azure Function.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="fd7c5-453">Créez une fonction déclenchée par HTTP.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="fd7c5-454">Importez les kits de développement logiciel (SDK) Azure pour Resource Manager, Web Apps et Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="fd7c5-455">Développer du code pour :</span><span class="sxs-lookup"><span data-stu-id="fd7c5-455">Develop code to:</span></span>

   - <span data-ttu-id="fd7c5-456">Vous authentifier auprès de votre abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="fd7c5-457">Utiliser un paramètre qui active ou désactive les points de terminaison Traffic Manager afin de diriger le trafic vers Azure ou Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="fd7c5-458">Enregistrer votre code et ajouter l’URL de l’application de fonction ainsi que les paramètres appropriés à la section **Webhook** des paramètres de règle d’alerte Application Insights.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="fd7c5-459">Le trafic est redirigé automatiquement lorsqu’une alerte Application Insights se déclenche.</span><span class="sxs-lookup"><span data-stu-id="fd7c5-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fd7c5-460">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="fd7c5-460">Next steps</span></span>

- <span data-ttu-id="fd7c5-461">Pour plus d’informations sur les modèles Azure Cloud, consultez [Modèles de conception cloud](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="fd7c5-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
