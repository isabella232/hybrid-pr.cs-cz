---
title: Nasazení vysoce dostupného řešení MongoDB do Azure a centra Azure Stack
description: Naučte se nasadit řešení MongoDB s vysokou dostupností do Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852503"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="83159-103">Nasazení vysoce dostupného řešení MongoDB do Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="83159-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="83159-104">Tento článek vás provede automatizovaným nasazením clusteru MongoDB s vysokou dostupností (HA) s lokalitou zotavení po havárii (DR) v rámci dvou Azure Stack hub prostředí.</span><span class="sxs-lookup"><span data-stu-id="83159-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="83159-105">Další informace o MongoDB a vysoké dostupnosti najdete v tématu věnovaném [členům sady replik](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="83159-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="83159-106">V tomto řešení vytvoříte ukázkové prostředí pro:</span><span class="sxs-lookup"><span data-stu-id="83159-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="83159-107">Orchestrujte nasazení v rámci dvou Azure Stackch Center.</span><span class="sxs-lookup"><span data-stu-id="83159-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="83159-108">K minimalizaci problémů s závislostmi s profily rozhraní API Azure použijte Docker.</span><span class="sxs-lookup"><span data-stu-id="83159-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="83159-109">Nasaďte základní vysoce dostupný cluster MongoDB s lokalitou pro zotavení po havárii.</span><span class="sxs-lookup"><span data-stu-id="83159-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="83159-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="83159-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="83159-111">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="83159-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="83159-112">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="83159-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="83159-113">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="83159-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="83159-114">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="83159-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="83159-115">Architektura pro MongoDB s rozbočovačem Azure Stack</span><span class="sxs-lookup"><span data-stu-id="83159-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![vysoce dostupná architektura MongoDB v centru Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="83159-117">Předpoklady pro MongoDB s rozbočovačem Azure Stack</span><span class="sxs-lookup"><span data-stu-id="83159-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="83159-118">Dva připojené systémy integrovaných Azure Stack hub (centrum Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="83159-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="83159-119">Toto nasazení nefunguje na Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="83159-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="83159-120">Další informace o centru Azure Stack najdete v tématu [co je Azure Stack hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="83159-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="83159-121">Předplatné tenanta v každém centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="83159-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="83159-122">**Poznamenejte si každé ID předplatného a Azure Resource Manager koncový bod pro každé centrum Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="83159-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="83159-123">Instanční objekt služby Azure Active Directory (Azure AD), který má oprávnění k předplatnému tenanta pro každé centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="83159-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="83159-124">Pokud jsou centra Azure Stack nasazená v různých klientech služby Azure AD, možná budete muset vytvořit dva instanční objekty.</span><span class="sxs-lookup"><span data-stu-id="83159-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="83159-125">Informace o tom, jak vytvořit instanční objekt pro centrum Azure Stack, najdete v tématu [použití identity aplikace pro přístup k prostředkům Azure Stack hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="83159-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="83159-126">**Poznamenejte si ID aplikace, tajný klíč klienta a název tenanta (xxxxx.onmicrosoft.com) daného instančního objektu.**</span><span class="sxs-lookup"><span data-stu-id="83159-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="83159-127">Ubuntu 16,04 se zaAzure Stack do každého tržiště centra.</span><span class="sxs-lookup"><span data-stu-id="83159-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="83159-128">Další informace o syndikaci na webu Marketplace najdete v tématu [stažení položek Marketplace do centra Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="83159-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="83159-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) nainstalované na místním počítači.</span><span class="sxs-lookup"><span data-stu-id="83159-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="83159-130">Získat image Docker</span><span class="sxs-lookup"><span data-stu-id="83159-130">Get the Docker image</span></span>

<span data-ttu-id="83159-131">Image Docker pro každé nasazení eliminují problémy závislosti mezi různými verzemi Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="83159-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="83159-132">Ujistěte se, že Docker for Windows používá kontejnery Windows.</span><span class="sxs-lookup"><span data-stu-id="83159-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="83159-133">Spuštěním následujícího příkazu na příkazovém řádku se zvýšenými oprávněními Získejte kontejner Docker se skripty nasazení.</span><span class="sxs-lookup"><span data-stu-id="83159-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="83159-134">Nasazení clusterů</span><span class="sxs-lookup"><span data-stu-id="83159-134">Deploy the clusters</span></span>

1. <span data-ttu-id="83159-135">Po úspěšném dokončení image kontejneru spusťte image.</span><span class="sxs-lookup"><span data-stu-id="83159-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="83159-136">Po spuštění kontejneru se v kontejneru udělí terminál PowerShellu se zvýšenými oprávněními.</span><span class="sxs-lookup"><span data-stu-id="83159-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="83159-137">Změňte adresáře tak, aby se získaly do skriptu nasazení.</span><span class="sxs-lookup"><span data-stu-id="83159-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="83159-138">Spusťte nasazení.</span><span class="sxs-lookup"><span data-stu-id="83159-138">Run the deployment.</span></span> <span data-ttu-id="83159-139">Zadejte přihlašovací údaje a názvy prostředků tam, kde je to potřeba.</span><span class="sxs-lookup"><span data-stu-id="83159-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="83159-140">HA odkazuje na centrum Azure Stack, ve kterém se cluster HA nasadí.</span><span class="sxs-lookup"><span data-stu-id="83159-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="83159-141">Nástroj DR odkazuje na centrum Azure Stack, do kterého bude nasazen cluster DR.</span><span class="sxs-lookup"><span data-stu-id="83159-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. <span data-ttu-id="83159-142">Zadejte, pokud chcete, aby se `Y` nainstaloval poskytovatel NuGet, který se aktivuje z profilu rozhraní API "2018-03-01-hybrid" modulů, které se mají nainstalovat.</span><span class="sxs-lookup"><span data-stu-id="83159-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="83159-143">Nejprve se nasadí prostředky HA.</span><span class="sxs-lookup"><span data-stu-id="83159-143">The HA resources will deploy first.</span></span> <span data-ttu-id="83159-144">Monitorujte nasazení a počkejte na jeho dokončení.</span><span class="sxs-lookup"><span data-stu-id="83159-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="83159-145">Jakmile se zobrazí zpráva s oznámením, že nasazení HA bylo dokončeno, můžete na portálu HA Azure Stack centra ověřit, jestli jsou nasazené prostředky.</span><span class="sxs-lookup"><span data-stu-id="83159-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="83159-146">Pokračujte v nasazení prostředků zotavení po havárii a rozhodněte se, jestli chcete povolit pole pro skok v Azure Stackovém centru DR pro interakci s clusterem.</span><span class="sxs-lookup"><span data-stu-id="83159-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="83159-147">Počkejte, než se dokončí nasazení prostředků DR.</span><span class="sxs-lookup"><span data-stu-id="83159-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="83159-148">Po dokončení nasazení prostředků DR se kontejner ukončí.</span><span class="sxs-lookup"><span data-stu-id="83159-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="83159-149">Další kroky</span><span class="sxs-lookup"><span data-stu-id="83159-149">Next steps</span></span>

- <span data-ttu-id="83159-150">Pokud jste v Azure Stackovém centru pro zotavení po havárii povolili virtuální počítač se seznamem odkazů, můžete se připojit přes SSH a komunikovat s clusterem MongoDB instalací rozhraní příkazového řádku Mongo.</span><span class="sxs-lookup"><span data-stu-id="83159-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="83159-151">Další informace o interakci s MongoDB najdete v tématu [prostředí Mongo](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="83159-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="83159-152">Další informace o hybridních cloudových aplikacích najdete v tématu [hybridní cloudová řešení.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="83159-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="83159-153">Upravte kód na tuto ukázku na [GitHubu](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="83159-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>