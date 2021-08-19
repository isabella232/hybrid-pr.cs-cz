---
title: Model detekce stop s využitím Azure a Azure Stack Hub
description: Naučte se používat Azure a Azure Stack Hub k implementaci řešení detekce stop založených na AI pro analýzu provozu v maloobchodě.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281274"
---
# <a name="footfall-detection-pattern"></a>Model detekce stop

Tento model poskytuje přehled pro implementaci řešení detekce stop založených na AI pro analýzu provozu návštěvníků v maloobchodních prodejnách. Řešení generuje přehledy z reálných akcí pomocí Azure, Azure Stack Hub a sady AI Dev Kit Custom Vision AI.

## <a name="context-and-problem"></a>Kontext a problém

Contoso Stores by chtěla získat přehled o tom, jak zákazníci získávají své aktuální produkty ve vztahu k rozložení obchodu. Nemohou umístit pracovníky do každé části a je neefektivní, aby tým analytiků zhodnotěl záběry z celého obchodu. Kromě toho žádná z jejich obchodů nemá dostatek šířky pásma pro streamování videa ze všech fotoaparátů do cloudu pro analýzu.

Společnost Contoso by chtěla najít nerušiný způsob, který je pro ochranu osobních údajů přívětivý k určení demografických údajů, loajalality a reakcí svých zákazníků při prodeji displejů a produktů.

## <a name="solution"></a>Řešení

Tento model analýzy maloobchodního prodeje používá vrstvený přístup k odvozování na hraničních zařízeních. Pomocí sady Custom Vision AI Dev Kit se k analýze do privátního soukromého Azure Stack Hub, na které běží Azure Cognitive Services. Anonymizovaná agregovaná data se odesílaují do Azure k agregaci ve všech obchodech a vizualizacích v Power BI. Kombinace hraničního a veřejného cloudu umožňuje společnosti Contoso využívat výhod moderní technologie AI a současně zajistit soulad se svými firemními zásadami a respektovat ochranu osobních údajů zákazníků.

[![Řešení vzorů detekce stop](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Tady je souhrn toho, jak řešení funguje:

1. Sada Custom Vision AI Dev Kit získá konfiguraci z IoT Hub, která nainstaluje IoT Edge Runtime a ML model.
2. Pokud model uvidí osobu, posouvá obrázek a nahraje ho do Azure Stack Hub objektů blob.
3. Služba blob aktivuje funkci Azure na Azure Stack Hub.
4. Funkce Azure volá kontejner s rozhraním API pro rozpoznávání tváře, aby z obrázku získat demografické údaje a data o emocích.
5. Data jsou anonymizovaná a odesílaná do clusteru Azure Event Hubs clusteru.
6. Cluster Event Hubs data do služby Stream Analytics.
7. Stream Analytics agreguje data a předá je do Power BI.

## <a name="components"></a>Komponenty

Toto řešení používá následující komponenty:

| Vrstva | Komponenta | Popis |
|----------|-----------|-------------|
| Hardware v obchodě | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Poskytuje filtrování v obchodě pomocí místního modelu ML, který zachycuje pouze obrázky osob pro analýzu. Bezpečně zřízené a aktualizované prostřednictvím IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs poskytuje škálovatelnou platformu pro ingestování anonymizovaných dat, která se hladce integruje s Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Úloha Azure Stream Analytics agreguje anonymizovaná data a seskupí je do 15sekudových oken pro vizualizaci. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI poskytuje snadno použitelný řídicí panel pro zobrazení výstupu z Azure Stream Analytics. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | Poskytovatel App Service prostředků (RP) poskytuje základ pro hraniční komponenty, včetně hostování a správy funkcí pro webové aplikace, rozhraní API a funkce. |
| | Azure Kubernetes Service modulu služby Azure Kubernetes Service [(AKS)](https://github.com/Azure/aks-engine) | AKS RP s clusterem AKS-Engine nasazeným do Azure Stack Hub poskytuje škálovatelný a odolný modul pro spuštění kontejneru rozhraní API pro rozpoznávání tváře. |
| | Azure Cognitive Services [rozhraní API pro rozpoznávání tváře](/azure/cognitive-services/face/face-how-to-install-containers)| The Azure Cognitive Services RP with Face API containers provides demographic, emotion, and unique visitor detection on Contoso's private network. |
| | Blob Storage | Obrázky zachycené ze sady AI Dev Kit se Azure Stack Hub úložiště objektů blob. |
| | Azure Functions | Funkce Azure spuštěná v Azure Stack Hub přijímá vstupy z úložiště objektů blob a spravuje interakce s rozhraním API pro rozpoznávání tváře. Generuje anonymizovaná data do clusteru Event Hubs umístěném v Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problémy a důležité informace

Při rozhodování o tom, jak toto řešení implementovat, zvažte následující body:

### <a name="scalability"></a>Škálovatelnost

Pokud chcete toto řešení škálovat napříč několika fotoaparáty a umístěními, musíte zajistit, aby všechny komponenty zvládly zvýšené zatížení. Možná budete muset provádět tyto akce:

- Zvyšte počet jednotek Stream Analytics streamování.
- Horizontální navýšení velikosti nasazení rozhraní API pro rozpoznávání tváře
- Zvyšte propustnost Event Hubs clusteru.
- V extrémních případech může být potřeba Azure Functions na virtuální počítač.

### <a name="availability"></a>Dostupnost

Vzhledem k tomu, že je toto řešení vrstvené, je důležité se zamyslet nad tím, jak řešit selhání sítě nebo napájení. V závislosti na obchodních potřebách můžete chtít implementovat mechanismus pro místní ukládání imagí do mezipaměti a po návratu Azure Stack Hub předávat dál. Pokud je umístění dostatečně velké, nasazení Data Box Edge s kontejnerem rozhraní API pro rozpoznávání tváře do tohoto umístění může být lepší volbou.

### <a name="manageability"></a>Možnosti správy

Toto řešení může zahrnovat mnoho zařízení a umístění, což může být nepřesné. [Služby Azure IoT](/azure/iot-fundamentals/) je možné použít k automatickému přenesení nových umístění a zařízení do online režimu a k jejich aktualizace.

### <a name="security"></a>Zabezpečení

Toto řešení zachycuje image zákazníků, takže zabezpečení je prvořadé. Ujistěte se, že jsou všechny účty úložiště zabezpečené správnými zásadami přístupu, a pravidelně obměny klíčů. Zajistěte, aby účty úložiště a Event Hubs měly zásady uchovávání informací, které splňují firemní a vládní předpisy na ochranu osobních údajů. Nezapomeňte také vrstvovat úrovně přístupu uživatelů. Vrstvení zajišťuje, že uživatelé mají přístup jenom k datům, která potřebují pro svou roli.

## <a name="next-steps"></a>Další kroky

Další informace o tématech uvedených v tomto článku:

- Podívejte se [na model vrstvené datové](https://aka.ms/tiereddatadeploy)úrovně, který využívá model detekce srážek.
- Další informace o Custom Vision custom vision najdete na Custom Vision [AI Dev Kit.](https://azure.github.io/Vision-AI-DevKit-Pages/) 

Až budete připraveni otestovat příklad řešení, pokračujte v průvodci nasazením detekce stop [.](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection) Průvodce nasazením obsahuje podrobné pokyny pro nasazení a testování komponent.