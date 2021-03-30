---
title: Diriger le trafic avec une application géodistribuée en utilisant Azure et Azure Stack Hub
description: Découvrez comment diriger le trafic vers des points de terminaison spécifiques avec une solution d’application géodistribuée à l’aide d’Azure et d’Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895429"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="665ce-103">Diriger le trafic avec une application géodistribuée en utilisant Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="665ce-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="665ce-104">Découvrez comment diriger le trafic vers des points de terminaison spécifiques en fonction de différentes mesures à l’aide du modèle d’applications géolocalisées.</span><span class="sxs-lookup"><span data-stu-id="665ce-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="665ce-105">En créant un profil Traffic Manager avec configuration du routage et du point de terminaison basée sur la géolocalisation, vous êtes certain que les informations sont dirigées vers les points de terminaison en fonction des exigences régionales, des réglementations organisationnelles et internationales et de vos besoins en matière de données.</span><span class="sxs-lookup"><span data-stu-id="665ce-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="665ce-106">Dans cette solution, vous allez générer un exemple d’environnement pour :</span><span class="sxs-lookup"><span data-stu-id="665ce-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="665ce-107">Créer une application géodistribuée.</span><span class="sxs-lookup"><span data-stu-id="665ce-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="665ce-108">Utiliser Traffic Manager pour cibler votre application.</span><span class="sxs-lookup"><span data-stu-id="665ce-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="665ce-109">Utiliser le modèle d’application géolocalisée.</span><span class="sxs-lookup"><span data-stu-id="665ce-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="665ce-110">Avec le modèle géodistribué, l’application couvre plusieurs régions.</span><span class="sxs-lookup"><span data-stu-id="665ce-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="665ce-111">Vous pouvez utiliser le cloud public par défaut, mais certains utilisateurs ont besoin que leurs données restent dans leur région.</span><span class="sxs-lookup"><span data-stu-id="665ce-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="665ce-112">Dans ce cas, orientez les utilisateurs vers le cloud le plus approprié en fonction de leurs besoins.</span><span class="sxs-lookup"><span data-stu-id="665ce-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="665ce-113">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="665ce-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="665ce-114">Considérations relatives à l’extensibilité</span><span class="sxs-lookup"><span data-stu-id="665ce-114">Scalability considerations</span></span>

<span data-ttu-id="665ce-115">La solution que vous allez créer avec cet article n’a pas pour objectif de s’adapter à la scalabilité.</span><span class="sxs-lookup"><span data-stu-id="665ce-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="665ce-116">Toutefois, utilisée en combinaison avec d’autres solutions Azure et locales, elle permet de s’adapter à ces exigences d’extensibilité.</span><span class="sxs-lookup"><span data-stu-id="665ce-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="665ce-117">Pour plus d’informations sur la création d’une solution hybride avec mise à l’échelle automatique via Traffic Manager, consultez [Créer des solutions de mise à l’échelle multicloud avec Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="665ce-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="665ce-118">Considérations relatives à la disponibilité</span><span class="sxs-lookup"><span data-stu-id="665ce-118">Availability considerations</span></span>

<span data-ttu-id="665ce-119">Cette solution ne prend pas directement en charge les besoins de disponibilité.</span><span class="sxs-lookup"><span data-stu-id="665ce-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="665ce-120">Toutefois, vous pouvez implémenter des solutions Azure et locales dans cette solution pour garantir la haute disponibilité de tous les composants impliqués.</span><span class="sxs-lookup"><span data-stu-id="665ce-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="665ce-121">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="665ce-121">When to use this pattern</span></span>

- <span data-ttu-id="665ce-122">Les succursales internationales de votre organisation exigent des politiques de sécurité et de distribution régionales personnalisées.</span><span class="sxs-lookup"><span data-stu-id="665ce-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="665ce-123">Chacun des bureaux de votre organisation extrait des données sur les employés, l’activité et les installations. Cela nécessite de rendre compte de l’activité en lien avec les réglementations locales et les fuseaux horaires.</span><span class="sxs-lookup"><span data-stu-id="665ce-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="665ce-124">Les exigences de grande échelle sont satisfaites par la mise à l’échelle horizontale (scale-out) des applications, avec plusieurs déploiements dans une ou plusieurs régions, pour gérer des besoins extrêmes en termes de charge.</span><span class="sxs-lookup"><span data-stu-id="665ce-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="665ce-125">Planification de la topologie</span><span class="sxs-lookup"><span data-stu-id="665ce-125">Planning the topology</span></span>

<span data-ttu-id="665ce-126">Avant de créer une empreinte d’application distribuée, il est utile de disposer des informations suivantes :</span><span class="sxs-lookup"><span data-stu-id="665ce-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="665ce-127">**Domaine personnalisé pour l’application :** quel est le nom de domaine personnalisé que les clients utiliseront pour accéder à l’application ?</span><span class="sxs-lookup"><span data-stu-id="665ce-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="665ce-128">Pour l’exemple d’application, le nom de domaine personnalisé est *www\.scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="665ce-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="665ce-129">**Domaine Traffic Manager :** vous devez choisir un nom de domaine au moment de la création d’un [profil Azure Traffic Manager](/azure/traffic-manager/traffic-manager-manage-profiles).</span><span class="sxs-lookup"><span data-stu-id="665ce-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="665ce-130">Ce nom est associé au suffixe *trafficmanager.net* pour enregistrer une entrée de domaine gérée par Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="665ce-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="665ce-131">Dans l’exemple d’application, le nom choisi est *scalable-ase-demo*.</span><span class="sxs-lookup"><span data-stu-id="665ce-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="665ce-132">Par conséquent, le nom de domaine complet géré par Traffic Manager est *scalable-ase-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="665ce-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="665ce-133">**Stratégie de mise à l’échelle de l’empreinte de l’application :** décidez si l’empreinte de l’application doit être distribuée dans plusieurs environnements App Service au sein d’une ou de plusieurs régions, ou dans un mélange des deux approches.</span><span class="sxs-lookup"><span data-stu-id="665ce-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="665ce-134">La décision doit être prise en fonction des attentes depuis l’emplacement d’origine du trafic du client, ainsi que de la manière dont peut évoluer le reste de l’application prenant en charge l’infrastructure principale.</span><span class="sxs-lookup"><span data-stu-id="665ce-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="665ce-135">Par exemple, avec une application à 100 % sans état, une application peut être adaptée à très grande échelle à l’aide d’une combinaison de plusieurs environnements App Service par région Azure, puis multipliée par les environnements App Service déployés dans plusieurs régions Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="665ce-136">Avec plus de 15 régions Azure globales disponibles, les clients peuvent véritablement créer une empreinte d’application à échelle mondiale.</span><span class="sxs-lookup"><span data-stu-id="665ce-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="665ce-137">Pour l’exemple d’application utilisé ici, trois environnements App Service ont été créés dans une seule région Azure (USA Centre Sud).</span><span class="sxs-lookup"><span data-stu-id="665ce-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="665ce-138">**Convention de nommage pour les environnements App Service :** chaque environnement App Service requiert un nom unique.</span><span class="sxs-lookup"><span data-stu-id="665ce-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="665ce-139">Au-delà d’un ou de deux environnements App Service, une convention de nommage identifiant chaque environnement App Service s’avère utile.</span><span class="sxs-lookup"><span data-stu-id="665ce-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="665ce-140">Pour l’exemple d’application utilisé ici, une convention d’affectation de noms simple a été utilisée.</span><span class="sxs-lookup"><span data-stu-id="665ce-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="665ce-141">Les noms des trois environnements App Service sont respectivement *fe1ase*, *fe2ase* et *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="665ce-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="665ce-142">**Convention d’affectation de noms pour les applications :** comme plusieurs instances de l’application vont être déployées, un nom est requis pour chacune d’entre elles.</span><span class="sxs-lookup"><span data-stu-id="665ce-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="665ce-143">Avec App Service Environment pour PowerApps, le même nom d’application peut être utilisé dans plusieurs environnements.</span><span class="sxs-lookup"><span data-stu-id="665ce-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="665ce-144">Étant donné que chaque environnement App Service comporte un suffixe de domaine unique, les développeurs peuvent choisir de réutiliser le même nom d’application dans chaque environnement.</span><span class="sxs-lookup"><span data-stu-id="665ce-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="665ce-145">Par exemple, un développeur peut avoir des applications nommées comme suit : *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, et ainsi de suite.</span><span class="sxs-lookup"><span data-stu-id="665ce-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="665ce-146">Pour l’application utilisée ici, chaque instance de l’application a un nom unique.</span><span class="sxs-lookup"><span data-stu-id="665ce-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="665ce-147">Les noms d’instance application utilisés sont *webfrontend1*, *webfrontend2* et *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="665ce-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="665ce-148">![Diagramme des piliers hybrides](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="665ce-148">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="665ce-149">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="665ce-150">Azure Stack Hub offre à votre environnement local l’agilité et l’innovation du cloud computing grâce au seul cloud hybride qui vous permette de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="665ce-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="665ce-151">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) pour la conception, le déploiement et le fonctionnement des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="665ce-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="665ce-152">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="665ce-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="665ce-153">Première partie : Créer une application géolocalisée</span><span class="sxs-lookup"><span data-stu-id="665ce-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="665ce-154">Dans cette partie, vous allez créer une application web.</span><span class="sxs-lookup"><span data-stu-id="665ce-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="665ce-155">Créer des applications web et publier.</span><span class="sxs-lookup"><span data-stu-id="665ce-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="665ce-156">Ajouter du code à Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="665ce-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="665ce-157">Pointez la génération de l’application sur plusieurs cibles du cloud.</span><span class="sxs-lookup"><span data-stu-id="665ce-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="665ce-158">Gérer et configurer le processus CD.</span><span class="sxs-lookup"><span data-stu-id="665ce-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="665ce-159">Prérequis</span><span class="sxs-lookup"><span data-stu-id="665ce-159">Prerequisites</span></span>

