---
title: Modèle de mise à l’échelle multicloud dans Azure Stack Hub
description: Découvrez comment créer une application multicloud scalable sur Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910428"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="5c33f-103">Modèle de mise à l’échelle multicloud</span><span class="sxs-lookup"><span data-stu-id="5c33f-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="5c33f-104">Ajoutez automatiquement des ressources à une application existante pour faire face à une hausse de la charge.</span><span class="sxs-lookup"><span data-stu-id="5c33f-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="5c33f-105">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="5c33f-105">Context and problem</span></span>

<span data-ttu-id="5c33f-106">Votre application ne peut pas augmenter sa capacité pour répondre à une hausse inattendue de la demande.</span><span class="sxs-lookup"><span data-stu-id="5c33f-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="5c33f-107">En raison de ce manque de scalabilité, les utilisateurs ne peuvent donc pas accéder à l’application pendant les pics d’utilisation.</span><span class="sxs-lookup"><span data-stu-id="5c33f-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="5c33f-108">L’application peut répondre aux demandes d’un nombre fixe d’utilisateurs.</span><span class="sxs-lookup"><span data-stu-id="5c33f-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="5c33f-109">Les entreprises internationales ont besoin d’applications cloud sécurisées, fiables et disponibles.</span><span class="sxs-lookup"><span data-stu-id="5c33f-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="5c33f-110">La réponse à la hausse de la demande et l’utilisation de l’infrastructure appropriée pour prendre en charge cette demande sont des aspects critiques.</span><span class="sxs-lookup"><span data-stu-id="5c33f-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="5c33f-111">Les entreprises ont des difficultés à équilibrer les coûts et la maintenance en ce qui concerne la sécurité, le stockage et la disponibilité en temps réel des données métier.</span><span class="sxs-lookup"><span data-stu-id="5c33f-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="5c33f-112">Vous ne pourrez peut-être pas exécuter votre application dans le cloud public.</span><span class="sxs-lookup"><span data-stu-id="5c33f-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="5c33f-113">Toutefois, ceci peut ne pas être économiquement faisable pour que l’entreprise puisse maintenir la capacité nécessaire dans son environnement local afin de gérer les pics de demande de l’application.</span><span class="sxs-lookup"><span data-stu-id="5c33f-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="5c33f-114">Avec ce modèle, vous pouvez tirer parti de l’élasticité du cloud public pour votre solution locale.</span><span class="sxs-lookup"><span data-stu-id="5c33f-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="5c33f-115">Solution</span><span class="sxs-lookup"><span data-stu-id="5c33f-115">Solution</span></span>

<span data-ttu-id="5c33f-116">Le modèle de mise à l’échelle multicloud étend une application située dans un cloud local avec les ressources du cloud public.</span><span class="sxs-lookup"><span data-stu-id="5c33f-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="5c33f-117">Le modèle se déclenche en réponse à une hausse ou une baisse de la demande, puis ajoute ou supprime respectivement des ressources dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="5c33f-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="5c33f-118">Ces ressources assurent une redondance, une disponibilité rapide et un routage géoconforme.</span><span class="sxs-lookup"><span data-stu-id="5c33f-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Modèle de mise à l’échelle multicloud](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="5c33f-120">Ce modèle s’applique uniquement aux applications sans état de votre application.</span><span class="sxs-lookup"><span data-stu-id="5c33f-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="5c33f-121">Components</span><span class="sxs-lookup"><span data-stu-id="5c33f-121">Components</span></span>

<span data-ttu-id="5c33f-122">Le modèle de mise à l’échelle multicloud inclut les composants suivants.</span><span class="sxs-lookup"><span data-stu-id="5c33f-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="5c33f-123">En dehors du cloud</span><span class="sxs-lookup"><span data-stu-id="5c33f-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="5c33f-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="5c33f-124">Traffic Manager</span></span>

