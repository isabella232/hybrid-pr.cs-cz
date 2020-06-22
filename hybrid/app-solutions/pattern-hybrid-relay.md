---
title: Vzor hybridního přenosu v Azure a centra Azure Stack
description: Pomocí vzoru hybridního přenosu v Azure a centra Azure Stack se připojte k hraničním prostředkům chráněným branami firewall.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910128"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="8afea-103">Vzor hybridního přenosu</span><span class="sxs-lookup"><span data-stu-id="8afea-103">Hybrid relay pattern</span></span>

<span data-ttu-id="8afea-104">Naučte se, jak se připojit k hraničním prostředkům nebo zařízením chráněným branami firewall pomocí modelu hybridního přenosu a Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="8afea-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="8afea-105">Kontext a problém</span><span class="sxs-lookup"><span data-stu-id="8afea-105">Context and problem</span></span>

<span data-ttu-id="8afea-106">Hraniční zařízení jsou často za podnikovou bránou firewall nebo zařízením NAT.</span><span class="sxs-lookup"><span data-stu-id="8afea-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="8afea-107">I když jsou zabezpečené, nemusí být schopni komunikovat s veřejným cloudem nebo hraničními zařízeními v jiných podnikových sítích.</span><span class="sxs-lookup"><span data-stu-id="8afea-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="8afea-108">Může být nutné zajistit zabezpečený způsob vystavení určitých portů a funkcí uživatelům ve veřejném cloudu.</span><span class="sxs-lookup"><span data-stu-id="8afea-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="8afea-109">Řešení</span><span class="sxs-lookup"><span data-stu-id="8afea-109">Solution</span></span>