<span data-ttu-id="665ce-160">Un abonnement Azure et l’installation d’Azure Stack Hub sont requis.</span><span class="sxs-lookup"><span data-stu-id="665ce-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="665ce-161">Procédure de création d’une application géolocalisée</span><span class="sxs-lookup"><span data-stu-id="665ce-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="665ce-162">Obtenir un domaine personnalisé et configurer DNS</span><span class="sxs-lookup"><span data-stu-id="665ce-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="665ce-163">Mettez à jour le fichier de zone DNS pour le domaine.</span><span class="sxs-lookup"><span data-stu-id="665ce-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="665ce-164">Azure AD peut ensuite vérifier la propriété du nom de domaine personnalisé.</span><span class="sxs-lookup"><span data-stu-id="665ce-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="665ce-165">Utilisez [Azure DNS](/azure/dns/dns-getstarted-portal) pour les enregistrements DNS Azure/Microsoft 365/externes dans Azure, ou ajoutez l’entrée DNS à [un autre bureau d’enregistrement DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="665ce-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="665ce-166">Inscrivez un domaine personnalisé auprès d’un bureau d’enregistrement public.</span><span class="sxs-lookup"><span data-stu-id="665ce-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="665ce-167">Connectez-vous au Bureau d’enregistrement des noms de domaine pour le domaine.</span><span class="sxs-lookup"><span data-stu-id="665ce-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="665ce-168">Un administrateur approuvé devra peut-être effectuer les mises à jour DNS.</span><span class="sxs-lookup"><span data-stu-id="665ce-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="665ce-169">Mettez à jour le fichier de zone DNS du domaine en ajoutant l’entrée DNS fournie par Azure AD.</span><span class="sxs-lookup"><span data-stu-id="665ce-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="665ce-170">Elle ne modifie pas les comportements comme le routage du courrier ou l’hébergement web.</span><span class="sxs-lookup"><span data-stu-id="665ce-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="665ce-171">Créer des applications web et publier</span><span class="sxs-lookup"><span data-stu-id="665ce-171">Create web apps and publish</span></span>

