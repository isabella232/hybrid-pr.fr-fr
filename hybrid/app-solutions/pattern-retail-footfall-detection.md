---
title: Modèle de détection des pas à l’aide d’Azure et d’Azure Stack Hub
description: Découvrez comment utiliser Azure et Azure Stack Hub pour implémenter une solution de détection des pas basé sur l’intelligence artificielle afin d’analyser la fréquentation des magasins.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895327"
---
# <a name="footfall-detection-pattern"></a>Modèle de détection des pas

Ce modèle fournit une présentation de l’implémentation d’une solution de détection de pas basée sur l’intelligence artificielle pour l’analyse du trafic des visiteurs dans les magasins. La solution fournit des insights à partir d’actions du monde réel, en utilisant Azure, Azure Stack Hub et le kit de développement d’intelligence artificielle personnalisé Custom Vision.

## <a name="context-and-problem"></a>Contexte et problème

Les magasins Contoso aimeraient avoir un aperçu de la façon dont les clients accueillent leurs produits actuels par rapport à l’agencement des magasins. Ces magasins ne peuvent pas affecter du personnel dans chaque section et il serait trop fastidieux de demander à une équipe d’analystes d’examiner les séquences vidéo d’un magasin tout entier. Par ailleurs, aucun des magasins ne dispose d'une bande passante suffisante pour diffuser la vidéo de toutes les caméras dans le cloud à des fins d'analyse.

Contoso aimerait trouver une façon discrète et respectueuse de la vie privée d’identifier certaines caractéristiques de sa clientèle, notamment le nombre de clients, leur loyauté et leurs réactions face aux présentoirs et produits proposés en magasin.

## <a name="solution"></a>Solution

Ce modèle d’analyse commerciale utilise une approche à plusieurs niveaux pour l’inférence à la périphérie. En utilisant le kit de développement personnalisé Custom Vision, seules les images affichant un visage humain sont envoyées pour analyse à une instance Azure Stack Hub privée qui exécute Azure Cognitive Services. Les données anonymisées et agrégées sont envoyées à Azure pour être agrégées dans tous les magasins puis visualisées dans Power BI. La combinaison d’un cloud périphérique et d’un cloud public permet à Contoso de tirer parti de la technologie IA moderne tout en restant en conformité avec ses stratégies d’entreprise et en respectant la vie privée de ses clients.

