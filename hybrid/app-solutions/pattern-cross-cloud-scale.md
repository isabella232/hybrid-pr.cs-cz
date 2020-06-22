---
title: Vzor škálování mezi cloudy v Azure Stackovém centru
description: Naučte se vytvářet škálovatelné aplikace pro více cloudů v Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910124"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="bfb6d-103">Vzor škálování mezi cloudy</span><span class="sxs-lookup"><span data-stu-id="bfb6d-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="bfb6d-104">Automatické přidání prostředků do existující aplikace, aby se přizpůsobilo zvýšení zátěže.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="bfb6d-105">Kontext a problém</span><span class="sxs-lookup"><span data-stu-id="bfb6d-105">Context and problem</span></span>

<span data-ttu-id="bfb6d-106">Vaše aplikace nemůže zvýšit kapacitu pro splnění neočekávaného zvýšení poptávky.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="bfb6d-107">Výsledkem tohoto nedostatku je to, že uživatelé nedosáhnou aplikace během špičky využití.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="bfb6d-108">Aplikace může obsluhovat pevný počet uživatelů.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="bfb6d-109">Globální podniky vyžadují zabezpečené, spolehlivé a dostupné cloudové aplikace.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="bfb6d-110">Schůzka roste na vyžádání a používání správné infrastruktury pro podporu této poptávky je kritická.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="bfb6d-111">Firmy se bojovat na rovnováhu mezi náklady a údržbou pomocí zabezpečení, úložiště a dostupnosti obchodních dat v reálném čase.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="bfb6d-112">Možná nebudete moct aplikaci spustit ve veřejném cloudu.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="bfb6d-113">Nemusí ale být hospodářsky proveditelné, aby společnost udržovala kapacitu potřebnou v místním prostředí, aby mohla zpracovávat špičky v poptávce pro aplikaci.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="bfb6d-114">V tomto modelu můžete použít pružnost veřejného cloudu s vaším místním řešením.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="bfb6d-115">Řešení</span><span class="sxs-lookup"><span data-stu-id="bfb6d-115">Solution</span></span>

