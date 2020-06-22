---
title: Modèle de relais hybride dans Azure et Azure Stack Hub
description: Utilisez le modèle de relais hybride dans Azure et Azure Stack Hub pour vous connecter à des ressources périphériques protégées par des pare-feu.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910434"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="9cde3-103">Modèle Relais hybride</span><span class="sxs-lookup"><span data-stu-id="9cde3-103">Hybrid relay pattern</span></span>

<span data-ttu-id="9cde3-104">Découvrez comment vous connecter à des ressources ou des appareils de périphérie protégés par des pare-feu à l’aide du modèle de relais hybride et de Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="9cde3-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="9cde3-105">Contexte et problème</span><span class="sxs-lookup"><span data-stu-id="9cde3-105">Context and problem</span></span>

<span data-ttu-id="9cde3-106">Les appareils à la périphérie sont souvent derrière un pare-feu d’entreprise ou un périphérique NAT.</span><span class="sxs-lookup"><span data-stu-id="9cde3-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="9cde3-107">Bien qu’ils soient sécurisés, ils peuvent ne pas être en mesure de communiquer avec le cloud public ou les appareils périphériques d’autres réseaux d’entreprise.</span><span class="sxs-lookup"><span data-stu-id="9cde3-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="9cde3-108">Il peut être nécessaire d’exposer certains ports et fonctionnalités aux utilisateurs dans le cloud public de manière sécurisée.</span><span class="sxs-lookup"><span data-stu-id="9cde3-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="9cde3-109">Solution</span><span class="sxs-lookup"><span data-stu-id="9cde3-109">Solution</span></span>

