---
title: Modèle d’entraînement d’un modèle Machine Learning en périphérie
description: Apprenez à effectuer l’entraînement de modèles Machine Learning en périphérie avec Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910413"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="e916f-103">Modèle d’entraînement d’un modèle Machine Learning en périphérie</span><span class="sxs-lookup"><span data-stu-id="e916f-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="e916f-104">Générez des modèles Machine Learning portables à partir de données présentes uniquement en local.</span><span class="sxs-lookup"><span data-stu-id="e916f-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="e916f-105">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="e916f-105">Context and problem</span></span>

<span data-ttu-id="e916f-106">De nombreuses organisations aimeraient pouvoir extraire des insights de leurs données locales ou héritées à l'aide d'outils que leurs équipes de scientifiques des données comprennent.</span><span class="sxs-lookup"><span data-stu-id="e916f-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="e916f-107">[Azure Machine Learning](/azure/machine-learning/) fournit des outils cloud natifs pour entraîner, régler et déployer des modèles Machine Learning et Deep Learning.</span><span class="sxs-lookup"><span data-stu-id="e916f-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="e916f-108">Cependant, certaines données sont trop volumineuses pour être envoyées dans le cloud, ou ne peuvent l’être pour des raisons réglementaires.</span><span class="sxs-lookup"><span data-stu-id="e916f-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="e916f-109">Grâce à ce modèle, les scientifiques des données peuvent utiliser Azure Machine Learning pour effectuer l'apprentissage de modèles à l'aide de données et de calculs locaux.</span><span class="sxs-lookup"><span data-stu-id="e916f-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="e916f-110">Solution</span><span class="sxs-lookup"><span data-stu-id="e916f-110">Solution</span></span>

<span data-ttu-id="e916f-111">Le modèle d'apprentissage en périphérie utilise une machine virtuelle exécutée sur Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e916f-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="e916f-112">La machine virtuelle est inscrite en tant que cible de calcul dans Azure Machine Learning, ce qui lui permet d’accéder aux données disponibles uniquement en local.</span><span class="sxs-lookup"><span data-stu-id="e916f-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="e916f-113">Dans ce cas, les données sont stockées dans le stockage d’objets blob d’Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e916f-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="e916f-114">Au terme de son apprentissage, le modèle est enregistré auprès d'Azure ML, conteneurisé et ajouté à une instance d'Azure Container Registry à des fins de déploiement.</span><span class="sxs-lookup"><span data-stu-id="e916f-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="e916f-115">Pour cette itération du modèle, la machine virtuelle d’entraînement Azure Stack Hub doit être accessible sur l’Internet public.</span><span class="sxs-lookup"><span data-stu-id="e916f-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="e916f-116">[![Architecture d’entraînement d’un modèle Machine Learning en périphérie](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="e916f-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="e916f-117">Voici comment fonctionne le modèle :</span><span class="sxs-lookup"><span data-stu-id="e916f-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="e916f-118">La machine virtuelle Azure Stack Hub est déployée et enregistrée en tant que cible de calcul auprès d'Azure ML.</span><span class="sxs-lookup"><span data-stu-id="e916f-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="e916f-119">Une expérience est créée dans Azure ML, avec la machine virtuelle Azure Stack Hub comme cible de calcul.</span><span class="sxs-lookup"><span data-stu-id="e916f-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="e916f-120">Au terme de son entraînement, le modèle est inscrit et conteneurisé.</span><span class="sxs-lookup"><span data-stu-id="e916f-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="e916f-121">Le modèle peut maintenant être déployé localement ou dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="e916f-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="e916f-122">Components</span><span class="sxs-lookup"><span data-stu-id="e916f-122">Components</span></span>

<span data-ttu-id="e916f-123">Cette solution utilise les composants suivants :</span><span class="sxs-lookup"><span data-stu-id="e916f-123">This solution uses the following components:</span></span>

