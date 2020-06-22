---
title: Déployer une solution de détection des pas basée sur l’intelligence artificielle dans Azure et Azure Stack Hub
description: Découvrez comment déployer une solution de détection des pas basée sur l’intelligence artificielle afin d’analyser la fréquentation des magasins de détail avec Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910326"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="b57cd-103">Déployer une solution de détection des pas basé sur l’intelligence artificielle à l'aide d'Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="b57cd-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="b57cd-104">Cet article explique comment déployer une solution basée sur l’intelligence artificielle qui génère des insights à partir d’actions du monde réel en utilisant Azure, Azure Stack Hub et le kit de développement IA Custom Vision.</span><span class="sxs-lookup"><span data-stu-id="b57cd-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="b57cd-105">Dans cette solution, vous allez apprendre à :</span><span class="sxs-lookup"><span data-stu-id="b57cd-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="b57cd-106">Déployer des offres CNAB (Cloud Native Application Bundles) en périphérie.</span><span class="sxs-lookup"><span data-stu-id="b57cd-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="b57cd-107">Déployer une application qui s’étend aux limites du cloud.</span><span class="sxs-lookup"><span data-stu-id="b57cd-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="b57cd-108">Utiliser le kit de développement IA Custom Vision pour l’inférence en périphérie.</span><span class="sxs-lookup"><span data-stu-id="b57cd-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="b57cd-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="b57cd-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="b57cd-110">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="b57cd-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="b57cd-111">Azure Stack Hub offre à votre environnement local l’agilité et l’innovation du cloud computing grâce au seul cloud hybride qui vous permette de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="b57cd-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="b57cd-112">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) en matière de conception, de déploiement et d’exploitation des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="b57cd-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="b57cd-113">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="b57cd-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="b57cd-114">Prérequis</span><span class="sxs-lookup"><span data-stu-id="b57cd-114">Prerequisites</span></span>

