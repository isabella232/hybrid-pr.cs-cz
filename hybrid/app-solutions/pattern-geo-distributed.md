---
title: Model geografické distribuované aplikace v centru Azure Stack
description: Přečtěte si o modelu geograficky distribuovaných aplikací pro inteligentní Edge pomocí Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910052"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="33d94-103">Model geograficky distribuované aplikace</span><span class="sxs-lookup"><span data-stu-id="33d94-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="33d94-104">Naučte se poskytovat koncové body aplikací napříč několika oblastmi a směrovat provoz uživatelů na základě umístění a požadavků na dodržování předpisů.</span><span class="sxs-lookup"><span data-stu-id="33d94-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="33d94-105">Kontext a problém</span><span class="sxs-lookup"><span data-stu-id="33d94-105">Context and problem</span></span>

<span data-ttu-id="33d94-106">Organizace, které mají rozsáhlou škálu geografických oblastí, se snaží bezpečně a přesně distribuovat a umožňují přístup k datům a současně zajišťují požadavky na zabezpečení, dodržování předpisů a výkon na uživatele, umístění a zařízení přes hranice.</span><span class="sxs-lookup"><span data-stu-id="33d94-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="33d94-107">Řešení</span><span class="sxs-lookup"><span data-stu-id="33d94-107">Solution</span></span>

