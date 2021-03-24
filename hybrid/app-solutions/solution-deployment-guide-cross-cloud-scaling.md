---
title: Nasazení aplikace, která škáluje mezi cloudy v Azure a centra Azure Stack
description: Naučte se, jak nasadit aplikaci, která škáluje více cloudů v Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895410"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="9907b-103">Nasazení aplikace, která škáluje více cloudů pomocí Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="9907b-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="9907b-104">Naučte se vytvářet řešení pro více cloudů, aby bylo možné ručně aktivovaný proces přepnutí z webové aplikace hostovaného centra Azure Stack do hostované webové aplikace Azure pomocí automatického škálování prostřednictvím Traffic Manageru.</span><span class="sxs-lookup"><span data-stu-id="9907b-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="9907b-105">Tento proces zajišťuje flexibilní a škálovatelný cloudový nástroj při zatížení.</span><span class="sxs-lookup"><span data-stu-id="9907b-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="9907b-106">V tomto modelu nemusí být váš tenant připravený na spuštění vaší aplikace ve veřejném cloudu.</span><span class="sxs-lookup"><span data-stu-id="9907b-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="9907b-107">Nemusí ale být hospodářsky proveditelné, aby společnost udržovala kapacitu potřebnou v místním prostředí, aby mohla zpracovávat špičky v poptávce pro aplikaci.</span><span class="sxs-lookup"><span data-stu-id="9907b-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="9907b-108">Váš tenant může využít pružnost veřejného cloudu s místním řešením.</span><span class="sxs-lookup"><span data-stu-id="9907b-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="9907b-109">V tomto řešení sestavíte ukázkové prostředí pro:</span><span class="sxs-lookup"><span data-stu-id="9907b-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="9907b-110">Vytvořte si webovou aplikaci s více uzly.</span><span class="sxs-lookup"><span data-stu-id="9907b-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="9907b-111">Nakonfigurujte a spravujte proces průběžného nasazování (CD).</span><span class="sxs-lookup"><span data-stu-id="9907b-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="9907b-112">Publikujte webovou aplikaci do centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="9907b-113">Vytvořte verzi.</span><span class="sxs-lookup"><span data-stu-id="9907b-113">Create a release.</span></span>
> - <span data-ttu-id="9907b-114">Naučte se monitorovat a sledovat vaše nasazení.</span><span class="sxs-lookup"><span data-stu-id="9907b-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="9907b-115">![Diagram hybridních pilířů](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="9907b-115">![hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="9907b-116">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="9907b-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="9907b-117">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="9907b-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="9907b-118">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="9907b-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="9907b-119">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="9907b-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="9907b-120">Předpoklady</span><span class="sxs-lookup"><span data-stu-id="9907b-120">Prerequisites</span></span>

