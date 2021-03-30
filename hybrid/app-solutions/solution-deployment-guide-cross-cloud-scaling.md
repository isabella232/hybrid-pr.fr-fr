---
title: Déployer une application qui effectue une mise à l’échelle multicloud dans Azure et Azure Stack Hub
description: Découvrez comment déployer une application qui effectue une mise à l’échelle multicloud dans Azure et Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895412"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Déployer une application qui effectue une mise à l’échelle multicloud à l’aide d’Azure et d’Azure Stack Hub

Découvrez comment créer une solution multicloud pour fournir un processus déclenché manuellement permettant de passer d’une application web hébergée sur Azure Stack Hub à une application web hébergée sur Azure avec mise à l’échelle automatique via Traffic Manager. Ce processus garantit la disponibilité d’un utilitaire cloud flexible et évolutif sous charge.

Avec ce modèle, il se peut que votre locataire ne soit pas prêt à exécuter votre application dans le cloud public. Toutefois, ceci peut ne pas être économiquement faisable pour que l’entreprise puisse maintenir la capacité nécessaire dans son environnement local afin de gérer les pics de demande de l’application. Votre locataire peut faire usage de l’élasticité du cloud public avec sa solution locale.

Dans cette solution, vous allez générer un exemple d’environnement pour :

> [!div class="checklist"]
> - Créer une application web à plusieurs nœuds.
> - Configurer et gérer le processus de déploiement continu (CD).
> - Publier l’application web dans Azure Stack Hub.
> - Créer une mise en production.
> - Apprenez à surveiller et à suivre vos déploiements.