<span data-ttu-id="8afea-110">Vzor hybridního přenosu používá Azure Relay k navázání tunelu WebSockets mezi dvěma koncovými body, které nemůžou přímo komunikovat.</span><span class="sxs-lookup"><span data-stu-id="8afea-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="8afea-111">Zařízení, která nejsou místní, ale potřebují se připojit k místnímu koncovému bodu, se připojí ke koncovému bodu ve veřejném cloudu.</span><span class="sxs-lookup"><span data-stu-id="8afea-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="8afea-112">Tento koncový bod přesměruje provoz v předdefinovaných trasách přes zabezpečený kanál.</span><span class="sxs-lookup"><span data-stu-id="8afea-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="8afea-113">Koncový bod v místním prostředí obdrží provoz a směruje ho do správného cíle.</span><span class="sxs-lookup"><span data-stu-id="8afea-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![Architektura řešení vzorů hybridního přenosu](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="8afea-115">Tady je postup, jak funguje vzor hybridního přenosu:</span><span class="sxs-lookup"><span data-stu-id="8afea-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="8afea-116">Zařízení se připojí k virtuálnímu počítači (VM) v Azure na předdefinovaném portu.</span><span class="sxs-lookup"><span data-stu-id="8afea-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="8afea-117">Provoz se přepošle Azure Relay v Azure.</span><span class="sxs-lookup"><span data-stu-id="8afea-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="8afea-118">Virtuální počítač na rozbočovači Azure Stack, který už navázal dlouhodobé připojení k Azure Relay, přijme provoz a předá ho k cíli.</span><span class="sxs-lookup"><span data-stu-id="8afea-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="8afea-119">Místní služba nebo koncový bod zpracovává požadavek.</span><span class="sxs-lookup"><span data-stu-id="8afea-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="8afea-120">Komponenty</span><span class="sxs-lookup"><span data-stu-id="8afea-120">Components</span></span>

<span data-ttu-id="8afea-121">Toto řešení používá následující komponenty:</span><span class="sxs-lookup"><span data-stu-id="8afea-121">This solution uses the following components:</span></span>

| <span data-ttu-id="8afea-122">Vrstva</span><span class="sxs-lookup"><span data-stu-id="8afea-122">Layer</span></span> | <span data-ttu-id="8afea-123">Součást</span><span class="sxs-lookup"><span data-stu-id="8afea-123">Component</span></span> | <span data-ttu-id="8afea-124">Description</span><span class="sxs-lookup"><span data-stu-id="8afea-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="8afea-125">Azure</span><span class="sxs-lookup"><span data-stu-id="8afea-125">Azure</span></span> | <span data-ttu-id="8afea-126">Virtuální počítač Azure</span><span class="sxs-lookup"><span data-stu-id="8afea-126">Azure VM</span></span> | <span data-ttu-id="8afea-127">Virtuální počítač Azure poskytuje veřejně přístupný koncový bod pro místní prostředek.</span><span class="sxs-lookup"><span data-stu-id="8afea-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="8afea-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="8afea-128">Azure Relay</span></span> | <span data-ttu-id="8afea-129">[Azure Relay](/azure/azure-relay/) poskytuje infrastrukturu pro udržování tunelového propojení a připojení mezi virtuálním počítačem Azure a virtuálním počítačem centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="8afea-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="8afea-130">Centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="8afea-130">Azure Stack Hub</span></span> | <span data-ttu-id="8afea-131">Compute</span><span class="sxs-lookup"><span data-stu-id="8afea-131">Compute</span></span> | <span data-ttu-id="8afea-132">Virtuální počítač centra Azure Stack poskytuje na straně serveru tunelové propojení hybridního přenosu.</span><span class="sxs-lookup"><span data-stu-id="8afea-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="8afea-133">Storage</span><span class="sxs-lookup"><span data-stu-id="8afea-133">Storage</span></span> | <span data-ttu-id="8afea-134">Cluster modulu AKS nasazený do centra Azure Stack poskytuje škálovatelný a odolný modul pro spuštění kontejneru Face API.</span><span class="sxs-lookup"><span data-stu-id="8afea-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="8afea-135">Problémy a důležité informace</span><span class="sxs-lookup"><span data-stu-id="8afea-135">Issues and considerations</span></span>

<span data-ttu-id="8afea-136">Při rozhodování, jak implementovat toto řešení, vezměte v úvahu následující body:</span><span class="sxs-lookup"><span data-stu-id="8afea-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="8afea-137">Škálovatelnost</span><span class="sxs-lookup"><span data-stu-id="8afea-137">Scalability</span></span>

<span data-ttu-id="8afea-138">Tento model umožňuje pouze mapování portů 1:1 na straně klienta a serveru.</span><span class="sxs-lookup"><span data-stu-id="8afea-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="8afea-139">Pokud je například port 80 tunelování pro jednu službu na koncovém bodu Azure, nelze ji použít pro jinou službu.</span><span class="sxs-lookup"><span data-stu-id="8afea-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="8afea-140">Mapování portů by mělo být naplánováno odpovídajícím způsobem.</span><span class="sxs-lookup"><span data-stu-id="8afea-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="8afea-141">Azure Relay a virtuální počítače by měly být vhodně škálované na zpracování provozu.</span><span class="sxs-lookup"><span data-stu-id="8afea-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="8afea-142">Dostupnost</span><span class="sxs-lookup"><span data-stu-id="8afea-142">Availability</span></span>

<span data-ttu-id="8afea-143">Tato tunelová propojení a připojení nejsou redundantní.</span><span class="sxs-lookup"><span data-stu-id="8afea-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="8afea-144">Abyste zajistili vysokou dostupnost, možná budete chtít implementovat kód kontroly chyb.</span><span class="sxs-lookup"><span data-stu-id="8afea-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="8afea-145">Další možností je vytvořit fond virtuálních počítačů připojených k Azure Relay za nástroj pro vyrovnávání zatížení.</span><span class="sxs-lookup"><span data-stu-id="8afea-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="8afea-146">Možnosti správy</span><span class="sxs-lookup"><span data-stu-id="8afea-146">Manageability</span></span>

<span data-ttu-id="8afea-147">Toto řešení může zahrnovat mnoho zařízení a umístění, což by mohlo mít nepraktický.</span><span class="sxs-lookup"><span data-stu-id="8afea-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="8afea-148">Služby IoT v Azure můžou automaticky přenést nová umístění a zařízení do režimu online a udržovat je v aktuálním stavu.</span><span class="sxs-lookup"><span data-stu-id="8afea-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="8afea-149">Zabezpečení</span><span class="sxs-lookup"><span data-stu-id="8afea-149">Security</span></span>

<span data-ttu-id="8afea-150">Tento model, jak ukazuje, umožňuje neomezený přístup k portu na interním zařízení z Edge.</span><span class="sxs-lookup"><span data-stu-id="8afea-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="8afea-151">Zvažte přidání mechanismu ověřování ke službě na interním zařízení nebo před koncový bod hybridního přenosu.</span><span class="sxs-lookup"><span data-stu-id="8afea-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="8afea-152">Další kroky</span><span class="sxs-lookup"><span data-stu-id="8afea-152">Next steps</span></span>

<span data-ttu-id="8afea-153">Další informace o tématech zavedených v tomto článku:</span><span class="sxs-lookup"><span data-stu-id="8afea-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="8afea-154">Tento model používá Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="8afea-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="8afea-155">Další informace najdete v dokumentaci k [Azure Relay](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="8afea-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="8afea-156">Další informace o osvědčených postupech najdete v tématu [aspekty návrhu hybridní aplikace](overview-app-design-considerations.md) a Získejte odpovědi na další otázky.</span><span class="sxs-lookup"><span data-stu-id="8afea-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="8afea-157">Další informace o celém portfoliu produktů a řešení najdete v [Azure Stack rodině produktů a řešení](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="8afea-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="8afea-158">Až budete připraveni otestovat příklad řešení, pokračujte v [Průvodci nasazením řešení hybridního přenosu](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="8afea-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="8afea-159">Průvodce nasazením poskytuje podrobné pokyny pro nasazení a testování jeho komponent.</span><span class="sxs-lookup"><span data-stu-id="8afea-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>