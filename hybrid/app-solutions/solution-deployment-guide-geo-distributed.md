---
title: Přímý provoz s geograficky distribuovanou aplikací pomocí Azure a centra Azure Stack
description: Naučte se směrovat provoz do konkrétních koncových bodů pomocí geograficky distribuované aplikace s využitím Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 27d07070becfa902a715b451baae7c81c7e4b46f
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886828"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="6bee6-103">Přímý provoz s geograficky distribuovanou aplikací pomocí Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="6bee6-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="6bee6-104">Naučte se směrovat provoz do konkrétních koncových bodů na základě různých metrik pomocí vzoru geograficky distribuovaných aplikací.</span><span class="sxs-lookup"><span data-stu-id="6bee6-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="6bee6-105">Když vytvoříte profil Traffic Manager s využitím geografického směrování a konfigurace koncového bodu, zajistíte směrování informací na koncové body na základě regionálních požadavků, podnikových a mezinárodních předpisů a vašich datových potřeb.</span><span class="sxs-lookup"><span data-stu-id="6bee6-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="6bee6-106">V tomto řešení sestavíte ukázkové prostředí pro:</span><span class="sxs-lookup"><span data-stu-id="6bee6-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6bee6-107">Vytvořte geograficky distribuovanou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="6bee6-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="6bee6-108">Použijte Traffic Manager k zacílení aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="6bee6-109">Použití vzoru geograficky distribuovaných aplikací</span><span class="sxs-lookup"><span data-stu-id="6bee6-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="6bee6-110">Pomocí geograficky distribuovaného vzoru vaše aplikace zahrnuje oblasti.</span><span class="sxs-lookup"><span data-stu-id="6bee6-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="6bee6-111">Můžete být ve výchozím nastavení veřejným cloudem, ale někteří uživatelé mohou vyžadovat, aby data zůstala ve své oblasti.</span><span class="sxs-lookup"><span data-stu-id="6bee6-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="6bee6-112">Na základě svých požadavků můžete směrovat uživatele na nejvhodnější Cloud.</span><span class="sxs-lookup"><span data-stu-id="6bee6-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="6bee6-113">Problémy a důležité informace</span><span class="sxs-lookup"><span data-stu-id="6bee6-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="6bee6-114">Aspekty zabezpečení</span><span class="sxs-lookup"><span data-stu-id="6bee6-114">Scalability considerations</span></span>