<span data-ttu-id="665ce-172">Configurez une intégration/livraison continue (CI/CD) hybride pour déployer Web App sur Azure et Azure Stack Hub, et pour envoyer les modifications aux deux clouds.</span><span class="sxs-lookup"><span data-stu-id="665ce-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="665ce-173">Vous avez besoin d’Azure Stack Hub avec les images appropriées syndiquées pour s’exécuter (Windows Server et SQL), et App Service doit être déployé.</span><span class="sxs-lookup"><span data-stu-id="665ce-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="665ce-174">Pour plus d’informations, consultez [Conditions préalables au déploiement d’App Service sur Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="665ce-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="665ce-175">Ajouter du code à Azure Repos</span><span class="sxs-lookup"><span data-stu-id="665ce-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="665ce-176">Connectez-vous à Visual Studio avec un **compte ayant les droits de création de projet** sur Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="665ce-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="665ce-177">La CI/CD hybride peut s’appliquer tant au code d’application qu’au code d’infrastructure.</span><span class="sxs-lookup"><span data-stu-id="665ce-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="665ce-178">Utilisez les [modèles Azure Resource Manager](https://azure.microsoft.com/resources/templates/) pour les développements cloud privé et hébergé.</span><span class="sxs-lookup"><span data-stu-id="665ce-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Se connecter à un projet dans Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="665ce-180">**Clonez le référentiel** en créant et en ouvrant l’application web par défaut.</span><span class="sxs-lookup"><span data-stu-id="665ce-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Cloner un référentiel dans Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="665ce-182">Création d’un déploiement d’application web dans les deux clouds</span><span class="sxs-lookup"><span data-stu-id="665ce-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="665ce-183">Modifiez le fichier **WebApplication.csproj** : Sélectionnez `Runtimeidentifier`, puis ajoutez `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="665ce-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="665ce-184">(Consultez la documentation sur le [déploiement autonome](/dotnet/core/deploying/deploy-with-vs#simpleSelf).)</span><span class="sxs-lookup"><span data-stu-id="665ce-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Modifier un fichier de projet d’application web dans Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="665ce-186">**Archivez le code dans Azure Repos** avec Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="665ce-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="665ce-187">Vérifiez que le **code d’application** a été archivé dans Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="665ce-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="665ce-188">Créer la définition de build</span><span class="sxs-lookup"><span data-stu-id="665ce-188">Create the build definition</span></span>

1. <span data-ttu-id="665ce-189">**Connectez-vous à Azure Pipelines** pour vérifier la possibilité de créer des définitions de build.</span><span class="sxs-lookup"><span data-stu-id="665ce-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="665ce-190">Ajoutez le code `-r win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="665ce-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="665ce-191">Cet ajout est nécessaire pour déclencher un déploiement autonome avec .NET Core.</span><span class="sxs-lookup"><span data-stu-id="665ce-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Ajouter du code à la définition de build dans Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="665ce-193">**Exécutez la build**.</span><span class="sxs-lookup"><span data-stu-id="665ce-193">**Run the build**.</span></span> <span data-ttu-id="665ce-194">Le processus de [build de déploiement autonome](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publie des artefacts qui peuvent s’exécuter sur Azure et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="665ce-195">Utilisation d’un agent hébergé sur Azure</span><span class="sxs-lookup"><span data-stu-id="665ce-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="665ce-196">L’utilisation d’un agent hébergé dans Azure Pipelines est une option pratique pour créer et déployer des applications web.</span><span class="sxs-lookup"><span data-stu-id="665ce-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="665ce-197">La maintenance et les mises à niveau sont effectuées automatiquement par Microsoft Azure, ce qui permet de développer, de tester et de déployer sans interruption.</span><span class="sxs-lookup"><span data-stu-id="665ce-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="665ce-198">Gérer et configurer le processus CD</span><span class="sxs-lookup"><span data-stu-id="665ce-198">Manage and configure the CD process</span></span>

<span data-ttu-id="665ce-199">Azure DevOps Services fournit un pipeline hautement configurable et gérable pour des mises en production sur plusieurs environnements, comme des environnements de développement, de préproduction, d’assurance qualité et de production, avec des demandes d’approbation à des étapes spécifiques.</span><span class="sxs-lookup"><span data-stu-id="665ce-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="665ce-200">Créer une définition de mise en production</span><span class="sxs-lookup"><span data-stu-id="665ce-200">Create release definition</span></span>

1. <span data-ttu-id="665ce-201">Sélectionnez le bouton **plus** pour ajouter une nouvelle mise en production sous l’onglet **Mises en production** dans la section **Build et mise en production** d’Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="665ce-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Créer une définition de mise en production dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="665ce-203">Appliquez le modèle de déploiement Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="665ce-203">Apply the Azure App Service Deployment template.</span></span>

   ![Appliquer un modèle de déploiement Azure App Service dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="665ce-205">Sous **Ajouter un artefact**, ajoutez l’artefact pour l’application de build Azure Cloud.</span><span class="sxs-lookup"><span data-stu-id="665ce-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Ajouter un artefact à la build de cloud Azure dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="665ce-207">Sous l’onglet Pipeline, sélectionnez le lien **Phase, Tâche** de l’environnement, et définissez les valeurs de l’environnement cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Définir des valeurs d’environnement cloud Azure dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="665ce-209">Définissez le **nom d’environnement** et sélectionnez l’**abonnement Azure** pour le point de terminaison du Cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Sélectionner un abonnement Azure pour le point de terminaison du cloud Azure dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="665ce-211">Sous **Nom de l’App Service**, définissez le nom de service de l’application Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Définir un nom Azure App Service dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="665ce-213">Entrez VS2017 hébergé sous **File d’attente de l’agent** pour l’environnement hébergé dans le cloud Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Définir une file d’attente d’agents pour l’environnement hébergé dans le cloud Azure dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="665ce-215">Dans le menu Déployer Azure App Service, sélectionnez **le package ou le dossier** valide pour l’environnement.</span><span class="sxs-lookup"><span data-stu-id="665ce-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="665ce-216">Sélectionnez **OK** pour **l’emplacement du dossier**.</span><span class="sxs-lookup"><span data-stu-id="665ce-216">Select **OK** to **folder location**.</span></span>
  
      ![Sélectionner un package ou un dossier pour l’environnement Azure App Service dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Boîte de dialogue du sélecteur de dossiers 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="665ce-219">Enregistrez toutes les modifications et revenez au **pipeline de mises en production**.</span><span class="sxs-lookup"><span data-stu-id="665ce-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Enregistrer les modifications dans le pipeline de mise en production dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="665ce-221">Ajoutez un nouvel artefact en sélectionnant la build pour l’application Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Ajouter un nouvel artefact pour une application Azure Stack Hub dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="665ce-223">Ajoutez un environnement supplémentaire en appliquant le déploiement Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="665ce-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Ajouter un environnement à un déploiement Azure App Service dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="665ce-225">Nommez le nouvel environnement Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-225">Name the new environment Azure Stack Hub.</span></span>

    ![Nommer un environnement dans un déploiement Azure App Service dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="665ce-227">Recherchez l’environnement Azure Stack Hub sous l’onglet **Tâche**.</span><span class="sxs-lookup"><span data-stu-id="665ce-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Environnement Azure Stack Hub dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="665ce-229">Sélectionnez l’abonnement pour le point de terminaison Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Sélectionner l’abonnement pour le point de terminaison Azure Stack Hub dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="665ce-231">Définissez le nom de l’application web Azure Stack Hub en tant que nom App Service.</span><span class="sxs-lookup"><span data-stu-id="665ce-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Définir un nom d’application web Azure Stack Hub dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="665ce-233">Sélectionnez l’agent Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-233">Select the Azure Stack Hub agent.</span></span>

    ![Sélectionner l’agent Azure Stack Hub dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="665ce-235">Sous la section Déployer Azure App Service, sélectionnez **le package ou le dossier** valides pour l’environnement.</span><span class="sxs-lookup"><span data-stu-id="665ce-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="665ce-236">Sélectionnez **OK** pour l’emplacement du dossier.</span><span class="sxs-lookup"><span data-stu-id="665ce-236">Select **OK** to folder location.</span></span>

    ![Sélectionner un dossier pour un déploiement Azure App Service dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Boîte de dialogue du sélecteur de dossiers 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="665ce-239">Sous l’onglet Variable, ajoutez une variable nommée `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, définissez sa valeur sur **true** et sa portée sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Ajouter une variable à un déploiement Azure App Service dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="665ce-241">Sélectionnez l’icône du déclencheur de déploiement **Continu** dans les deux artefacts et activez le déclencheur de déploiement **Continu**.</span><span class="sxs-lookup"><span data-stu-id="665ce-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Sélectionner un déclencheur de déploiement continu dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="665ce-243">Sélectionnez l’icône des conditions **Prédéploiement** dans l’environnement Azure Stack Hub et définissez le déclencheur sur **Après la mise en production**.</span><span class="sxs-lookup"><span data-stu-id="665ce-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Sélectionner des conditions préalables au déploiement dans Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="665ce-245">Enregistrez toutes les modifications.</span><span class="sxs-lookup"><span data-stu-id="665ce-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="665ce-246">Certains paramètres des tâches peuvent avoir été automatiquement définis en tant que [variables d’environnement](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) lors de la création d’une définition de mise en production à partir d’un modèle.</span><span class="sxs-lookup"><span data-stu-id="665ce-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="665ce-247">Ces paramètres ne peuvent pas être modifiés dans les paramètres de la tâche. Au lieu de cela, l’élément d’environnement parent doit être sélectionné pour modifier ces paramètres.</span><span class="sxs-lookup"><span data-stu-id="665ce-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="665ce-248">Deuxième partie : Mettre à jour les options de l’application web</span><span class="sxs-lookup"><span data-stu-id="665ce-248">Part 2: Update web app options</span></span>

<span data-ttu-id="665ce-249">[Azure App Service](/azure/app-service/overview) offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques.</span><span class="sxs-lookup"><span data-stu-id="665ce-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="665ce-251">Mappez un nom DNS personnalisé existant à Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="665ce-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="665ce-252">Utilisez un **enregistrement CNAME** ou un **enregistrement A** pour mapper un nom DNS personnalisé à App Service.</span><span class="sxs-lookup"><span data-stu-id="665ce-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="665ce-253">Mapper un nom DNS personnalisé existant à des applications web Azure</span><span class="sxs-lookup"><span data-stu-id="665ce-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="665ce-254">Utilisez un enregistrement CNAME pour tous les noms DNS personnalisés, à l’exception d’un domaine racine (par exemple, northwind.com).</span><span class="sxs-lookup"><span data-stu-id="665ce-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="665ce-255">Pour migrer un site actif et son nom de domaine DNS vers App Service, voir [Migrer un nom DNS actif vers Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="665ce-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="665ce-256">Prérequis</span><span class="sxs-lookup"><span data-stu-id="665ce-256">Prerequisites</span></span>

<span data-ttu-id="665ce-257">Pour suivre cette solution :</span><span class="sxs-lookup"><span data-stu-id="665ce-257">To complete this solution:</span></span>

- <span data-ttu-id="665ce-258">[Créez une application App Service](/azure/app-service/), ou utilisez une application créée pour une autre solution.</span><span class="sxs-lookup"><span data-stu-id="665ce-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="665ce-259">Achetez un nom de domaine et fournissez un accès au registre DNS au fournisseur de domaine.</span><span class="sxs-lookup"><span data-stu-id="665ce-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="665ce-260">Mettez à jour le fichier de zone DNS pour le domaine.</span><span class="sxs-lookup"><span data-stu-id="665ce-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="665ce-261">Azure AD vérifie la propriété du nom de domaine personnalisé.</span><span class="sxs-lookup"><span data-stu-id="665ce-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="665ce-262">Utilisez [Azure DNS](/azure/dns/dns-getstarted-portal) pour les enregistrements DNS Azure/Microsoft 365/externes dans Azure, ou ajoutez l’entrée DNS à [un autre bureau d’enregistrement DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="665ce-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="665ce-263">Inscrivez un domaine personnalisé auprès d’un bureau d’enregistrement public.</span><span class="sxs-lookup"><span data-stu-id="665ce-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="665ce-264">Connectez-vous au Bureau d’enregistrement des noms de domaine pour le domaine.</span><span class="sxs-lookup"><span data-stu-id="665ce-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="665ce-265">(Il se peut qu’un administrateur approuvé doive effectuer les mises à jour DNS.)</span><span class="sxs-lookup"><span data-stu-id="665ce-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="665ce-266">Mettez à jour le fichier de zone DNS du domaine en ajoutant l’entrée DNS fournie par Azure AD.</span><span class="sxs-lookup"><span data-stu-id="665ce-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="665ce-267">Par exemple, pour ajouter des entrées DNS pour northwindcloud.com et www\.northwindcloud.com, configurez les paramètres DNS pour le domaine racine northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="665ce-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="665ce-268">Vous pouvez acheter un nom de domaine à l’aide du [portail Azure](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="665ce-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="665ce-269">Pour mapper un nom DNS personnalisé à une application web, le [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) de l’application web doit être un niveau payant (**Partagé**, **De base**, **Standard** ou **Premium**).</span><span class="sxs-lookup"><span data-stu-id="665ce-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="665ce-270">Créer et mapper des enregistrements CNAME et A</span><span class="sxs-lookup"><span data-stu-id="665ce-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="665ce-271">Accès aux enregistrements DNS avec le fournisseur de domaine</span><span class="sxs-lookup"><span data-stu-id="665ce-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="665ce-272">Utilisez Azure DNS pour configurer un nom DNS personnalisé pour les applications web Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="665ce-273">Pour plus d’informations, consultez [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain) (Utiliser DNS Azure pour fournir des paramètres de domaine personnalisé pour un service Azure).</span><span class="sxs-lookup"><span data-stu-id="665ce-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="665ce-274">Connectez-vous au site web du fournisseur principal.</span><span class="sxs-lookup"><span data-stu-id="665ce-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="665ce-275">Trouvez la page de gestion des enregistrements DNS.</span><span class="sxs-lookup"><span data-stu-id="665ce-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="665ce-276">Chaque fournisseur de domaine a sa propre interface d’enregistrements DNS.</span><span class="sxs-lookup"><span data-stu-id="665ce-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="665ce-277">Recherchez les zones du site qui portent les mentions **Nom de domaine**, **DNS** ou **Gestion du nom de serveur**.</span><span class="sxs-lookup"><span data-stu-id="665ce-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="665ce-278">Vous pouvez consulter la page d’enregistrements DNS dans **Mes domaines**.</span><span class="sxs-lookup"><span data-stu-id="665ce-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="665ce-279">Trouvez le lien nommé **Fichier de zone**, **Enregistrements DNS** ou **Configuration avancée**.</span><span class="sxs-lookup"><span data-stu-id="665ce-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="665ce-280">La capture d’écran suivante est un exemple d’une page d’enregistrements DNS :</span><span class="sxs-lookup"><span data-stu-id="665ce-280">The following screenshot is an example of a DNS records page:</span></span>

![Exemple de page d’enregistrements DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="665ce-282">Dans le bureau d’enregistrement de noms de domaine, sélectionnez **Ajouter ou Créer** pour créer un enregistrement.</span><span class="sxs-lookup"><span data-stu-id="665ce-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="665ce-283">Certains fournisseurs ont différents liens pour ajouter divers types d’enregistrements.</span><span class="sxs-lookup"><span data-stu-id="665ce-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="665ce-284">Consultez la documentation du fournisseur.</span><span class="sxs-lookup"><span data-stu-id="665ce-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="665ce-285">Ajoutez un enregistrement CNAME pour mapper un sous-domaine au nom d’hôte par défaut de l’application.</span><span class="sxs-lookup"><span data-stu-id="665ce-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="665ce-286">Pour l’exemple de domaine www\.northwindcloud.com, ajoutez un enregistrement CNAME qui mappe le nom à `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="665ce-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="665ce-287">Après avoir ajouté l’enregistrement CNAME, la page d’enregistrements DNS ressemble à l’exemple suivant :</span><span class="sxs-lookup"><span data-stu-id="665ce-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Navigation au sein du portail pour accéder à l’application Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="665ce-289">Activer le mappage d’enregistrement CNAME dans Azure</span><span class="sxs-lookup"><span data-stu-id="665ce-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="665ce-290">Dans un nouvel onglet, connectez-vous au portail Azure.</span><span class="sxs-lookup"><span data-stu-id="665ce-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="665ce-291">Accédez à App Services.</span><span class="sxs-lookup"><span data-stu-id="665ce-291">Go to App Services.</span></span>

3. <span data-ttu-id="665ce-292">Sélectionnez l’application web.</span><span class="sxs-lookup"><span data-stu-id="665ce-292">Select web app.</span></span>

4. <span data-ttu-id="665ce-293">Dans le volet de navigation gauche de la page d’application du portail Azure, sélectionnez **Domaines personnalisés**.</span><span class="sxs-lookup"><span data-stu-id="665ce-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="665ce-294">Cliquez sur l’icône **+** en regard de **Ajouter un nom d’hôte**.</span><span class="sxs-lookup"><span data-stu-id="665ce-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="665ce-295">Tapez le nom de domaine complet, par exemple `www.northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="665ce-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="665ce-296">Sélectionnez **Valider**.</span><span class="sxs-lookup"><span data-stu-id="665ce-296">Select **Validate**.</span></span>

8. <span data-ttu-id="665ce-297">Si cela est indiqué, ajoutez des enregistrements supplémentaires d’autres types (`A` ou `TXT`) aux enregistrements DNS du bureau d’enregistrement de noms de domaine.</span><span class="sxs-lookup"><span data-stu-id="665ce-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="665ce-298">Azure fournit les valeurs et les types de ces enregistrements :</span><span class="sxs-lookup"><span data-stu-id="665ce-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="665ce-299">a.</span><span class="sxs-lookup"><span data-stu-id="665ce-299">a.</span></span>  <span data-ttu-id="665ce-300">Un enregistrement **A** pour effectuer un mappage vers l’adresse IP de l’application.</span><span class="sxs-lookup"><span data-stu-id="665ce-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="665ce-301">b.</span><span class="sxs-lookup"><span data-stu-id="665ce-301">b.</span></span>  <span data-ttu-id="665ce-302">Un enregistrement **TXT** pour effectuer un mappage vers le nom d’hôte par défaut de l’application `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="665ce-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="665ce-303">App Service utilise cet enregistrement uniquement au moment de la configuration, pour vérifier la propriété du domaine personnalisé.</span><span class="sxs-lookup"><span data-stu-id="665ce-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="665ce-304">Après vérification, supprimez l’enregistrement TXT.</span><span class="sxs-lookup"><span data-stu-id="665ce-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="665ce-305">Effectuez cette tâche dans l’onglet du bureau d’enregistrement de domaine et validez à nouveau pour que le bouton **Ajouter un nom d’hôte** soit activé.</span><span class="sxs-lookup"><span data-stu-id="665ce-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="665ce-306">Assurez-vous que **Type d’enregistrement du nom d’hôte** est défini sur **CNAME** (www.exemple.com ou tout sous-domaine).</span><span class="sxs-lookup"><span data-stu-id="665ce-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="665ce-307">Sélectionnez **Ajouter un nom d’hôte**.</span><span class="sxs-lookup"><span data-stu-id="665ce-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="665ce-308">Tapez le nom de domaine complet, par exemple `northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="665ce-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="665ce-309">Sélectionnez **Valider**.</span><span class="sxs-lookup"><span data-stu-id="665ce-309">Select **Validate**.</span></span> <span data-ttu-id="665ce-310">**Ajouter** est activé.</span><span class="sxs-lookup"><span data-stu-id="665ce-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="665ce-311">Assurez-vous que **Type d’enregistrement du nom d’hôte** est défini sur **Enregistrement A** (exemple.com).</span><span class="sxs-lookup"><span data-stu-id="665ce-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="665ce-312">**Ajoutez un nom d’hôte**.</span><span class="sxs-lookup"><span data-stu-id="665ce-312">**Add hostname**.</span></span>

    <span data-ttu-id="665ce-313">Un certain temps peut être nécessaire pour que les nouveaux noms d’hôte soient reflétés sur la page **Domaines personnalisés** de votre application.</span><span class="sxs-lookup"><span data-stu-id="665ce-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="665ce-314">Essayez d’actualiser le navigateur pour mettre à jour les données.</span><span class="sxs-lookup"><span data-stu-id="665ce-314">Try refreshing the browser to update the data.</span></span>
  
    ![Domaines personnalisés](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="665ce-316">En cas d’erreur, une notification d’erreur de vérification s’affiche en bas de la page.</span><span class="sxs-lookup"><span data-stu-id="665ce-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Erreur de vérification du domaine](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="665ce-318">Les étapes ci-dessus peuvent être répétées pour mapper un domaine contenant un caractère générique (\*.northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="665ce-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="665ce-319">Cela vous permet d’ajouter n’importe quel sous-domaine à cet App Service sans avoir à créer un enregistrement CNAME distinct pour chacun.</span><span class="sxs-lookup"><span data-stu-id="665ce-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="665ce-320">Suivez les instructions du bureau d’enregistrement pour configurer ce paramètre.</span><span class="sxs-lookup"><span data-stu-id="665ce-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="665ce-321">Test dans un navigateur</span><span class="sxs-lookup"><span data-stu-id="665ce-321">Test in a browser</span></span>

<span data-ttu-id="665ce-322">Dans votre navigateur, accédez aux noms DNS configurés précédemment (par exemple, `northwindcloud.com`ou `www.northwindcloud.com`).</span><span class="sxs-lookup"><span data-stu-id="665ce-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="665ce-323">Troisième partie : Lier un certificat SSL personnalisé</span><span class="sxs-lookup"><span data-stu-id="665ce-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="665ce-324">Dans cette partie, nous allons :</span><span class="sxs-lookup"><span data-stu-id="665ce-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="665ce-325">lier le certificat SSL personnalisé à App Service ;</span><span class="sxs-lookup"><span data-stu-id="665ce-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="665ce-326">appliquer le protocole HTTPS pour l’application ;</span><span class="sxs-lookup"><span data-stu-id="665ce-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="665ce-327">automatiser une liaison de certificat SSL avec des scripts.</span><span class="sxs-lookup"><span data-stu-id="665ce-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="665ce-328">Si nécessaire, obtenez un certificat SSL dans le portail Azure et liez-le à l’application web.</span><span class="sxs-lookup"><span data-stu-id="665ce-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="665ce-329">Pour plus d’informations, voir le [didacticiel sur App Service Certificates](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="665ce-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="665ce-330">Prérequis</span><span class="sxs-lookup"><span data-stu-id="665ce-330">Prerequisites</span></span>

<span data-ttu-id="665ce-331">Pour effectuer cette solution :</span><span class="sxs-lookup"><span data-stu-id="665ce-331">To complete this  solution:</span></span>

- <span data-ttu-id="665ce-332">[Créez une application App Service](/azure/app-service/).</span><span class="sxs-lookup"><span data-stu-id="665ce-332">[Create an App Service app.](/azure/app-service/)</span></span>
- [<span data-ttu-id="665ce-333">Mappez un nom DNS personnalisé à votre application web.</span><span class="sxs-lookup"><span data-stu-id="665ce-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="665ce-334">Acquérez un certificat SSL auprès d’une autorité de certification approuvée et utilisez la clé pour signer la demande.</span><span class="sxs-lookup"><span data-stu-id="665ce-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="665ce-335">Conditions requises pour le certificat SSL</span><span class="sxs-lookup"><span data-stu-id="665ce-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="665ce-336">Pour l’utiliser dans App Service, un certificat doit remplir toutes les conditions suivantes :</span><span class="sxs-lookup"><span data-stu-id="665ce-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="665ce-337">être signé par une autorité de certification approuvée ;</span><span class="sxs-lookup"><span data-stu-id="665ce-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="665ce-338">être exporté sous la forme d’un fichier PFX protégé par mot de passe ;</span><span class="sxs-lookup"><span data-stu-id="665ce-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="665ce-339">contenir une clé privée d’une longueur minimale de 2048 bits ;</span><span class="sxs-lookup"><span data-stu-id="665ce-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="665ce-340">contenir tous les certificats intermédiaires dans la chaîne de certificats.</span><span class="sxs-lookup"><span data-stu-id="665ce-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="665ce-341">Les **certificats de chiffrement à courbe elliptique (ECC)** sont compatibles avec App Service, mais ne sont pas abordés dans ce guide.</span><span class="sxs-lookup"><span data-stu-id="665ce-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="665ce-342">Consultez une autorité de certification pour bénéficier d’une assistance lors de la création de certificats ECC.</span><span class="sxs-lookup"><span data-stu-id="665ce-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="665ce-343">Préparation de l’application web</span><span class="sxs-lookup"><span data-stu-id="665ce-343">Prepare the web app</span></span>

<span data-ttu-id="665ce-344">Pour lier un certificat SSL personnalisé à votre application web, le [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) doit se trouver dans le niveau **De base**, **Standard** ou **Premium**.</span><span class="sxs-lookup"><span data-stu-id="665ce-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="665ce-345">Connexion à Azure</span><span class="sxs-lookup"><span data-stu-id="665ce-345">Sign in to Azure</span></span>

1. <span data-ttu-id="665ce-346">Ouvrez le [portail Azure](https://portal.azure.com/) et accédez à l’application web.</span><span class="sxs-lookup"><span data-stu-id="665ce-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="665ce-347">Dans le menu de gauche, sélectionnez **App Services**, puis le nom de l’application web.</span><span class="sxs-lookup"><span data-stu-id="665ce-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Sélectionner l’application web dans le portail Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="665ce-349">Vérification du niveau tarifaire</span><span class="sxs-lookup"><span data-stu-id="665ce-349">Check the pricing tier</span></span>

1. <span data-ttu-id="665ce-350">Dans le volet de navigation de gauche de la page d’application web, accédez à la section **Paramètres** et sélectionnez **Monter en puissance (plan App Service)** .</span><span class="sxs-lookup"><span data-stu-id="665ce-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menu Scale-up dans l’application web](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="665ce-352">Vérifiez que l’application web n’est pas de niveau **Gratuit** ou **Partagé**.</span><span class="sxs-lookup"><span data-stu-id="665ce-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="665ce-353">Le niveau actuel de l’application web est encadré d’un rectangle bleu foncé.</span><span class="sxs-lookup"><span data-stu-id="665ce-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Vérifier le niveau tarifaire dans l’application web](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="665ce-355">Le certificat SSL personnalisé n’est pas pris en charge aux niveaux **Gratuit** et **Partagé**.</span><span class="sxs-lookup"><span data-stu-id="665ce-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="665ce-356">Pour contourner ce problème, suivez la procédure décrite dans la section suivante ou dans la page **Choisir votre niveau de tarification**, puis passez à [Charger et lier votre certificat SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="665ce-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="665ce-357">Évolution de votre plan App Service</span><span class="sxs-lookup"><span data-stu-id="665ce-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="665ce-358">Sélectionnez l’un des niveaux **De base**, **Standard** ou **Premium**.</span><span class="sxs-lookup"><span data-stu-id="665ce-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="665ce-359">Sélectionnez **Sélectionner**.</span><span class="sxs-lookup"><span data-stu-id="665ce-359">Select **Select**.</span></span>

![Choisir un niveau tarifaire pour votre application web](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="665ce-361">L’opération de mise à l’échelle est terminée lorsque la notification s’affiche.</span><span class="sxs-lookup"><span data-stu-id="665ce-361">The scale operation is complete when notification is displayed.</span></span>

![Notification de montée en puissance](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="665ce-363">Liaison de votre certificat SSL et fusion des certificats intermédiaires</span><span class="sxs-lookup"><span data-stu-id="665ce-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="665ce-364">Fusionnez plusieurs certificats dans la chaîne.</span><span class="sxs-lookup"><span data-stu-id="665ce-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="665ce-365">**Ouvrez chaque certificat** reçu dans un éditeur de texte.</span><span class="sxs-lookup"><span data-stu-id="665ce-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="665ce-366">Créez un fichier pour le certificat fusionné appelé *mergedcertificate.crt*.</span><span class="sxs-lookup"><span data-stu-id="665ce-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="665ce-367">Dans un éditeur de texte, copiez le contenu de chaque certificat dans ce fichier.</span><span class="sxs-lookup"><span data-stu-id="665ce-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="665ce-368">L’ordre de vos certificats devrait suivre l’ordre dans la chaîne d’approbation, commençant par votre certificat et finissant par le certificat racine.</span><span class="sxs-lookup"><span data-stu-id="665ce-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="665ce-369">Cela ressemble à l’exemple suivant :</span><span class="sxs-lookup"><span data-stu-id="665ce-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="665ce-370">Exportation du certificat vers PFX</span><span class="sxs-lookup"><span data-stu-id="665ce-370">Export certificate to PFX</span></span>

<span data-ttu-id="665ce-371">Exportez le certificat SSL fusionné avec la clé privée générée par le certificat.</span><span class="sxs-lookup"><span data-stu-id="665ce-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="665ce-372">Un fichier de clé privée est créé via OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="665ce-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="665ce-373">Pour exporter le certificat vers PFX, exécutez la commande suivante, en remplaçant les espaces réservés `<private-key-file>` et `<merged-certificate-file>` par le chemin de clé privée et le fichier de certificat fusionné :</span><span class="sxs-lookup"><span data-stu-id="665ce-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="665ce-374">Lorsque vous y êtes invité, définissez un mot de passe d’exportation pour charger votre certificat SSL dans App Service ultérieurement.</span><span class="sxs-lookup"><span data-stu-id="665ce-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="665ce-375">Lorsque vous utilisez IIS ou **Certreq.exe** pour générer la demande de certificat, installez le certificat sur un ordinateur local, puis [exportez le certificat au format PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="665ce-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="665ce-376">Charger le certificat SSL</span><span class="sxs-lookup"><span data-stu-id="665ce-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="665ce-377">Dans le volet de navigation gauche, sélectionnez **Paramètres SSL**.</span><span class="sxs-lookup"><span data-stu-id="665ce-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="665ce-378">Sélectionnez **Charger un certificat**.</span><span class="sxs-lookup"><span data-stu-id="665ce-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="665ce-379">Dans **Fichier de certificat PFX**, sélectionnez le fichier PFX.</span><span class="sxs-lookup"><span data-stu-id="665ce-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="665ce-380">Dans **Mot de passe du certificat**, tapez le mot de passe créé lors de l’exportation du fichier PFX.</span><span class="sxs-lookup"><span data-stu-id="665ce-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="665ce-381">Sélectionnez **Télécharger**.</span><span class="sxs-lookup"><span data-stu-id="665ce-381">Select **Upload**.</span></span>

    ![Charger un certificat SSL](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="665ce-383">Lorsque App Service finit de charger le certificat, celui-ci apparaît dans la page **Paramètres SSL**.</span><span class="sxs-lookup"><span data-stu-id="665ce-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Paramètres SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="665ce-385">Liaison de votre certificat SSL</span><span class="sxs-lookup"><span data-stu-id="665ce-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="665ce-386">Dans la section **Liaisons SSL**, sélectionnez **Ajouter une liaison**.</span><span class="sxs-lookup"><span data-stu-id="665ce-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="665ce-387">Si le certificat a été chargé, mais n’apparaît pas dans les noms de domaine dans la liste déroulante **Nom d’hôte**, essayez d’actualiser la page du navigateur.</span><span class="sxs-lookup"><span data-stu-id="665ce-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="665ce-388">Dans la page **Ajouter une liaison SSL**, utilisez les listes déroulantes pour sélectionner le nom de domaine à sécuriser et le certificat à utiliser.</span><span class="sxs-lookup"><span data-stu-id="665ce-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="665ce-389">Dans **Type SSL**, choisissez d’utiliser [**l’indication du nom du serveur (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) ou le protocole SSL basé sur IP.</span><span class="sxs-lookup"><span data-stu-id="665ce-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="665ce-390">**SSL basé sur SNI** : plusieurs liaisons SSL basées sur SNI peuvent être ajoutées.</span><span class="sxs-lookup"><span data-stu-id="665ce-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="665ce-391">Cette option permet de sécuriser plusieurs domaines sur la même adresse IP avec plusieurs certificats SSL.</span><span class="sxs-lookup"><span data-stu-id="665ce-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="665ce-392">La plupart des navigateurs actuels (y compris Internet Explorer, Chrome, Firefox et Opera) prennent en charge SNI (plus d’informations sur la prise en charge des navigateurs dans [Indication du nom du serveur](https://wikipedia.org/wiki/Server_Name_Indication)).</span><span class="sxs-lookup"><span data-stu-id="665ce-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="665ce-393">**SSL basé sur IP** : une seule liaison SSL basée sur IP peut être ajoutée.</span><span class="sxs-lookup"><span data-stu-id="665ce-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="665ce-394">Cette option permet de sécuriser une adresse IP publique dédiée avec un seul certificat SSL.</span><span class="sxs-lookup"><span data-stu-id="665ce-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="665ce-395">Pour sécuriser plusieurs domaines, vous devez tous les sécuriser en utilisant le même certificat SSL.</span><span class="sxs-lookup"><span data-stu-id="665ce-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="665ce-396">SSL basé sur IP est l’option habituelle pour la liaison SSL.</span><span class="sxs-lookup"><span data-stu-id="665ce-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="665ce-397">Sélectionnez **Ajouter une liaison**.</span><span class="sxs-lookup"><span data-stu-id="665ce-397">Select **Add Binding**.</span></span>

    ![Ajouter une liaison SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="665ce-399">Lorsque App Service finit de charger votre certificat, celui-ci apparaît dans les sections **Liaisons SSL**.</span><span class="sxs-lookup"><span data-stu-id="665ce-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Le chargement des liaisons SSL est terminé](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="665ce-401">Remappage d’un enregistrement pour IP SSL</span><span class="sxs-lookup"><span data-stu-id="665ce-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="665ce-402">Si vous n’utilisez pas SSL basé sur IP dans l’application web, passez à [Tester HTTPS pour votre domaine personnalisé](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="665ce-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="665ce-403">Par défaut, l’application web utilise une adresse IP publique partagée.</span><span class="sxs-lookup"><span data-stu-id="665ce-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="665ce-404">Lorsque le certificat est lié avec SSL basé sur IP, App Service crée une adresse IP dédiée pour l’application web.</span><span class="sxs-lookup"><span data-stu-id="665ce-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="665ce-405">Lorsqu’un enregistrement A est mappé sur l’application web, le registre de domaine doit être mis à jour avec l’adresse IP dédiée.</span><span class="sxs-lookup"><span data-stu-id="665ce-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="665ce-406">La page **Domaine personnalisé** est mise à jour avec la nouvelle adresse IP dédiée.</span><span class="sxs-lookup"><span data-stu-id="665ce-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="665ce-407">Copiez cette [adresse IP](/azure/app-service/app-service-web-tutorial-custom-domain), puis remappez [l’enregistrement A](/azure/app-service/app-service-web-tutorial-custom-domain) sur cette nouvelle adresse IP.</span><span class="sxs-lookup"><span data-stu-id="665ce-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="665ce-408">Test du protocole HTTPS</span><span class="sxs-lookup"><span data-stu-id="665ce-408">Test HTTPS</span></span>

<span data-ttu-id="665ce-409">Dans différents navigateurs, accédez à `https://<your.custom.domain>` pour vérifier que l’application web est fournie.</span><span class="sxs-lookup"><span data-stu-id="665ce-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![accéder à l’application web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="665ce-411">Les erreurs de validation de certificat éventuelles peuvent être dues à un certificat auto-signé ou à des certificats intermédiaires mis de côté lors de l’exportation vers le fichier PFX.</span><span class="sxs-lookup"><span data-stu-id="665ce-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="665ce-412">Appliquer le protocole HTTPS</span><span class="sxs-lookup"><span data-stu-id="665ce-412">Enforce HTTPS</span></span>

<span data-ttu-id="665ce-413">Par défaut, chacun peut accéder à l’application web avec HTTP.</span><span class="sxs-lookup"><span data-stu-id="665ce-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="665ce-414">Vous pouvez rediriger toutes les requêtes HTTP vers le port HTTPS.</span><span class="sxs-lookup"><span data-stu-id="665ce-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="665ce-415">Dans la page d’application web, sélectionnez **Paramètres SSL**.</span><span class="sxs-lookup"><span data-stu-id="665ce-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="665ce-416">Ensuite, dans **HTTPS uniquement**, sélectionnez **Activer**.</span><span class="sxs-lookup"><span data-stu-id="665ce-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Appliquer le protocole HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="665ce-418">Lorsque l’opération est terminée, accédez à l’une des URL HTTP pointant vers l’application.</span><span class="sxs-lookup"><span data-stu-id="665ce-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="665ce-419">Par exemple :</span><span class="sxs-lookup"><span data-stu-id="665ce-419">For example:</span></span>

- <span data-ttu-id="665ce-420"> https://<nom_application>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="665ce-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="665ce-421">Appliquer le protocole TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="665ce-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="665ce-422">L’application autorise [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 par défaut, qui n’est plus considéré comme sécurisé par les normes du secteur telles que [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard).</span><span class="sxs-lookup"><span data-stu-id="665ce-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="665ce-423">Pour appliquer des versions ultérieures de TLS, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="665ce-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="665ce-424">Dans le volet de navigation gauche de la page d’application web, sélectionnez **Paramètres SSL**.</span><span class="sxs-lookup"><span data-stu-id="665ce-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="665ce-425">Dans **Version TLS**, sélectionnez la version minimale de TLS.</span><span class="sxs-lookup"><span data-stu-id="665ce-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Appliquer le protocole TLS 1.1 ou 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="665ce-427">Créer un profil Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="665ce-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="665ce-428">Sélectionnez **Créer une ressource** > **Mise en réseau** > **Profil Traffic Manager** > **Créer**.</span><span class="sxs-lookup"><span data-stu-id="665ce-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="665ce-429">Dans **Créer un profil Traffic Manager**, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="665ce-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="665ce-430">Sous **Nom**, entrez un nom pour le profil.</span><span class="sxs-lookup"><span data-stu-id="665ce-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="665ce-431">Ce nom doit être unique au sein de la zone trafficmanager.net et produit le nom DNS trafficmanager.net qui est utilisé pour accéder au profil Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="665ce-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="665ce-432">Sous **Méthode de routage**, sélectionnez la **méthode de routage géographique**.</span><span class="sxs-lookup"><span data-stu-id="665ce-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="665ce-433">Sous **Abonnement**, sélectionnez l’abonnement sous lequel créer ce profil.</span><span class="sxs-lookup"><span data-stu-id="665ce-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="665ce-434">Sous **Groupe de ressources**, créez un groupe de ressources où placer ce profil.</span><span class="sxs-lookup"><span data-stu-id="665ce-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="665ce-435">Sous **Emplacement du groupe de ressources**, sélectionnez l’emplacement du groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="665ce-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="665ce-436">Ce paramètre fait référence à l’emplacement du groupe de ressources et n’a pas d’impact sur le profil Traffic Manager déployé globalement.</span><span class="sxs-lookup"><span data-stu-id="665ce-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="665ce-437">Sélectionnez **Create** (Créer).</span><span class="sxs-lookup"><span data-stu-id="665ce-437">Select **Create**.</span></span>

    7. <span data-ttu-id="665ce-438">Lorsque le déploiement global du profil Traffic Manager est terminé, il est répertorié comme l’une des ressources dans le groupe de ressources concerné.</span><span class="sxs-lookup"><span data-stu-id="665ce-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Groupes de ressources dans le panneau Créer un profil Traffic Manager](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="665ce-440">Ajouter des points de terminaison Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="665ce-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="665ce-441">Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** créé dans la section précédente, puis sélectionnez le profil Traffic Manager dans les résultats affichés.</span><span class="sxs-lookup"><span data-stu-id="665ce-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="665ce-442">Dans **Profil Traffic Manager**, à la section **Paramètres**, sélectionnez **Points de terminaison**.</span><span class="sxs-lookup"><span data-stu-id="665ce-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="665ce-443">Sélectionnez **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="665ce-443">Select **Add**.</span></span>

4. <span data-ttu-id="665ce-444">Ajout du point de terminaison Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="665ce-445">Pour **Type**, sélectionnez **Point de terminaison externe**.</span><span class="sxs-lookup"><span data-stu-id="665ce-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="665ce-446">Fournissez un **nom** pour ce point de terminaison, dans l’idéal, le nom du déploiement Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="665ce-447">Pour le nom de domaine complet (**FQDN**), utilisez l’URL externe de l’application web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="665ce-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="665ce-448">Sous Géocartographie, sélectionnez la région ou le continent où se trouve la ressource.</span><span class="sxs-lookup"><span data-stu-id="665ce-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="665ce-449">Par exemple, **Europe**.</span><span class="sxs-lookup"><span data-stu-id="665ce-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="665ce-450">Dans la liste déroulante Pays/Région qui s’affiche, sélectionnez le pays qui s’applique à ce point de terminaison.</span><span class="sxs-lookup"><span data-stu-id="665ce-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="665ce-451">Par exemple, **Allemagne**.</span><span class="sxs-lookup"><span data-stu-id="665ce-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="665ce-452">Vérifiez que la case **Ajouter comme désactivé** est désélectionnée.</span><span class="sxs-lookup"><span data-stu-id="665ce-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="665ce-453">Sélectionnez **OK**.</span><span class="sxs-lookup"><span data-stu-id="665ce-453">Select **OK**.</span></span>

12. <span data-ttu-id="665ce-454">Ajout du point de terminaison Azure :</span><span class="sxs-lookup"><span data-stu-id="665ce-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="665ce-455">Sous **Type**, sélectionnez **Point de terminaison Azure**.</span><span class="sxs-lookup"><span data-stu-id="665ce-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="665ce-456">Entrez un **Nom** pour le point de terminaison.</span><span class="sxs-lookup"><span data-stu-id="665ce-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="665ce-457">Sous **Type de ressource cible**, sélectionnez **App Service**.</span><span class="sxs-lookup"><span data-stu-id="665ce-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="665ce-458">Sous **Ressource cible**, sélectionnez **Choisir un service d’application** pour afficher la liste des applications Web sous le même abonnement.</span><span class="sxs-lookup"><span data-stu-id="665ce-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="665ce-459">Dans **Ressources**, sélectionnez le service d’application utilisé en tant que premier point de terminaison.</span><span class="sxs-lookup"><span data-stu-id="665ce-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="665ce-460">Sous Géocartographie, sélectionnez la région ou le continent où se trouve la ressource.</span><span class="sxs-lookup"><span data-stu-id="665ce-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="665ce-461">Par exemple, **Amérique du Nord/Amérique Centrale/Antilles**.</span><span class="sxs-lookup"><span data-stu-id="665ce-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="665ce-462">Dans la liste déroulante Pays/Région qui s’affiche, laissez ce champ vide pour sélectionner le regroupement régional complet au-dessus.</span><span class="sxs-lookup"><span data-stu-id="665ce-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="665ce-463">Vérifiez que la case **Ajouter comme désactivé** est désélectionnée.</span><span class="sxs-lookup"><span data-stu-id="665ce-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="665ce-464">Sélectionnez **OK**.</span><span class="sxs-lookup"><span data-stu-id="665ce-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="665ce-465">Créez au moins un point de terminaison en choisissant la portée géographique Tous (Monde) comme point de terminaison par défaut de la ressource.</span><span class="sxs-lookup"><span data-stu-id="665ce-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="665ce-466">Une fois l’ajout des deux points de terminaison terminé, ceux-ci s’affichent dans **Profil Traffic Manager** avec leur état de surveillance défini sur **En ligne**.</span><span class="sxs-lookup"><span data-stu-id="665ce-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![État du point de terminaison de profil Traffic Manager](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="665ce-468">Les multinationales s’appuient sur les fonctionnalités de géodistribution Azure</span><span class="sxs-lookup"><span data-stu-id="665ce-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="665ce-469">En dirigeant le trafic de données à l’aide d’Azure Traffic Manager et de points de terminaison spécifiquement répartis à travers le monde, les multinationales peuvent se conformer aux réglementations régionales et assurer la conformité et la protection des données, indispensables au succès des sites d’activité locaux et distants.</span><span class="sxs-lookup"><span data-stu-id="665ce-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="665ce-470">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="665ce-470">Next steps</span></span>

- <span data-ttu-id="665ce-471">Pour plus d’informations sur les modèles Azure Cloud, consultez [Modèles de conception cloud](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="665ce-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
