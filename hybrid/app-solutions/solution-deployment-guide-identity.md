---
title: Configurer une identité cloud hybride pour les applications Azure et Azure Stack Hub
description: Apprenez à configurer une identité cloud hybride pour les applications Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895344"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Configurer une identité cloud hybride pour les applications Azure et Azure Stack Hub

Apprenez à configurer une identité cloud hybride pour vos applications Azure et Azure Stack Hub.

Il existe deux moyens d'accorder l'accès à vos applications à la fois dans le service Azure global et dans Azure Stack Hub.

 * Si Azure Stack Hub dispose d'une connexion permanente à Internet, vous pouvez utiliser Azure Active Directory (Azure AD).
 * Si Azure Stack Hub est déconnecté d'Internet, vous pouvez opter pour les services de fédération Active Directory (AD FS).

Les principaux de service vous permettent d'accorder l'accès à vos applications Azure Stack Hub à des fins de déploiement ou de configuration à l'aide d'Azure Resource Manager dans Azure Stack Hub.

Dans cette solution, vous allez générer un exemple d’environnement pour :

> [!div class="checklist"]
> - établir une identité hybride dans Azure global et Azure Stack Hub ;
> - récupérer un jeton afin d'accéder à l'API Azure Stack Hub.

Pour suivre les étapes de cette solution, vous devez disposer d'autorisations d'opérateur Azure Stack Hub.

> [!Tip]  
> ![Diagramme des piliers hybrides](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub est une extension d’Azure. Azure Stack Hub apporte l’agilité et l’innovation du cloud computing à votre environnement local et active le seul cloud hybride qui vous permet de créer et de déployer des applications hybrides en tout lieu.  
> 
> L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) pour la conception, le déploiement et le fonctionnement des applications hybrides. Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Créer un principal du service pour Azure AD sur le portail

Si vous avez déployé Azure Stack Hub en utilisant Azure AD en tant que magasin d'identités, vous pouvez créer des principaux de service de la même façon que pour Azure. La rubrique [Utiliser une identité d’application pour accéder aux ressources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) montre comment effectuer les étapes via le portail. Avant de commencer, vérifiez que vous disposez des [autorisations Azure AD requises](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions).

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Créer un principal du service pour AD FS avec PowerShell

Si vous avez déployé Azure Stack Hub avec AD FS, vous pouvez utiliser PowerShell pour créer un principal de service, attribuer un rôle d'accès et vous connecter à partir de PowerShell à l'aide de cette identité. La rubrique [Utiliser une identité d’application pour accéder aux ressources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) montre comment procéder à l’aide de PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Utiliser l'API Azure Stack Hub

La solution [API Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use) décrit le processus de récupération d'un jeton pour accéder à l'API Azure Stack Hub.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Se connecter à Azure Stack Hub avec PowerShell

Le guide de démarrage rapide à suivre pour [devenir rapidement opérationnel avec PowerShell dans Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) vous explique comment installer Azure PowerShell et vous connecter à votre installation Azure Stack Hub.

### <a name="prerequisites"></a>Prérequis

Vous devez disposer d’une installation Azure Stack Hub connectée à Azure Active Directory, avec un abonnement auquel vous pouvez accéder. Si Azure Stack Hub n’est pas installé, vous pouvez suivre ces instructions pour configurer un [kit de développement Azure Stack (ASDK)](/azure-stack/asdk/asdk-install).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Se connecter à Azure Stack Hub à l'aide de code

Pour vous connecter à Azure Stack Hub à l’aide de code, utilisez l’API des points de terminaison Azure Resource Manager afin d’obtenir les points de terminaison d’authentification et Graph de votre installation Azure Stack Hub. Authentifiez-vous ensuite par le biais de demandes REST. Vous trouverez un exemple d’application cliente sur [GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>Si le kit de développement logiciel (SDK) Azure de la langue de votre choix ne prend pas en charge les profils d'API Azure, il risque de ne pas fonctionner avec Azure Stack Hub. Pour en savoir plus sur les profils d’API Azure, consultez l’article [Gérer les profils de version des API](/azure-stack/user/azure-stack-version-profiles).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d'informations sur la gestion des identités dans Azure Stack Hub, consultez [Architecture d'identité pour Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).
- Pour plus d’informations sur les modèles Azure Cloud, consultez [Modèles de conception cloud](/azure/architecture/patterns).
