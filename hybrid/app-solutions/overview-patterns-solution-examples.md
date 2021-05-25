---
title: Příklady hybridních vzorů a řešení pro Azure a Azure Stack Hub
description: Přehled hybridních vzorů a příkladů řešení pro výuku a vytváření hybridních řešení v Azure a Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343854"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Příklady a vzory hybridních řešení pro Azure a Azure Stack

Microsoft poskytuje Azure a Azure Stack a řešení jako jeden konzistentní ekosystém Azure. Rodina Microsoft Azure Stack je rozšířením Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Hybridní cloud a hybridní aplikace

Azure Stack hybridního cloudu cloud computing a hraničních zařízení flexibilitu místních prostředí a *hraničních zařízení.* Azure Stack Hub, Azure Stack HCI a Azure Stack Edge rozšiřují Azure z cloudu do suverénních datacenter, poboček, terénu i mimo ni. Díky této různorodé sadě funkcí můžete:

- Opakovaně používat kód a spouštět aplikace nativní pro cloud konzistentně napříč Azure a místními prostředími.
- Spouštění tradičních virtualizovaných úloh s volitelnými připojeními ke službám Azure
- Přenášet data do cloudu nebo je uchovat v suverénním datacentru, aby se zachovala kompatibilita.
- Spouštění hardwarově akcelerovaných úloh strojového učení, kontejnerizovaných nebo virtualizovaných úloh na inteligentních hraničních zařízeních

Aplikace, které pokrývají cloudy, se také označují jako *hybridní aplikace*. V Azure můžete vytvářet hybridní cloudové aplikace a nasazovat je do připojeného nebo odpojeného datacentra, které se nachází kdekoli.

Scénáře hybridních aplikací se výrazně liší podle prostředků, které jsou k dispozici pro vývoj. Zahrnují také aspekty, jako je zeměpisná oblast, zabezpečení, přístup k internetu a další. I když zde popsané vzory a příklady řešení nemusí řešit všechny požadavky, poskytují pokyny a příklady pro zkoumání a opakované použití při implementaci hybridních řešení.

## <a name="solution-patterns"></a>Vzory řešení

Vzory řešení zobecněné opakovatelné pokyny k návrhu z reálných zákaznických scénářů a prostředí. Vzor je abstraktní a umožňuje, aby byl použitelný pro různé typy scénářů nebo vertikální obory. Každý vzor dokumentuje kontext a problém a poskytuje přehled o příkladech řešení. Příklad řešení je určen jako možná implementace vzoru.

Existují dva typy článků o vzorcích:

- Jeden vzor: poskytuje pokyny k návrhu pro jediný scénář pro obecné účely.
- Multi-Pattern: poskytuje pokyny pro návrh, kde se používá aplikace s více vzorci. Tento model se často vyžaduje pro řešení složitějších scénářů nebo problémů specifických pro konkrétní odvětví.

## <a name="solution-deployment-guides"></a>Průvodci nasazením řešení

Podrobný průvodce nasazením vám pomůže při nasazení příkladu řešení. Průvodce může také odkazovat na ukázku kódu, který je uložen v [úložišti ukázek řešení](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)GitHub.

## <a name="next-steps"></a>Další kroky

- Další informace o celém portfoliu produktů a řešení najdete v [Azure Stack rodině produktů a řešení](/azure-stack) .
- Další informace najdete v částech "vzory" a "Průvodci nasazením řešení" v obsahu.
- Přečtěte si o [otázkách návrhu hybridní aplikace](overview-app-design-considerations.md) a prozkoumejte pilíře kvality softwaru pro navrhování, nasazování a provozování hybridních aplikací.
- [Nastavte vývojové prostředí na Azure Stack](/azure-stack/user/azure-stack-dev-start) a [nasaďte svoji první aplikaci](/azure-stack/user/azure-stack-dev-start-deploy-app) na Azure Stack.
