---
title: Neuložené zjišťování pomocí Azure a Azure Stack Edge
description: Naučte se používat Azure a Azure Stack Edge Services k implementaci detekce z provozu.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343871"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Zjišťování z hranice hraničního modelu z neaktivního zásobníku

Tento model znázorňuje, jak zjistit, jestli jsou položky police mimo skladové položky pomocí Azure Stack hraničních zařízení nebo Azure IoT Edgech síťových fotoaparátů.

## <a name="context-and-problem"></a>Kontext a problém

Fyzické maloobchodní obchody ztratí prodej, protože když zákazníci hledají položku, neexistují na police. Položka však může být na zadní straně úložiště a nebyla přeuložená. Úložiště by chtěla používat své zaměstnance efektivněji a automaticky dostávat oznámení, když je potřeba přezásobovat položky.

## <a name="solution"></a>Řešení

Příklad řešení používá hraniční zařízení, například Azure Stack Edge v každém obchodě, což efektivně zpracovává data z fotoaparátů v úložišti. Tento optimalizovaný návrh umožňuje úložištím odesílat do cloudu pouze relevantní události a obrázky. Návrh šetří šířku pásma, prostor úložiště a zajišťuje ochranu osobních údajů zákazníků. Když jsou snímky čteny z jednotlivých kamer, model ML zpracuje obrázek a vrátí z oblasti akcií. Obrázek a mimo oblast zásob se zobrazí v místní webové aplikaci. Tato data se dají odeslat do prostředí Time Series Insight, kde se zobrazí přehledy o Power BI.

![Neuložený v architektuře hraničních řešení](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Jak řešení funguje:

1. Image se zaznamenávají ze síťové kamery přes protokol HTTP nebo RTSP.
2. Velikost bitové kopie se změní a pošle se do ovladače odvození, který komunikuje s modelem ML a určí, jestli jsou dostupné žádné z uložených imagí.
3. Model ML vrátí jakékoli z burzovních oblastí.
4. Ovladač Inferencing nahraje nezpracovaný obrázek do objektu BLOB (Pokud je zadaný) a pošle výsledky z modelu do Azure IoT Hub a s procesorem ohraničujícího pole na zařízení.
5. Procesor ohraničujících uzlů přidává do obrázku ohraničovací rámečky a cestu k imagi ukládá do mezipaměti v paměti databáze.
6. Webová aplikace se dotazuje na obrázky a zobrazuje je v přijatém pořadí.
7. Zprávy z IoT Hub se agregují v Time Series Insights.
8. Power BI se v průběhu času zobrazí interaktivní sestava s daty z Time Series Insights.


## <a name="components"></a>Komponenty

Toto řešení používá následující komponenty:

| Vrstva | Komponenta | Popis |
|----------|-----------|-------------|
| Místní hardware | Síťová kamera | K odvozování obrázků se vyžaduje síťová kamera s kanálem HTTP nebo RTSP. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) o zřizování zařízení a zasílání zpráv pro hraniční zařízení. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) uloží zprávy z IoT Hub pro vizualizaci. |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) poskytuje obchodní sestavy o neskladně obchodních událostech. Power BI poskytuje snadno použitelný řídicí panel pro zobrazení výstupu z Azure Stream Analytics. |
| Azure Stack Edge nebo<br>Azure IoT Edge zařízení | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) orchestruje modul runtime pro místní kontejnery a zpracovává správu a aktualizace zařízení.|
| | Projekt Azure brainwave | Na zařízení Azure Stack Edge používá [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) pole FPGA (Field-Programmable Gate Array) k urychlení odvozování ML.|

## <a name="issues-and-considerations"></a>Problémy a důležité informace

Při rozhodování o tom, jak toto řešení implementovat, zvažte následující body:

### <a name="scalability"></a>Škálovatelnost

Většina modelů strojového učení může běžet pouze s určitým počtem snímků za sekundu v závislosti na poskytnutém hardwaru. Určete optimální vzorkovací frekvenci z fotoaparátů, abyste zajistili, že se kanál ML nebude zálohovat. Různé typy hardwaru budou zpracovávat různé počty fotoaparátů a snímek.

### <a name="availability"></a>Dostupnost

Je důležité zvážit, co se může stát, když hraniční zařízení ztratí připojení. Zvažte, jaká data mohou být ztracena z Time Series Insights a Power BI řídicím panelu. Ukázkové řešení, které je uvedené, není navržené tak, aby bylo vysoce dostupné.

### <a name="manageability"></a>Možnosti správy

Toto řešení může zahrnovat mnoho zařízení a umístění, což by mohlo mít nepraktický. Služby IoT v Azure můžou automaticky přenést nová umístění a zařízení do režimu online a udržovat je v aktuálním stavu. Musí následovat i správné postupy správného řízení dat.

### <a name="security"></a>Zabezpečení

Tento model zpracovává potenciálně citlivá data. Ujistěte se, že jsou klíče pravidelně otočené a že jsou správně nastavená oprávnění pro účet Azure Storage a místní sdílené složky.

## <a name="next-steps"></a>Další kroky

Další informace o tématech zavedených v tomto článku:
- V tomto vzoru se používá několik služeb souvisejících s IoT, včetně [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)a [Azure Time Series Insights](/azure/time-series-insights/).
- Další informace o Microsoft Project Brainwave najdete [v oznámení na blogu](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) a s využitím [Projectu Brainwave video k rezervaci Azure akcelerovaných Machine Learning](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Další informace o osvědčených postupech a odpovědi na další otázky najdete v tématu [aspekty návrhu hybridní aplikace](overview-app-design-considerations.md) .
- Další informace o celém portfoliu produktů a řešení najdete v [Azure Stack rodině produktů a řešení](/azure-stack) .

Až budete připraveni otestovat příklad řešení, pokračujte v [Průvodci nasazením řešení Edge ml Inferencing](https://aka.ms/edgeinferencingdeploy). Průvodce nasazením poskytuje podrobné pokyny pro nasazení a testování jeho komponent.