<span data-ttu-id="33d94-108">Model směrování geografického provozu Azure Stack centra nebo geografické distribuované aplikace umožňuje směrovat provoz na konkrétní koncové body na základě různých metrik.</span><span class="sxs-lookup"><span data-stu-id="33d94-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="33d94-109">Vytvoření Traffic Manager s využitím geograficky směrování a konfigurace koncového bodu směruje provoz do koncových bodů na základě regionálních požadavků, podnikových a mezinárodních předpisů a datových potřeb.</span><span class="sxs-lookup"><span data-stu-id="33d94-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Geograficky distribuovaný model](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="33d94-111">Komponenty</span><span class="sxs-lookup"><span data-stu-id="33d94-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="33d94-112">Mimo Cloud</span><span class="sxs-lookup"><span data-stu-id="33d94-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="33d94-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="33d94-113">Traffic Manager</span></span>

<span data-ttu-id="33d94-114">V diagramu se Traffic Manager nachází mimo veřejný cloud, ale musí být schopný koordinovat provoz v místním datacentru i ve veřejném cloudu.</span><span class="sxs-lookup"><span data-stu-id="33d94-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="33d94-115">Nástroj pro vyrovnávání zatížení směruje provoz do geografických umístění.</span><span class="sxs-lookup"><span data-stu-id="33d94-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="33d94-116">DNS (Domain Name System)</span><span class="sxs-lookup"><span data-stu-id="33d94-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="33d94-117">Název domény systému nebo DNS zodpovídá za překlad (nebo překladu) názvu webu nebo služby na jeho IP adresu.</span><span class="sxs-lookup"><span data-stu-id="33d94-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="33d94-118">Veřejný cloud</span><span class="sxs-lookup"><span data-stu-id="33d94-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="33d94-119">Koncový bod cloudu</span><span class="sxs-lookup"><span data-stu-id="33d94-119">Cloud Endpoint</span></span>

<span data-ttu-id="33d94-120">Veřejné IP adresy se používají ke směrování příchozího provozu prostřednictvím Traffic Manageru do koncového bodu prostředků cloudové aplikace.</span><span class="sxs-lookup"><span data-stu-id="33d94-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="33d94-121">Místní cloudy</span><span class="sxs-lookup"><span data-stu-id="33d94-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="33d94-122">Místní koncový bod</span><span class="sxs-lookup"><span data-stu-id="33d94-122">Local endpoint</span></span>

<span data-ttu-id="33d94-123">Veřejné IP adresy se používají ke směrování příchozího provozu prostřednictvím Traffic Manageru do koncového bodu prostředků cloudové aplikace.</span><span class="sxs-lookup"><span data-stu-id="33d94-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="33d94-124">Problémy a důležité informace</span><span class="sxs-lookup"><span data-stu-id="33d94-124">Issues and considerations</span></span>

<span data-ttu-id="33d94-125">Když se budete rozhodovat, jak tento model implementovat, měli byste vzít v úvahu následující skutečnosti:</span><span class="sxs-lookup"><span data-stu-id="33d94-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="33d94-126">Škálovatelnost</span><span class="sxs-lookup"><span data-stu-id="33d94-126">Scalability</span></span>

<span data-ttu-id="33d94-127">Tento model zpracovává směr geografického provozu, nikoli škálování pro zvýšení provozu.</span><span class="sxs-lookup"><span data-stu-id="33d94-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="33d94-128">Tento model však můžete zkombinovat s jinými řešeními Azure a místními řešeními.</span><span class="sxs-lookup"><span data-stu-id="33d94-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="33d94-129">Tento model se dá použít například se vzorem škálování mezi cloudy.</span><span class="sxs-lookup"><span data-stu-id="33d94-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="33d94-130">Dostupnost</span><span class="sxs-lookup"><span data-stu-id="33d94-130">Availability</span></span>

<span data-ttu-id="33d94-131">Zajistěte, aby lokálně nasazené aplikace byly nakonfigurované pro vysokou dostupnost prostřednictvím místní konfigurace hardwaru a nasazení softwaru.</span><span class="sxs-lookup"><span data-stu-id="33d94-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="33d94-132">Možnosti správy</span><span class="sxs-lookup"><span data-stu-id="33d94-132">Manageability</span></span>

<span data-ttu-id="33d94-133">Vzor zajišťuje bezproblémové řízení a známé rozhraní mezi prostředími.</span><span class="sxs-lookup"><span data-stu-id="33d94-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="33d94-134">Kdy se má tento model použít</span><span class="sxs-lookup"><span data-stu-id="33d94-134">When to use this pattern</span></span>

- <span data-ttu-id="33d94-135">Moje organizace má mezinárodní pobočky vyžadující vlastní regionální zásady zabezpečení a distribuce.</span><span class="sxs-lookup"><span data-stu-id="33d94-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="33d94-136">Každá z kanceláří mojí organizace si vyžádá data o zaměstnancích, firmách a obchodních přístavech a vyžaduje, aby se na základě místních předpisů a časového pásma vycházely</span><span class="sxs-lookup"><span data-stu-id="33d94-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="33d94-137">Vysoce škálovatelné požadavky můžou být splněné horizontálním škálováním aplikací, přičemž v jedné oblasti a v různých oblastech se provádí nasazení s více aplikacemi, aby se mohly zvládnout extrémní požadavky na zatížení.</span><span class="sxs-lookup"><span data-stu-id="33d94-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="33d94-138">Aplikace musí být vysoce dostupné a reagovat na požadavky klientů i v případě výpadků v jedné oblasti.</span><span class="sxs-lookup"><span data-stu-id="33d94-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="33d94-139">Další kroky</span><span class="sxs-lookup"><span data-stu-id="33d94-139">Next steps</span></span>

<span data-ttu-id="33d94-140">Další informace o tématech zavedených v tomto článku:</span><span class="sxs-lookup"><span data-stu-id="33d94-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="33d94-141">Další informace o tom, jak tento nástroj pro vyrovnávání zatížení využívající službu DNS funguje, najdete v tématu [Přehled Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) .</span><span class="sxs-lookup"><span data-stu-id="33d94-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="33d94-142">Další informace o osvědčených postupech a získání odpovědí na případné další otázky najdete v tématu [aspekty návrhu hybridní aplikace](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="33d94-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="33d94-143">Další informace o celém portfoliu produktů a řešení najdete v [Azure Stack rodině produktů a řešení](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="33d94-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="33d94-144">Až budete připraveni otestovat příklad řešení, pokračujte pomocí [Průvodce nasazením geograficky distribuovaných aplikací](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="33d94-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="33d94-145">Průvodce nasazením poskytuje podrobné pokyny pro nasazení a testování jeho komponent.</span><span class="sxs-lookup"><span data-stu-id="33d94-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="33d94-146">Naučíte se, jak směrovat provoz do konkrétních koncových bodů na základě různých metrik pomocí vzoru geograficky distribuované aplikace.</span><span class="sxs-lookup"><span data-stu-id="33d94-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="33d94-147">Když vytvoříte profil Traffic Manager s využitím geografického směrování a konfigurace koncového bodu, zajistíte směrování informací na koncové body na základě regionálních požadavků, podnikových a mezinárodních předpisů a vašich datových potřeb.</span><span class="sxs-lookup"><span data-stu-id="33d94-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
