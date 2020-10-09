---
title: Nasazení skupiny dostupnosti SQL Server 2016 do Azure a centra Azure Stack
description: Naučte se nasadit skupinu dostupnosti SQL Server 2016 do Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852469"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="eeac6-103">Nasazení skupiny dostupnosti SQL Server 2016 do Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="eeac6-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="eeac6-104">Tento článek vás provede automatizovaným nasazením základního vysoce dostupného clusteru s vysokou dostupností (HA) SQL Server 2016 Enterprise s asynchronním serverem pro zotavení po havárii ve dvou Azure Stack hub prostředích.</span><span class="sxs-lookup"><span data-stu-id="eeac6-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="eeac6-105">Další informace o SQL Server 2016 a vysoké dostupnosti najdete v tématu [skupiny dostupnosti Always On: řešení zotavení po havárii s vysokou dostupností](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="eeac6-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="eeac6-106">V tomto řešení sestavíte ukázkové prostředí pro:</span><span class="sxs-lookup"><span data-stu-id="eeac6-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="eeac6-107">Orchestrujte nasazení v rámci dvou Azure Stackch Center.</span><span class="sxs-lookup"><span data-stu-id="eeac6-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="eeac6-108">K minimalizaci problémů s závislostmi s profily rozhraní API Azure použijte Docker.</span><span class="sxs-lookup"><span data-stu-id="eeac6-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="eeac6-109">Nasaďte základní vysoce dostupný SQL Server 2016 Enterprise cluster s lokalitou pro zotavení po havárii.</span><span class="sxs-lookup"><span data-stu-id="eeac6-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="eeac6-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="eeac6-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="eeac6-111">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="eeac6-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="eeac6-112">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="eeac6-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="eeac6-113">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="eeac6-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="eeac6-114">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="eeac6-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="eeac6-115">Architektura pro SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="eeac6-115">Architecture for SQL Server 2016</span></span>