<span data-ttu-id="b57cd-115">Avant de commencer à utiliser ce guide de déploiement, vous devez :</span><span class="sxs-lookup"><span data-stu-id="b57cd-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="b57cd-116">Consulter la rubrique [Modèle de détection des pas](pattern-retail-footfall-detection.md).</span><span class="sxs-lookup"><span data-stu-id="b57cd-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="b57cd-117">Obtenir un accès utilisateur à un kit de développement Azure Stack (ASDK) ou à une instance de système intégré Azure Stack Hub avec :</span><span class="sxs-lookup"><span data-stu-id="b57cd-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="b57cd-118">[Azure App Service sur le fournisseur de ressources Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview.md) installé.</span><span class="sxs-lookup"><span data-stu-id="b57cd-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="b57cd-119">Vous devez disposer d'un accès opérateur à votre instance Azure Stack Hub ou vous rapprocher de votre administrateur pour qu'il l'installe.</span><span class="sxs-lookup"><span data-stu-id="b57cd-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="b57cd-120">Abonnement à une offre fournissant App Service et un quota de stockage.</span><span class="sxs-lookup"><span data-stu-id="b57cd-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="b57cd-121">Vous devez disposer d'un accès opérateur pour créer une offre.</span><span class="sxs-lookup"><span data-stu-id="b57cd-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="b57cd-122">Obtenir un accès à un abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="b57cd-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="b57cd-123">Si vous n’avez pas d’abonnement Azure, créez un [compte d'évaluation gratuit](https://azure.microsoft.com/free/) avant de commencer.</span><span class="sxs-lookup"><span data-stu-id="b57cd-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="b57cd-124">Créez deux principaux de service dans votre répertoire :</span><span class="sxs-lookup"><span data-stu-id="b57cd-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="b57cd-125">Un principal de service configuré pour une utilisation avec les ressources Azure, avec accès à l’étendue de l’abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="b57cd-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="b57cd-126">Un principal de service configuré pour une utilisation avec les ressources Azure Stack Hub, avec accès à l’étendue de l’abonnement Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b57cd-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="b57cd-127">Pour en savoir plus sur la création de principaux de service et l’autorisation d’accès, consultez [Utiliser une identité d’application pour accéder aux ressources](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="b57cd-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="b57cd-128">Si vous préférez utiliser Azure CLI, consultez [Créer un principal du service avec Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="b57cd-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="b57cd-129">Déployer Azure Cognitive Services dans Azure ou Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b57cd-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="b57cd-130">Commencez par vous [renseigner sur Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="b57cd-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="b57cd-131">Consultez ensuite [Déployer Azure Cognitive Services sur Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) pour déployer Cognitive Services sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b57cd-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="b57cd-132">Vous devez d’abord vous inscrire pour accéder à la préversion.</span><span class="sxs-lookup"><span data-stu-id="b57cd-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="b57cd-133">Clonez ou téléchargez un kit de développement IA Azure Custom Vision non configuré.</span><span class="sxs-lookup"><span data-stu-id="b57cd-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="b57cd-134">Pour plus d’informations, consultez [Kit de développement IA Custom Vision](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="b57cd-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="b57cd-135">Créez un compte Power BI.</span><span class="sxs-lookup"><span data-stu-id="b57cd-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="b57cd-136">Clé d’abonnement et URL de point de terminaison API Visage Azure Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="b57cd-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="b57cd-137">Vous pouvez obtenir les deux avec la version d’évaluation gratuite [Essayez Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api),</span><span class="sxs-lookup"><span data-stu-id="b57cd-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="b57cd-138">ou suivre les instructions contenues dans [Créer un compte Cognitive Services](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="b57cd-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="b57cd-139">Installez les ressources de développement suivantes :</span><span class="sxs-lookup"><span data-stu-id="b57cd-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="b57cd-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="b57cd-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="b57cd-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="b57cd-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="b57cd-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="b57cd-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="b57cd-143">Vous utilisez Porter pour déployer des applications cloud à l’aide des manifestes de bundle CNAB qui vous sont fournis.</span><span class="sxs-lookup"><span data-stu-id="b57cd-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="b57cd-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="b57cd-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="b57cd-145">Azure IoT Tools pour Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="b57cd-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="b57cd-146">Extension Python pour Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="b57cd-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="b57cd-147">Python</span><span class="sxs-lookup"><span data-stu-id="b57cd-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="b57cd-148">Déployer l’application cloud hybride</span><span class="sxs-lookup"><span data-stu-id="b57cd-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="b57cd-149">Tout d’abord, vous utilisez l’interface CLI Porter pour générer un jeu d’informations d’identification, puis vous déployez l’application cloud.</span><span class="sxs-lookup"><span data-stu-id="b57cd-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="b57cd-150">Clonez ou téléchargez l’exemple de code de solution depuis https://github.com/azure-samples/azure-intelligent-edge-patterns.</span><span class="sxs-lookup"><span data-stu-id="b57cd-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="b57cd-151">Porter génère un jeu d’informations d’identification qui automatise le déploiement de l’application.</span><span class="sxs-lookup"><span data-stu-id="b57cd-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="b57cd-152">Avant d’exécuter la commande de génération des informations d’identification, assurez-vous que les éléments suivants sont disponibles :</span><span class="sxs-lookup"><span data-stu-id="b57cd-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="b57cd-153">Principal de service pour accéder aux ressources Azure, ID du principal du service, clé et DNS du locataire notamment.</span><span class="sxs-lookup"><span data-stu-id="b57cd-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="b57cd-154">ID d’abonnement de votre abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="b57cd-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="b57cd-155">Principal de service pour accéder aux ressources Azure Stack Hub, ID du principal du service, clé et DNS du locataire notamment.</span><span class="sxs-lookup"><span data-stu-id="b57cd-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="b57cd-156">ID d’abonnement de votre compte Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="b57cd-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="b57cd-157">Clé et URL de point de terminaison des ressources API Visage Azure Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="b57cd-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="b57cd-158">Exécutez le processus de génération d'informations d’identification Porter et suivez les invites :</span><span class="sxs-lookup"><span data-stu-id="b57cd-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="b57cd-159">Porter requiert également l’exécution d’un ensemble de paramètres.</span><span class="sxs-lookup"><span data-stu-id="b57cd-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="b57cd-160">Créez un fichier texte de paramètres et entrez les paires nom/valeur suivantes.</span><span class="sxs-lookup"><span data-stu-id="b57cd-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="b57cd-161">Rapprochez-vous de votre administrateur Azure Stack Hub si vous avez besoin d’aide concernant les valeurs requises.</span><span class="sxs-lookup"><span data-stu-id="b57cd-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="b57cd-162">La valeur `resource suffix` est utilisée pour veiller à ce que les ressources de votre déploiement possèdent des noms uniques dans Azure.</span><span class="sxs-lookup"><span data-stu-id="b57cd-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="b57cd-163">Il doit s’agir d’une chaîne unique de lettres et de chiffres, ne dépassant pas 8 caractères.</span><span class="sxs-lookup"><span data-stu-id="b57cd-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="b57cd-164">Enregistrez le fichier texte et notez son chemin d’accès.</span><span class="sxs-lookup"><span data-stu-id="b57cd-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="b57cd-165">Vous êtes maintenant prêt à déployer l’application cloud hybride à l’aide de Porter.</span><span class="sxs-lookup"><span data-stu-id="b57cd-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="b57cd-166">Exécutez la commande d’installation et observez le déploiement des ressources sur Azure et Azure Stack Hub :</span><span class="sxs-lookup"><span data-stu-id="b57cd-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="b57cd-167">Une fois le déploiement terminé, notez les valeurs suivantes :</span><span class="sxs-lookup"><span data-stu-id="b57cd-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="b57cd-168">Chaîne de connexion de la caméra.</span><span class="sxs-lookup"><span data-stu-id="b57cd-168">The camera's connection string.</span></span>
    - <span data-ttu-id="b57cd-169">Chaîne de connexion de compte de stockage d'image.</span><span class="sxs-lookup"><span data-stu-id="b57cd-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="b57cd-170">Noms des groupes de ressources.</span><span class="sxs-lookup"><span data-stu-id="b57cd-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="b57cd-171">Préparer le kit de développement IA Custom Vision</span><span class="sxs-lookup"><span data-stu-id="b57cd-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="b57cd-172">Configurez ensuite le kit de développement IA Custom Vision comme expliqué dans le [démarrage rapide du kit de développement IA Custom Vision](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="b57cd-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="b57cd-173">Vous pouvez également configurer et tester votre caméra à l’aide de la chaîne de connexion fournie à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="b57cd-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="b57cd-174">Déployer l’application de caméra</span><span class="sxs-lookup"><span data-stu-id="b57cd-174">Deploy the camera app</span></span>

<span data-ttu-id="b57cd-175">Utilisez l’interface CLI Porter pour générer un jeu d’informations d’identification, puis déployez l’application de caméra.</span><span class="sxs-lookup"><span data-stu-id="b57cd-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="b57cd-176">Porter génère un jeu d’informations d’identification qui automatise le déploiement de l’application.</span><span class="sxs-lookup"><span data-stu-id="b57cd-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="b57cd-177">Avant d’exécuter la commande de génération des informations d’identification, assurez-vous que les éléments suivants sont disponibles :</span><span class="sxs-lookup"><span data-stu-id="b57cd-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="b57cd-178">Principal de service pour accéder aux ressources Azure, ID du principal du service, clé et DNS du locataire notamment.</span><span class="sxs-lookup"><span data-stu-id="b57cd-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="b57cd-179">ID d’abonnement de votre abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="b57cd-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="b57cd-180">Chaîne de connexion du compte de stockage des images fournie lors du déploiement de l’application cloud.</span><span class="sxs-lookup"><span data-stu-id="b57cd-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="b57cd-181">Exécutez le processus de génération d'informations d’identification Porter et suivez les invites :</span><span class="sxs-lookup"><span data-stu-id="b57cd-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="b57cd-182">Porter requiert également l’exécution d’un ensemble de paramètres.</span><span class="sxs-lookup"><span data-stu-id="b57cd-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="b57cd-183">Créez un fichier texte de paramètres et entrez le texte suivant.</span><span class="sxs-lookup"><span data-stu-id="b57cd-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="b57cd-184">Rapprochez-vous de votre administrateur Azure Stack Hub si vous ignorez certaines des valeurs requises.</span><span class="sxs-lookup"><span data-stu-id="b57cd-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="b57cd-185">La valeur `deployment suffix` est utilisée pour veiller à ce que les ressources de votre déploiement possèdent des noms uniques dans Azure.</span><span class="sxs-lookup"><span data-stu-id="b57cd-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="b57cd-186">Il doit s’agir d’une chaîne unique de lettres et de chiffres, ne dépassant pas 8 caractères.</span><span class="sxs-lookup"><span data-stu-id="b57cd-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="b57cd-187">Enregistrez le fichier texte et notez son chemin d’accès.</span><span class="sxs-lookup"><span data-stu-id="b57cd-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="b57cd-188">Vous êtes maintenant prêt à déployer l’application de caméra à l’aide de Porter.</span><span class="sxs-lookup"><span data-stu-id="b57cd-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="b57cd-189">Exécutez la commande d’installation et observez le déploiement IoT Edge.</span><span class="sxs-lookup"><span data-stu-id="b57cd-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="b57cd-190">Pour vérifier que le déploiement de la caméra est terminé, consultez le flux de caméra sur `https://<camera-ip>:3000/`, où `<camara-ip>` correspond à l’adresse IP de la caméra.</span><span class="sxs-lookup"><span data-stu-id="b57cd-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="b57cd-191">Cette étape peut durer jusqu’à 10 minutes.</span><span class="sxs-lookup"><span data-stu-id="b57cd-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="b57cd-192">Configuration d’Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="b57cd-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="b57cd-193">Maintenant que les données sont transmises de la caméra à Azure Stream Analytics, vous devez l'autoriser manuellement à communiquer avec Power BI.</span><span class="sxs-lookup"><span data-stu-id="b57cd-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="b57cd-194">À partir du portail Azure, ouvrez **Toutes les ressources** et le travail *process-footfall\[\]votresuffixe*.</span><span class="sxs-lookup"><span data-stu-id="b57cd-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="b57cd-195">Dans la section **Topologie de la tâche** du volet du travail Stream Analytics, sélectionnez l’option **Sorties**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="b57cd-196">Sélectionnez le récepteur de sortie **traffic-output**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="b57cd-197">Sélectionnez **Renouveler une autorisation** et connectez-vous à votre compte Power BI.</span><span class="sxs-lookup"><span data-stu-id="b57cd-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Invite de renouvellement d’autorisation dans Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="b57cd-199">Enregistrez les paramètres de sortie.</span><span class="sxs-lookup"><span data-stu-id="b57cd-199">Save the output settings.</span></span>

6. <span data-ttu-id="b57cd-200">Accédez au volet **Vue d’ensemble** et sélectionnez **Démarrer** pour commencer à envoyer des données à Power BI.</span><span class="sxs-lookup"><span data-stu-id="b57cd-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="b57cd-201">Sélectionnez **Maintenant** pour l’heure de début de sortie du travail, puis sélectionnez **Démarrer**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="b57cd-202">Vous pouvez voir l’état du travail dans la barre de notification.</span><span class="sxs-lookup"><span data-stu-id="b57cd-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="b57cd-203">Créer un tableau de bord Power BI</span><span class="sxs-lookup"><span data-stu-id="b57cd-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="b57cd-204">Lorsque le travail est terminé, accédez à [Power BI](https://powerbi.com/), puis connectez-vous avec votre compte professionnel ou scolaire.</span><span class="sxs-lookup"><span data-stu-id="b57cd-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="b57cd-205">Si la requête du travail Stream Analytics génère des résultats, le jeu de données *footfall-dataset* que vous avez créé doit s’afficher dans l’onglet **Jeux de données**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="b57cd-206">À partir de votre espace de travail Power BI, sélectionnez **+ Créer** pour créer un nouveau tableau de bord intitulé *Analyse des pas*.</span><span class="sxs-lookup"><span data-stu-id="b57cd-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="b57cd-207">En haut de la fenêtre, sélectionnez **Ajouter une vignette**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="b57cd-208">Ensuite, sélectionnez **Données de streaming personnalisées**, puis **Suivant**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="b57cd-209">Choisissez **football-dataset** dans **Vos jeux de données**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="b57cd-210">Sélectionnez **Carte** dans la liste déroulante **Type de visualisation**, puis ajoutez **âge** à **Champs**.</span><span class="sxs-lookup"><span data-stu-id="b57cd-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="b57cd-211">Sélectionnez **Suivant** afin de saisir un nom pour la vignette, puis **Appliquer** pour créer la vignette.</span><span class="sxs-lookup"><span data-stu-id="b57cd-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="b57cd-212">Vous pouvez ajouter des champs et cartes supplémentaires, selon vos besoins.</span><span class="sxs-lookup"><span data-stu-id="b57cd-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="b57cd-213">Tester votre solution</span><span class="sxs-lookup"><span data-stu-id="b57cd-213">Test Your Solution</span></span>

<span data-ttu-id="b57cd-214">Observez la manière dont les données des cartes que vous avez créées dans Power BI évoluent en fonction des personnes se déplaçant devant la caméra.</span><span class="sxs-lookup"><span data-stu-id="b57cd-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="b57cd-215">Une fois enregistrées, les inférences peuvent mettre jusqu’à 20 secondes à s'afficher.</span><span class="sxs-lookup"><span data-stu-id="b57cd-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="b57cd-216">Supprimer votre solution</span><span class="sxs-lookup"><span data-stu-id="b57cd-216">Remove Your Solution</span></span>

<span data-ttu-id="b57cd-217">Si vous souhaitez supprimer votre solution, exécutez les commandes suivantes à l’aide de Porter, en utilisant les mêmes fichiers de paramètres que ceux créés pour le déploiement :</span><span class="sxs-lookup"><span data-stu-id="b57cd-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="b57cd-218">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="b57cd-218">Next steps</span></span>

- <span data-ttu-id="b57cd-219">Découvrez les [considérations sur la conception d’applications hybrides].(overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="b57cd-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="b57cd-220">Passez en revue et proposez des améliorations pour [le code de cet exemple sur GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="b57cd-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
