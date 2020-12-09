---
title: Kubernetes vzor vysoké dostupnosti s využitím Azure a centra Azure Stack
description: Naučte se, jak řešení clusteru Kubernetes poskytuje vysokou dostupnost pomocí Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 454cc0a0531882b7a8ec050a461420ce13eebcfe
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911790"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Model clusteru Kubernetes s vysokou dostupností

Tento článek popisuje, jak architektovat a provozovat vysoce dostupnou infrastrukturu založenou na Kubernetes pomocí modulu Azure Kubernetes Service (AKS) na rozbočovači Azure Stack. Tento scénář je společný pro organizace s důležitými úlohami ve vysoce omezených a regulovaných prostředích. Organizace v doménách, jako jsou finance, obrana a státní správa.

## <a name="context-and-problem"></a>Kontext a problém

Mnoho organizací vyvíjí nativní řešení cloudu, které využívá špičkové služby a technologie, jako je Kubernetes. I když Azure poskytuje datacentra ve většině oblastí světa, někdy existují oblasti použití hraničního prostředí a scénáře, kdy je potřeba, aby podnikové kritické aplikace běžely v určitém umístění. K důležitým hlediskům patří:

- Citlivost na umístění
- Latence mezi aplikací a místními systémy
- Zachování šířky pásma
- Připojení
- Zákonné nebo zákonné požadavky

Azure, v kombinaci s Azure Stack hub, řeší většinu těchto otázek. V následující části je popsána široká sada možností, rozhodnutí a důležitých informací pro úspěšnou implementaci Kubernetes běžící v centru Azure Stack.

## <a name="solution"></a>Řešení

Tento vzor předpokládá, že musíme řešit striktní sadu omezení. Aplikace musí běžet místně a veškerá osobní data musí mít přístup k veřejným cloudovým službám. Monitorování a další data, která nejsou PII, se dají do Azure Odeslat a zpracovat tam. Externím službám, jako jsou veřejné Container Registry nebo jiné, se dá přistupovat, ale můžou se filtrovat přes bránu firewall nebo proxy server.

Ukázková aplikace uvedená tady (na základě [služby Azure Kubernetes Service Workshop](/learn/modules/aks-workshop/)) je navržená tak, aby používala řešení nativní pro Kubernetes, kdykoli to bude možné. Tento návrh zabraňuje zamčení dodavatele namísto použití služeb nativního pro platformu. Jako příklad používá aplikace jako hostitele back-end MongoDB Database namísto služby PaaS nebo externí databázové služby.

[![Hybridní vzor aplikace](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Předchozí diagram znázorňuje architekturu aplikace ukázkové aplikace spuštěné v Kubernetes na rozbočovači Azure Stack. Aplikace se skládá z několika součástí, včetně:

 1) Cluster AKS založený na stroji Kubernetes na rozbočovači Azure Stack.
 2) [správce certifikace](https://www.jetstack.io/cert-manager/), který poskytuje sadu nástrojů pro správu certifikátů v Kubernetes, který slouží k automatickému vyžádání certifikátů pomocí šifrování.
 3) Obor názvů Kubernetes, který obsahuje komponenty aplikace pro front-end (hodnocení – Web), rozhraní API (hodnocení – API) a databázi (hodnocení-MongoDB).
 4) Kontroler příchozích dat, který směruje provoz HTTP/HTTPS do koncových bodů v rámci clusteru Kubernetes.

Ukázková aplikace se používá k ilustraci architektury aplikace. Všechny komponenty jsou příklady. Architektura obsahuje jenom jedno nasazení aplikace. Abychom dosáhli vysoké dostupnosti (HA), spustíme nasazení minimálně dvakrát na dvou různých instancích centra Azure Stack – můžou běžet buď ve stejném umístění, nebo ve dvou (nebo více) různých lokalitách:

![Architektura infrastruktury](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Služby, jako jsou Azure Container Registry, Azure Monitor a další, se hostují mimo Azure Stack centrum v Azure nebo místně. Tento hybridní návrh chrání řešení před výpadkem jedné instance centra Azure Stack.

## <a name="components"></a>Komponenty

Celková architektura se skládá z následujících součástí:

**Centrum Azure Stack** je rozšířením Azure, které může spouštět úlohy v místním prostředí tím, že poskytuje služby Azure ve vašem datovém centru. Další informace najdete v článku [Přehled centra Azure Stack](/azure-stack/operator/azure-stack-overview) .

Modul **AKS (Azure Kubernetes Service Engine)** je modul, na kterém se nachází služba Azure Kubernetes Service (AKS), která je v Azure ještě dnes dostupná. V případě centra Azure Stack umožňuje modul AKS nasadit, škálovat a upgradovat plně vybavené a samoobslužné clustery Kubernetes s využitím IaaS možností centra Azure Stack. Další informace najdete v článku [Přehled modulu AKS](https://github.com/Azure/aks-engine) .

Pokud se chcete dozvědět víc o rozdílech mezi modulem AKS v Azure a modulem AKS v centru Azure Stack, přečtěte si o [známých problémech a omezeních](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) .

Služba **Azure Virtual Network (VNET)** slouží k poskytování síťové infrastruktury v každém centru Azure Stack pro Virtual Machines (virtuální počítače) hostující infrastrukturu clusteru Kubernetes.

**Azure Load Balancer** se používá pro koncový bod rozhraní API Kubernetes a kontroler Nginx pro příchozí přenosy. Nástroj pro vyrovnávání zatížení směruje externí provoz (například Internet) do uzlů a virtuálních počítačů, které nabízejí konkrétní službu.

**Azure Container Registry (ACR)** se používá k ukládání privátních imagí Docker a grafů Helm, které se nasazují do clusteru. Modul AKS se může ověřit s Container Registry pomocí identity Azure AD. Kubernetes nevyžaduje ACR. Můžete použít jiné Registry kontejnerů, jako je například Docker Hub.

**Azure Repos** je sada nástrojů pro správu verzí, které lze použít ke správě kódu. Můžete také použít GitHub nebo jiná úložiště založená na Gitu. Další informace najdete v článku [přehled Azure Repos](/azure/devops/repos/get-started/what-is-repos) .

**Azure Pipelines** je součástí Azure DevOps Services a spouští automatizované sestavení, testy a nasazení. Můžete také použít řešení CI/CD třetí strany, jako je například Jenkinse. Další informace najdete v článku [Přehled kanálu Azure](/azure/devops/pipelines/get-started/what-is-azure-pipelines) .

**Azure monitor** shromažďuje a ukládá metriky a protokoly, včetně metrik platforem pro služby Azure v telemetrie řešení a aplikace. Tato data slouží k monitorování aplikace, nastavení výstrah a řídicích panelů a k analýze selhání hlavní příčiny. Azure Monitor se integruje s Kubernetes ke shromažďování metrik z řadičů, uzlů a kontejnerů a také protokolů kontejnerů a protokolů hlavního uzlu. Další informace najdete v článku [přehled Azure monitor](/azure/azure-monitor/overview) .

**Azure Traffic Manager** je nástroj pro vyrovnávání zatížení založený na DNS, který umožňuje distribuci provozu optimálně do služeb napříč různými oblastmi Azure nebo nasazeními centra Azure Stack. Traffic Manager také nabízí vysokou dostupnost a rychlost odezvy. Koncové body aplikace musí být přístupné z vnějšku. K dispozici jsou také další místní řešení.

**Kontroler** příchozího přenosu dat (Kubernetes) zpřístupňuje trasy HTTP (S) službám v clusteru Kubernetes. Pro tento účel se dá použít Nginx nebo jakýkoli vhodný kontroler příchozího přenosu dat.

**Helm** je správce balíčků pro nasazení Kubernetes, který poskytuje způsob, jak seskupit různé objekty Kubernetes jako nasazení, služby, tajné kódy do jediného "grafu". Můžete publikovat, nasazovat, řídit správu verzí a aktualizovat objekt grafu. Azure Container Registry lze použít jako úložiště pro ukládání zabalených Helm grafů.

## <a name="design-considerations"></a>Na co dát pozor při navrhování

Tento model následuje několik nejdůležitějších doporučení, která jsou podrobněji vysvětlena v dalších částech tohoto článku:

- Aplikace používá Kubernetes řešení, aby se předešlo zamčení dodavatele.
- Aplikace používá architekturu mikroslužeb.
- Rozbočovač Azure Stack nepotřebuje příchozí, ale umožňuje odchozí připojení k Internetu.

Tyto doporučené postupy se budou vztahovat i na reálné úlohy a scénáře.

## <a name="scalability-considerations"></a>Aspekty zabezpečení

Škálovatelnost je důležité zajistit, aby uživatelé měli konzistentní, spolehlivý a dobře fungující přístup k aplikaci.

Vzorový scénář pokrývá škálovatelnost na více vrstvách aplikačního zásobníku. Tady je podrobný přehled různých vrstev:

| Úroveň architektury | Ovlivňuje | Jak? |
| --- | --- | ---
| Aplikace | Aplikace | Horizontální škálování na základě počtu lusků/replik/Container Instances * |
| Cluster | Cluster Kubernetes | Počet uzlů (mezi 1 a 50), virtuální počítače-SKU-velikosti a fondy uzlů (modul AKS na rozbočovači Azure Stack aktuálně podporuje pouze jeden fond uzlů); použití příkazu Scale AKS Engine (ruční) |
| Infrastruktura | Azure Stack Hub | Počet uzlů, kapacity a jednotek škálování v rámci nasazení Azure Stackového centra |

\* Používá se Kubernetes ' Horizontal pod autoscaleer (HPA); Automatizovaná škála založená na metrikách nebo vertikální škálování podle velikosti kontejnerových instancí (CPU/paměť).

**Centrum Azure Stack (úroveň infrastruktury)**

Infrastruktura centra Azure Stack je základem této implementace, protože centrum Azure Stack běží na fyzickém hardwaru v datacentru. Při výběru hardwaru vašeho rozbočovače je třeba provést volby pro CPU, hustotu paměti, konfiguraci úložiště a počet serverů. Další informace o škálovatelnosti centra Azure Stack najdete v následujících materiálech:

- [Přehled plánování kapacity pro Azure Stack centra](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Přidání dalších uzlů jednotek škálování do centra Azure Stack](/azure-stack/operator/azure-stack-add-scale-node)

**Cluster Kubernetes (úroveň clusteru)**

Samotný cluster Kubernetes se skládá z a je postaven na IaaS komponentách Azure (Stack), včetně výpočetních, úložných a síťových prostředků. Řešení Kubernetes zahrnují hlavní a pracovní uzly, které se nasazují jako virtuální počítače v Azure (a centrum Azure Stack).

- [Řídicí uzly ovládacích prvků](/azure/aks/concepts-clusters-workloads#control-plane) (hlavní) poskytují základní služby Kubernetes a orchestraci úloh aplikací.
- [Pracovní uzly](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (Worker) spouštějí úlohy vaší aplikace.

Při výběru velikostí virtuálních počítačů pro počáteční nasazení je k dispozici několik důležitých informací:  

- **Náklady** – při plánování pracovních uzlů mějte na paměti celkové náklady na virtuální počítač, který se vám bude účtovat. Pokud například úlohy vaší aplikace vyžadují omezené prostředky, měli byste naplánovat nasazení virtuálních počítačů s menší velikostí. Centrum Azure Stack, jako je Azure, se obvykle účtuje na základě spotřeby, takže vhodně navýšení velikosti virtuálních počítačů pro role Kubernetes je klíčové pro optimalizaci nákladů na spotřebu. 

- **Škálovatelnost clusteru** se dosahuje tím, že se škáluje a rozchází na počet hlavních a pracovních uzlů nebo přidáním dalších fondů uzlů (není k dispozici v Azure Stack hub ještě dnes). Škálování clusteru se dá udělat na základě dat o výkonu, shromážděných pomocí kontejneru Insights (Azure Monitor + Log Analytics). 

    Pokud vaše aplikace potřebuje více (nebo méně) prostředků, můžete horizontální navýšení kapacity (nebo v nich) horizontálně škálovat (mezi 1 a 50 uzly). Pokud potřebujete více než 50 uzlů, můžete vytvořit další cluster v samostatném předplatném. Nemůžete vertikálně škálovat skutečné virtuální počítače na jinou velikost virtuálního počítače bez opětovného nasazení clusteru.

    Škálování se provádí ručně pomocí AKS pomocníka pro modul, který se původně použil k nasazení clusteru Kubernetes. Další informace najdete v tématu [škálování clusterů Kubernetes](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md) .

- **Kvóty** – uvažujte o [kvótách](/azure-stack/operator/azure-stack-quota-types) , které jste nakonfigurovali při plánování nasazení AKS na svém rozbočovači Azure Stack. Ujistěte se, že má každé [předplatné](/azure-stack/operator/service-plan-offer-subscription-overview) správné plány a nakonfigurované kvóty. Předplatné bude muset při horizontálním navýšení kapacity přizpůsobit množství výpočetních prostředků, úložiště a dalších služeb potřebných pro vaše clustery.

- **Aplikační úlohy** – [Koncepty clusterů a úloh](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) najdete v tématu Koncepty Kubernetes Core pro dokument služby Azure Kubernetes. Tento článek vám pomůže určit rozsah správné velikosti virtuálních počítačů na základě výpočetních a paměťových potřeb vaší aplikace.  

**Aplikace (úroveň aplikace)**

V aplikační vrstvě používáme Kubernetes [horizontálně pod autoscaleer (hPa)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). HPA může zvýšit nebo snížit počet replik (pod/Container Instances) v našem nasazení na základě různých metrik (například využití procesoru).

Další možností je škálovat instance kontejnerů vertikálně. To se dá udělat tak, že změníte velikost procesoru a paměti, která je požadovaná a dostupná pro konkrétní nasazení. Další informace najdete v tématu [Správa prostředků pro kontejnery](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) v Kubernetes.IO.

## <a name="networking-and-connectivity-considerations"></a>Požadavky na síť a připojení

Síťové a síťové připojení má vliv i na tři vrstvy zmíněné dříve pro Kubernetes v Azure Stackovém centru. V následující tabulce jsou uvedeny vrstvy a služby, které obsahují:

| Vrstva | Ovlivňuje | Co? |
| --- | --- | ---
| Aplikace | Aplikace | Jak aplikace je přístupná? Bude zpřístupněno pro Internet? |
| Cluster | Cluster Kubernetes | Kubernetes API, virtuální počítač modulu AKS, načítání imagí kontejnerů (odchozí přenos), odesílání dat monitorování a telemetrie (výstup) |
| Infrastruktura | Azure Stack Hub | Přístupnost koncových bodů správy centra Azure Stack jako koncových bodů portálu a Azure Resource Manager |

**Aplikace**

V případě aplikační vrstvy je nejdůležitějším aspektem, zda je aplikace vystavená a přístupná z Internetu. V Kubernetes perspektivě Internet Accessibility znamená vystavení nasazení nebo pod pomocí služby Kubernetes nebo řadiče příchozího přenosu dat.

> [!NOTE]
> K vystavování služeb Kubernetes doporučujeme použití řadičů příchozího přenosu dat, protože počet veřejných IP adres front-endu na rozbočovači Azure Stack je omezený na 5. Tento návrh také omezuje počet služeb Kubernetes (s typem vyrovnávání zatížení) na 5, který bude pro mnoho nasazení příliš malý. Další informace najdete v [dokumentaci k modulu AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) .

Vystavení aplikace pomocí veřejné IP adresy prostřednictvím Load Balancer nebo nessecarily, znamená, že je aplikace teď přístupná prostřednictvím Internetu. Je možné, aby centrum Azure Stack mít veřejnou IP adresu, která je viditelná jenom na místním intranetu – ne všechny veřejné IP adresy jsou skutečně internetové.

Předchozí blok považuje příchozí přenos do aplikace za provozu. Dalším tématem, které je potřeba vzít v úvahu pro úspěšné nasazení Kubernetes, je odchozí a výstupní provoz. Tady je několik případů použití, které vyžadují odchozí přenosy:

- Přijímání imagí kontejneru uložených na Dockerhubu nebo Azure Container Registry
- Načítání Helm grafů
- Emitování Application Insights dat (nebo jiných dat monitorování)

Některá podniková prostředí můžou vyžadovat použití _transparentních_ nebo _netransparentních_ proxy serverů. Tyto servery vyžadují konkrétní konfiguraci různých komponent našeho clusteru. Dokumentace k modulům AKS obsahuje různé informace o tom, jak se kterými se vejdou síťové proxy. Další podrobnosti najdete v tématu [modul AKS a proxy servery](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md) .

A konečně provoz mezi clustery musí být mezi Azure Stack instancemi centra. Ukázkové nasazení se skládá z jednotlivých Kubernetes clusterů běžících na jednotlivých instancích centra Azure Stack. Provoz mezi nimi, jako je například provoz replikace mezi dvěma databázemi, je "externí provoz". Externí provoz musí být směrován prostřednictvím sítě Site-to-Site VPN nebo veřejných IP adres centra Azure Stack pro připojení Kubernetes ve dvou instancích centra Azure Stack:

![přenos mezi clustery mezi a uvnitř](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

Cluster Kubernetes nemusí být nutně přístupný prostřednictvím Internetu. Relevantní část je rozhraní API Kubernetes používané k provozování clusteru, například pomocí nástroje `kubectl` . Koncový bod rozhraní API Kubernetes musí být přístupný všem, kdo provozuje cluster nebo nasazuje aplikace a služby nad ním. V tomto tématu se podrobněji věnujeme DevOps perspektivě v níže uvedené části věnované [nasazení (CI/CD)](#deployment-cicd-considerations) .

Na úrovni clusteru existuje také několik informací o odchozím provozu:

- Aktualizace uzlů (pro Ubuntu)
- Monitorování dat (odeslání do Azure LogAnalytics)
- Další agenti vyžadující odchozí přenosy (specifické pro každé prostředí nasazení)

Před nasazením clusteru Kubernetes pomocí modulu AKS si naplánujte finální návrh sítě. Místo vytvoření vyhrazeného Virtual Network může být efektivnější nasadit cluster do existující sítě. Můžete například využít existující připojení VPN typu Site-to-site, které je už nakonfigurované v prostředí centra Azure Stack.

**Infrastruktura**  

Infrastruktura odkazuje na přístup k koncovým bodům správy centra Azure Stack. K koncovým bodům patří portály tenanta a správců a koncové body správce Azure Resource Manager a tenanta. Tyto koncové body se vyžadují pro provoz Azure Stackho centra a jeho základních služeb.

## <a name="data-and-storage-considerations"></a>Požadavky na data a úložiště

Dvě instance naší aplikace se nasadí na dva jednotlivé clustery Kubernetes ve dvou Azure Stackch instancích centra. Tento návrh bude vyžadovat, abyste měli v úvahu, jak replikovat a synchronizovat data mezi nimi.

S Azure máme integrovanou možnost replikace úložiště napříč několika oblastmi a zónami v rámci cloudu. V současné době s centrem Azure Stack neexistují žádné nativní způsoby, jak replikovat úložiště mezi dvěma různými instancemi centra Azure Stack – tvoří dva nezávislé cloudy s žádným způsobem, jak je spravovat jako sadu. Plánování odolnosti aplikací spuštěných v rámci centra Azure Stack vynutí tuto nezávislost v návrhu a nasazeních aplikací.

Ve většině případů nebude replikace úložiště nutná pro zajištění odolné a vysoce dostupné aplikace nasazené v AKS. V návrhu aplikací byste ale měli zvážit nezávislé úložiště na instanci centra Azure Stack. Pokud se tento návrh týká nasazení řešení v centru Azure Stack, existují možná řešení od partnerů Microsoftu, která poskytují přílohy úložiště. Přílohy úložiště poskytují řešení replikace úložiště v rámci několika Azure Stackch Center a Azure. Další informace najdete v tématu [Partnerská řešení](#partner-solutions).

V naší architektuře byly zváženy tyto vrstvy:

**Konfigurace**

Konfigurace zahrnuje konfiguraci centra Azure Stack, modulu AKS a samotného clusteru Kubernetes. Konfigurace by měla být co nejvíc automatizovaná a uložená jako infrastruktura jako kód v systému správy verzí založeném na Gitu, jako je Azure DevOps nebo GitHub. Tato nastavení se nedají snadno synchronizovat mezi několika nasazeními. Proto doporučujeme uložit a použít konfiguraci z vnějšku a pomocí kanálu DevOps.

**Aplikace**

Aplikace by měla být uložená v úložišti založeném na Gitu. Pokaždé, když dojde k novému nasazení, změny aplikace nebo zotavení po havárii, můžete je snadno nasadit pomocí Azure Pipelines.

**Data**

Data jsou nejdůležitějším aspektem většiny návrhů aplikací. Data aplikací musí zůstat synchronizovaná mezi různými instancemi aplikace. Pokud dojde k výpadku, potřebují data taky strategii zálohování a zotavení po havárii.

Dosahování tohoto návrhu je silně závislé na technologických volbách. Tady je několik příkladů řešení pro implementaci databáze ve vysoce dostupném režimu v centru Azure Stack:

- [Nasazení skupiny dostupnosti SQL Server 2016 do Azure a centra Azure Stack](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Nasazení vysoce dostupného řešení MongoDB do Azure a centra Azure Stack](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Důležité informace týkající se práce s daty napříč více umístěními jsou ještě složitějším aspektem vysoce dostupného a odolného řešení. Rozmyslete si:

- Latence a síťové připojení mezi Azure Stackmi rozbočovači.
- Dostupnost identit pro služby a oprávnění. Každá instance centra Azure Stack se integruje s externím adresářem. Během nasazování se rozhodnete použít buď Azure Active Directory (Azure AD) nebo Active Directory Federation Services (AD FS) (ADFS). V takovém případě je možné použít jedinou identitu, která může pracovat s několika nezávislými instancemi centra Azure Stack.

## <a name="business-continuity-and-disaster-recovery"></a>Provozní kontinuita a zotavení po havárii

Provozní kontinuita a zotavení po havárii (BCDR) je důležité téma v Azure Stackovém centru i v Azure. Hlavní rozdíl je v tom, že v Azure Stackovém centru musí operátor spravovat celý proces BCDR. V Azure jsou části BCDR automaticky spravovány Microsoftem.

BCDR má vliv na tytéž oblasti, které jsou uvedené v předchozích částech [data a úložiště](#data-and-storage-considerations):

- Infrastruktura/konfigurace
- Dostupnost aplikace
- Data aplikací

Jak je uvedeno v předchozí části, tyto oblasti jsou zodpovědností operátora centra Azure Stack a mohou se lišit mezi organizacemi. Plánování BCDR podle vašich dostupných nástrojů a procesů.

**Infrastruktura a konfigurace**

Tato část se zabývá fyzickou a logickou infrastrukturou a konfigurací centra Azure Stack. Zahrnuje akce v prostorách správce a tenanta.

Operátor centra Azure Stack (nebo správce) zodpovídá za údržbu instancí centra Azure Stack. Včetně součástí sítě, úložiště, identity a dalších témat, která jsou mimo rámec tohoto článku. Další informace o konkrétních operacích Azure Stack centra najdete v následujících zdrojích informací:

- [Obnovení dat v Azure Stack centru pomocí služby Infrastructure Backup](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Povolení zálohování centra Azure Stack z portálu pro správu](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Obnovení z katastrofické ztráty dat](/azure-stack/operator/azure-stack-backup-recover-data)
- [Osvědčené postupy pro službu Infrastructure Backup](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack hub je platforma a prostředky infrastruktury, na kterých budou nasazené aplikace Kubernetes. Vlastníkem aplikace pro aplikaci Kubernetes bude uživatel centra Azure Stack s přístupem uděleným k nasazení infrastruktury aplikace potřebné pro řešení. Aplikační infrastruktura v tomto případě znamená cluster Kubernetes nasazený pomocí modulu AKS a okolní služby. Tyto součásti budou nasazeny do centra Azure Stack s omezením nabídky centra Azure Stack. Ujistěte se, že nabídka přijatá vlastníkem aplikace Kubernetes má dostatečnou kapacitu (vyjádřená v kvótách centra Azure Stack) pro nasazení celého řešení. Jak je doporučeno v předchozí části, nasazení aplikace by mělo být automatizované pomocí kanálů pro nasazení infrastruktury jako kódu a nasazení, jako je Azure DevOps Azure Pipelines.

Další informace o nabídkách a kvótách centra Azure Stack najdete v tématu [Přehled služeb, plánů, nabídek a předplatných centra Azure Stack](/azure-stack/operator/service-plan-offer-subscription-overview) .

Je důležité bezpečně uložit a uložit konfiguraci modulu AKS, včetně jeho výstupů. Tyto soubory obsahují důvěrné informace, které se používají pro přístup ke clusteru Kubernetes, takže je potřeba je chránit před zveřejněním bez oprávnění správce.

**Dostupnost aplikace**

Aplikace by neměla spoléhat na zálohy nasazené instance. Jako standardní postup si aplikaci znovu nasaďte úplně podle vzorů infrastruktury jako kódu. Můžete například znovu nasadit pomocí Azure DevOps Azure Pipelines. Postup BCDR by měl zahrnovat opětovné nasazení aplikace do stejného nebo jiného clusteru Kubernetes.

**Data aplikací**

Data aplikací jsou kritickou součástí pro minimalizaci ztráty dat. V předchozí části byly popsány techniky pro replikaci a synchronizaci dat mezi dvěma (nebo více) instancemi aplikace. V závislosti na infrastruktuře databáze (MySQL, MongoDB, MSSQL nebo jiné), která se používá k ukládání dat, budou k dispozici různé techniky dostupnosti a zálohování databáze, ze kterých si můžete vybrat.

K dosažení integrity se doporučuje použít tyto možnosti:
- Nativní řešení zálohování pro určitou databázi.
- Řešení zálohování, které oficiálně podporuje zálohování a obnovení typu databáze používaného vaší aplikací.

> [!IMPORTANT]
> Neukládejte data záloh ve stejné instanci centra Azure Stack, kde se nachází data aplikace. Úplný výpadek instance centra Azure Stack by také ohrozil vaše zálohy.

## <a name="availability-considerations"></a>Aspekty dostupnosti

Kubernetes na rozbočovači Azure Stack nasazeném prostřednictvím modulu AKS není spravovaná služba. Jedná se o automatizované nasazení a konfiguraci clusteru Kubernetes s využitím infrastruktury Azure jako služby (IaaS). V takovém případě poskytuje stejnou dostupnost jako základní infrastruktura.

Infrastruktura centra Azure Stack je už odolná vůči selháním a poskytuje možnosti, jako jsou skupiny dostupnosti k distribuci komponent napříč více [doménami selhání a aktualizačními doménami](/azure-stack/user/azure-stack-vm-considerations#high-availability). Ale základní technologie (Clustering s podporou převzetí služeb při selhání) stále při selhání hardwaru u virtuálních počítačů na ovlivněném fyzickém serveru způsobí nějaké výpadky.

Je dobrým zvykem nasadit cluster produkčního prostředí Kubernetes i zatížení na dva (nebo víc) clustery. Tyto clustery by se měly hostovat v různých umístěních nebo datových centrech a využívat technologie, jako je Azure Traffic Manager, ke směrování uživatelů na základě doby odezvy clusteru nebo na základě geografického prostředí.

![Řízení toků provozu pomocí Traffic Manager](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Zákazníci, kteří mají jeden cluster Kubernetes, se obvykle připojují k IP adrese služby nebo názvu DNS dané aplikace. V nasazení s více clustery by se zákazníci měli připojit k Traffic Manager názvu DNS, který odkazuje na služby a příchozí přenosy na jednotlivých clusterech Kubernetes.

![Použití Traffic Manager ke směrování do místního clusteru](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Tento model je také [osvědčeným postupem pro (spravované) clustery AKS v Azure](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

Samotný cluster Kubernetes, který je nasazený prostřednictvím modulu AKS, by se měl skládat z aspoň tří hlavních uzlů a dvou pracovních uzlů.

## <a name="identity-and-security-considerations"></a>Požadavky na identitu a zabezpečení

Informace o identitách a zabezpečení jsou důležitá témata. Hlavně v případě, že řešení zahrnuje nezávislé Azure Stack instance centra. Kubernetes a Azure (včetně centra Azure Stack) mají pro řízení přístupu na základě role (RBAC) odlišné mechanismy:

- Azure RBAC řídí přístup k prostředkům v Azure (a centra Azure Stack), včetně možnosti vytvářet nové prostředky Azure. Oprávnění lze přiřadit uživatelům, skupinám nebo instančním objektům. (Instanční objekt je identita zabezpečení používaná aplikacemi.)
- Kubernetes RBAC řídí oprávnění k rozhraní Kubernetes API. Například vytvoření lusků a výpisů lusků jsou akce, které mohou být autorizovány (nebo odepřeny) uživateli prostřednictvím RBAC. K přiřazení oprávnění Kubernetes uživatelům můžete vytvářet role a vazby rolí.

**Azure Stack identitu centra a RBAC**

Centrum Azure Stack poskytuje dvě volby zprostředkovatele identity. Použitý poskytovatel závisí na prostředí a na tom, jestli je spuštěný v připojeném nebo odpojeném prostředí:

- Azure AD – dá se použít jenom v připojeném prostředí.
- ADFS na tradiční doménovou strukturu služby Active Directory – dá se použít v připojeném i odpojeném prostředí.

Zprostředkovatel identity spravuje uživatele a skupiny, včetně ověřování a autorizace pro přístup k prostředkům. Přístup se dá udělit Azure Stack prostředkům centra, jako jsou předplatná, skupiny prostředků a jednotlivé prostředky, jako jsou virtuální počítače nebo nástroje pro vyrovnávání zatížení. Chcete-li mít konzistentní model přístupu, měli byste zvážit použití stejných skupin (buď přímo, nebo vnořené) pro všechna centra Azure Stack. Tady je příklad konfigurace:

![vnořené skupiny AAD se službou Azure Stack hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Příklad obsahuje vyhrazenou skupinu (pomocí AAD nebo ADFS) pro konkrétní účel. Například pokud chcete poskytnout oprávnění přispěvatele pro skupinu prostředků, která obsahuje naši infrastrukturu clusteru Kubernetes na konkrétní instanci centra Azure Stack (tady je K8s Přispěvatel clusteru). Tyto skupiny jsou potom vnořené do celkové skupiny, která obsahuje "podskupiny" pro každé centrum Azure Stack.

Náš ukázkový uživatel teď bude mít oprávnění Přispěvatel pro obě skupiny prostředků, které obsahují celou sadu prostředků infrastruktury Kubernetes. Uživatel bude mít přístup k prostředkům v obou instancích centra Azure Stack, protože instance sdílejí stejného poskytovatele identity.

> [!IMPORTANT]
> Tato oprávnění se týkají pouze centra Azure Stack a některých prostředků nasazených nad ním. Uživatel, který má tuto úroveň přístupu, může provádět spoustu škod, ale nemůže získat přístup k virtuálním počítačům Kubernetes IaaS ani k rozhraní Kubernetes API bez dalšího přístupu k nasazení Kubernetes.

**Kubernetes identity a RBAC**

Cluster Kubernetes ve výchozím nastavení nepoužívá stejného poskytovatele identity jako Azure Stack centrum. Virtuální počítače hostující cluster Kubernetes, hlavní uzel a pracovní uzly používají klíč SSH, který je určen během nasazování clusteru. Tento klíč SSH se musí připojit k těmto uzlům pomocí SSH.

Rozhraní Kubernetes API (například k používání pomocí `kubectl` ) je chráněno účty služeb, včetně výchozího účtu služby Správce clusteru. Přihlašovací údaje pro tento účet služby se zpočátku ukládají do `.kube/config` souboru na hlavních Kubernetes uzlech.

**Správa tajných klíčů a přihlašovací údaje aplikací**

K ukládání tajných klíčů jako připojovacích řetězců nebo přihlašovacích údajů databáze je k dispozici několik možností, včetně:

- Azure Key Vault
- Tajné klíče Kubernetes
- řešení třetích stran, jako je HashiCorp trezor (běží na Kubernetes)

Neukládejte tajné klíče ani přihlašovací údaje ve formátu prostého textu v konfiguračních souborech, kódu aplikace nebo ve skriptech. A neukládat je do systému správy verzí. Místo toho by měla automatizace nasazení získávat tajné klíče podle potřeby.

## <a name="patch-and-update"></a>Oprava a aktualizace

Proces **opravy a aktualizace (PNU)** ve službě Azure Kubernetes je částečně automatizovaný. Upgrady verze Kubernetes se spouštějí ručně, zatímco se aktualizace zabezpečení aplikují automaticky. Tyto aktualizace můžou zahrnovat opravy zabezpečení operačního systému nebo aktualizace jádra. AKS neprovede automatické restartování těchto uzlů Linux k dokončení procesu aktualizace. 

Proces PNU pro cluster Kubernetes nasazený pomocí modulu AKS v centru Azure Stack není spravován a zodpovídá za operátora clusteru. 

AKS Engine pomáhá s těmito dvěma nejdůležitějšími úkoly:

- [Upgrade na novější verzi image operačního systému Kubernetes a Base OS](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Upgrade jenom základní image operačního systému](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Novější image základního operačního systému obsahují nejnovější opravy zabezpečení operačního systému a aktualizace jádra. 

Mechanismus [bezobslužného upgradu](https://wiki.debian.org/UnattendedUpgrades) automaticky nainstaluje aktualizace zabezpečení vydané před tím, než bude k dispozici nová verze základní image operačního systému v tržišti Azure Stack hub. Bezobslužný upgrade je ve výchozím nastavení povolený a instaluje aktualizace zabezpečení automaticky, ale nerestartuje uzly clusteru Kubernetes. Restartování uzlů se dá automatizovat pomocí Open-source [ **k** Ubernetes **opětovného** spuštění **D** aemon (kured))](/azure/aks/node-updates-kured). Kured sleduje uzly Linux, které vyžadují restart, a pak automaticky zpracuje přeplánování běžícího procesu lusků a restartování uzlu.

## <a name="deployment-cicd-considerations"></a>Požadavky na nasazení (CI/CD)

Centrum Azure a Azure Stack zveřejňují stejná Azure Resource Manager rozhraní REST API. Tato rozhraní API jsou řešena jako jakýkoli jiný cloud Azure (Azure, Azure Čína 21Vianet, Azure Government). Mezi cloudy můžou být rozdíly ve verzích rozhraní API a služba Azure Stack hub poskytuje jenom podmnožinu služeb. Identifikátor URI koncového bodu správy se také liší pro každý Cloud a každou instanci centra Azure Stack.

Kromě uvedených drobných rozdílů nabízí Azure Resource Manager rozhraní REST API konzistentní způsob, jak pracovat s centrem Azure i Azure Stack. Tady se dá použít stejná sada nástrojů, jak by se používala s jakýmkoli jiným cloudem Azure. K nasazení a orchestraci služeb pro Azure Stack Hub můžete použít Azure DevOps, nástroje jako Jenkinse nebo PowerShell.

**Důležité informace**

Jedním z hlavních rozdílů v případě, že je Azure Stack nasazení centra, je otázka dostupnosti Internetu. Přístupnost internetu určuje, jestli se má pro úlohy CI/CD vybrat agent sestavení hostovaná Microsoftem nebo místním hostem.

Samoobslužný Agent může běžet na Azure Stackovém centru (jako virtuální počítač IaaS) nebo v podsíti sítě, která má přístup k centru Azure Stack. Další informace o rozdílech najdete v části [agenti Azure Pipelines](/azure/devops/pipelines/agents/agents) .

Následující obrázek vám pomůže rozhodnout se, jestli potřebujete agenta sestavení hostovaného v místním prostředí nebo Microsoft hosta:

![Agenti sestavení v místním prostředí – Ano nebo ne](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Mají přístup k koncovým bodům správy centra Azure Stack přes Internet?
  - Ano: k připojení k centru Azure Stack můžete použít Azure Pipelines s agenty hostovanými Microsoftem.
  - Ne: potřebujeme samoobslužné agenty, kteří se můžou připojit k koncovým bodům správy centra Azure Stack.
- Je náš cluster Kubernetes přístupný prostřednictvím Internetu?
  - Ano: k přímé interakci s koncovým bodem rozhraní Kubernetes API můžeme použít Azure Pipelines s agenty hostovanými Microsoftem.
  - Ne: potřebujeme samoobslužné agenty, kteří se můžou připojit ke koncovému bodu rozhraní API clusteru Kubernetes.

Ve scénářích, kdy jsou koncové body správy centra Azure Stack a rozhraní Kubernetes API přístupné prostřednictvím Internetu, může nasazení použít agenta hostovaného Microsoftem. Toto nasazení bude mít za následek architekturu aplikace následujícím způsobem:

[![Přehled veřejné architektury](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Pokud Azure Resource Manager koncovým bodům, rozhraní Kubernetes API nebo obojím nemáte přímo přístup přes Internet, můžeme pro spuštění kroků kanálu využít samoobslužného agenta sestavení. Tento návrh potřebuje méně připojení a dá se nasadit jenom s místními připojeními k síti do koncových bodů Azure Resource Manager a rozhraní Kubernetes API:

[![Přehled architektury on-Prem](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Jaké jsou informace o odpojených scénářích?** Ve scénářích, kdy Azure Stack hub nebo Kubernetes nebo oba nemají koncové body správy, je stále možné využít Azure DevOps pro vaše nasazení. Můžete buď použít samoobslužný fond agentů (což je DevOps agent, který je spuštěný místně, nebo Azure Stack v samotném rozbočovači), nebo zcela místně hostovaný Azure DevOps Server. Samoobslužný agent potřebuje jenom odchozí připojení HTTPS (TCP/443) k Internetu.

Vzor může používat cluster Kubernetes (nasazený a orchestrující modul AKS) v každé instanci centra Azure Stack. Zahrnuje aplikaci, která se skládá z front-endové služby back-endu (například MongoDB) a řadiče příchozího přenosu založeného na Nginx. Místo používání databáze hostované v clusteru K8s můžete využít "externí úložiště dat". Mezi možnosti databáze patří MySQL, SQL Server nebo jakýkoli druh databáze hostovaný mimo centrum Azure Stack nebo v IaaS. Zde uvedené konfigurace nejsou v oboru.

## <a name="partner-solutions"></a>Partnerská řešení

Existují Partnerská řešení Microsoftu, která můžou roztáhnout možnosti centra Azure Stack. Tato řešení byla užitečná při nasazení aplikací spuštěných v clusterech Kubernetes.  

## <a name="storage-and-data-solutions"></a>Řešení úložiště a dat

Jak je popsáno v tématu [požadavky na data a úložiště](#data-and-storage-considerations), Azure Stack hub aktuálně nemá nativní řešení pro replikaci úložiště mezi několika instancemi. Na rozdíl od Azure neexistuje schopnost replikace úložiště napříč několika oblastmi. V Azure Stackovém centru je každá instance svým vlastním jedinečným cloudem. Ale řešení jsou dostupná od partnerů Microsoftu, která umožňují replikaci úložiště napříč Azure Stackmi rozbočovači a Azure. 

**ŠKÁLOVATELNOST**

[Škálovatelnost](https://www.scality.com/) přináší úložiště na úrovni webu, které používá digitální firmy od 2009. KANÁL pro škálovatelnost, který má naše softwarově definované úložiště, převede komoditní servery na neomezený fond úložiště pro libovolný typ dat – soubor a objekt – v řádu petabajtů měřítku.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) zjednodušuje podnikové úložiště s neomezeným škálovatelným úložištěm, které konsoliduje obrovských datových sad do jediného snadno spravovaného prostředí.

## <a name="next-steps"></a>Další kroky

Další informace o konceptech představených v tomto článku:

- [Škálování mezi cloudy](pattern-cross-cloud-scale.md) a [geograficky distribuované aplikace](pattern-geo-distributed.md) v centru Azure Stack.
- [Architektura mikroslužeb ve službě Azure Kubernetes (AKS)](/azure/architecture/reference-architectures/microservices/aks).

Až budete připraveni otestovat příklad řešení, pokračujte pomocí [Průvodce nasazením clusteru s vysokou dostupností Kubernetes](solution-deployment-guide-highly-available-kubernetes.md). Průvodce nasazením poskytuje podrobné pokyny pro nasazení a testování jeho komponent.