[![Solution de modèle de détection des pas](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Voici un résumé du fonctionnement de la solution :

1. Le kit de développement personnalisé Vision AI reçoit une configuration de la part d’IoT Hub, qui installe le runtime IoT Edge et un modèle ML.
2. Si le modèle détecte une personne, il prend une photo et la charge sur le stockage Blob Azure Stack Hub.
3. Le service Blob déclenche une fonction Azure sur Azure Stack Hub.
4. La fonction Azure appelle un conteneur avec l'API Visage pour obtenir des données démographiques et émotionnelles de l'image.
5. Les données sont anonymisées et envoyées à un cluster Azure Event Hubs.
6. Le cluster Event Hubs transmet les données à Stream Analytics.
7. Stream Analytics agrège les données et les transmet à Power BI.

## <a name="components"></a>Components

Cette solution utilise les composants suivants :

| Couche | Composant | Description |
|----------|-----------|-------------|
| Matériel en magasin | [Kit de développement d’intelligence artificielle personnalisé Vision AI](https://azure.github.io/Vision-AI-DevKit-Pages/) | Fournit un filtrage en magasin à l'aide d'un modèle ML local qui capture des images de personnes uniquement à des fins d'analyse. Configuré et mis à jour de façon sécurisée via l’IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs fournit une plate-forme évolutive pour l'acquisition de données anonymisées qui s'intègre parfaitement à Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Une tâche Azure Stream Analytics agrège les données anonymisées et les regroupe dans des fenêtres de 15 secondes pour les visualiser. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI fournit une interface de tableau de bord conviviale pour visualiser les résultats issus d’Azure Stream Analytics. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | Le fournisseur de ressources App Service fournit une base pour les composants périphériques, notamment les fonctionnalités d’hébergement et de gestion pour les applications web/API et les fonctions. |
| | Cluster du moteur Azure Kubernetes Service [(AKS)](https://github.com/Azure/aks-engine) | Le fournisseur de ressources AKS avec le cluster du moteur AKS déployé dans Azure Stack Hub fournit un moteur résilient et scalable pour exécuter le conteneur de l’API Visage. |
| | Azure Cognitive Services [Conteneurs de l’API Visage](/azure/cognitive-services/face/face-how-to-install-containers)| Le fournisseur de ressources Azure Cognitive Services avec les conteneurs de l’API Visage offre une détection démographique, émotionnelle et unique des visiteurs sur le réseau privé de Contoso. |
| | Stockage Blob | Les images capturées à partir du kit de développement d’intelligence artificielle sont chargées sur le stockage Blob Azure Stack Hub. |
| | Azure Functions | Une fonction Azure s’exécutant sur Azure Stack Hub reçoit les données du stockage blob et gère les interactions avec l’API Visage. Elle transmet des données anonymisées vers un cluster Event Hubs qui se trouve dans Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problèmes et considérations

Prenez en compte des points suivants lors du choix de l'implémentation de cette solution :

### <a name="scalability"></a>Extensibilité

Pour permettre à cette solution de s'adapter à plusieurs caméras et emplacements, vous devez vous assurer que tous les composants peuvent gérer cette surcharge. Vous devrez peut-être prendre des mesures comme :

- Augmenter le nombre d’unités de streaming Stream Analytics.
- Effectuer un scale-out du déploiement de l’API Visage.
- Augmenter le débit du cluster Event Hubs.
- Dans les cas extrêmes, il peut être nécessaire de migrer des fonctions Azure vers une machine virtuelle.

### <a name="availability"></a>Disponibilité

Comme cette solution comporte plusieurs niveaux, il est important de réfléchir à la façon de faire face aux pannes de réseau ou d'électricité. Selon les besoins métier, vous pouvez mettre en place un mécanisme permettant de mettre en cache des images localement, puis de les transférer vers Azure Stack Hub quand la connectivité est rétablie. Si l'emplacement est suffisamment grand, le déploiement d'un appareil Data Box Edge avec le conteneur API Visage sur cet emplacement peut être préférable.

### <a name="manageability"></a>Simplicité de gestion

Cette solution peut couvrir de nombreux appareils et emplacements, et donc devenir difficile à gérer. Les [services IoT d’Azure](/azure/iot-fundamentals/) peuvent être utilisés pour mettre automatiquement en ligne de nouveaux emplacements et appareils et les maintenir à jour.

### <a name="security"></a>Sécurité

Cette solution capturant des images des clients, la sécurité revêt ici une importance critique. Assurez-vous que tous les comptes de stockage sont sécurisés à l’aide des stratégies d’accès appropriées et modifiez les clés régulièrement. Veillez à ce que les comptes de stockage et Event Hubs disposent de stratégies de rétention conformes aux règlements sur la protection des informations personnelles définies par l'entreprise et le gouvernement. Veillez également à hiérarchiser les niveaux d’accès utilisateur. La hiérarchisation garantit que les utilisateurs ont uniquement accès aux données dont ils ont besoin pour leur rôle.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les sujets abordés dans cet article :

- Consultez le [modèle Données hiérarchisées](https://aka.ms/tiereddatadeploy), qui est exploité par le modèle de détection des pas.
- Pour en savoir plus sur l’utilisation de Custom Vision, consultez le [Kit de développement d’intelligence artificielle Custom Vision](https://azure.github.io/Vision-AI-DevKit-Pages/). 

Lorsque vous êtes prêt à tester l’exemple de solution, poursuivez avec le [Guide de déploiement de la solution de détection des pas](solution-deployment-guide-retail-footfall-detection.md). Ce guide de déploiement fournit des instructions pas à pas sur le déploiement et sur le test de ses composants.