- <span data-ttu-id="9907b-121">Předplatné Azure.</span><span class="sxs-lookup"><span data-stu-id="9907b-121">Azure subscription.</span></span> <span data-ttu-id="9907b-122">V případě potřeby vytvořte si [bezplatný účet](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) před tím, než začnete.</span><span class="sxs-lookup"><span data-stu-id="9907b-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="9907b-123">Integrovaný systém Azure Stack nebo nasazení Azure Stack Development Kit (ASDK) hub.</span><span class="sxs-lookup"><span data-stu-id="9907b-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="9907b-124">Pokyny k instalaci centra Azure Stack najdete v tématu [instalace ASDK](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="9907b-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install).</span></span>
  - <span data-ttu-id="9907b-125">ASDK skript pro automatizaci po nasazení najdete tady: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="9907b-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="9907b-126">Dokončení této instalace může trvat několik hodin.</span><span class="sxs-lookup"><span data-stu-id="9907b-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="9907b-127">Nasaďte [App Service](/azure-stack/operator/azure-stack-app-service-deploy) služby PaaS do centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="9907b-128">[Vytvářejte plány/nabídky](/azure-stack/operator/service-plan-offer-subscription-overview) v prostředí Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="9907b-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="9907b-129">[Vytvořte předplatné tenanta](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) v prostředí Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="9907b-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="9907b-130">Vytvořte webovou aplikaci v rámci předplatného tenanta.</span><span class="sxs-lookup"><span data-stu-id="9907b-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="9907b-131">Poznamenejte si novou adresu URL webové aplikace, abyste ji mohli později použít.</span><span class="sxs-lookup"><span data-stu-id="9907b-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="9907b-132">Nasaďte Azure Pipelines virtuální počítač (VM) v rámci předplatného tenanta.</span><span class="sxs-lookup"><span data-stu-id="9907b-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="9907b-133">Vyžaduje se virtuální počítač se systémem Windows Server 2016 s rozhraním .NET 3,5.</span><span class="sxs-lookup"><span data-stu-id="9907b-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="9907b-134">Tento virtuální počítač bude sestaven v rámci předplatného tenanta na Azure Stack hub jako privátní agent sestavení.</span><span class="sxs-lookup"><span data-stu-id="9907b-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="9907b-135">[Windows Server 2016 s IMAGÍ SQL 2017 VM](/azure-stack/operator/azure-stack-add-vm-image) je k dispozici v tržišti Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="9907b-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="9907b-136">Pokud tento obrázek není k dispozici, pracujte s operátorem centra Azure Stack, abyste se ujistili, že je přidaný do prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="9907b-137">Problémy a důležité informace</span><span class="sxs-lookup"><span data-stu-id="9907b-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="9907b-138">Škálovatelnost</span><span class="sxs-lookup"><span data-stu-id="9907b-138">Scalability</span></span>

<span data-ttu-id="9907b-139">Klíčovou součástí škálování mezi cloudy je schopnost doručovat okamžitou a spolehlivou škálu mezi veřejnou a místní cloudovou infrastrukturou a poskytovat tak konzistentní a spolehlivé služby.</span><span class="sxs-lookup"><span data-stu-id="9907b-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="9907b-140">Dostupnost</span><span class="sxs-lookup"><span data-stu-id="9907b-140">Availability</span></span>

<span data-ttu-id="9907b-141">Zajistěte, aby lokálně nasazené aplikace byly nakonfigurované pro vysokou dostupnost prostřednictvím místní konfigurace hardwaru a nasazení softwaru.</span><span class="sxs-lookup"><span data-stu-id="9907b-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="9907b-142">Možnosti správy</span><span class="sxs-lookup"><span data-stu-id="9907b-142">Manageability</span></span>

<span data-ttu-id="9907b-143">Řešení mezi cloudy zajišťuje bezproblémové řízení a známé rozhraní mezi prostředími.</span><span class="sxs-lookup"><span data-stu-id="9907b-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="9907b-144">PowerShell se doporučuje pro správu různých platforem.</span><span class="sxs-lookup"><span data-stu-id="9907b-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="9907b-145">Škálování napříč cloudy</span><span class="sxs-lookup"><span data-stu-id="9907b-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="9907b-146">Získat vlastní doménu a nakonfigurovat DNS</span><span class="sxs-lookup"><span data-stu-id="9907b-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="9907b-147">Aktualizujte soubor zóny DNS pro doménu.</span><span class="sxs-lookup"><span data-stu-id="9907b-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="9907b-148">Azure AD ověří vlastnictví vlastního názvu domény.</span><span class="sxs-lookup"><span data-stu-id="9907b-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="9907b-149">Použijte [Azure DNS](/azure/dns/dns-getstarted-portal) pro azure/Microsoft 365/externí záznamy DNS v rámci Azure nebo přidejte položku DNS v [jiném registrátoru DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="9907b-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="9907b-150">Zaregistrujte vlastní doménu s veřejným registrátorem.</span><span class="sxs-lookup"><span data-stu-id="9907b-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="9907b-151">Přihlaste se k registrátorovi názvu domény.</span><span class="sxs-lookup"><span data-stu-id="9907b-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="9907b-152">K provedení aktualizací DNS může být nutný schválený správce.</span><span class="sxs-lookup"><span data-stu-id="9907b-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="9907b-153">Aktualizujte soubor zóny DNS pro doménu tak, že přidáte položku DNS, kterou poskytuje Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9907b-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="9907b-154">(Položka DNS nebude mít vliv na směrování e-mailu nebo na chování webového hostování.)</span><span class="sxs-lookup"><span data-stu-id="9907b-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="9907b-155">Vytvoření výchozí webové aplikace s více uzly v centru Azure Stack</span><span class="sxs-lookup"><span data-stu-id="9907b-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="9907b-156">Nastavte hybridní průběžnou integraci a průběžné nasazování (CI/CD), abyste nasadili webové aplikace do Azure a Azure Stack hub a mohli do obou cloudů doručovat změny.</span><span class="sxs-lookup"><span data-stu-id="9907b-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="9907b-157">Azure Stack centrum se správnými obrázky publikovanými pro spuštění (Windows Server a SQL) a vyžaduje se nasazení App Service.</span><span class="sxs-lookup"><span data-stu-id="9907b-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="9907b-158">Další informace najdete v dokumentaci App Service [předpoklady pro nasazení App Service na Azure Stack hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="9907b-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="9907b-159">Přidat kód pro Azure Repos</span><span class="sxs-lookup"><span data-stu-id="9907b-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="9907b-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="9907b-160">Azure Repos</span></span>

1. <span data-ttu-id="9907b-161">Přihlaste se k Azure Repos pomocí účtu, který má práva na vytvoření projektu na Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="9907b-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="9907b-162">Hybridní CI/CD může platit pro kód aplikace i kód infrastruktury.</span><span class="sxs-lookup"><span data-stu-id="9907b-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="9907b-163">Použijte [šablony Azure Resource Manager](https://azure.microsoft.com/resources/templates/) pro vývoj privátního i hostovaného cloudu.</span><span class="sxs-lookup"><span data-stu-id="9907b-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Připojit se k projektu na Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="9907b-165">**Naklonujte úložiště** vytvořením a otevřením výchozí webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="9907b-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klonování úložiště ve službě Azure Web App](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="9907b-167">Vytvoření samoobslužného nasazení webové aplikace pro App Services v obou cloudech</span><span class="sxs-lookup"><span data-stu-id="9907b-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="9907b-168">Upravte soubor **WebApplication. csproj** .</span><span class="sxs-lookup"><span data-stu-id="9907b-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="9907b-169">Vyberte `Runtimeidentifier` a přidejte `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="9907b-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="9907b-170">(Viz dokumentace k [samoobslužnému nasazení](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)</span><span class="sxs-lookup"><span data-stu-id="9907b-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Upravit soubor projektu webové aplikace](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="9907b-172">Vrácením kódu se změnami Azure Repos používání Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="9907b-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="9907b-173">Ověřte, že je kód aplikace zkontrolovaný Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="9907b-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="9907b-174">Vytvoření definice sestavení</span><span class="sxs-lookup"><span data-stu-id="9907b-174">Create the build definition</span></span>

1. <span data-ttu-id="9907b-175">Přihlaste se k Azure Pipelines a potvrďte možnost vytvářet definice sestavení.</span><span class="sxs-lookup"><span data-stu-id="9907b-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="9907b-176">Přidejte kód **-r Win10-x64** .</span><span class="sxs-lookup"><span data-stu-id="9907b-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="9907b-177">Tento dodatek je nezbytný pro aktivaci samostatného nasazení pomocí .NET Core.</span><span class="sxs-lookup"><span data-stu-id="9907b-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Přidání kódu do webové aplikace](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="9907b-179">Spusťte sestavení.</span><span class="sxs-lookup"><span data-stu-id="9907b-179">Run the build.</span></span> <span data-ttu-id="9907b-180">Proces [sestavení samostatného nasazení](/dotnet/core/deploying/deploy-with-vs#simpleSelf) bude publikovat artefakty, které běží v Azure a centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="9907b-181">Použití hostovaného agenta Azure</span><span class="sxs-lookup"><span data-stu-id="9907b-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="9907b-182">Použití hostovaného agenta sestavení v Azure Pipelines je pohodlný způsob pro sestavování a nasazování webových aplikací.</span><span class="sxs-lookup"><span data-stu-id="9907b-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="9907b-183">Údržba a upgrady se provádí automaticky Microsoft Azure, což umožňuje průběžné a nepřerušované vývojové cykly.</span><span class="sxs-lookup"><span data-stu-id="9907b-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="9907b-184">Správa a konfigurace procesu CD</span><span class="sxs-lookup"><span data-stu-id="9907b-184">Manage and configure the CD process</span></span>

<span data-ttu-id="9907b-185">Azure Pipelines a Azure DevOps Services poskytují vysoce konfigurovatelný a spravovatelný kanál pro vydání do více prostředí, jako jsou vývojové, pracovní, QA a produkční prostředí. zahrnutí požadavku na schválení v určitých fázích.</span><span class="sxs-lookup"><span data-stu-id="9907b-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="9907b-186">Vytvořit definici vydané verze</span><span class="sxs-lookup"><span data-stu-id="9907b-186">Create release definition</span></span>

1. <span data-ttu-id="9907b-187">Kliknutím na tlačítko **plus** přidejte novou verzi na kartě **vydání** v části **sestavení a vydání** Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="9907b-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Vytvoření definice verze](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="9907b-189">Použijte šablonu nasazení Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="9907b-189">Apply the Azure App Service Deployment template.</span></span>

   ![Použít šablonu nasazení Azure App Service](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="9907b-191">V části **Přidat artefakt** přidejte artefakt pro aplikaci Azure Cloud Build.</span><span class="sxs-lookup"><span data-stu-id="9907b-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Přidání artefaktu do cloudového sestavení Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="9907b-193">Na kartě kanál vyberte **fáze,** odkaz na úlohu prostředí a nastavte hodnoty cloudového prostředí Azure.</span><span class="sxs-lookup"><span data-stu-id="9907b-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Nastavení hodnot cloudového prostředí Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="9907b-195">Nastavte **Název prostředí** a vyberte **předplatné Azure** pro koncový bod cloudu Azure.</span><span class="sxs-lookup"><span data-stu-id="9907b-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Výběr předplatného Azure pro koncový bod cloudu Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="9907b-197">V části **název služby App Service** nastavte požadovaný název služby Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="9907b-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Nastavit název služby Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="9907b-199">Do pole **fronta agenta** pro hostované cloudové prostředí Azure zadejte "hostované VS2017".</span><span class="sxs-lookup"><span data-stu-id="9907b-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Nastavení fronty agenta pro hostované cloudové prostředí Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="9907b-201">V nabídce nasadit Azure App Service vyberte pro prostředí platný **balíček nebo složku** .</span><span class="sxs-lookup"><span data-stu-id="9907b-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="9907b-202">Vyberte **OK** do **umístění složky**.</span><span class="sxs-lookup"><span data-stu-id="9907b-202">Select **OK** to **folder location**.</span></span>
  
      ![Vyberte balíček nebo složku pro Azure App Service prostředí.](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Dialogové okno pro výběr složky 1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="9907b-205">Uložte všechny změny a vraťte se do **kanálu uvolnění**.</span><span class="sxs-lookup"><span data-stu-id="9907b-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Uložit změny v kanálu vydání](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="9907b-207">Přidejte nový artefakt, který vybírá sestavení pro aplikaci Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="9907b-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Přidat nový artefakt pro aplikaci Azure Stack hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="9907b-209">Přidejte další prostředí pomocí Azure App Service nasazení.</span><span class="sxs-lookup"><span data-stu-id="9907b-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Přidání prostředí do nasazení Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="9907b-211">Pojmenujte nové prostředí "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="9907b-211">Name the new environment "Azure Stack".</span></span>

    ![Název prostředí při nasazení Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="9907b-213">Na kartě **úloha** Najděte Azure Stack prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack prostředí](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="9907b-215">Vyberte předplatné pro Azure Stack koncový bod.</span><span class="sxs-lookup"><span data-stu-id="9907b-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Vyberte předplatné pro Azure Stack koncový bod.](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="9907b-217">Jako název služby App Service nastavte Azure Stack název webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="9907b-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="9907b-218">![Nastavit Azure Stack název webové aplikace](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="9907b-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="9907b-219">Vyberte agenta Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-219">Select the Azure Stack agent.</span></span>

    ![Vybrat agenta Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="9907b-221">V části nasadit Azure App Service vyberte platný **balíček nebo složku** pro prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="9907b-222">Vyberte **OK** do umístění složky.</span><span class="sxs-lookup"><span data-stu-id="9907b-222">Select **OK** to folder location.</span></span>

    ![Vyberte složku pro nasazení Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Dialogové okno pro výběr složky 2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="9907b-225">V části karta proměnné přidejte proměnnou s názvem `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , nastavte její hodnotu na **true** a rozsah na Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Přidat proměnnou do nasazení aplikace Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="9907b-227">V obou artefaktech vyberte ikonu triggeru **průběžného** nasazování a povolte aktivační událost **pokračování** nasazení.</span><span class="sxs-lookup"><span data-stu-id="9907b-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Vybrat aktivační událost průběžného nasazování](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="9907b-229">Vyberte ikonu podmínky **před nasazením** v prostředí Azure Stack a nastavte Trigger na **po vydání.**</span><span class="sxs-lookup"><span data-stu-id="9907b-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Vybrat podmínky předběžného nasazení](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="9907b-231">Uložte všechny změny.</span><span class="sxs-lookup"><span data-stu-id="9907b-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="9907b-232">Některá nastavení pro úlohy mohla být při vytváření definice verze ze šablony automaticky definována jako [proměnné prostředí](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) .</span><span class="sxs-lookup"><span data-stu-id="9907b-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="9907b-233">Tato nastavení se nedají upravit v nastavení úlohy. místo toho je nutné vybrat nadřazenou položku prostředí pro úpravu těchto nastavení.</span><span class="sxs-lookup"><span data-stu-id="9907b-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="9907b-234">Publikování do centra Azure Stack pomocí sady Visual Studio</span><span class="sxs-lookup"><span data-stu-id="9907b-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="9907b-235">Vytvořením koncových bodů může Azure DevOps Services Build nasazovat aplikace služby Azure do centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="9907b-236">Azure Pipelines se připojí k agentu sestavení, který se připojí k centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="9907b-237">Přihlaste se k Azure DevOps Services a přejdete na stránku nastavení aplikace.</span><span class="sxs-lookup"><span data-stu-id="9907b-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="9907b-238">V **Nastavení** vyberte **zabezpečení**.</span><span class="sxs-lookup"><span data-stu-id="9907b-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="9907b-239">V **VSTS skupin** vyberte **Tvůrce koncových bodů**.</span><span class="sxs-lookup"><span data-stu-id="9907b-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="9907b-240">Na kartě **Členové** vyberte **Přidat**.</span><span class="sxs-lookup"><span data-stu-id="9907b-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="9907b-241">V části **Přidat uživatele a skupiny** zadejte uživatelské jméno a vyberte tohoto uživatele ze seznamu uživatelů.</span><span class="sxs-lookup"><span data-stu-id="9907b-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="9907b-242">Vyberte **Uložit změny**.</span><span class="sxs-lookup"><span data-stu-id="9907b-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="9907b-243">V seznamu **skupiny VSTS** vyberte možnost **Správci koncových bodů**.</span><span class="sxs-lookup"><span data-stu-id="9907b-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="9907b-244">Na kartě **Členové** vyberte **Přidat**.</span><span class="sxs-lookup"><span data-stu-id="9907b-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="9907b-245">V části **Přidat uživatele a skupiny** zadejte uživatelské jméno a vyberte tohoto uživatele ze seznamu uživatelů.</span><span class="sxs-lookup"><span data-stu-id="9907b-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="9907b-246">Vyberte **Uložit změny**.</span><span class="sxs-lookup"><span data-stu-id="9907b-246">Select **Save changes**.</span></span>

<span data-ttu-id="9907b-247">Teď, když existují informace o koncovém bodu, je Azure Pipelines připojení k rozbočovači Azure Stack připraveno k použití.</span><span class="sxs-lookup"><span data-stu-id="9907b-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="9907b-248">Agent sestavení v centru Azure Stack získá pokyny od Azure Pipelines a potom agent přenáší informace koncového bodu pro komunikaci se službou Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="9907b-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="9907b-249">Vývoj buildu aplikace</span><span class="sxs-lookup"><span data-stu-id="9907b-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="9907b-250">Azure Stack centrum se správnými obrázky publikovanými pro spuštění (Windows Server a SQL) a vyžaduje se nasazení App Service.</span><span class="sxs-lookup"><span data-stu-id="9907b-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="9907b-251">Další informace najdete v tématu [předpoklady pro nasazení App Service v centru Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="9907b-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

<span data-ttu-id="9907b-252">K nasazení do obou cloudů použijte [Azure Resource Manager šablony](https://azure.microsoft.com/resources/templates/) , jako je kód webové aplikace z Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="9907b-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="9907b-253">Přidat kód do projektu Azure Repos</span><span class="sxs-lookup"><span data-stu-id="9907b-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="9907b-254">Přihlaste se k Azure Repos pomocí účtu, který má práva pro vytváření projektů v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="9907b-255">**Naklonujte úložiště** vytvořením a otevřením výchozí webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="9907b-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="9907b-256">Vytvoření samoobslužného nasazení webové aplikace pro App Services v obou cloudech</span><span class="sxs-lookup"><span data-stu-id="9907b-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="9907b-257">Upravte soubor **WebApplication. csproj** : vyberte `Runtimeidentifier` a pak přidejte `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="9907b-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="9907b-258">Další informace najdete v dokumentaci k [samoobslužnému nasazení](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="9907b-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="9907b-259">Použijte Team Explorer ke kontrole kódu do Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="9907b-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="9907b-260">Potvrďte, že kód aplikace byl zkontrolován Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="9907b-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="9907b-261">Vytvoření definice sestavení</span><span class="sxs-lookup"><span data-stu-id="9907b-261">Create the build definition</span></span>

1. <span data-ttu-id="9907b-262">Přihlaste se k Azure Pipelines pomocí účtu, který může vytvořit definici sestavení.</span><span class="sxs-lookup"><span data-stu-id="9907b-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="9907b-263">Přejít na stránku **sestavení webové aplikace** pro projekt.</span><span class="sxs-lookup"><span data-stu-id="9907b-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="9907b-264">V **argumentech** přidejte kód **-r Win10-x64** .</span><span class="sxs-lookup"><span data-stu-id="9907b-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="9907b-265">Tento dodatek je nutný k aktivaci samostatného nasazení pomocí .NET Core.</span><span class="sxs-lookup"><span data-stu-id="9907b-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="9907b-266">Spusťte sestavení.</span><span class="sxs-lookup"><span data-stu-id="9907b-266">Run the build.</span></span> <span data-ttu-id="9907b-267">Proces [sestavení samostatného nasazení](/dotnet/core/deploying/deploy-with-vs#simpleSelf) bude publikovat artefakty, které se dají spouštět v Azure a centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="9907b-268">Použití hostovaného agenta sestavení Azure</span><span class="sxs-lookup"><span data-stu-id="9907b-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="9907b-269">Použití hostovaného agenta sestavení v Azure Pipelines je pohodlný způsob pro sestavování a nasazování webových aplikací.</span><span class="sxs-lookup"><span data-stu-id="9907b-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="9907b-270">Údržba a upgrady se provádí automaticky Microsoft Azure, což umožňuje průběžné a nepřerušované vývojové cykly.</span><span class="sxs-lookup"><span data-stu-id="9907b-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="9907b-271">Konfigurace procesu průběžného nasazování (CD)</span><span class="sxs-lookup"><span data-stu-id="9907b-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="9907b-272">Azure Pipelines a Azure DevOps Services poskytují vysoce konfigurovatelný a spravovatelný kanál pro vydání do více prostředí, jako je vývoj, příprava, zabezpečování kvality (QA) a produkce.</span><span class="sxs-lookup"><span data-stu-id="9907b-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="9907b-273">Tento proces může zahrnovat vyžadování schválení v určitých fázích životního cyklu aplikace.</span><span class="sxs-lookup"><span data-stu-id="9907b-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="9907b-274">Vytvořit definici vydané verze</span><span class="sxs-lookup"><span data-stu-id="9907b-274">Create release definition</span></span>

<span data-ttu-id="9907b-275">Vytvoření definice verze je posledním krokem v procesu sestavování aplikace.</span><span class="sxs-lookup"><span data-stu-id="9907b-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="9907b-276">Tato definice verze slouží k vytvoření vydání a nasazení sestavení.</span><span class="sxs-lookup"><span data-stu-id="9907b-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="9907b-277">Přihlaste se k Azure Pipelines a v projektu klikněte na **sestavení a vydání** .</span><span class="sxs-lookup"><span data-stu-id="9907b-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="9907b-278">Na kartě **vydané verze** vyberte **[+]** a pak vyberte **vytvořit definici vydané verze**.</span><span class="sxs-lookup"><span data-stu-id="9907b-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="9907b-279">V nabídce **Vybrat šablonu** zvolte **Azure App Service nasazení** a pak vyberte **použít**.</span><span class="sxs-lookup"><span data-stu-id="9907b-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="9907b-280">V části **Přidat artefakt** ze **zdroje (definice sestavení)** vyberte aplikaci Azure Cloud Build.</span><span class="sxs-lookup"><span data-stu-id="9907b-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="9907b-281">Na kartě **kanál** vyberte odkaz **1 fáze**, **1 úloha** a **Zobrazte úlohy prostředí**.</span><span class="sxs-lookup"><span data-stu-id="9907b-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="9907b-282">Na kartě **úlohy** zadejte Azure jako **Název prostředí** a v seznamu **předplatných Azure** vyberte AzureCloud Traders-Web EP.</span><span class="sxs-lookup"><span data-stu-id="9907b-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="9907b-283">Zadejte **název služby Azure App Service**, který je `northwindtraders` na následujícím snímku obrazovky.</span><span class="sxs-lookup"><span data-stu-id="9907b-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="9907b-284">V případě fáze agenta vyberte možnost **hostované VS2017** ze seznamu **fronta agenta** .</span><span class="sxs-lookup"><span data-stu-id="9907b-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="9907b-285">V části **nasadit Azure App Service** Vyberte platný **balíček nebo složku** pro prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="9907b-286">V **oblasti** **Vybrat soubor nebo složku** vyberte **OK** .</span><span class="sxs-lookup"><span data-stu-id="9907b-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="9907b-287">Uložte všechny změny a vraťte se zpět do **kanálu**.</span><span class="sxs-lookup"><span data-stu-id="9907b-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="9907b-288">Na kartě **kanál** vyberte **Přidat artefakt** a ze seznamu **zdroj (definice sestavení)** zvolte **plavidlo NorthwindCloud Traders** .</span><span class="sxs-lookup"><span data-stu-id="9907b-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="9907b-289">V nabídce **Vybrat šablonu** přidejte další prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="9907b-290">Vyberte **nasazení Azure App Service** a pak vyberte **použít**.</span><span class="sxs-lookup"><span data-stu-id="9907b-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="9907b-291">`Azure Stack Hub`Jako **Název prostředí** zadejte.</span><span class="sxs-lookup"><span data-stu-id="9907b-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="9907b-292">Na kartě **úlohy** vyhledejte a vyberte centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="9907b-293">V seznamu **předplatné Azure** vyberte **AzureStack Traders-Vessel EP** pro koncový bod centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="9907b-294">Jako **název služby App Service** zadejte název webové aplikace centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="9907b-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="9907b-295">V části **Výběr agenta** vyberte v seznamu **fronta agenta** **AzureStack-b Douglas jedle** .</span><span class="sxs-lookup"><span data-stu-id="9907b-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="9907b-296">Pro **Azure App Service nasazení** vyberte pro prostředí platný **balíček nebo složku** .</span><span class="sxs-lookup"><span data-stu-id="9907b-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="9907b-297">V části **Vybrat soubor nebo složku** vyberte **OK** pro **umístění** složky.</span><span class="sxs-lookup"><span data-stu-id="9907b-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="9907b-298">Na kartě **Proměnná** Najděte proměnnou s názvem `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="9907b-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="9907b-299">Nastavte hodnotu proměnné na **true** a nastavte její obor na **Azure Stack hub**.</span><span class="sxs-lookup"><span data-stu-id="9907b-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="9907b-300">Na kartě **kanál** vyberte ikonu **triggeru průběžného nasazování** pro NorthwindCloud Traders-Web artefakt a nastavte **aktivační událost průběžného nasazování** na **povoleno**.</span><span class="sxs-lookup"><span data-stu-id="9907b-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="9907b-301">Totéž udělejte pro **NorthwindCloud obchodníci – artefakt plavidla** .</span><span class="sxs-lookup"><span data-stu-id="9907b-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="9907b-302">V prostředí Azure Stack hub vyberte ikonu **podmínky před nasazením** nastavit Trigger na **po vydání**.</span><span class="sxs-lookup"><span data-stu-id="9907b-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="9907b-303">Uložte všechny změny.</span><span class="sxs-lookup"><span data-stu-id="9907b-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="9907b-304">Některá nastavení pro úlohy vydaných verzí se při vytváření definice verze ze šablony automaticky definují jako [proměnné prostředí](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) .</span><span class="sxs-lookup"><span data-stu-id="9907b-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="9907b-305">Tato nastavení nelze upravovat v nastavení úlohy, ale lze je upravit v položkách nadřazeného prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="9907b-306">Vytvoření vydané verze</span><span class="sxs-lookup"><span data-stu-id="9907b-306">Create a release</span></span>

1. <span data-ttu-id="9907b-307">Na kartě **kanál** otevřete seznam vydaných **verzí** a vyberte **vytvořit vydání**.</span><span class="sxs-lookup"><span data-stu-id="9907b-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="9907b-308">Zadejte popis vydané verze, zkontrolujte, že jsou vybrané správné artefakty, a pak vyberte **vytvořit**.</span><span class="sxs-lookup"><span data-stu-id="9907b-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="9907b-309">Po chvíli se zobrazí banner s oznámením, že se vytvořila nová vydaná verze a že se název verze zobrazuje jako odkaz.</span><span class="sxs-lookup"><span data-stu-id="9907b-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="9907b-310">Kliknutím na odkaz zobrazíte stránku se souhrnem vydání.</span><span class="sxs-lookup"><span data-stu-id="9907b-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="9907b-311">Na stránce souhrnu vydání najdete podrobnosti o vydané verzi.</span><span class="sxs-lookup"><span data-stu-id="9907b-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="9907b-312">Na následujícím snímku obrazovky "Release-2" v části **prostředí** se zobrazuje **stav nasazení** pro Azure jako probíhající a stav centra Azure Stack je "úspěch".</span><span class="sxs-lookup"><span data-stu-id="9907b-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="9907b-313">Když se stav nasazení prostředí Azure změní na úspěšné, zobrazí se informační zpráva s oznámením, že verze je připravená ke schválení.</span><span class="sxs-lookup"><span data-stu-id="9907b-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="9907b-314">Když nasazení čeká na vyřízení nebo se nezdařilo, zobrazí se ikona s modrou **(i)** informacemi.</span><span class="sxs-lookup"><span data-stu-id="9907b-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="9907b-315">Když najedete myší na ikonu, zobrazí se automaticky otevírané okno, které obsahuje důvod zpoždění nebo chyby.</span><span class="sxs-lookup"><span data-stu-id="9907b-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="9907b-316">Další zobrazení, jako je například seznam vydání, zobrazují také ikonu, která indikuje, že schválení čeká na vyřízení.</span><span class="sxs-lookup"><span data-stu-id="9907b-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="9907b-317">Automaticky otevírané okno pro tuto ikonu zobrazuje název prostředí a další podrobnosti týkající se nasazení.</span><span class="sxs-lookup"><span data-stu-id="9907b-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="9907b-318">Správce může snadno zobrazit celkový průběh vydaných verzí a zjistit, které verze čekají na schválení.</span><span class="sxs-lookup"><span data-stu-id="9907b-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="9907b-319">Monitorování a sledování nasazení</span><span class="sxs-lookup"><span data-stu-id="9907b-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="9907b-320">Na stránce Souhrn **vydání 2** vyberte **protokoly**.</span><span class="sxs-lookup"><span data-stu-id="9907b-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="9907b-321">Během nasazení se na této stránce zobrazuje živý protokol z agenta.</span><span class="sxs-lookup"><span data-stu-id="9907b-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="9907b-322">V levém podokně se zobrazuje stav každé operace v nasazení pro každé prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="9907b-323">Vyberte ikonu osoby ve sloupci **Akce** pro schválení před nasazením nebo po nasazení, abyste viděli, kdo toto nasazení schválil (nebo zamítl), a zprávu, kterou poskytli.</span><span class="sxs-lookup"><span data-stu-id="9907b-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="9907b-324">Po dokončení nasazení se v pravém podokně zobrazí celý soubor protokolu.</span><span class="sxs-lookup"><span data-stu-id="9907b-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="9907b-325">V levém podokně vyberte libovolný **Krok** , abyste viděli soubor protokolu pro jeden krok, jako je například **Inicializace úlohy**.</span><span class="sxs-lookup"><span data-stu-id="9907b-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="9907b-326">Možnost zobrazit jednotlivé protokoly usnadňuje trasování a ladění částí celkového nasazení.</span><span class="sxs-lookup"><span data-stu-id="9907b-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="9907b-327">**Uložte** soubor protokolu pro krok nebo **stáhněte všechny protokoly jako soubor zip**.</span><span class="sxs-lookup"><span data-stu-id="9907b-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="9907b-328">Otevřete kartu **Souhrn** , kde najdete obecné informace o vydané verzi.</span><span class="sxs-lookup"><span data-stu-id="9907b-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="9907b-329">Toto zobrazení obsahuje podrobnosti o sestavení, prostředích, do kterých byla nasazena, stav nasazení a další informace o vydané verzi.</span><span class="sxs-lookup"><span data-stu-id="9907b-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="9907b-330">Vyberte odkaz prostředí (**Azure** nebo **centrum Azure Stack**), ve kterém se zobrazí informace o stávajících a nevyřízených nasazeních do konkrétního prostředí.</span><span class="sxs-lookup"><span data-stu-id="9907b-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="9907b-331">Tato zobrazení slouží jako rychlý způsob, jak kontrolovat, zda bylo stejné sestavení nasazeno v obou prostředích.</span><span class="sxs-lookup"><span data-stu-id="9907b-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="9907b-332">Otevřete **nasazenou produkční aplikaci** v prohlížeči.</span><span class="sxs-lookup"><span data-stu-id="9907b-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="9907b-333">Například pro web Azure App Services otevřete adresu URL `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="9907b-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="9907b-334">Integrace Azure a centra Azure Stack přináší škálovatelné řešení mezi cloudy.</span><span class="sxs-lookup"><span data-stu-id="9907b-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="9907b-335">Flexibilní a robustní cloudová služba poskytuje zabezpečení dat, zálohování a redundanci, konzistentní a rychlé dostupnosti, škálovatelné úložiště a distribuci a směrování vyhovující geografickým požadavkům.</span><span class="sxs-lookup"><span data-stu-id="9907b-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="9907b-336">Tento ručně aktivovaný proces zajišťuje spolehlivé a efektivní přepínání zatížení mezi hostovanými webovými aplikacemi a okamžitou dostupností důležitých dat.</span><span class="sxs-lookup"><span data-stu-id="9907b-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9907b-337">Další kroky</span><span class="sxs-lookup"><span data-stu-id="9907b-337">Next steps</span></span>

- <span data-ttu-id="9907b-338">Další informace o vzorech cloudu Azure najdete v tématu [vzory návrhu cloudu](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="9907b-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