<span data-ttu-id="5c33f-125">Dans le diagramme, il se situe en dehors du groupe de clouds publics, mais il doit pouvoir coordonner le trafic dans le centre de données local et dans le cloud public.</span><span class="sxs-lookup"><span data-stu-id="5c33f-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="5c33f-126">L’équilibreur offre une haute disponibilité pour l’application en supervisant les points de terminaison et en assurant la redistribution du basculement en cas de besoin.</span><span class="sxs-lookup"><span data-stu-id="5c33f-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="5c33f-127">DNS (Domain Name System)</span><span class="sxs-lookup"><span data-stu-id="5c33f-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="5c33f-128">Le DNS (Domain Name System) se charge de traduire (ou résoudre) un nom de site web ou de service en une adresse IP.</span><span class="sxs-lookup"><span data-stu-id="5c33f-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="5c33f-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="5c33f-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="5c33f-130">Serveur de builds hébergé</span><span class="sxs-lookup"><span data-stu-id="5c33f-130">Hosted build server</span></span>

<span data-ttu-id="5c33f-131">Environnement d’hébergement de votre pipeline de build.</span><span class="sxs-lookup"><span data-stu-id="5c33f-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="5c33f-132">Ressources d’application</span><span class="sxs-lookup"><span data-stu-id="5c33f-132">App resources</span></span>

<span data-ttu-id="5c33f-133">Les ressources d’application doivent pouvoir effectuer un scale-in et un scale-out, à l’image des groupes de machines virtuelles identiques et des conteneurs.</span><span class="sxs-lookup"><span data-stu-id="5c33f-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="5c33f-134">Nom de domaine personnalisé</span><span class="sxs-lookup"><span data-stu-id="5c33f-134">Custom domain name</span></span>

<span data-ttu-id="5c33f-135">Utilisez un nom de domaine personnalisé pour le Glob de routage des requêtes.</span><span class="sxs-lookup"><span data-stu-id="5c33f-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="5c33f-136">Adresses IP publiques</span><span class="sxs-lookup"><span data-stu-id="5c33f-136">Public IP addresses</span></span>

<span data-ttu-id="5c33f-137">Les adresses IP publiques sont utilisées pour router le trafic entrant via Traffic Manager vers le point de terminaison des ressources d’application du cloud public.</span><span class="sxs-lookup"><span data-stu-id="5c33f-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="5c33f-138">Cloud local</span><span class="sxs-lookup"><span data-stu-id="5c33f-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="5c33f-139">Serveur de builds hébergé</span><span class="sxs-lookup"><span data-stu-id="5c33f-139">Hosted build server</span></span>

<span data-ttu-id="5c33f-140">Environnement d’hébergement de votre pipeline de build.</span><span class="sxs-lookup"><span data-stu-id="5c33f-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="5c33f-141">Ressources d’application</span><span class="sxs-lookup"><span data-stu-id="5c33f-141">App resources</span></span>

<span data-ttu-id="5c33f-142">Les ressources d’application doivent pouvoir effectuer un scale-in et un scale-out, à l’image des groupes de machines virtuelles identiques et des conteneurs.</span><span class="sxs-lookup"><span data-stu-id="5c33f-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="5c33f-143">Nom de domaine personnalisé</span><span class="sxs-lookup"><span data-stu-id="5c33f-143">Custom domain name</span></span>

<span data-ttu-id="5c33f-144">Utilisez un nom de domaine personnalisé pour le Glob de routage des requêtes.</span><span class="sxs-lookup"><span data-stu-id="5c33f-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="5c33f-145">Adresses IP publiques</span><span class="sxs-lookup"><span data-stu-id="5c33f-145">Public IP addresses</span></span>

<span data-ttu-id="5c33f-146">Les adresses IP publiques sont utilisées pour router le trafic entrant via Traffic Manager vers le point de terminaison des ressources d’application du cloud public.</span><span class="sxs-lookup"><span data-stu-id="5c33f-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="5c33f-147">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="5c33f-147">Issues and considerations</span></span>

<span data-ttu-id="5c33f-148">Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="5c33f-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="5c33f-149">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="5c33f-149">Scalability</span></span>

<span data-ttu-id="5c33f-150">Le composant clé de la mise à l’échelle multicloud est la possibilité de fournir une mise à l’échelle à la demande.</span><span class="sxs-lookup"><span data-stu-id="5c33f-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="5c33f-151">La mise à l’échelle doit se produire entre l’infrastructure cloud publique et locale et fournir un service cohérent et fiable à la demande.</span><span class="sxs-lookup"><span data-stu-id="5c33f-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="5c33f-152">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="5c33f-152">Availability</span></span>