![Centrum Azure Stack SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="eeac6-117">Předpoklady pro SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="eeac6-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="eeac6-118">Dva připojené systémy integrovaných Azure Stack hub (centrum Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="eeac6-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="eeac6-119">Toto nasazení nefunguje na Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="eeac6-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="eeac6-120">Další informace o centru Azure Stack najdete v tématu [přehled Azure Stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="eeac6-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="eeac6-121">Předplatné tenanta v každém centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="eeac6-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="eeac6-122">**Poznamenejte si každé ID předplatného a Azure Resource Manager koncový bod pro každé centrum Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="eeac6-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="eeac6-123">Instanční objekt služby Azure Active Directory (Azure AD), který má oprávnění k předplatnému tenanta pro každé centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="eeac6-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="eeac6-124">Pokud jsou centra Azure Stack nasazená v různých klientech služby Azure AD, možná budete muset vytvořit dva instanční objekty.</span><span class="sxs-lookup"><span data-stu-id="eeac6-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="eeac6-125">Informace o tom, jak vytvořit instanční objekt pro centrum Azure Stack, najdete v tématu [Vytvoření instančních objektů a udělení přístupu aplikacím k prostředkům služby Azure Stack hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="eeac6-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="eeac6-126">**Poznamenejte si ID aplikace, tajný klíč klienta a název tenanta (xxxxx.onmicrosoft.com) daného instančního objektu.**</span><span class="sxs-lookup"><span data-stu-id="eeac6-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="eeac6-127">SQL Server 2016 Enterprise se do každého tržiště centra Azure Stack zasyndikátoval.</span><span class="sxs-lookup"><span data-stu-id="eeac6-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="eeac6-128">Další informace o syndikaci na webu Marketplace najdete v tématu [stažení položek Marketplace do centra Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="eeac6-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="eeac6-129">**Ujistěte se, že má vaše organizace odpovídající licence SQL.**</span><span class="sxs-lookup"><span data-stu-id="eeac6-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="eeac6-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) nainstalované na místním počítači.</span><span class="sxs-lookup"><span data-stu-id="eeac6-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="eeac6-131">Získat image Docker</span><span class="sxs-lookup"><span data-stu-id="eeac6-131">Get the Docker image</span></span>

<span data-ttu-id="eeac6-132">Image Docker pro každé nasazení eliminují problémy závislosti mezi různými verzemi Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="eeac6-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="eeac6-133">Ujistěte se, že Docker for Windows používá kontejnery Windows.</span><span class="sxs-lookup"><span data-stu-id="eeac6-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="eeac6-134">Spuštěním následujícího skriptu na příkazovém řádku se zvýšenými oprávněními Získejte kontejner Docker se skripty nasazení.</span><span class="sxs-lookup"><span data-stu-id="eeac6-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="eeac6-135">Nasazení skupiny dostupnosti</span><span class="sxs-lookup"><span data-stu-id="eeac6-135">Deploy the availability group</span></span>

1. <span data-ttu-id="eeac6-136">Po úspěšném dokončení image kontejneru spusťte image.</span><span class="sxs-lookup"><span data-stu-id="eeac6-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="eeac6-137">Po spuštění kontejneru se v kontejneru udělí terminál PowerShellu se zvýšenými oprávněními.</span><span class="sxs-lookup"><span data-stu-id="eeac6-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="eeac6-138">Změňte adresáře tak, aby se získaly do skriptu nasazení.</span><span class="sxs-lookup"><span data-stu-id="eeac6-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="eeac6-139">Spusťte nasazení.</span><span class="sxs-lookup"><span data-stu-id="eeac6-139">Run the deployment.</span></span> <span data-ttu-id="eeac6-140">Zadejte přihlašovací údaje a názvy prostředků tam, kde je to potřeba.</span><span class="sxs-lookup"><span data-stu-id="eeac6-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="eeac6-141">HA odkazuje na centrum Azure Stack, ve kterém se cluster HA nasadí.</span><span class="sxs-lookup"><span data-stu-id="eeac6-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="eeac6-142">Nástroj DR odkazuje na centrum Azure Stack, do kterého bude nasazen cluster DR.</span><span class="sxs-lookup"><span data-stu-id="eeac6-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="eeac6-143">Zadejte, pokud chcete, aby se `Y` nainstaloval poskytovatel NuGet, který se aktivuje z profilu rozhraní API "2018-03-01-hybrid" modulů, které se mají nainstalovat.</span><span class="sxs-lookup"><span data-stu-id="eeac6-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="eeac6-144">Počkejte, až se nasazení prostředků dokončí.</span><span class="sxs-lookup"><span data-stu-id="eeac6-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="eeac6-145">Po dokončení nasazení prostředků DR se kontejner ukončí.</span><span class="sxs-lookup"><span data-stu-id="eeac6-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="eeac6-146">Prozkoumejte nasazení zobrazením prostředků na portálu centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="eeac6-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="eeac6-147">Připojte se k jedné z instancí SQL v prostředí HA a zkontrolujte skupinu dostupnosti pomocí SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="eeac6-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="eeac6-149">Další kroky</span><span class="sxs-lookup"><span data-stu-id="eeac6-149">Next steps</span></span>

- <span data-ttu-id="eeac6-150">Použijte SQL Server Management Studio k ručnímu převzetí služeb při selhání clusteru.</span><span class="sxs-lookup"><span data-stu-id="eeac6-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="eeac6-151">Viz [provedení vynuceného ručního převzetí služeb při selhání skupiny dostupnosti Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017) .</span><span class="sxs-lookup"><span data-stu-id="eeac6-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="eeac6-152">Přečtěte si další informace o hybridních cloudových aplikacích.</span><span class="sxs-lookup"><span data-stu-id="eeac6-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="eeac6-153">Podívejte se na [hybridní cloudová řešení.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="eeac6-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="eeac6-154">Použijte vlastní data nebo upravte kód v této ukázce na [GitHubu](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="eeac6-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>