| <span data-ttu-id="e916f-124">Couche</span><span class="sxs-lookup"><span data-stu-id="e916f-124">Layer</span></span> | <span data-ttu-id="e916f-125">Composant</span><span class="sxs-lookup"><span data-stu-id="e916f-125">Component</span></span> | <span data-ttu-id="e916f-126">Description</span><span class="sxs-lookup"><span data-stu-id="e916f-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="e916f-127">Azure</span><span class="sxs-lookup"><span data-stu-id="e916f-127">Azure</span></span> | <span data-ttu-id="e916f-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="e916f-128">Azure Machine Learning</span></span> | <span data-ttu-id="e916f-129">[Azure Machine Learning](/azure/machine-learning/) orchestre l'apprentissage du modèle ML.</span><span class="sxs-lookup"><span data-stu-id="e916f-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="e916f-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="e916f-130">Azure Container Registry</span></span> | <span data-ttu-id="e916f-131">Azure ML empaquette le modèle dans un conteneur et le stocke dans une instance d'[Azure Container Registry](/azure/container-registry/) à des fins de déploiement.</span><span class="sxs-lookup"><span data-stu-id="e916f-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="e916f-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="e916f-132">Azure Stack Hub</span></span> | <span data-ttu-id="e916f-133">App Service</span><span class="sxs-lookup"><span data-stu-id="e916f-133">App Service</span></span> | <span data-ttu-id="e916f-134">[Azure Stack Hub avec App Service](/azure-stack/operator/azure-stack-app-service-overview) fournit une base aux composants situés en périphérie.</span><span class="sxs-lookup"><span data-stu-id="e916f-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="e916f-135">Calcul</span><span class="sxs-lookup"><span data-stu-id="e916f-135">Compute</span></span> | <span data-ttu-id="e916f-136">Une machine virtuelle Azure Stack Hub exécutant Ubuntu avec Docker est utilisée pour effectuer l'apprentissage du modèle ML.</span><span class="sxs-lookup"><span data-stu-id="e916f-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="e916f-137">Stockage</span><span class="sxs-lookup"><span data-stu-id="e916f-137">Storage</span></span> | <span data-ttu-id="e916f-138">Les données privées peuvent être hébergées dans le stockage blob d'Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e916f-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="e916f-139">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="e916f-139">Issues and considerations</span></span>

<span data-ttu-id="e916f-140">Prenez en compte des points suivants lors du choix de l'implémentation de cette solution :</span><span class="sxs-lookup"><span data-stu-id="e916f-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="e916f-141">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="e916f-141">Scalability</span></span>

<span data-ttu-id="e916f-142">Pour permettre la mise à l’échelle de cette solution, vous devez créer une machine virtuelle de taille appropriée sur Azure Stack Hub pour l’entraînement.</span><span class="sxs-lookup"><span data-stu-id="e916f-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="e916f-143">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="e916f-143">Availability</span></span>

<span data-ttu-id="e916f-144">Assurez-vous que les scripts d'apprentissage et les machines virtuelles Azure Stack Hub ont accès aux données locales utilisées pour l'apprentissage.</span><span class="sxs-lookup"><span data-stu-id="e916f-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="e916f-145">Simplicité de gestion</span><span class="sxs-lookup"><span data-stu-id="e916f-145">Manageability</span></span>

<span data-ttu-id="e916f-146">Assurez-vous que les modèles et les expériences sont correctement enregistrés, versionnés et étiquetés afin d'éviter toute confusion lors du déploiement du modèle.</span><span class="sxs-lookup"><span data-stu-id="e916f-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="e916f-147">Sécurité</span><span class="sxs-lookup"><span data-stu-id="e916f-147">Security</span></span>

<span data-ttu-id="e916f-148">Ce modèle permet à Azure Machine Learning d’accéder localement à d’éventuelles données sensibles.</span><span class="sxs-lookup"><span data-stu-id="e916f-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="e916f-149">Vérifiez que le compte utilisé pour la connexion SSH à la machine virtuelle Azure Stack Hub dispose d’un mot de passe fort et que les scripts d’entraînement ne conservent ni ne chargent de données dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="e916f-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e916f-150">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="e916f-150">Next steps</span></span>

<span data-ttu-id="e916f-151">Pour en savoir plus sur les sujets abordés dans cet article :</span><span class="sxs-lookup"><span data-stu-id="e916f-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="e916f-152">Consultez la [documentation Azure Machine Learning](/azure/machine-learning) pour bénéficier d'une vue d'ensemble sur le Machine Learning et les sujets connexes.</span><span class="sxs-lookup"><span data-stu-id="e916f-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="e916f-153">Consultez [Azure Container Registry](/azure/container-registry/) pour apprendre à créer, stocker et gérer des images pour les déploiements de conteneurs.</span><span class="sxs-lookup"><span data-stu-id="e916f-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="e916f-154">Consultez [App Service sur Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) pour en savoir plus sur le fournisseur de ressources et son déploiement.</span><span class="sxs-lookup"><span data-stu-id="e916f-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="e916f-155">Consultez [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) pour en savoir plus sur les bonnes pratiques et obtenir des réponses à d’autres questions.</span><span class="sxs-lookup"><span data-stu-id="e916f-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="e916f-156">Consultez [Famille de produits et de solutions Azure Stack](/azure-stack) pour en savoir plus sur l’ensemble du portefeuille de produits et de solutions.</span><span class="sxs-lookup"><span data-stu-id="e916f-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="e916f-157">Lorsque vous êtes prêt à tester l'exemple de solution, poursuivez avec le [Guide de déploiement consacré à l'apprentissage d'un modèle ML en périphérie](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="e916f-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="e916f-158">Ce guide de déploiement fournit des instructions pas à pas sur le déploiement et sur le test de ses composants.</span><span class="sxs-lookup"><span data-stu-id="e916f-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