<span data-ttu-id="6bee6-115">Řešení, které v tomto článku sestavíte, se nevejde na škálovatelnost.</span><span class="sxs-lookup"><span data-stu-id="6bee6-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="6bee6-116">Pokud se ale používá v kombinaci s jinými řešeními Azure a místními řešeními, můžete přizpůsobit požadavky na škálovatelnost.</span><span class="sxs-lookup"><span data-stu-id="6bee6-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="6bee6-117">Informace o vytvoření hybridního řešení s automatickým škálováním prostřednictvím Traffic Manageru najdete v tématu [vytváření řešení pro škálování mezi cloudy pomocí Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="6bee6-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="6bee6-118">Aspekty dostupnosti</span><span class="sxs-lookup"><span data-stu-id="6bee6-118">Availability considerations</span></span>

<span data-ttu-id="6bee6-119">Stejně jako v případě, že se jedná o požadavky na škálovatelnost, toto řešení přímo neřeší dostupnost.</span><span class="sxs-lookup"><span data-stu-id="6bee6-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="6bee6-120">V rámci tohoto řešení je ale možné implementovat Azure a místní řešení, aby se zajistila vysoká dostupnost všech součástí, které se týkají.</span><span class="sxs-lookup"><span data-stu-id="6bee6-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="6bee6-121">Kdy se má tento model použít</span><span class="sxs-lookup"><span data-stu-id="6bee6-121">When to use this pattern</span></span>

- <span data-ttu-id="6bee6-122">Vaše organizace má mezinárodní pobočky vyžadující vlastní regionální zásady zabezpečení a distribuce.</span><span class="sxs-lookup"><span data-stu-id="6bee6-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="6bee6-123">Každá z poboček vaší organizace si vyžádá data o zaměstnancích, firmách a obchodních přístavech, což vyžaduje, aby se aktivita vytváření sestav na místní a časová pásma.</span><span class="sxs-lookup"><span data-stu-id="6bee6-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="6bee6-124">Vysoce škálovatelné požadavky jsou splněné horizontálním škálováním aplikací s více nasazeními aplikací v jedné oblasti a mezi oblastmi, které zpracovávají extrémní požadavky na zatížení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="6bee6-125">Plánování topologie</span><span class="sxs-lookup"><span data-stu-id="6bee6-125">Planning the topology</span></span>

<span data-ttu-id="6bee6-126">Před vytvořením kapacity distribuovaných aplikací vám pomůže tyto věci:</span><span class="sxs-lookup"><span data-stu-id="6bee6-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="6bee6-127">**Vlastní doména pro aplikaci:** Jaký je vlastní název domény, který budou zákazníci používat pro přístup k aplikaci?</span><span class="sxs-lookup"><span data-stu-id="6bee6-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="6bee6-128">Pro ukázkovou aplikaci je vlastní název domény *www \. scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="6bee6-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="6bee6-129">**Doména Traffic Manager:** Při vytváření [profilu Traffic Manager Azure](/azure/traffic-manager/traffic-manager-manage-profiles)se vybere název domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="6bee6-130">Tento název se používá v kombinaci s příponou *trafficmanager.NET* k registraci položky domény spravované pomocí Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="6bee6-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="6bee6-131">Pro ukázkovou aplikaci je zvolený název *škálovatelný-pomocný-demo*.</span><span class="sxs-lookup"><span data-stu-id="6bee6-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="6bee6-132">Výsledkem je, že úplný název domény, který je spravovaný nástrojem Traffic Manager, je *Scalable-ASE-demo.trafficmanager.NET*.</span><span class="sxs-lookup"><span data-stu-id="6bee6-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="6bee6-133">**Strategie škálování aplikace:** Rozhodněte se, jestli budou nároky na aplikace distribuované napříč několika App Service prostředími v jedné oblasti, několika oblastech nebo kombinací obou přístupů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="6bee6-134">Rozhodnutí by mělo být založené na očekáváních, kde se bude nacházet na provozu zákazníků, a na tom, jak se může dál škálovat i zbytek podpory back-endové infrastruktury aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="6bee6-135">Například s bezstavovou aplikací 100% se dá aplikace hromadně škálovat s využitím kombinace více App Service prostředí na oblast Azure vynásobená App Service prostředími nasazenými napříč několika oblastmi Azure.</span><span class="sxs-lookup"><span data-stu-id="6bee6-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="6bee6-136">Díky více než 15 globálním oblastem Azure, ze kterých si můžete vybrat, můžou zákazníci skutečně vytvořit vysoce škálovatelné nároky na aplikace na úrovni Hyper.</span><span class="sxs-lookup"><span data-stu-id="6bee6-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="6bee6-137">Pro ukázkovou aplikaci, která se tady používá, se v jedné oblasti Azure (Střed USA – jih) vytvořila tři App Service prostředí.</span><span class="sxs-lookup"><span data-stu-id="6bee6-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="6bee6-138">**Konvence pojmenování pro prostředí App Service:** Každé App Service prostředí vyžaduje jedinečný název.</span><span class="sxs-lookup"><span data-stu-id="6bee6-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="6bee6-139">Mimo jedno nebo dvě App Service prostředí je vhodné, abyste měli zásady vytváření názvů, které vám pomůžou identifikovat každé App Service prostředí.</span><span class="sxs-lookup"><span data-stu-id="6bee6-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="6bee6-140">Pro ukázkovou aplikaci, která se tady používá, se použila jednoduchá konvence pojmenování.</span><span class="sxs-lookup"><span data-stu-id="6bee6-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="6bee6-141">Názvy tří App Service prostředí jsou *fe1ase*, *fe2ase*a *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="6bee6-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="6bee6-142">**Konvence pojmenování pro aplikace:** Vzhledem k tomu, že bude nasazeno několik instancí aplikace, je pro každou instanci nasazené aplikace nutné zadat název.</span><span class="sxs-lookup"><span data-stu-id="6bee6-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="6bee6-143">S App Service Environment pro Power Apps je možné použít stejný název aplikace i v různých prostředích.</span><span class="sxs-lookup"><span data-stu-id="6bee6-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="6bee6-144">Vzhledem k tomu, že každé App Service prostředí má jedinečnou příponu domény, mohou vývojáři zvolit, aby v každém prostředí znovu převedly stejný název aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="6bee6-145">Například vývojář může mít aplikace, které jsou pojmenovány takto: *MyApp.foo1.p.azurewebsites.NET*, *MyApp.foo2.p.azurewebsites.NET*, *MyApp.foo3.p.azurewebsites.NET*a tak dále.</span><span class="sxs-lookup"><span data-stu-id="6bee6-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="6bee6-146">Pro aplikaci, která se tady používá, má každá instance aplikace jedinečný název.</span><span class="sxs-lookup"><span data-stu-id="6bee6-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="6bee6-147">Používané názvy instancí aplikace jsou *webfrontend1*, *webfrontend2*a *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="6bee6-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="6bee6-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="6bee6-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="6bee6-149">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="6bee6-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="6bee6-150">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="6bee6-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="6bee6-151">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="6bee6-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="6bee6-152">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="6bee6-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="6bee6-153">Část 1: vytvoření geografické distribuované aplikace</span><span class="sxs-lookup"><span data-stu-id="6bee6-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="6bee6-154">V této části vytvoříte webovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="6bee6-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6bee6-155">Vytváření webových aplikací a publikování.</span><span class="sxs-lookup"><span data-stu-id="6bee6-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="6bee6-156">Přidejte kód pro Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="6bee6-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="6bee6-157">Najeďte na sestavení aplikace na více cloudových cílů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="6bee6-158">Správa a konfigurace procesu CD</span><span class="sxs-lookup"><span data-stu-id="6bee6-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="6bee6-159">Předpoklady</span><span class="sxs-lookup"><span data-stu-id="6bee6-159">Prerequisites</span></span>

<span data-ttu-id="6bee6-160">Vyžaduje se instalace předplatného Azure a centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6bee6-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="6bee6-161">Postup geografické distribuované aplikace</span><span class="sxs-lookup"><span data-stu-id="6bee6-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="6bee6-162">Získání vlastní domény a konfigurace DNS</span><span class="sxs-lookup"><span data-stu-id="6bee6-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="6bee6-163">Aktualizujte soubor zóny DNS pro doménu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="6bee6-164">Azure AD pak může ověřit vlastnictví vlastního názvu domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="6bee6-165">Použijte [Azure DNS](/azure/dns/dns-getstarted-portal) pro azure/Microsoft 365/externí záznamy DNS v rámci Azure nebo přidejte položku DNS v [jiném registrátoru DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="6bee6-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="6bee6-166">Zaregistrujte vlastní doménu s veřejným registrátorem.</span><span class="sxs-lookup"><span data-stu-id="6bee6-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="6bee6-167">Přihlaste se k registrátorovi názvu domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="6bee6-168">K provedení aktualizací DNS může být nutný schválený správce.</span><span class="sxs-lookup"><span data-stu-id="6bee6-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="6bee6-169">Aktualizujte soubor zóny DNS pro doménu tak, že přidáte položku DNS, kterou poskytuje Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6bee6-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="6bee6-170">Položka DNS nemění chování, například směrování pošty nebo hostování webu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="6bee6-171">Vytváření webových aplikací a publikování</span><span class="sxs-lookup"><span data-stu-id="6bee6-171">Create web apps and publish</span></span>

<span data-ttu-id="6bee6-172">Nastavte hybridní průběžnou integraci/průběžné doručování (CI/CD), abyste nasadili webovou aplikaci do Azure a centra Azure Stack a automaticky nastavili změny do obou cloudů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="6bee6-173">Azure Stack centrum se správnými obrázky publikovanými pro spuštění (Windows Server a SQL) a vyžaduje se nasazení App Service.</span><span class="sxs-lookup"><span data-stu-id="6bee6-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="6bee6-174">Další informace najdete v tématu [předpoklady pro nasazení App Service v centru Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="6bee6-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="6bee6-175">Přidat kód pro Azure Repos</span><span class="sxs-lookup"><span data-stu-id="6bee6-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="6bee6-176">Přihlaste se k aplikaci Visual Studio pomocí **účtu s právy pro vytváření projektu** na Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="6bee6-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="6bee6-177">CI/CD se může vztahovat jak na kód aplikace, tak pro kód infrastruktury.</span><span class="sxs-lookup"><span data-stu-id="6bee6-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="6bee6-178">Použijte [šablony Azure Resource Manager](https://azure.microsoft.com/resources/templates/) pro vývoj privátního i hostovaného cloudu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Připojení k projektu v aplikaci Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="6bee6-180">**Naklonujte úložiště** vytvořením a otevřením výchozí webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klonovat úložiště v aplikaci Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="6bee6-182">Vytvoření nasazení webové aplikace v obou cloudech</span><span class="sxs-lookup"><span data-stu-id="6bee6-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="6bee6-183">Upravte soubor **WebApplication. csproj** : vyberte `Runtimeidentifier` a přidejte `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="6bee6-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="6bee6-184">(Viz dokumentace k [samoobslužnému nasazení](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .)</span><span class="sxs-lookup"><span data-stu-id="6bee6-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Upravit soubor projektu webové aplikace v aplikaci Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="6bee6-186">**Vrácením kódu se změnami Azure Repos** používání Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="6bee6-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="6bee6-187">Potvrďte, že **kód aplikace** byl zkontrolován do Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="6bee6-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="6bee6-188">Vytvoření definice sestavení</span><span class="sxs-lookup"><span data-stu-id="6bee6-188">Create the build definition</span></span>

1. <span data-ttu-id="6bee6-189">**Přihlaste se k Azure Pipelines** a potvrďte schopnost vytvářet definice sestavení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="6bee6-190">Přidejte `-r win10-x64` kód.</span><span class="sxs-lookup"><span data-stu-id="6bee6-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="6bee6-191">Tento dodatek je nezbytný pro aktivaci samostatného nasazení pomocí .NET Core.</span><span class="sxs-lookup"><span data-stu-id="6bee6-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Přidejte kód do definice sestavení v Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="6bee6-193">**Spusťte sestavení**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-193">**Run the build**.</span></span> <span data-ttu-id="6bee6-194">Proces [sestavení samostatného nasazení](/dotnet/core/deploying/deploy-with-vs#simpleSelf) bude publikovat artefakty, které se dají spouštět v Azure a centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6bee6-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="6bee6-195">Použití hostovaného agenta Azure</span><span class="sxs-lookup"><span data-stu-id="6bee6-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="6bee6-196">Použití hostovaného agenta v Azure Pipelines je pohodlnou možností pro sestavování a nasazování webových aplikací.</span><span class="sxs-lookup"><span data-stu-id="6bee6-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="6bee6-197">Údržba a upgrady se automaticky provádí Microsoft Azure, což umožňuje nepřerušované nasazení, testování a nasazování.</span><span class="sxs-lookup"><span data-stu-id="6bee6-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="6bee6-198">Správa a konfigurace procesu CD</span><span class="sxs-lookup"><span data-stu-id="6bee6-198">Manage and configure the CD process</span></span>

<span data-ttu-id="6bee6-199">Azure DevOps Services poskytují vysoce konfigurovatelný a spravovatelný kanál pro vydání do více prostředí, jako jsou vývojové, pracovní, QA a produkční prostředí. zahrnutí požadavku na schválení v určitých fázích.</span><span class="sxs-lookup"><span data-stu-id="6bee6-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="6bee6-200">Vytvořit definici vydané verze</span><span class="sxs-lookup"><span data-stu-id="6bee6-200">Create release definition</span></span>

1. <span data-ttu-id="6bee6-201">Kliknutím na tlačítko **plus** přidejte novou verzi na kartě **vydání** v části **sestavení a vydání** Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="6bee6-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Vytvoření definice verze v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="6bee6-203">Použijte šablonu nasazení Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="6bee6-203">Apply the Azure App Service Deployment template.</span></span>

   ![Použít šablonu nasazení Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="6bee6-205">V části **Přidat artefakt**přidejte artefakt pro aplikaci Azure Cloud Build.</span><span class="sxs-lookup"><span data-stu-id="6bee6-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Přidání artefaktu do cloudového sestavení Azure v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="6bee6-207">Na kartě kanál vyberte **fáze,** odkaz na úlohu prostředí a nastavte hodnoty cloudového prostředí Azure.</span><span class="sxs-lookup"><span data-stu-id="6bee6-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Nastavení hodnot cloudového prostředí Azure v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="6bee6-209">Nastavte **Název prostředí** a vyberte **předplatné Azure** pro koncový bod cloudu Azure.</span><span class="sxs-lookup"><span data-stu-id="6bee6-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Vyberte předplatné Azure pro koncový bod cloudu Azure v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="6bee6-211">V části **název služby App Service**nastavte požadovaný název služby Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="6bee6-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Nastavte název služby Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="6bee6-213">Do pole **fronta agenta** pro hostované cloudové prostředí Azure zadejte "hostované VS2017".</span><span class="sxs-lookup"><span data-stu-id="6bee6-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Nastavte frontu agentů pro prostředí hostované v cloudu Azure v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="6bee6-215">V nabídce nasadit Azure App Service vyberte pro prostředí platný **balíček nebo složku** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="6bee6-216">Vyberte **OK** do **umístění složky**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-216">Select **OK** to **folder location**.</span></span>
  
      ![Vyberte balíček nebo složku pro prostředí Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Vyberte balíček nebo složku pro prostředí Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="6bee6-219">Uložte všechny změny a vraťte se do **kanálu uvolnění**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Uložení změn v kanálu vydání v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="6bee6-221">Přidejte nový artefakt, který vybírá sestavení pro aplikaci Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6bee6-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Přidání nového artefaktu pro aplikaci Azure Stackového centra v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="6bee6-223">Přidejte další prostředí pomocí Azure App Service nasazení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Přidání prostředí do nasazení Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="6bee6-225">Pojmenujte nové prostředí Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6bee6-225">Name the new environment Azure Stack Hub.</span></span>

    ![Název prostředí při nasazení Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="6bee6-227">Na kartě **úloha** najděte prostředí Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6bee6-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure Stack prostředí centra v Azure DevOps Services v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="6bee6-229">Vyberte předplatné pro koncový bod centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6bee6-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Vyberte předplatné pro koncový bod centra Azure Stack v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="6bee6-231">Jako název služby App Service nastavte název webové aplikace centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6bee6-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Nastavit název webové aplikace Azure Stack hub v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="6bee6-233">Vyberte agenta centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6bee6-233">Select the Azure Stack Hub agent.</span></span>

    ![Vybrat agenta centra Azure Stack v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="6bee6-235">V části nasadit Azure App Service vyberte platný **balíček nebo složku** pro prostředí.</span><span class="sxs-lookup"><span data-stu-id="6bee6-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="6bee6-236">Vyberte **OK** do umístění složky.</span><span class="sxs-lookup"><span data-stu-id="6bee6-236">Select **OK** to folder location.</span></span>

    ![Vyberte složku pro nasazení Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Vyberte složku pro nasazení Azure App Service v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="6bee6-239">V části karta proměnné přidejte proměnnou s názvem `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , nastavte její hodnotu na **true**a obor na Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6bee6-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Přidání proměnné do nasazení aplikace Azure v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="6bee6-241">V obou artefaktech vyberte ikonu triggeru **průběžného** nasazování a povolte aktivační událost **pokračování** nasazení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Vyberte aktivační událost průběžného nasazování v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="6bee6-243">Vyberte ikonu podmínky **před nasazením** v prostředí Azure Stack hub a nastavte Trigger na **po vydání.**</span><span class="sxs-lookup"><span data-stu-id="6bee6-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Vyberte podmínky předběžného nasazení v Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="6bee6-245">Uložte všechny změny.</span><span class="sxs-lookup"><span data-stu-id="6bee6-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="6bee6-246">Některá nastavení pro úlohy mohla být při vytváření definice verze ze šablony automaticky definována jako [proměnné prostředí](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) .</span><span class="sxs-lookup"><span data-stu-id="6bee6-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="6bee6-247">Tato nastavení se nedají upravit v nastavení úlohy. místo toho je nutné vybrat nadřazenou položku prostředí pro úpravu těchto nastavení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="6bee6-248">Část 2: aktualizace možností webové aplikace</span><span class="sxs-lookup"><span data-stu-id="6bee6-248">Part 2: Update web app options</span></span>

<span data-ttu-id="6bee6-249">[Azure App Service ](/azure/app-service/overview) je vysoce škálovatelná služba s automatickými opravami pro hostování webů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="6bee6-251">Mapování existujícího vlastního názvu DNS na Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="6bee6-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="6bee6-252">Použijte **záznam CNAME** a **záznam** a k mapování vlastního názvu DNS na App Service.</span><span class="sxs-lookup"><span data-stu-id="6bee6-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="6bee6-253">Mapování existujícího vlastního názvu DNS na Azure Web Apps</span><span class="sxs-lookup"><span data-stu-id="6bee6-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="6bee6-254">Použijte záznam CNAME pro všechny vlastní názvy DNS kromě kořenové domény (například northwind.com).</span><span class="sxs-lookup"><span data-stu-id="6bee6-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="6bee6-255">Pokud chcete do služby App Service migrovat živý web a jeho název domény DNS, přečtěte si téma [Migrace aktivního názvu DNS do služby Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="6bee6-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="6bee6-256">Předpoklady</span><span class="sxs-lookup"><span data-stu-id="6bee6-256">Prerequisites</span></span>

<span data-ttu-id="6bee6-257">Dokončení tohoto řešení:</span><span class="sxs-lookup"><span data-stu-id="6bee6-257">To complete this solution:</span></span>

- <span data-ttu-id="6bee6-258">[Vytvořte aplikaci App Service](/azure/app-service/)nebo použijte aplikaci vytvořenou pro jiné řešení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="6bee6-259">Zakupte název domény a zajistěte přístup k registru DNS pro poskytovatele domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="6bee6-260">Aktualizujte soubor zóny DNS pro doménu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="6bee6-261">Azure AD ověří vlastnictví vlastního názvu domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="6bee6-262">Použijte [Azure DNS](/azure/dns/dns-getstarted-portal) pro azure/Microsoft 365/externí záznamy DNS v rámci Azure nebo přidejte položku DNS v [jiném registrátoru DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="6bee6-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="6bee6-263">Zaregistrujte vlastní doménu s veřejným registrátorem.</span><span class="sxs-lookup"><span data-stu-id="6bee6-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="6bee6-264">Přihlaste se k registrátorovi názvu domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="6bee6-265">(K provedení aktualizací DNS může být nutný schválený správce.)</span><span class="sxs-lookup"><span data-stu-id="6bee6-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="6bee6-266">Aktualizujte soubor zóny DNS pro doménu tak, že přidáte položku DNS, kterou poskytuje Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6bee6-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="6bee6-267">Pokud například chcete přidat položky DNS pro northwindcloud.com a webové \. northwindcloud.com, nakonfigurujte nastavení DNS pro kořenovou doménu northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="6bee6-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="6bee6-268">Název domény může být zakoupen pomocí [Azure Portal](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="6bee6-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="6bee6-269">Abyste mohli mapovat vlastní název DNS na webovou aplikaci, [plán služby App Service](https://azure.microsoft.com/pricing/details/app-service/) příslušné webové aplikace musí být na placené úrovni (**Shared**, **Basic**, **Standard** nebo **Premium**).</span><span class="sxs-lookup"><span data-stu-id="6bee6-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="6bee6-270">Vytvoření a mapování záznamů CNAME a a</span><span class="sxs-lookup"><span data-stu-id="6bee6-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="6bee6-271">Přístup k záznamům DNS u poskytovatele domény</span><span class="sxs-lookup"><span data-stu-id="6bee6-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="6bee6-272">Pomocí Azure DNS můžete nakonfigurovat vlastní název DNS pro Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="6bee6-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="6bee6-273">Další informace najdete v tématu popisujícím [použití Azure DNS k určení nastavení vlastní domény pro službu Azure](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="6bee6-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="6bee6-274">Přihlaste se na web hlavního poskytovatele.</span><span class="sxs-lookup"><span data-stu-id="6bee6-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="6bee6-275">Vyhledejte stránku pro správu záznamů DNS.</span><span class="sxs-lookup"><span data-stu-id="6bee6-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="6bee6-276">Každý poskytovatel domény má vlastní rozhraní záznamů DNS.</span><span class="sxs-lookup"><span data-stu-id="6bee6-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="6bee6-277">Hledejte oblasti webu označené jako **Domain Name** (Název domény), **DNS** nebo **Name Server Management** (Správa názvových serverů).</span><span class="sxs-lookup"><span data-stu-id="6bee6-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="6bee6-278">Stránky záznamů DNS je možné zobrazit v **mých doménách**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="6bee6-279">Vyhledejte soubor s názvem **zóny**, **záznamy DNS**nebo **pokročilou konfiguraci**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="6bee6-280">Následující snímek obrazovky obsahuje příklad stránky záznamů DNS:</span><span class="sxs-lookup"><span data-stu-id="6bee6-280">The following screenshot is an example of a DNS records page:</span></span>

![Příklad stránky záznamů DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="6bee6-282">V registrátoru názvu domény vyberte **Přidat nebo vytvořit** a vytvořte záznam.</span><span class="sxs-lookup"><span data-stu-id="6bee6-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="6bee6-283">Někteří poskytovatelé nabízejí různé odkazy pro přidání různých typů záznamů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="6bee6-284">Projděte si dokumentaci k poskytovateli.</span><span class="sxs-lookup"><span data-stu-id="6bee6-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="6bee6-285">Přidejte záznam CNAME pro mapování subdomény na výchozí název hostitele aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="6bee6-286">V \. příkladech domény northwindcloud.com www přidejte záznam CNAME, na který se název mapuje `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="6bee6-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="6bee6-287">Po přidání CNAME bude stránka záznamů DNS vypadat jako v následujícím příkladu:</span><span class="sxs-lookup"><span data-stu-id="6bee6-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Přechod do aplikace Azure na portálu](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="6bee6-289">Povolení mapování záznamu CNAME v Azure</span><span class="sxs-lookup"><span data-stu-id="6bee6-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="6bee6-290">Na nové kartě se přihlaste k Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="6bee6-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="6bee6-291">Přejděte na App Services.</span><span class="sxs-lookup"><span data-stu-id="6bee6-291">Go to App Services.</span></span>

3. <span data-ttu-id="6bee6-292">Vyberte webová aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-292">Select web app.</span></span>

4. <span data-ttu-id="6bee6-293">V levém navigačním panelu na stránce aplikace na webu Azure Portal vyberte **Vlastní domény**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="6bee6-294">Vyberte **+** ikonu vedle **Přidat název hostitele**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="6bee6-295">Zadejte plně kvalifikovaný název domény, třeba `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="6bee6-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="6bee6-296">Vyberte **Ověřit**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-296">Select **Validate**.</span></span>

8. <span data-ttu-id="6bee6-297">Pokud je uvedeno jinak, přidejte další záznamy jiných typů ( `A` nebo `TXT` ) do záznamů DNS registrátora názvu domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="6bee6-298">Azure nabídne hodnoty a typy těchto záznamů:</span><span class="sxs-lookup"><span data-stu-id="6bee6-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="6bee6-299">a.</span><span class="sxs-lookup"><span data-stu-id="6bee6-299">a.</span></span>  <span data-ttu-id="6bee6-300">Záznam **A** pro mapování na IP adresu aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="6bee6-301">b.</span><span class="sxs-lookup"><span data-stu-id="6bee6-301">b.</span></span>  <span data-ttu-id="6bee6-302">Záznam **TXT** pro mapování na výchozí název hostitele aplikace `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="6bee6-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="6bee6-303">App Service používá tento záznam pouze v době konfigurace k ověření vlastního vlastnictví domény.</span><span class="sxs-lookup"><span data-stu-id="6bee6-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="6bee6-304">Po ověření odstraňte záznam TXT.</span><span class="sxs-lookup"><span data-stu-id="6bee6-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="6bee6-305">Dokončete tuto úlohu na kartě registrátor domény a znovu ověřte, dokud není aktivováno tlačítko **Přidat název hostitele** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="6bee6-306">Ujistěte se, že **typ záznamu názvu hostitele** je nastavený na **CNAME** (www.example.com nebo libovolná subdoména).</span><span class="sxs-lookup"><span data-stu-id="6bee6-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="6bee6-307">Vyberte **Přidat název hostitele**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="6bee6-308">Zadejte plně kvalifikovaný název domény, třeba `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="6bee6-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="6bee6-309">Vyberte **Ověřit**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-309">Select **Validate**.</span></span> <span data-ttu-id="6bee6-310">Je aktivováno **Přidání** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="6bee6-311">Ujistěte se, že **typ záznamu názvu hostitele** je nastavený na **záznam** (example.com).</span><span class="sxs-lookup"><span data-stu-id="6bee6-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="6bee6-312">**Přidejte název hostitele**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-312">**Add hostname**.</span></span>

    <span data-ttu-id="6bee6-313">Může trvat nějakou dobu, než se nové názvy hostitelů projeví na stránce **vlastní domény** aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="6bee6-314">Zkuste aktualizovat prohlížeč, aby se data aktualizovala.</span><span class="sxs-lookup"><span data-stu-id="6bee6-314">Try refreshing the browser to update the data.</span></span>
  
    ![Vlastní domény](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="6bee6-316">Pokud dojde k chybě, zobrazí se v dolní části stránky oznámení o chybě ověření.</span><span class="sxs-lookup"><span data-stu-id="6bee6-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Chyba ověřování domény](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="6bee6-318">Výše uvedené kroky se můžou opakovat, aby se namapovala doména se zástupnými znaky ( \* . northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="6bee6-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="6bee6-319">To umožňuje přidání jakýchkoli dalších subdomén do této služby App Service, aniž by bylo nutné pro každé z nich vytvořit samostatný záznam CNAME.</span><span class="sxs-lookup"><span data-stu-id="6bee6-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="6bee6-320">Nakonfigurujte toto nastavení podle pokynů registrátora.</span><span class="sxs-lookup"><span data-stu-id="6bee6-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="6bee6-321">Testování v prohlížeči</span><span class="sxs-lookup"><span data-stu-id="6bee6-321">Test in a browser</span></span>

<span data-ttu-id="6bee6-322">Přejděte na názvy DNS, které jste nakonfigurovali dříve (například `northwindcloud.com` nebo `www.northwindcloud.com` ).</span><span class="sxs-lookup"><span data-stu-id="6bee6-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="6bee6-323">Část 3: vazba vlastního certifikátu SSL</span><span class="sxs-lookup"><span data-stu-id="6bee6-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="6bee6-324">V této části budeme:</span><span class="sxs-lookup"><span data-stu-id="6bee6-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6bee6-325">Navažte vlastní certifikát SSL na App Service.</span><span class="sxs-lookup"><span data-stu-id="6bee6-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="6bee6-326">Vyvynuťte pro aplikaci protokol HTTPS.</span><span class="sxs-lookup"><span data-stu-id="6bee6-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="6bee6-327">Automatizujte vazbu certifikátu SSL pomocí skriptů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="6bee6-328">V případě potřeby Získejte certifikát SSL zákazníka v Azure Portal a navažte ho k webové aplikaci.</span><span class="sxs-lookup"><span data-stu-id="6bee6-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="6bee6-329">Další informace najdete v [kurzu App Servicech certifikátů](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="6bee6-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="6bee6-330">Předpoklady</span><span class="sxs-lookup"><span data-stu-id="6bee6-330">Prerequisites</span></span>

<span data-ttu-id="6bee6-331">Dokončení tohoto řešení:</span><span class="sxs-lookup"><span data-stu-id="6bee6-331">To complete this  solution:</span></span>

- [<span data-ttu-id="6bee6-332">Vytvořte aplikaci App Service.</span><span class="sxs-lookup"><span data-stu-id="6bee6-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="6bee6-333">Namapujte vlastní název DNS na webovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="6bee6-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="6bee6-334">Získejte certifikát SSL od důvěryhodné certifikační autority a použijte klíč k podepsání žádosti.</span><span class="sxs-lookup"><span data-stu-id="6bee6-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="6bee6-335">Požadavky na certifikát SSL</span><span class="sxs-lookup"><span data-stu-id="6bee6-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="6bee6-336">Pokud chcete ve službě App Service použít certifikát, musí splňovat všechny následující požadavky:</span><span class="sxs-lookup"><span data-stu-id="6bee6-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="6bee6-337">Podepsáno důvěryhodnou certifikační autoritou.</span><span class="sxs-lookup"><span data-stu-id="6bee6-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="6bee6-338">Exportováno jako soubor PFX chráněný heslem.</span><span class="sxs-lookup"><span data-stu-id="6bee6-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="6bee6-339">Obsahuje privátní klíč minimálně 2048 bitů dlouhého.</span><span class="sxs-lookup"><span data-stu-id="6bee6-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="6bee6-340">Obsahuje všechny zprostředkující certifikáty v řetězu certifikátů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="6bee6-341">**Certifikáty ECC (eliptický Curve Cryptography)** fungují s App Service, ale nejsou součástí této příručky.</span><span class="sxs-lookup"><span data-stu-id="6bee6-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="6bee6-342">Pomoc při vytváření certifikátů ECC získáte od certifikační autority.</span><span class="sxs-lookup"><span data-stu-id="6bee6-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="6bee6-343">Příprava webové aplikace</span><span class="sxs-lookup"><span data-stu-id="6bee6-343">Prepare the web app</span></span>

<span data-ttu-id="6bee6-344">Aby bylo možné vytvořit navázání vlastního certifikátu SSL k webové aplikaci, musí být [plán App Service](https://azure.microsoft.com/pricing/details/app-service/) v úrovni **Basic**, **Standard**nebo **Premium** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="6bee6-345">Přihlášení k Azure</span><span class="sxs-lookup"><span data-stu-id="6bee6-345">Sign in to Azure</span></span>

1. <span data-ttu-id="6bee6-346">Otevřete [Azure Portal](https://portal.azure.com/) a pokračujte na webovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="6bee6-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="6bee6-347">V nabídce vlevo vyberte **App Services**a pak vyberte název webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="6bee6-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Vyberte možnost Webová aplikace v Azure Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="6bee6-349">Kontrola cenové úrovně</span><span class="sxs-lookup"><span data-stu-id="6bee6-349">Check the pricing tier</span></span>

1. <span data-ttu-id="6bee6-350">V levé navigační části stránky webové aplikace se posuňte do části **Nastavení** a vyberte **škálovat nahoru (App Service plán)**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Škálování nabídky ve webové aplikaci](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="6bee6-352">Ujistěte se, že webová aplikace není na úrovni **Free** nebo **Shared** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="6bee6-353">Aktuální úroveň webové aplikace je zvýrazněna v tmavě modrém poli.</span><span class="sxs-lookup"><span data-stu-id="6bee6-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Kontrolovat cenovou úroveň ve webové aplikaci](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="6bee6-355">Vlastní protokol SSL není podporován na úrovni **Free** nebo **Shared** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="6bee6-356">Pokud chcete provést škálování, postupujte podle kroků v následující části nebo na stránce **vyberte cenovou úroveň** a přejděte k části [odeslání a vytvoření vazby certifikátu SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="6bee6-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="6bee6-357">Vertikální navýšení kapacity plánu služby App Service</span><span class="sxs-lookup"><span data-stu-id="6bee6-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="6bee6-358">Vyberte některou z úrovní **Basic**, **Standard** nebo **Premium**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="6bee6-359">Vyberte **Vybrat**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-359">Select **Select**.</span></span>

![Výběr cenové úrovně pro vaši webovou aplikaci](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="6bee6-361">Operace škálování je dokončena, když se zobrazí oznámení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-361">The scale operation is complete when notification is displayed.</span></span>

![Oznámení vertikálního navýšení kapacity](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="6bee6-363">Vytvoření vazby certifikátu SSL a sloučení zprostředkujících certifikátů</span><span class="sxs-lookup"><span data-stu-id="6bee6-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="6bee6-364">Sloučí více certifikátů v řetězu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="6bee6-365">**Otevřete každý certifikát** , který jste obdrželi v textovém editoru.</span><span class="sxs-lookup"><span data-stu-id="6bee6-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="6bee6-366">Vytvořte soubor pro sloučený certifikát nazvaný *mergedcertificate. CRT*.</span><span class="sxs-lookup"><span data-stu-id="6bee6-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="6bee6-367">V textovém editoru zkopírujte do tohoto souboru obsah jednotlivých certifikátů.</span><span class="sxs-lookup"><span data-stu-id="6bee6-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="6bee6-368">Pořadí certifikátů by mělo odpovídat jejich pořadí v řetězu certifikátů, počínaje vaším certifikátem a konče kořenovým certifikátem.</span><span class="sxs-lookup"><span data-stu-id="6bee6-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="6bee6-369">Soubor bude vypadat jako v následujícím příkladu:</span><span class="sxs-lookup"><span data-stu-id="6bee6-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="6bee6-370">Export certifikátu do formátu PFX</span><span class="sxs-lookup"><span data-stu-id="6bee6-370">Export certificate to PFX</span></span>

<span data-ttu-id="6bee6-371">Exportujte sloučený certifikát SSL s privátním klíčem vygenerovaným certifikátem.</span><span class="sxs-lookup"><span data-stu-id="6bee6-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="6bee6-372">Soubor privátního klíče se vytvoří prostřednictvím OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="6bee6-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="6bee6-373">Pokud chcete certifikát exportovat do souboru PFX, spusťte následující příkaz a nahraďte zástupné symboly `<private-key-file>` a `<merged-certificate-file>` cestu k privátnímu klíči a souborem sloučeného certifikátu:</span><span class="sxs-lookup"><span data-stu-id="6bee6-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="6bee6-374">Po zobrazení výzvy definujte heslo pro export pro nahrání certifikátu SSL pro App Service později.</span><span class="sxs-lookup"><span data-stu-id="6bee6-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="6bee6-375">Když se k vygenerování žádosti o certifikát používá služba IIS nebo **Certreq.exe** , nainstalujte certifikát do místního počítače a pak [certifikát EXPORTUJTE do souboru PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="6bee6-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="6bee6-376">Nahrajte certifikát SSL.</span><span class="sxs-lookup"><span data-stu-id="6bee6-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="6bee6-377">V levém navigačním panelu webové aplikace vyberte **Nastavení SSL** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="6bee6-378">Vyberte **Odeslat certifikát**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="6bee6-379">V **souboru certifikátu PFX**vyberte soubor PFX.</span><span class="sxs-lookup"><span data-stu-id="6bee6-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="6bee6-380">Do pole **heslo certifikátu**zadejte heslo vytvořené při exportování souboru PFX.</span><span class="sxs-lookup"><span data-stu-id="6bee6-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="6bee6-381">Vyberte **Nahrát**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-381">Select **Upload**.</span></span>

    ![Odeslat certifikát SSL](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="6bee6-383">Až App Service dokončí nahrávání certifikátu, zobrazí se na stránce **Nastavení SSL** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Nastavení protokolu SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="6bee6-385">Vytvoření vazby certifikátu SSL</span><span class="sxs-lookup"><span data-stu-id="6bee6-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="6bee6-386">V části **vazby SSL** vyberte **Přidat vazbu**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="6bee6-387">Pokud se certifikát nahrál, ale v rozevíracím seznamu název **hostitele** se nezobrazí v části názvy domén, zkuste aktualizovat stránku prohlížeče.</span><span class="sxs-lookup"><span data-stu-id="6bee6-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="6bee6-388">Na stránce **Přidat vazbu SSL** vyberte rozevírací seznam a vyberte název domény, který chcete zabezpečit, a certifikát, který chcete použít.</span><span class="sxs-lookup"><span data-stu-id="6bee6-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="6bee6-389">V části **Typ SSL** vyberte, jestli se má použít SSL na základě [**Indikace názvu serveru (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) nebo IP adresy.</span><span class="sxs-lookup"><span data-stu-id="6bee6-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="6bee6-390">**SSL založené na sni**: přidat se dá víc vazeb SSL založených na sni.</span><span class="sxs-lookup"><span data-stu-id="6bee6-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="6bee6-391">Tato možnost umožňuje zabezpečení několika domén na stejné IP adrese pomocí několika certifikátů SSL.</span><span class="sxs-lookup"><span data-stu-id="6bee6-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="6bee6-392">Většina moderních prohlížečů (včetně prohlížečů Internet Explorer, Chrome, Firefox a Opera) podporuje SNI (ucelenější informace o podpoře prohlížečů najdete v článku o [Indikaci názvu serveru](https://wikipedia.org/wiki/Server_Name_Indication)).</span><span class="sxs-lookup"><span data-stu-id="6bee6-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="6bee6-393">**SSL na základě IP adresy**: dá se přidat jenom jedna vazba SSL založená na IP adrese.</span><span class="sxs-lookup"><span data-stu-id="6bee6-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="6bee6-394">Tato možnost umožňuje zabezpečení vyhrazené veřejné IP adresy pouze jedním certifikátem SSL.</span><span class="sxs-lookup"><span data-stu-id="6bee6-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="6bee6-395">Chcete-li zabezpečit více domén, zabezpečte je pomocí stejného certifikátu SSL.</span><span class="sxs-lookup"><span data-stu-id="6bee6-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="6bee6-396">Protokol SSL založený na protokolu IP je tradiční volbou pro vazby SSL.</span><span class="sxs-lookup"><span data-stu-id="6bee6-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="6bee6-397">Vyberte **Přidat vazbu**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-397">Select **Add Binding**.</span></span>

    ![Přidat vazbu SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="6bee6-399">Až App Service dokončí nahrávání certifikátu, zobrazí se v oddílech **vazeb SSL** .</span><span class="sxs-lookup"><span data-stu-id="6bee6-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Nahrávání vazeb SSL bylo dokončeno.](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="6bee6-401">Přemapování záznamu A pro IP SSL</span><span class="sxs-lookup"><span data-stu-id="6bee6-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="6bee6-402">Pokud se ve webové aplikaci nepoužívá protokol SSL založený na protokolu IP, přeskočte na [test HTTPS pro vaši vlastní doménu](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="6bee6-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="6bee6-403">Ve výchozím nastavení webová aplikace používá sdílenou veřejnou IP adresu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="6bee6-404">Pokud je certifikát vázán pomocí protokolu SSL založeného na protokolu IP, App Service vytvoří novou a vyhrazenou IP adresu pro webovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="6bee6-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="6bee6-405">Pokud je záznam A mapován na webovou aplikaci, je nutné aktualizovat registr domény pomocí vyhrazené IP adresy.</span><span class="sxs-lookup"><span data-stu-id="6bee6-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="6bee6-406">Stránka **vlastní doména** je aktualizována novou vyhrazenou IP adresou.</span><span class="sxs-lookup"><span data-stu-id="6bee6-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="6bee6-407">Zkopírujte tuto [IP adresu](/azure/app-service/app-service-web-tutorial-custom-domain)a pak přemapujte [záznam a](/azure/app-service/app-service-web-tutorial-custom-domain) na tuto novou IP adresu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="6bee6-408">Test HTTPS</span><span class="sxs-lookup"><span data-stu-id="6bee6-408">Test HTTPS</span></span>

<span data-ttu-id="6bee6-409">V různých prohlížečích přejdete na adresu, `https://<your.custom.domain>` abyste zajistili, že se webová aplikace doručí.</span><span class="sxs-lookup"><span data-stu-id="6bee6-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Přejít k webové aplikaci](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="6bee6-411">Pokud dojde k chybám ověření certifikátu, může to být způsobeno certifikátem podepsaným svým držitelem nebo při exportu do souboru PFX byly pozměněny zprostředkující certifikáty.</span><span class="sxs-lookup"><span data-stu-id="6bee6-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="6bee6-412">Vynucení protokolu HTTPS</span><span class="sxs-lookup"><span data-stu-id="6bee6-412">Enforce HTTPS</span></span>

<span data-ttu-id="6bee6-413">Ve výchozím nastavení má kdokoli přístup k webové aplikaci přes HTTP.</span><span class="sxs-lookup"><span data-stu-id="6bee6-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="6bee6-414">Je možné přesměrovat všechny požadavky HTTP na port HTTPS.</span><span class="sxs-lookup"><span data-stu-id="6bee6-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="6bee6-415">Na stránce webová aplikace vyberte **Nastavení SL**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="6bee6-416">Pak v části **Pouze HTTPS** vyberte **Zapnuto**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Vynucení protokolu HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="6bee6-418">Po dokončení operace přejděte na libovolnou adresu URL protokolu HTTP, která odkazuje na aplikaci.</span><span class="sxs-lookup"><span data-stu-id="6bee6-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="6bee6-419">Příklad:</span><span class="sxs-lookup"><span data-stu-id="6bee6-419">For example:</span></span>

- <span data-ttu-id="6bee6-420">https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="6bee6-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="6bee6-421">Vynucení protokolu TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="6bee6-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="6bee6-422">Aplikace ve výchozím nastavení povolí protokol [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0, který se už nepovažuje za zabezpečený oborovou normou (například [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span><span class="sxs-lookup"><span data-stu-id="6bee6-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="6bee6-423">Pokud chcete vynucovat vyšší verze protokolu TLS, postupujte následovně:</span><span class="sxs-lookup"><span data-stu-id="6bee6-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="6bee6-424">Na stránce webová aplikace v levém navigačním panelu vyberte **Nastavení SSL**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="6bee6-425">V části **verze TLS**vyberte minimální verzi TLS.</span><span class="sxs-lookup"><span data-stu-id="6bee6-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Vynucení protokolu TLS 1.1 nebo 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="6bee6-427">Vytvoření profilu Traffic Manageru</span><span class="sxs-lookup"><span data-stu-id="6bee6-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="6bee6-428">Vyberte **vytvořit prostředek**  >  **sítě**  >  **Traffic Manager profil**  >  **vytvořit**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="6bee6-429">Část **Vytvořit profil služby Traffic Manager** vyplňte následovně:</span><span class="sxs-lookup"><span data-stu-id="6bee6-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="6bee6-430">Do pole **název**zadejte název profilu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="6bee6-431">Tento název musí být jedinečný v rámci zóny přenosů manager.net a má za následek název DNS, trafficmanager.net, který se používá pro přístup k profilu Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="6bee6-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="6bee6-432">V části **způsob směrování**vyberte **metodu geografického směrování**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="6bee6-433">V části **předplatné**vyberte předplatné, ve kterém chcete vytvořit tento profil.</span><span class="sxs-lookup"><span data-stu-id="6bee6-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="6bee6-434">V části **Skupina prostředků** vytvořte novou skupinu prostředků, do které chcete profil umístit.</span><span class="sxs-lookup"><span data-stu-id="6bee6-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="6bee6-435">V poli **Umístění skupiny prostředků** vyberte umístění skupiny prostředků.</span><span class="sxs-lookup"><span data-stu-id="6bee6-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="6bee6-436">Toto nastavení odkazuje na umístění skupiny prostředků a nemá žádný vliv na globálně nasazený profil Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="6bee6-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="6bee6-437">Vyberte **Vytvořit**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-437">Select **Create**.</span></span>

    7. <span data-ttu-id="6bee6-438">Po dokončení globálního nasazení profilu Traffic Manager se v příslušné skupině prostředků zobrazí jako jeden z prostředků.</span><span class="sxs-lookup"><span data-stu-id="6bee6-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Skupiny prostředků v profilu Create Traffic Manager](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="6bee6-440">Přidání koncových bodů služby Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="6bee6-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="6bee6-441">Na panelu hledání na portálu vyhledejte název **profilu Traffic Manager** vytvořeného v předchozí části a v zobrazených výsledcích vyberte profil Traffic Manageru.</span><span class="sxs-lookup"><span data-stu-id="6bee6-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="6bee6-442">V **Traffic Manager profil**v části **Nastavení** vyberte **koncové body**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="6bee6-443">Vyberte **Přidat**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-443">Select **Add**.</span></span>

4. <span data-ttu-id="6bee6-444">Přidává se koncový bod centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6bee6-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="6bee6-445">Jako **typ**vyberte **externí koncový bod**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="6bee6-446">Zadejte **název** tohoto koncového bodu, v ideálním případě název centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6bee6-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="6bee6-447">Pro plně kvalifikovaný název domény (**FQDN**) použijte externí adresu URL pro webovou aplikaci Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="6bee6-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="6bee6-448">V části geografické mapování vyberte oblast nebo kontinent, kde se prostředek nachází.</span><span class="sxs-lookup"><span data-stu-id="6bee6-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="6bee6-449">Například **Evropa.**</span><span class="sxs-lookup"><span data-stu-id="6bee6-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="6bee6-450">V rozevíracím seznamu země/oblast, který se zobrazí, vyberte zemi, která se vztahuje k tomuto koncovému bodu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="6bee6-451">Například **Německo**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="6bee6-452">Políčko **Přidat jako zakázaný** ponechte nezaškrtnuté.</span><span class="sxs-lookup"><span data-stu-id="6bee6-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="6bee6-453">Vyberte **OK**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-453">Select **OK**.</span></span>

12. <span data-ttu-id="6bee6-454">Přidání Koncový bod Azure:</span><span class="sxs-lookup"><span data-stu-id="6bee6-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="6bee6-455">Jako **typ**vyberte **koncový bod Azure**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="6bee6-456">Zadejte **název** koncového bodu.</span><span class="sxs-lookup"><span data-stu-id="6bee6-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="6bee6-457">Jako **typ cílového prostředku**vyberte **App Service**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="6bee6-458">V části **cílový prostředek**vyberte **možnost zvolit službu App Service** , abyste zobrazili výpis Web Apps v rámci stejného předplatného.</span><span class="sxs-lookup"><span data-stu-id="6bee6-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="6bee6-459">V části **prostředek**vyberte službu App Service, která se používá jako první koncový bod.</span><span class="sxs-lookup"><span data-stu-id="6bee6-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="6bee6-460">V části geografické mapování vyberte oblast nebo kontinent, kde se prostředek nachází.</span><span class="sxs-lookup"><span data-stu-id="6bee6-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="6bee6-461">Například **Severní Amerika/Střední Amerika/Karibská oblast.**</span><span class="sxs-lookup"><span data-stu-id="6bee6-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="6bee6-462">V rozevíracím seznamu země/oblast, který se zobrazí, ponechte toto políčko prázdné, pokud chcete vybrat všechna výše uvedená oblastní seskupení.</span><span class="sxs-lookup"><span data-stu-id="6bee6-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="6bee6-463">Políčko **Přidat jako zakázaný** ponechte nezaškrtnuté.</span><span class="sxs-lookup"><span data-stu-id="6bee6-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="6bee6-464">Vyberte **OK**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="6bee6-465">Vytvořte alespoň jeden koncový bod s geografickým rozsahem všech (World), který bude sloužit jako výchozí koncový bod pro daný prostředek.</span><span class="sxs-lookup"><span data-stu-id="6bee6-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="6bee6-466">Když se dokončí přidávání obou koncových bodů, zobrazí se v **profilu Traffic Manager** spolu s jejich stavem monitorování jako **online**.</span><span class="sxs-lookup"><span data-stu-id="6bee6-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Stav koncového bodu profilu Traffic Manager](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="6bee6-468">Globální podnik spoléhá na možnosti geografické distribuce Azure</span><span class="sxs-lookup"><span data-stu-id="6bee6-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="6bee6-469">Přímý přenos dat prostřednictvím Azure Traffic Manager a koncových bodů specifických pro geografie umožňuje globálním podnikům dodržovat regionální předpisy a zachovat předpisy a zajistit jejich kompatibilitu a zabezpečení, což je zásadní pro úspěch místních i vzdálených obchodních umístění.</span><span class="sxs-lookup"><span data-stu-id="6bee6-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6bee6-470">Další kroky</span><span class="sxs-lookup"><span data-stu-id="6bee6-470">Next steps</span></span>

- <span data-ttu-id="6bee6-471">Další informace o vzorech cloudu Azure najdete v tématu [vzory návrhu cloudu](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="6bee6-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