<span data-ttu-id="9cde3-110">Le modèle de relais hybride utilise Azure Relay pour établir un tunnel WebSocket entre deux points de terminaison qui ne peuvent pas communiquer directement.</span><span class="sxs-lookup"><span data-stu-id="9cde3-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="9cde3-111">Les appareils qui ne sont pas locaux, mais qui doivent se connecter à un point de terminaison local, se connectent à un point de terminaison dans le cloud public.</span><span class="sxs-lookup"><span data-stu-id="9cde3-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="9cde3-112">Ce point de terminaison redirige le trafic sur les itinéraires prédéfinis sur un canal sécurisé.</span><span class="sxs-lookup"><span data-stu-id="9cde3-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="9cde3-113">Un point de terminaison à l’intérieur de l’environnement local reçoit le trafic et l’achemine vers la destination correcte.</span><span class="sxs-lookup"><span data-stu-id="9cde3-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![Architecture de la solution de modèle de relais hybride](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="9cde3-115">Voici comment fonctionne le modèle de relais hybride :</span><span class="sxs-lookup"><span data-stu-id="9cde3-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="9cde3-116">Un appareil se connecte à la machine virtuelle dans Azure, sur un port prédéfini.</span><span class="sxs-lookup"><span data-stu-id="9cde3-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="9cde3-117">Le trafic est transféré vers le Azure Relay dans Azure.</span><span class="sxs-lookup"><span data-stu-id="9cde3-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="9cde3-118">La machine virtuelle sur Azure Stack Hub, qui a déjà établi une connexion de longue durée au Azure Relay, reçoit le trafic et le transfère vers la destination.</span><span class="sxs-lookup"><span data-stu-id="9cde3-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="9cde3-119">Le point de terminaison ou le service local traite la demande.</span><span class="sxs-lookup"><span data-stu-id="9cde3-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="9cde3-120">Components</span><span class="sxs-lookup"><span data-stu-id="9cde3-120">Components</span></span>

<span data-ttu-id="9cde3-121">Cette solution utilise les composants suivants :</span><span class="sxs-lookup"><span data-stu-id="9cde3-121">This solution uses the following components:</span></span>

| <span data-ttu-id="9cde3-122">Couche</span><span class="sxs-lookup"><span data-stu-id="9cde3-122">Layer</span></span> | <span data-ttu-id="9cde3-123">Composant</span><span class="sxs-lookup"><span data-stu-id="9cde3-123">Component</span></span> | <span data-ttu-id="9cde3-124">Description</span><span class="sxs-lookup"><span data-stu-id="9cde3-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="9cde3-125">Azure</span><span class="sxs-lookup"><span data-stu-id="9cde3-125">Azure</span></span> | <span data-ttu-id="9cde3-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="9cde3-126">Azure VM</span></span> | <span data-ttu-id="9cde3-127">Une machine virtuelle Azure fournit un point de terminaison accessible publiquement pour la ressource locale.</span><span class="sxs-lookup"><span data-stu-id="9cde3-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="9cde3-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="9cde3-128">Azure Relay</span></span> | <span data-ttu-id="9cde3-129">Une [Azure Relay](/azure/azure-relay/) fournit l’infrastructure pour maintenir le tunnel et la connexion entre la machine virtuelle Azure et la machine virtuelle Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="9cde3-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="9cde3-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="9cde3-130">Azure Stack Hub</span></span> | <span data-ttu-id="9cde3-131">Calcul</span><span class="sxs-lookup"><span data-stu-id="9cde3-131">Compute</span></span> | <span data-ttu-id="9cde3-132">Une machine virtuelle Azure Stack Hub fournit le côté serveur du tunnel de relais hybride.</span><span class="sxs-lookup"><span data-stu-id="9cde3-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="9cde3-133">Stockage</span><span class="sxs-lookup"><span data-stu-id="9cde3-133">Storage</span></span> | <span data-ttu-id="9cde3-134">Le cluster du moteur AKS déployé dans Azure Stack Hub fournit un moteur résilient et scalable pour exécuter le conteneur de l’API Visage.</span><span class="sxs-lookup"><span data-stu-id="9cde3-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="9cde3-135">Problèmes et considérations</span><span class="sxs-lookup"><span data-stu-id="9cde3-135">Issues and considerations</span></span>

<span data-ttu-id="9cde3-136">Prenez en compte des points suivants lors du choix de l'implémentation de cette solution :</span><span class="sxs-lookup"><span data-stu-id="9cde3-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="9cde3-137">Extensibilité</span><span class="sxs-lookup"><span data-stu-id="9cde3-137">Scalability</span></span>

<span data-ttu-id="9cde3-138">Ce modèle autorise uniquement les mappages de port 1:1 sur le client et le serveur.</span><span class="sxs-lookup"><span data-stu-id="9cde3-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="9cde3-139">Par exemple, si le port 80 est tunnelé pour un service sur le point de terminaison Azure, il ne peut pas être utilisé pour un autre service.</span><span class="sxs-lookup"><span data-stu-id="9cde3-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="9cde3-140">Les mappages de ports doivent être planifiés en conséquence.</span><span class="sxs-lookup"><span data-stu-id="9cde3-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="9cde3-141">Les Azure Relay et les machines virtuelles doivent être mis à l’échelle de manière appropriée pour gérer le trafic.</span><span class="sxs-lookup"><span data-stu-id="9cde3-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="9cde3-142">Disponibilité</span><span class="sxs-lookup"><span data-stu-id="9cde3-142">Availability</span></span>

<span data-ttu-id="9cde3-143">Ces tunnels et connexions ne sont pas redondants.</span><span class="sxs-lookup"><span data-stu-id="9cde3-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="9cde3-144">Pour garantir une haute disponibilité, vous pouvez implémenter un code de vérification des erreurs.</span><span class="sxs-lookup"><span data-stu-id="9cde3-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="9cde3-145">Une autre option consiste à avoir un pool de machines virtuelles connectées à Azure Relay derrière un équilibreur de charge.</span><span class="sxs-lookup"><span data-stu-id="9cde3-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="9cde3-146">Simplicité de gestion</span><span class="sxs-lookup"><span data-stu-id="9cde3-146">Manageability</span></span>

<span data-ttu-id="9cde3-147">Cette solution peut couvrir de nombreux appareils et emplacements, et donc devenir difficile à gérer.</span><span class="sxs-lookup"><span data-stu-id="9cde3-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="9cde3-148">Les services IoT d’Azure peuvent être utilisés pour mettre automatiquement en ligne de nouveaux emplacements et appareils et les maintenir à jour.</span><span class="sxs-lookup"><span data-stu-id="9cde3-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="9cde3-149">Sécurité</span><span class="sxs-lookup"><span data-stu-id="9cde3-149">Security</span></span>

<span data-ttu-id="9cde3-150">Ce modèle, tel que présenté, permet un accès non autorisé à un port sur un appareil interne à partir de la périphérie.</span><span class="sxs-lookup"><span data-stu-id="9cde3-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="9cde3-151">Envisagez d’ajouter un mécanisme d’authentification au service sur l’appareil interne, ou devant le point de terminaison de relais hybride.</span><span class="sxs-lookup"><span data-stu-id="9cde3-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9cde3-152">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="9cde3-152">Next steps</span></span>

<span data-ttu-id="9cde3-153">Pour en savoir plus sur les sujets abordés dans cet article :</span><span class="sxs-lookup"><span data-stu-id="9cde3-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="9cde3-154">Ce modèle utilise Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="9cde3-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="9cde3-155">Pour plus d’informations, consultez la [documentation Azure Relay](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="9cde3-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="9cde3-156">Consultez [Considérations relatives à la conception des applications hybrides](overview-app-design-considerations.md) pour en savoir plus sur les bonnes pratiques et obtenir des réponses à d’autres questions.</span><span class="sxs-lookup"><span data-stu-id="9cde3-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="9cde3-157">Consultez [Famille de produits et de solutions Azure Stack](/azure-stack) pour en savoir plus sur l’ensemble du portefeuille de produits et de solutions.</span><span class="sxs-lookup"><span data-stu-id="9cde3-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="9cde3-158">Lorsque vous êtes prêt à tester l’exemple de solution, poursuivez avec le [Guide de déploiement de la solution de relais hybride](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="9cde3-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="9cde3-159">Ce guide de déploiement fournit des instructions pas à pas sur le déploiement et sur le test de ses composants.</span><span class="sxs-lookup"><span data-stu-id="9cde3-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>