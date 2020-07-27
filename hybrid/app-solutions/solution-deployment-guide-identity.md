---
title: Configurer une identité cloud hybride pour les applications Azure et Azure Stack Hub
description: Apprenez à configurer une identité cloud hybride pour les applications Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477250"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="a3b4d-103">Configurer une identité cloud hybride pour les applications Azure et Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="a3b4d-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="a3b4d-104">Apprenez à configurer une identité cloud hybride pour vos applications Azure et Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="a3b4d-105">Il existe deux moyens d'accorder l'accès à vos applications à la fois dans le service Azure global et dans Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="a3b4d-106">Si Azure Stack Hub dispose d'une connexion permanente à Internet, vous pouvez utiliser Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="a3b4d-107">Si Azure Stack Hub est déconnecté d'Internet, vous pouvez opter pour les services de fédération Active Directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="a3b4d-108">Les principaux de service vous permettent d'accorder l'accès à vos applications Azure Stack Hub à des fins de déploiement ou de configuration à l'aide d'Azure Resource Manager dans Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="a3b4d-109">Dans cette solution, vous allez générer un exemple d’environnement pour :</span><span class="sxs-lookup"><span data-stu-id="a3b4d-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="a3b4d-110">établir une identité hybride dans Azure global et Azure Stack Hub ;</span><span class="sxs-lookup"><span data-stu-id="a3b4d-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="a3b4d-111">récupérer un jeton afin d'accéder à l'API Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="a3b4d-112">Pour suivre les étapes de cette solution, vous devez disposer d'autorisations d'opérateur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="a3b4d-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="a3b4d-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="a3b4d-114">Microsoft Azure Stack Hub est une extension d’Azure.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="a3b4d-115">Azure Stack Hub apporte l’agilité et l’innovation du cloud computing à votre environnement local et active le seul cloud hybride qui vous permet de créer et de déployer des applications hybrides en tout lieu.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="a3b4d-116">L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) en matière de conception, de déploiement et d’exploitation des applications hybrides.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="a3b4d-117">Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="a3b4d-118">Créer un principal du service pour Azure AD sur le portail</span><span class="sxs-lookup"><span data-stu-id="a3b4d-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="a3b4d-119">Si vous avez déployé Azure Stack Hub en utilisant Azure AD en tant que magasin d'identités, vous pouvez créer des principaux de service de la même façon que pour Azure.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="a3b4d-120">La rubrique [Utiliser une identité d’application pour accéder aux ressources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) montre comment effectuer les étapes via le portail.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="a3b4d-121">Avant de commencer, vérifiez que vous disposez des [autorisations Azure AD requises](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="a3b4d-122">Créer un principal du service pour AD FS avec PowerShell</span><span class="sxs-lookup"><span data-stu-id="a3b4d-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="a3b4d-123">Si vous avez déployé Azure Stack Hub avec AD FS, vous pouvez utiliser PowerShell pour créer un principal de service, attribuer un rôle d'accès et vous connecter à partir de PowerShell à l'aide de cette identité.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="a3b4d-124">La rubrique [Utiliser une identité d’application pour accéder aux ressources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) montre comment procéder à l’aide de PowerShell.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="a3b4d-125">Utiliser l'API Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="a3b4d-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="a3b4d-126">La solution [API Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use.md) décrit le processus de récupération d'un jeton pour accéder à l'API Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="a3b4d-127">Se connecter à Azure Stack Hub avec PowerShell</span><span class="sxs-lookup"><span data-stu-id="a3b4d-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="a3b4d-128">Le guide de démarrage rapide à suivre pour [devenir rapidement opérationnel avec PowerShell dans Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) vous explique comment installer Azure PowerShell et vous connecter à votre installation Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="a3b4d-129">Prérequis</span><span class="sxs-lookup"><span data-stu-id="a3b4d-129">Prerequisites</span></span>

<span data-ttu-id="a3b4d-130">Vous devez disposer d’une installation Azure Stack Hub connectée à Azure Active Directory, avec un abonnement auquel vous pouvez accéder.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="a3b4d-131">Si Azure Stack Hub n’est pas installé, vous pouvez suivre ces instructions pour configurer un [kit de développement Azure Stack (ASDK)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="a3b4d-132">Se connecter à Azure Stack Hub à l'aide de code</span><span class="sxs-lookup"><span data-stu-id="a3b4d-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="a3b4d-133">Pour vous connecter à Azure Stack Hub à l’aide de code, utilisez l’API des points de terminaison Azure Resource Manager afin d’obtenir les points de terminaison d’authentification et Graph de votre installation Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="a3b4d-134">Authentifiez-vous ensuite par le biais de demandes REST.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="a3b4d-135">Vous trouverez un exemple d’application cliente sur [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="a3b4d-136">Si le kit de développement logiciel (SDK) Azure de la langue de votre choix ne prend pas en charge les profils d'API Azure, il risque de ne pas fonctionner avec Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="a3b4d-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="a3b4d-137">Pour en savoir plus sur les profils d’API Azure, consultez l’article [Gérer les profils de version des API](/azure-stack/user/azure-stack-version-profiles.md).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a3b4d-138">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="a3b4d-138">Next steps</span></span>

- <span data-ttu-id="a3b4d-139">Pour plus d'informations sur la gestion des identités dans Azure Stack Hub, consultez [Architecture d'identité pour Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="a3b4d-140">Pour plus d’informations sur les modèles Azure Cloud, consultez [Modèles de conception cloud](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="a3b4d-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
