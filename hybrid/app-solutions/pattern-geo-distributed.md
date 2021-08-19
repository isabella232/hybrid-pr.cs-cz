---
title: Model geograficky distribuovaných aplikací v Azure Stack Hub
description: Seznamte se s modelem geograficky distribuovaných aplikací pro inteligentní hraniční zařízení s využitím Azure a Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281223"
---
# <a name="geo-distributed-app-pattern"></a>Model geograficky distribuované aplikace

Zjistěte, jak poskytovat koncové body aplikace napříč několika oblastmi a směrovat uživatelský provoz na základě potřeb z oblasti a dodržování předpisů.

## <a name="context-and-problem"></a>Kontext a problém

Organizace se širokým dosahem zeměpisných údajů se snaží bezpečně a přesně distribuovat a umožnit přístup k datům a zároveň zajistit požadovanou úroveň zabezpečení, dodržování předpisů a výkonu pro uživatele, umístění a zařízení přes hranice.

## <a name="solution"></a>Řešení

Model Azure Stack Hub geografického směrování provozu neboli geograficky distribuované aplikace umožňuje směrovat provoz do konkrétních koncových bodů na základě různých metrik. Vytvořením Traffic Manager geografickým směrováním a konfigurací koncových bodů se provoz směruje do koncových bodů na základě regionálních požadavků, firemních a mezinárodních předpisů a požadavků na data.

![Geograficky distribuovaný vzor](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Komponenty

### <a name="outside-the-cloud"></a>Mimo cloud

#### <a name="traffic-manager"></a>Traffic Manager

V diagramu se Traffic Manager nachází mimo veřejný cloud, ale musí být schopný koordinovat provoz v místním datacentru i ve veřejném cloudu. Balancer směruje provoz do geografických umístění.

#### <a name="domain-name-system-dns"></a>DNS (Domain Name System)

Dns (Domain Name System) zodpovídá za překlad (nebo překlad) názvu webu nebo služby na jeho IP adresu.

### <a name="public-cloud"></a>Veřejný cloud

#### <a name="cloud-endpoint"></a>Koncový bod cloudu

Veřejné IP adresy slouží ke směrování příchozího provozu přes Traffic Manager do koncového bodu prostředků aplikace veřejného cloudu.  

### <a name="local-clouds"></a>Místní cloudy

#### <a name="local-endpoint"></a>Místní koncový bod

Veřejné IP adresy slouží ke směrování příchozího provozu přes Traffic Manager do koncového bodu prostředků aplikace veřejného cloudu.

## <a name="issues-and-considerations"></a>Problémy a důležité informace

Když se budete rozhodovat, jak tento model implementovat, měli byste vzít v úvahu následující skutečnosti:

### <a name="scalability"></a>Škálovatelnost

Tento model zpracovává geografické směrování provozu, nikoli škálování, aby se splňovalo zvýšení provozu. Tento model ale můžete kombinovat s jinými řešeními Azure a místními řešeními. Tento model můžete například použít se vzorem škálování mezi cloudy.

### <a name="availability"></a>Dostupnost

Zajistěte, aby byly místně nasazené aplikace nakonfigurované pro vysokou dostupnost prostřednictvím místní konfigurace hardwaru a nasazení softwaru.

### <a name="manageability"></a>Možnosti správy

Tento model zajišťuje bezproblémovou správu a známé rozhraní mezi prostředími.

## <a name="when-to-use-this-pattern"></a>Kdy se má tento model použít

- Moje organizace má mezinárodní pobočky, které vyžadují vlastní místní zásady zabezpečení a distribuce.
- Každá firemní pobočka si vyžádá data zaměstnanců, podniků a zařízení, což vyžaduje aktivitu generování sestav podle místních předpisů a časového pásma.
- Požadavky ve velkém měřítku je možné splnit horizontálním horizontálním navýšením kapacity aplikací, kdy se v rámci jedné oblasti a napříč oblastmi provádí několik nasazení aplikací, aby bylo možné zvládat extrémní požadavky na zatížení.
- Aplikace musí být vysoce dostupné a musí reagovat na požadavky klientů i v případě výpadků v jedné oblasti.

## <a name="next-steps"></a>Další kroky

Další informace o tématech uvedených v tomto článku:

- Další informace [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) tomto nástroji pro vyrovnávání zatížení provozu na základě DNS najdete v přehledu konfigurace.
- Další [informace o osvědčených](overview-app-design-considerations.md) postupech a získání odpovědí na jakékoli další otázky najdete v tématu Aspekty návrhu hybridních aplikací.
- Další [informace Azure Stack o celém portfoliu produktů a](/azure-stack) řešení najdete v článku o celé Azure Stack produktech a řešeních.

Až budete připraveni otestovat příklad řešení, pokračujte v průvodci [nasazením geograficky distribuovaného řešení aplikací.](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed) Průvodce nasazením obsahuje podrobné pokyny pro nasazení a testování komponent. Naučíte se směrovat provoz do konkrétních koncových bodů na základě různých metrik pomocí vzoru geograficky distribuovaných aplikací. Vytvořením Traffic Manager profilu s geografickým směrováním a konfigurací koncových bodů zajistíte směrování informací do koncových bodů na základě místních požadavků, firemní a mezinárodní regulace a potřeb vašich dat.