> [!Tip]  
> ![Diagramme des piliers hybrides](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub est une extension d’Azure. Azure Stack Hub apporte l’agilité et l’innovation du cloud computing à votre environnement local et active le seul cloud hybride qui vous permet de créer et de déployer des applications hybrides en tout lieu.  
> 
> L’article [Considérations sur la conception d’applications hybrides](overview-app-design-considerations.md) se penche sur les fondements de la qualité logicielle (sélection élective, scalabilité, disponibilité, résilience, facilité de gestion et sécurité) pour la conception, le déploiement et le fonctionnement des applications hybrides. Les considérations de conception vous aident à optimiser la conception d’application hybride, en réduisant les risques dans les environnements de production.

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure. Si nécessaire, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Système intégré Azure Stack Hub ou déploiement du kit de développement Azure Stack (ASDK).
  - Pour obtenir des instructions concernant l’installation d’Azure Stack Hub, consultez [Installer le kit ASDK](/azure-stack/asdk/asdk-install).
  - Pour un script d’automatisation post-déploiement ASDK, accédez à : [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Cette installation peut prendre quelques heures.
- Déployez les services PaaS [App Service](/azure-stack/operator/azure-stack-app-service-deploy) sur Azure Stack Hub.
- [Créez des plans/offres](/azure-stack/operator/service-plan-offer-subscription-overview) dans l’environnement Azure Stack Hub.
- [Créez un abonnement de locataire](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) dans l'environnement Azure Stack Hub.
- Créez une application web dans l’abonnement du locataire. Notez l’URL de la nouvelle application web. Vous en aurez besoin plus tard.
- Déployez la machine virtuelle Azure Pipelines au sein de l’abonnement du locataire.
- Une machine virtuelle Windows Server 2016 avec .NET 3.5 est nécessaire. Cette machine virtuelle est créée dans l’abonnement du locataire sur Azure Stack Hub en tant qu’agent de build privé.
- [Windows Server 2016 avec l’image de machine virtuelle SQL 2017](/azure-stack/operator/azure-stack-add-vm-image) est disponible dans la Place de marché Azure Stack Hub. Si cette image n’est pas disponible, utilisez un opérateur Azure Stack Hub pour vous assurer qu’elle est ajoutée à l’environnement.

## <a name="issues-and-considerations"></a>Problèmes et considérations

### <a name="scalability"></a>Extensibilité

Le principal composant de la mise à l’échelle inter-cloud est la capacité à fournir une mise à l’échelle à la demande immédiate entre l’infrastructure cloud publique et locale, garantissant un service fiable et cohérent.

### <a name="availability"></a>Disponibilité

Vérifiez que les applications déployées en local sont configurées pour la haute disponibilité via la configuration matérielle locale et le déploiement de logiciels.

### <a name="manageability"></a>Simplicité de gestion

La solution dans le cloud garantit une gestion transparente et une interface familière entre les environnements. PowerShell est recommandé pour une gestion multiplateforme.

## <a name="cross-cloud-scaling"></a>Mise à l’échelle multicloud

### <a name="get-a-custom-domain-and-configure-dns"></a>Obtenir un domaine personnalisé et configurer DNS

Mettez à jour le fichier de zone DNS pour le domaine. Azure AD vérifie la propriété du nom de domaine personnalisé. Utilisez [Azure DNS](/azure/dns/dns-getstarted-portal) pour les enregistrements DNS Azure/Microsoft 365/externes dans Azure, ou ajoutez l’entrée DNS à [un autre bureau d’enregistrement DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

1. Inscrivez un domaine personnalisé auprès d’un bureau d’enregistrement public.
2. Connectez-vous au Bureau d’enregistrement des noms de domaine pour le domaine. Il se peut qu’un administrateur approuvé doive effectuer les mises à jour DNS.
3. Mettez à jour le fichier de zone DNS du domaine en ajoutant l’entrée DNS fournie par Azure AD. (L’entrée DNS n’affecte pas le routage des e-mails ni les comportements d’hébergement web.)

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Créer une application web à plusieurs nœuds par défaut dans Azure Stack Hub

Configurez l’intégration continue/le déploiement continu (CI/CD) hybride pour déployer des applications web sur Azure et Azure Stack Hub, et envoyer automatiquement les modifications aux deux clouds.

> [!Note]  
> Vous avez besoin d’Azure Stack Hub avec les images appropriées syndiquées pour s’exécuter (Windows Server et SQL), et App Service doit être déployé. Pour plus d’informations, consultez la documentation App Service [Prérequis pour le déploiement d’App Service sur Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).

### <a name="add-code-to-azure-repos"></a>Ajouter du code à Azure Repos

Azure Repos

1. Connectez-vous à Azure Repos avec un compte ayant les droits de création de projet sur Azure Repos.

    La CI/CD hybride peut s’appliquer tant au code d’application qu’au code d’infrastructure. Utilisez les [modèles Azure Resource Manager](https://azure.microsoft.com/resources/templates/) pour les développements cloud privé et hébergé.

    ![Se connecter à un projet sur Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Clonez le référentiel** en créant et en ouvrant l’application web par défaut.

    ![Cloner un référentiel dans une application web Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Créer un déploiement d’applications web autonomes pour App Services dans les deux clouds

1. Modifiez le fichier **WebApplication.csproj**. Sélectionnez `Runtimeidentifier`, puis ajoutez `win10-x64`. (Consultez la documentation sur le [déploiement autonome](/dotnet/core/deploying/deploy-with-vs#simpleSelf).)

    ![Modifier un fichier de projet d’application web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Archivez le code dans Azure Repos avec Team Explorer.

3. Vérifiez que le code d’application a été archivé dans Azure Repos.

## <a name="create-the-build-definition"></a>Créer la définition de build

1. Connectez-vous à Azure Pipelines pour vérifier la possibilité de créer des définitions de build.

2. Ajoutez le code **-r win10-x64**. Cet ajout est nécessaire pour déclencher un déploiement autonome avec .NET Core.

    ![Ajouter du code à l’application web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Exécutez la build. Le processus de [build de déploiement autonome](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publie des artefacts qui s’exécutent sur Azure et Azure Stack Hub.

## <a name="use-an-azure-hosted-agent"></a>Utiliser un agent hébergé sur Azure

L’utilisation d’un agent de build hébergé dans Azure Pipelines est une option pratique pour créer et déployer des applications web. La maintenance et les mises à niveau sont effectuées automatiquement par Microsoft Azure, ce qui permet un cycle de développement continu et sans interruption.

### <a name="manage-and-configure-the-cd-process"></a>Gérer et configurer le processus CD

Azure Pipelines et Azure DevOps Services fournissent un pipeline hautement configurable et gérable pour des mises en production sur plusieurs environnements, comme des environnements de développement, de préproduction, d’assurance qualité et de production, avec des demandes d’approbation à des étapes spécifiques.

## <a name="create-release-definition"></a>Créer une définition de mise en production

1. Sélectionnez le bouton **plus** pour ajouter une nouvelle mise en production sous l’onglet **Mises en production** dans la section **Build et mise en production** d’Azure DevOps Services.

    ![Création d’une définition de version](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Appliquez le modèle de déploiement Azure App Service.

   ![Appliquer un modèle de déploiement Azure App Service](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. Sous **Ajouter un artefact**, ajoutez l’artefact pour l’application de build Azure Cloud.

   ![Ajouter un artefact à la build Azure Cloud](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. Sous l’onglet Pipeline, sélectionnez le lien **Phase, Tâche** de l’environnement, et définissez les valeurs de l’environnement cloud Azure.

   ![Définir des valeurs d’environnement cloud Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Définissez le **nom d’environnement** et sélectionnez l’**abonnement Azure** pour le point de terminaison du Cloud Azure.

      ![Sélectionner un abonnement Azure pour le point de terminaison du Cloud Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. Sous **Nom de l’App Service**, définissez le nom de service de l’application Azure.

      ![Définir un nom de service d’application Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Entrez VS2017 hébergé sous **File d’attente de l’agent** pour l’environnement hébergé dans le cloud Azure.

      ![Définir une file d’attente pour l’environnement hébergé dans le cloud Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. Dans le menu Déployer Azure App Service, sélectionnez **le package ou le dossier** valide pour l’environnement. Sélectionnez **OK** pour **l’emplacement du dossier**.
  
      ![Sélectionner un package ou un dossier pour l’environnement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Boîte de dialogue du sélecteur de dossiers 1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Enregistrez toutes les modifications et revenez au **pipeline de mises en production**.

    ![Enregistrer les modifications dans le pipeline de mise en production](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Ajoutez un nouvel artefact en sélectionnant la build pour l’application Azure Stack Hub.

    ![Ajouter un nouvel artefact pour un application Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Ajoutez un environnement supplémentaire en appliquant le déploiement Azure App Service.

    ![Ajouter un environnement à un déploiement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Nommez le nouvel environnement « Azure Stack ».

    ![Nommer un environnement dans un déploiement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Recherchez l’environnement Azure Stack sous l’onglet **Tâche**.

    ![Environnement Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Sélectionnez l’abonnement pour le point de terminaison Azure Stack.

    ![Sélectionner l’abonnement pour le point de terminaison Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Définissez le nom de l’application web Azure Stack comme nom de l’App Service.
    ![Définir un nom d’application web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Sélectionnez l’agent Azure Stack.

    ![Sélectionner l’agent Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Sous la section Déployer Azure App Service, sélectionnez **le package ou le dossier** valides pour l’environnement. Sélectionnez **OK** pour l’emplacement du dossier.

    ![Sélectionner un dossier pour un déploiement Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Boîte de dialogue du sélecteur de dossiers 2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. Sous l’onglet Variable, ajoutez une variable nommée `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, définissez sa valeur sur **true** et sa portée sur Azure Stack.

    ![Ajouter une variable à un déploiement Azure App](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Sélectionnez l’icône du déclencheur de déploiement **Continu** dans les deux artefacts et activez le déclencheur de déploiement **Continu**.

    ![Sélectionner un déclencheur de déploiement continu](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Sélectionnez l’icône des conditions **Prédéploiement** dans l’environnement Azure Stack et définissez le déclencheur sur **Après la mise en production**.

    ![Sélectionner des conditions préalables au déploiement](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Enregistrez toutes les modifications.

> [!Note]  
> Certains paramètres des tâches peuvent avoir été automatiquement définis en tant que [variables d’environnement](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) lors de la création d’une définition de mise en production à partir d’un modèle. Ces paramètres ne peuvent pas être modifiés dans les paramètres de la tâche. Au lieu de cela, l’élément d’environnement parent doit être sélectionné pour modifier ces paramètres.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Publier sur Azure Stack Hub via Visual Studio

En créant des points de terminaison, une build Azure DevOps Services peut déployer des applications Azure App Service sur Azure Stack Hub. Azure Pipelines se connecte à l’agent de build, qui se connecte à Azure Stack Hub.

1. Connectez-vous à Azure DevOps Services et accédez à la page Paramètres de l’application.

2. Dans **Paramètres**, sélectionnez **Sécurité**.

3. Dans **Groupes VSTS**, sélectionnez **Créateurs de points de terminaison**.

4. Sous l’onglet **Membres**, sélectionnez **Ajouter**.

5. Dans **Ajouter des utilisateurs et groupes**, entrez un nom d’utilisateur et sélectionnez cet utilisateur dans la liste des utilisateurs.

6. Sélectionnez **Enregistrer les modifications**.

7. Dans la liste **Groupes VSTS**, sélectionnez **Administrateurs du point de terminaison**.

8. Sous l’onglet **Membres**, sélectionnez **Ajouter**.

9. Dans **Ajouter des utilisateurs et groupes**, entrez un nom d’utilisateur et sélectionnez cet utilisateur dans la liste des utilisateurs.

10. Sélectionnez **Enregistrer les modifications**.

Maintenant que les informations du point de terminaison ont été ajoutées, la connexion entre Azure Pipelines et Azure Stack Hub est prête à être utilisée. L’agent de build dans Azure Stack Hub reçoit des instructions d’Azure Pipelines, puis transmet les informations du point de terminaison pour la communication avec Azure Stack Hub.

## <a name="develop-the-app-build"></a>Développer la build de l’application

> [!Note]  
> Vous avez besoin d’Azure Stack Hub avec les images appropriées syndiquées pour s’exécuter (Windows Server et SQL), et App Service doit être déployé. Pour plus d’informations, consultez [Conditions préalables au déploiement d’App Service sur Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).

Utilisez des [modèles Azure Resource Manager](https://azure.microsoft.com/resources/templates/) comme du code d’application web d’Azure Repos pour le déploiement dans les deux clouds.

### <a name="add-code-to-an-azure-repos-project"></a>Ajouter du code à un projet Azure Repos

1. Connectez-vous à Azure Repos avec un compte ayant les droits de création de projet sur Azure Stack Hub.

2. **Clonez le référentiel** en créant et en ouvrant l’application web par défaut.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Créer un déploiement d’applications web autonomes pour App Services dans les deux clouds

1. Modifiez le fichier **WebApplication.csproj** : Sélectionnez `Runtimeidentifier`, puis ajoutez `win10-x64`. Pour plus d’informations, consultez la documentation sur le [déploiement autonome](/dotnet/core/deploying/deploy-with-vs#simpleSelf).

2. Utilisez Team Explorer pour archiver le code dans Azure Repos.

3. Vérifiez que le code d’application a été archivé dans Azure Repos.

### <a name="create-the-build-definition"></a>Créer la définition de build

1. Connectez-vous à Azure Pipelines avec un compte permettant de créer une définition de build.

2. Accédez à la page **Build Web Application** (Créer une application web) du projet.

3. Dans **Arguments**, ajoutez le code **-r win10-x64**. Cet ajout est obligatoire pour déclencher un déploiement autonome avec .NET Core.

4. Exécutez la build. Le processus de [build de déploiement autonome](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publie des artefacts qui peuvent s’exécuter sur Azure et Azure Stack Hub.

#### <a name="use-an-azure-hosted-build-agent"></a>Utiliser un agent de build hébergé Azure

L’utilisation d’un agent de build hébergé dans Azure Pipelines est une option pratique pour créer et déployer des applications web. La maintenance et les mises à niveau sont effectuées automatiquement par Microsoft Azure, ce qui permet un cycle de développement continu et sans interruption.

### <a name="configure-the-continuous-deployment-cd-process"></a>Configurer le processus de déploiement continu (CD)

Azure Pipelines et Azure DevOps Services fournissent un pipeline hautement configurable et gérable pour des mises en production sur plusieurs environnements, comme des environnements de développement, de préproduction, d’assurance qualité et de production. Ce processus peut inclure des demandes d’approbation à des étapes spécifiques du cycle de vie de l’application.

#### <a name="create-release-definition"></a>Créer une définition de mise en production

La création d’une définition de mise en production est la dernière étape du processus de création d’application. Cette définition de mise en production est utilisée pour créer une mise en production et déployer une build.

1. Connectez-vous à Azure Pipelines et accédez à **Build et mise en production** pour le projet.

2. Sous l’onglet **Versions**, sélectionnez **[ + ]** , puis choisissez **Créer une définition de mise en production**.

3. Sous **Sélectionner un modèle**, choisissez **Déploiement d'Azure App Service**, puis sélectionnez **Appliquer**.

4. Sous **Ajouter un artefact**, dans **Source (définition de build)** , sélectionnez l’application de build Azure Cloud.

5. Sous l’onglet **Pipeline**, sélectionnez le lien **1 Phase**, **1 Tâche** vers **Afficher les tâches d’environnement**.

6. Sous l’onglet **Tâches**, entrez Azure comme **nom d’environnement** et sélectionnez AzureCloud Traders-Web EP dans la liste **Abonnement Azure**.

7. Entrez le **nom de l’Azure App Service**, qui est `northwindtraders` dans la capture d’écran suivante.

8. Pour la phase d’agent, sélectionnez **VS2017 hébergé** dans la liste **File d’attente d’agents**.

9. Dans **Déployer Azure App Service**, sélectionnez le **package ou dossier** valide pour l’environnement.

10. Dans **Sélectionner un fichier ou un dossier**, sélectionnez **OK** pour **Emplacement**.

11. Enregistrez toutes les modifications et revenez à **Pipeline**.

12. Sous l’onglet **Pipeline**, sélectionnez **Ajouter un artefact** et choisissez **NorthwindCloud Traders-Vessel** dans la liste **Source (définition de build)** .

13. Sous **Sélectionner un modèle**, ajoutez un autre environnement. Choisissez **Déploiement d'Azure App Service**, puis sélectionnez **Appliquer**.

14. Entrez `Azure Stack Hub` comme **nom d’environnement**.

15. Sous l’onglet **Tâches**, recherchez et sélectionnez Azure Stack Hub.

16. Dans la liste **Abonnement Azure**, sélectionnez **AzureStack Traders-Vessel EP** pour le point de terminaison Azure Stack Hub.

17. Entrez le nom de l’application web Azure Stack Hub comme **nom de l’App Service**.

18. Sous **Sélection de l’Agent**, choisissez **AzureStack -b Douglas Fir** dans la liste **File d’attente d’agents**.

19. Pour **Déployer Azure App Service**, sélectionnez le **package ou dossier** valide pour l’environnement. Sous **Sélectionner un fichier ou un dossier**, sélectionnez **OK** pour le dossier **Emplacement**.

20. Sous l’onglet **Variable**, recherchez la variable nommée `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`. Définissez la valeur de la variable sur **true** et définissez sa portée sur **Azure Stack Hub**.

21. Sous l’onglet **Pipeline**, sélectionnez l’icône **Déclencheur de déploiement continu** pour l’artefact NorthwindCloud Traders-Web et définissez le **Déclencheur de déploiement continu** sur **Activé**. Procédez de la même manière pour l’artefact **NorthwindCloud Traders-Vessel**.

22. Pour l’environnement Azure Stack Hub, sélectionnez l’icône **Conditions préalables au déploiement** et définissez le déclencheur sur **Après la mise en production**.

23. Enregistrez toutes les modifications.

> [!Note]  
> Certains paramètres des tâches de mise en production sont automatiquement définis en tant que [variables d’environnement](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) lors de la création d’une définition de mise en production à partir d’un modèle. Ces paramètres ne peuvent pas être modifiés dans les paramètres de tâche, mais peuvent l’être dans les éléments d’environnement parent.

## <a name="create-a-release"></a>Créer une mise en production

1. Sous l’onglet **Pipeline**, ouvrez la liste **Version finale**, et choisissez **Créer une mise en production**.

2. Entrez une description de la mise en production, vérifiez que les artefacts corrects sont sélectionnés, puis sélectionnez **Créer**. Après quelques instants, une bannière s’affiche, indiquant que la nouvelle mise en production a été créée, et le nom de la mise en production est affichée sous forme de lien. Sélectionnez le lien pour consulter la page récapitulative de la mise en production.

3. La page récapitulative de mise en production fournit des détails sur la mise en production. Dans la capture d’écran suivante pour « Release-2 », la section **Environnements** indique que l’**état du déploiement** d’Azure est « IN PROGRESS » (En cours) et que l’état d’Azure Stack Hub est « SUCCEEDED » (Réussi). Lorsque l’état du déploiement de l’environnement Azure passe à « SUCCEEDED » (Réussi), une bannière indiquant que la mise en production est prête pour l’approbation s’affiche. Lorsqu’un déploiement est en attente ou a échoué, une icône d’informations **(i)** bleue est affichée. Pointez sur l’icône pour afficher une fenêtre contextuelle qui indique le motif du retard ou de l’échec.

4. D’autres vues, comme la liste des mises en production, affichent aussi une icône indiquant qu’une approbation est en attente. La fenêtre contextuelle de cette icône indique le nom de l’environnement et plus de détails sur le déploiement. Un administrateur peut facilement voir la progression globale des mises en production et savoir quelles mises en production sont en attente d’approbation.

## <a name="monitor-and-track-deployments"></a>Surveiller et suivre les déploiements

1. Sur la page récapitulative **Release-2**, sélectionnez **Journaux d’activité**. Pendant un déploiement, cette page affiche le journal en direct de l’agent. Le volet gauche indique l’état de chaque opération du déploiement pour chaque environnement.

2. Sélectionnez l’icône représentant une personne dans la colonne **Action** pour une approbation avant déploiement ou après déploiement afin de voir qui a approuvé (ou rejeté) le déploiement ainsi que le message que ces personnes ont fourni.

3. Lorsque le déploiement est terminé, l’intégralité du fichier journal s’affiche dans le volet droit. Sélectionnez une **étape** dans le volet gauche pour voir le fichier journal d’une étape spécifique comme **Initialiser le travail**. La possibilité de voir les journaux d’activité individuels facilite le suivi et le débogage de parties du déploiement global. **Enregistrez** le fichier journal d’une étape ou **téléchargez tous les journaux au format zip**.

4. Ouvrez l’onglet **Résumé** pour afficher des informations générales sur la mise en production. Cette vue montre les détails de la build, les environnements où elle a été déployée, l’état du déploiement et d’autres informations sur la mise en production.

5. Sélectionnez un lien d’environnement (**Azure** ou **Azure Stack Hub**) pour afficher des informations sur les déploiements existants et en attente dans un environnement spécifique. Utilisez ces vues pour vérifier rapidement que la même build a été déployée sur les deux environnements.

6. Ouvrez l’**application de production déployée** dans un navigateur. Par exemple, pour le site web Azure App Services, ouvrez l’URL `https://[your-app-name\].azurewebsites.net`.

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>L’intégration d’Azure et d’Azure Stack Hub fournit une solution multicloud scalable

Un service à plusieurs clouds flexible et fiable offre sécurité des données, sauvegarde et redondance, disponibilité rapide et cohérente, stockage et distribution évolutifs, et routage conforme géographiquement. Ce processus déclenché manuellement garantit une commutation de charge fiable et efficace entre les applications web hébergées, ainsi qu’une disponibilité immédiate des données critiques.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les modèles Azure Cloud, consultez [Modèles de conception cloud](/azure/architecture/patterns).