<span data-ttu-id="bfb6d-116">Vzor škálování mezi cloudy rozšiřuje aplikaci umístěnou v místním cloudu s využitím veřejných cloudových prostředků.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="bfb6d-117">Tento model se aktivuje zvýšením nebo snížením poptávky a v tomto pořadí přidá nebo odebere prostředky v cloudu.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="bfb6d-118">Tyto prostředky poskytují redundanci, rychlou dostupnost a geograficky vyhovující směrování.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Vzor škálování mezi cloudy](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="bfb6d-120">Tento model se vztahuje pouze na bezstavové komponenty vaší aplikace.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="bfb6d-121">Komponenty</span><span class="sxs-lookup"><span data-stu-id="bfb6d-121">Components</span></span>

<span data-ttu-id="bfb6d-122">Vzor škálování mezi cloudy se skládá z následujících součástí.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="bfb6d-123">Mimo Cloud</span><span class="sxs-lookup"><span data-stu-id="bfb6d-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="bfb6d-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="bfb6d-124">Traffic Manager</span></span>

<span data-ttu-id="bfb6d-125">V diagramu se nachází mimo skupinu veřejných cloudů, ale musela by koordinovat provoz v místním datacentru i ve veřejném cloudu.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="bfb6d-126">Nástroj pro vyrovnávání zatížení poskytuje vysokou dostupnost pro aplikace monitorováním koncových bodů a zajištěním přerozdělení převzetí služeb při selhání.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="bfb6d-127">DNS (Domain Name System)</span><span class="sxs-lookup"><span data-stu-id="bfb6d-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="bfb6d-128">Název domény systému nebo DNS zodpovídá za překlad (nebo překladu) názvu webu nebo služby na jeho IP adresu.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="bfb6d-129">Cloud</span><span class="sxs-lookup"><span data-stu-id="bfb6d-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="bfb6d-130">Hostovaný sestavovací Server</span><span class="sxs-lookup"><span data-stu-id="bfb6d-130">Hosted build server</span></span>

<span data-ttu-id="bfb6d-131">Prostředí pro hostování vašeho kanálu sestavení.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="bfb6d-132">Prostředky aplikace</span><span class="sxs-lookup"><span data-stu-id="bfb6d-132">App resources</span></span>

<span data-ttu-id="bfb6d-133">Prostředky aplikace musí umožňovat horizontální navýšení kapacity a horizontální navýšení kapacity, jako jsou sady škálování a kontejnery pro virtuální počítače.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="bfb6d-134">Vlastní název domény</span><span class="sxs-lookup"><span data-stu-id="bfb6d-134">Custom domain name</span></span>

<span data-ttu-id="bfb6d-135">Pro požadavky směrování glob použít vlastní název domény.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="bfb6d-136">Veřejné IP adresy</span><span class="sxs-lookup"><span data-stu-id="bfb6d-136">Public IP addresses</span></span>

<span data-ttu-id="bfb6d-137">Veřejné IP adresy se používají ke směrování příchozího provozu prostřednictvím Traffic Manageru do koncového bodu prostředků cloudové aplikace.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="bfb6d-138">Místní Cloud</span><span class="sxs-lookup"><span data-stu-id="bfb6d-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="bfb6d-139">Hostovaný sestavovací Server</span><span class="sxs-lookup"><span data-stu-id="bfb6d-139">Hosted build server</span></span>

<span data-ttu-id="bfb6d-140">Prostředí pro hostování vašeho kanálu sestavení.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="bfb6d-141">Prostředky aplikace</span><span class="sxs-lookup"><span data-stu-id="bfb6d-141">App resources</span></span>

<span data-ttu-id="bfb6d-142">Prostředky aplikace potřebují možnost horizontálního navýšení kapacity a horizontálního navýšení kapacity, jako jsou například sady škálování virtuálních počítačů a kontejnery.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="bfb6d-143">Vlastní název domény</span><span class="sxs-lookup"><span data-stu-id="bfb6d-143">Custom domain name</span></span>

<span data-ttu-id="bfb6d-144">Pro požadavky směrování glob použít vlastní název domény.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="bfb6d-145">Veřejné IP adresy</span><span class="sxs-lookup"><span data-stu-id="bfb6d-145">Public IP addresses</span></span>

<span data-ttu-id="bfb6d-146">Veřejné IP adresy se používají ke směrování příchozího provozu prostřednictvím Traffic Manageru do koncového bodu prostředků cloudové aplikace.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="bfb6d-147">Problémy a důležité informace</span><span class="sxs-lookup"><span data-stu-id="bfb6d-147">Issues and considerations</span></span>

<span data-ttu-id="bfb6d-148">Když se budete rozhodovat, jak tento model implementovat, měli byste vzít v úvahu následující skutečnosti:</span><span class="sxs-lookup"><span data-stu-id="bfb6d-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="bfb6d-149">Škálovatelnost</span><span class="sxs-lookup"><span data-stu-id="bfb6d-149">Scalability</span></span>

<span data-ttu-id="bfb6d-150">Klíčovou součástí škálování mezi cloudy je schopnost doručovat škálování na vyžádání.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="bfb6d-151">Škálování musí probíhat mezi veřejnou a místní cloudovou infrastrukturou a poskytovat konzistentní a spolehlivé služby na vyžádání.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="bfb6d-152">Dostupnost</span><span class="sxs-lookup"><span data-stu-id="bfb6d-152">Availability</span></span>

<span data-ttu-id="bfb6d-153">Zajistěte, aby lokálně nasazené aplikace byly nakonfigurované pro vysokou dostupnost prostřednictvím místní konfigurace hardwaru a nasazení softwaru.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="bfb6d-154">Možnosti správy</span><span class="sxs-lookup"><span data-stu-id="bfb6d-154">Manageability</span></span>

<span data-ttu-id="bfb6d-155">Vzor křížového cloudu zajišťuje bezproblémové řízení a známé rozhraní mezi prostředími.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="bfb6d-156">Kdy se má tento model použít</span><span class="sxs-lookup"><span data-stu-id="bfb6d-156">When to use this pattern</span></span>

<span data-ttu-id="bfb6d-157">Použijte tento model:</span><span class="sxs-lookup"><span data-stu-id="bfb6d-157">Use this pattern:</span></span>

- <span data-ttu-id="bfb6d-158">Když potřebujete zvýšit kapacitu vaší aplikace s neočekávanými požadavky nebo pravidelnými požadavky na vyžádání.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="bfb6d-159">Pokud nechcete investovat do prostředků, které se budou používat jenom během špičky.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="bfb6d-160">Platíte za to, co využijete.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-160">Pay for what you use.</span></span>

<span data-ttu-id="bfb6d-161">Tento model se nedoporučuje, pokud:</span><span class="sxs-lookup"><span data-stu-id="bfb6d-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="bfb6d-162">Vaše řešení vyžaduje, aby se uživatelé připojovali přes Internet.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="bfb6d-163">Vaše firma má místní předpisy, které vyžadují, aby původní připojení pocházelo z volání na pracovišti.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="bfb6d-164">Vaše síť bude mít normální kritická místa, která by omezila výkon škálování.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="bfb6d-165">Vaše prostředí je odpojené od Internetu a nemůže se připojit k veřejnému cloudu.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="bfb6d-166">Další kroky</span><span class="sxs-lookup"><span data-stu-id="bfb6d-166">Next steps</span></span>

<span data-ttu-id="bfb6d-167">Další informace o tématech zavedených v tomto článku:</span><span class="sxs-lookup"><span data-stu-id="bfb6d-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="bfb6d-168">Další informace o tom, jak tento nástroj pro vyrovnávání zatížení využívající službu DNS funguje, najdete v tématu [Přehled Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) .</span><span class="sxs-lookup"><span data-stu-id="bfb6d-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="bfb6d-169">Další informace o osvědčených postupech a získání odpovědí na jakékoli další otázky najdete v tématu [aspekty návrhu hybridní aplikace](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="bfb6d-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="bfb6d-170">Další informace o celém portfoliu produktů a řešení najdete v [Azure Stack rodině produktů a řešení](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="bfb6d-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="bfb6d-171">Až budete připraveni otestovat příklad řešení, pokračujte v [Průvodci nasazením řešení škálování mezi cloudy](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="bfb6d-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="bfb6d-172">Průvodce nasazením poskytuje podrobné pokyny pro nasazení a testování jeho komponent.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="bfb6d-173">Naučíte se, jak vytvořit řešení pro různé cloudy, které umožní ručně aktivovaný proces přepnutí z hostované webové aplikace Azure Stack hub do hostované webové aplikace Azure.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="bfb6d-174">Naučíte se také, jak používat automatické škálování prostřednictvím Traffic Manageru, což zajišťuje flexibilní a škálovatelný cloudový nástroj při zatížení.</span><span class="sxs-lookup"><span data-stu-id="bfb6d-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