<span data-ttu-id="5c33f-153">Vérifiez que les applications déployées en local sont configurées pour la haute disponibilité via la configuration matérielle locale et le déploiement de logiciels.</span><span class="sxs-lookup"><span data-stu-id="5c33f-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="5c33f-154">Simplicité de gestion</span><span class="sxs-lookup"><span data-stu-id="5c33f-154">Manageability</span></span>

<span data-ttu-id="5c33f-155">Le modèle multicloud garantit une gestion fluide et l’accès à une interface familière entre les environnements.</span><span class="sxs-lookup"><span data-stu-id="5c33f-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="5c33f-156">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="5c33f-156">When to use this pattern</span></span>

<span data-ttu-id="5c33f-157">Utilisez ce modèle :</span><span class="sxs-lookup"><span data-stu-id="5c33f-157">Use this pattern:</span></span>

- <span data-ttu-id="5c33f-158">Quand vous avez besoin d’augmenter la capacité de votre application pour faire face à des demandes inattendues ou régulières.</span><span class="sxs-lookup"><span data-stu-id="5c33f-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="5c33f-159">Quand vous ne souhaitez pas investir dans des ressources qui vont être utilisées uniquement durant les pics d’activité.</span><span class="sxs-lookup"><span data-stu-id="5c33f-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="5c33f-160">Payez pour ce que vous utilisez.</span><span class="sxs-lookup"><span data-stu-id="5c33f-160">Pay for what you use.</span></span>

<span data-ttu-id="5c33f-161">Ce modèle n’est pas recommandé quand :</span><span class="sxs-lookup"><span data-stu-id="5c33f-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="5c33f-162">Votre solution impose aux utilisateurs de se connecter via Internet</span><span class="sxs-lookup"><span data-stu-id="5c33f-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="5c33f-163">Votre entreprise doit se conformer à des réglementations locales qui imposent à la connexion d’origine de provenir d’un appel local</span><span class="sxs-lookup"><span data-stu-id="5c33f-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="5c33f-164">Votre réseau est confronté à des goulots d’étranglement réguliers qui limitent les performances de la mise à l’échelle.</span><span class="sxs-lookup"><span data-stu-id="5c33f-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="5c33f-165">Votre environnement est déconnecté d’Internet et ne peut pas atteindre le cloud public.</span><span class="sxs-lookup"><span data-stu-id="5c33f-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5c33f-166">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="5c33f-166">Next steps</span></span>

<span data-ttu-id="5c33f-167">Pour en savoir plus sur les sujets abordés dans cet article :</span><span class="sxs-lookup"><span data-stu-id="5c33f-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="5c33f-168">Pour en savoir plus sur le fonctionnement de cet équilibreur de charge du trafic basé sur DNS, consultez la [présentation d’Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="5c33f-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="5c33f-169">Consultez [Considérations relatives à la conception des applications hybrides](overview-app-design-considerations.md) pour en savoir plus sur les bonnes pratiques et obtenir des réponses à d’autres questions.</span><span class="sxs-lookup"><span data-stu-id="5c33f-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="5c33f-170">Consultez [Famille de produits et de solutions Azure Stack](/azure-stack) pour en savoir plus sur l’ensemble du portefeuille de produits et de solutions.</span><span class="sxs-lookup"><span data-stu-id="5c33f-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="5c33f-171">Lorsque vous êtes prêt à tester l’exemple de solution, poursuivez avec le [Guide de déploiement de la solution de mise à l’échelle multicloud](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="5c33f-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="5c33f-172">Ce guide de déploiement fournit des instructions pas à pas sur le déploiement et sur le test de ses composants.</span><span class="sxs-lookup"><span data-stu-id="5c33f-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="5c33f-173">Découvrez comment créer une solution multicloud pour fournir un processus déclenché manuellement permettant de passer d’une application web hébergée sur Azure Stack Hub à une application web hébergée sur Azure.</span><span class="sxs-lookup"><span data-stu-id="5c33f-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="5c33f-174">Découvrez également comment utiliser la mise à l’échelle automatique via Traffic Manager, en garantissant un utilitaire cloud flexible et scalable en cas de charge.</span><span class="sxs-lookup"><span data-stu-id="5c33f-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
