---
title: Modèle d’application géodistribuée dans Azure Stack Hub
description: En savoir plus sur le modèle d’application géodistribuée pour la périphérie intelligente utilisant Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910320"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="5d914-103">Procédure de création d’une application géodistribuée</span><span class="sxs-lookup"><span data-stu-id="5d914-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="5d914-104">Découvrez comment fournir des points de terminaison d’application dans plusieurs régions et acheminer le trafic utilisateur en fonction de l’emplacement et des besoins de conformité.</span><span class="sxs-lookup"><span data-stu-id="5d914-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="5d914-105">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="5d914-105">Context and problem</span></span>

<span data-ttu-id="5d914-106">Les organisations situées dans des régions géographiques éloignées s’efforcent de distribuer et d’activer l’accès aux données de manière sécurisée et précise, tout en garantissant les niveaux de sécurité, de conformité et de performance nécessaires en fonction des utilisateurs, des emplacements et des appareils, indépendamment des frontières.</span><span class="sxs-lookup"><span data-stu-id="5d914-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="5d914-107">Solution</span><span class="sxs-lookup"><span data-stu-id="5d914-107">Solution</span></span>

<span data-ttu-id="5d914-108">Le modèle de routage de trafic géographique Azure Stack Hub (ou applications géodistribuées) permet de diriger le trafic vers des points de terminaison spécifiques en fonction de diverses métriques.</span><span class="sxs-lookup"><span data-stu-id="5d914-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="5d914-109">La création d’un profil Traffic Manager avec configuration du routage et du point de terminaison en fonction de la région géographique permet de router le trafic vers les points de terminaison en fonction des exigences régionales, des réglementations organisationnelles et internationales et de vos besoins relatifs aux données.</span><span class="sxs-lookup"><span data-stu-id="5d914-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Modèle géodistribué](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="5d914-111">Components</span><span class="sxs-lookup"><span data-stu-id="5d914-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="5d914-112">En dehors du cloud</span><span class="sxs-lookup"><span data-stu-id="5d914-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="5d914-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="5d914-113">Traffic Manager</span></span>

<span data-ttu-id="5d914-114">Dans le diagramme, Traffic Manager se situe en dehors du cloud public, mais il doit pouvoir coordonner le trafic dans le centre de données local et dans le cloud public.</span><span class="sxs-lookup"><span data-stu-id="5d914-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="5d914-115">L’équilibreur route le trafic vers des emplacements géographiques.</span><span class="sxs-lookup"><span data-stu-id="5d914-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="5d914-116">DNS (Domain Name System)</span><span class="sxs-lookup"><span data-stu-id="5d914-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="5d914-117">Le DNS (Domain Name System) se charge de traduire (ou résoudre) un nom de site web ou de service en une adresse IP.</span><span class="sxs-lookup"><span data-stu-id="5d914-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="5d914-118">Cloud public</span><span class="sxs-lookup"><span data-stu-id="5d914-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="5d914-119">Point de terminaison cloud</span><span class="sxs-lookup"><span data-stu-id="5d914-119">Cloud Endpoint</span></span>

<span data-ttu-id="5d914-120">Les adresses IP publiques sont utilisées pour router le trafic entrant via Traffic Manager vers le point de terminaison des ressources d’application du cloud public.</span><span class="sxs-lookup"><span data-stu-id="5d914-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="5d914-121">Clouds locaux</span><span class="sxs-lookup"><span data-stu-id="5d914-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="5d914-122">Point de terminaison local</span><span class="sxs-lookup"><span data-stu-id="5d914-122">Local endpoint</span></span>

<span data-ttu-id="5d914-123">Les adresses IP publiques sont utilisées pour router le trafic entrant via Traffic Manager vers le point de terminaison des ressources d’application du cloud public.</span><span class="sxs-lookup"><span data-stu-id="5d914-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="5d914-124">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="5d914-124">Issues and considerations</span></span>

<span data-ttu-id="5d914-125">Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :</span><span class="sxs-lookup"><span data-stu-id="5d914-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="5d914-126">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="5d914-126">Scalability</span></span>

<span data-ttu-id="5d914-127">Le modèle gère le routage du trafic géographique plutôt que la mise à l’échelle pour répondre aux hausses du trafic.</span><span class="sxs-lookup"><span data-stu-id="5d914-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="5d914-128">Toutefois, vous pouvez combiner ce modèle avec d’autres solutions Azure et locales.</span><span class="sxs-lookup"><span data-stu-id="5d914-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="5d914-129">Par exemple, vous pouvez utiliser ce modèle avec le modèle de mise à l’échelle multicloud.</span><span class="sxs-lookup"><span data-stu-id="5d914-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="5d914-130">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="5d914-130">Availability</span></span>

<span data-ttu-id="5d914-131">Vérifiez que les applications déployées en local sont configurées pour la haute disponibilité via la configuration matérielle locale et le déploiement de logiciels.</span><span class="sxs-lookup"><span data-stu-id="5d914-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="5d914-132">Simplicité de gestion</span><span class="sxs-lookup"><span data-stu-id="5d914-132">Manageability</span></span>

<span data-ttu-id="5d914-133">Le modèle garantit une gestion transparente et l’accès à une interface familière entre les environnements.</span><span class="sxs-lookup"><span data-stu-id="5d914-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="5d914-134">Quand utiliser ce modèle</span><span class="sxs-lookup"><span data-stu-id="5d914-134">When to use this pattern</span></span>

- <span data-ttu-id="5d914-135">Mon organisation a des filiales internationales qui imposent des stratégies de sécurité et de distribution régionales personnalisées.</span><span class="sxs-lookup"><span data-stu-id="5d914-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="5d914-136">Chaque filiale de mon organisation extrait des données sur les employés, l’entreprise et les installations, ce qui nécessite la création de rapports d’activité en fonction de la réglementation locale et du fuseau horaire.</span><span class="sxs-lookup"><span data-stu-id="5d914-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="5d914-137">Les exigences à grande échelle peuvent être satisfaites par la mise à l’échelle horizontale des applications, avec plusieurs déploiements dans une seule ou plusieurs régions, pour gérer les charges extrêmes.</span><span class="sxs-lookup"><span data-stu-id="5d914-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="5d914-138">Les applications doivent être hautement disponibles et répondre aux requêtes des clients, même pendant les pannes dans une seule région.</span><span class="sxs-lookup"><span data-stu-id="5d914-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5d914-139">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="5d914-139">Next steps</span></span>

<span data-ttu-id="5d914-140">Pour en savoir plus sur les sujets abordés dans cet article :</span><span class="sxs-lookup"><span data-stu-id="5d914-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="5d914-141">Pour en savoir plus sur le fonctionnement de cet équilibreur de charge du trafic basé sur DNS, consultez la [présentation d’Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="5d914-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="5d914-142">Consultez [Considérations relatives à la conception des applications hybrides](overview-app-design-considerations.md) pour en savoir plus sur les bonnes pratiques et obtenir des réponses à d’autres questions.</span><span class="sxs-lookup"><span data-stu-id="5d914-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="5d914-143">Consultez [Famille de produits et de solutions Azure Stack](/azure-stack) pour en savoir plus sur l’ensemble du portefeuille de produits et de solutions.</span><span class="sxs-lookup"><span data-stu-id="5d914-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="5d914-144">Lorsque vous êtes prêt à tester l’exemple de solution, poursuivez avec le [Guide de déploiement d’une solution d’application géodistribuée](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="5d914-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="5d914-145">Ce guide de déploiement fournit des instructions pas à pas sur le déploiement et sur le test de ses composants.</span><span class="sxs-lookup"><span data-stu-id="5d914-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="5d914-146">Découvrez comment diriger le trafic vers des points de terminaison spécifiques en fonction de différentes métriques à l’aide du modèle d’application géodistribuée.</span><span class="sxs-lookup"><span data-stu-id="5d914-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="5d914-147">En créant un profil Traffic Manager avec configuration du routage et du point de terminaison basée sur la géolocalisation, vous êtes certain que les informations sont dirigées vers les points de terminaison en fonction des exigences régionales, des réglementations organisationnelles et internationales et de vos besoins en matière de données.</span><span class="sxs-lookup"><span data-stu-id="5d914-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
