---
title: Nasazení clusteru Kubernetes s vysokou dostupností na rozbočovači Azure Stack
description: Naučte se, jak nasadit řešení clusteru Kubernetes pro zajištění vysoké dostupnosti pomocí Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911734"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Nasazení clusteru Kubernetes s vysokou dostupností v centru Azure Stack

V tomto článku se dozvíte, jak vytvořit vysoce dostupné prostředí clusteru Kubernetes nasazené na více instancích centra Azure Stack v různých fyzických umístěních.

V tomto průvodci nasazením řešení se dozvíte, jak:

> [!div class="checklist"]
> - Stažení a příprava modulu AKS
> - Připojení k virtuálnímu počítači pomocníka AKS Engine
> - Nasazení clusteru Kubernetes
> - Připojení ke clusteru Kubernetes
> - Připojení Azure Pipelines ke clusteru Kubernetes
> - Konfigurace sledování
> - Nasazení aplikace
> - Automatické škálování aplikace
> - Konfigurace služby Traffic Manager
> - Upgrade Kubernetes
> - Škálování Kubernetes

> [!Tip]  
> ![Hybridní pilíře](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Centrum Microsoft Azure Stack je rozšířením Azure. Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.  
> 
> Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací. Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.

## <a name="prerequisites"></a>Předpoklady

Než začnete s tímto průvodcem nasazením, nezapomeňte:

- Projděte si článek o [vzoru clusteru Kubernetes s vysokou dostupností](pattern-highly-available-kubernetes.md) .
- Zkontrolujte obsah [doprovodného úložiště GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), které obsahuje další prostředky, na které se odkazuje v tomto článku.
- Mít účet, který má přístup k [portálu pro uživatele centra Azure Stack](/azure-stack/user/azure-stack-use-portal), s alespoň [oprávněními Přispěvatel](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>Stažení a příprava AKS Engine

Modul AKS je binární soubor, který se dá použít v jakémkoli hostiteli se systémem Windows nebo Linux, který se může dostat k koncovým bodům Azure Resource Manager centra Azure Stack. Tato příručka popisuje nasazení nového virtuálního počítače se systémem Linux (nebo Windows) v centru Azure Stack. Později se použije v případě, že AKS modul nasadí clustery Kubernetes.

> [!NOTE]
> Můžete také použít stávající virtuální počítač se systémem Windows nebo Linux k nasazení clusteru Kubernetes v Azure Stackovém centru pomocí modulu AKS.

Podrobný proces a požadavky na modul AKS jsou popsané tady:

* [Instalace modulu AKS v systému Linux do centra Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (nebo pomocí [systému Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

Modul AKS je pomocný nástroj pro nasazení a provozování (nespravovaných) Kubernetes clusterů (v Azure a centru Azure Stack).

Podrobnosti a rozdíly modulu AKS na rozbočovači Azure Stack jsou popsány zde:

* [Co je modul AKS v centru Azure Stack?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [AKS Engine v centru Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (na GitHubu)

Ukázkové prostředí použije Terraformu k automatizaci nasazení virtuálního počítače modulu AKS. [Podrobnosti a kód můžete najít v doprovodném úložišti GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

Výsledkem tohoto kroku je nová skupina prostředků v Azure Stack hub, která obsahuje pomocný virtuální počítač pro modul AKS a související prostředky:

![Prostředky virtuálních počítačů AKS Engine v centru Azure Stack](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Pokud potřebujete nasadit modul AKS v odpojeném prostředí gapped, přečtěte si téma [odpojené instance centra Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) , kde se dozvíte víc.

V dalším kroku použijeme nově nasazený virtuální počítač AKS Engine k nasazení clusteru Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Připojení k virtuálnímu počítači pomocníka AKS Engine

Nejprve se musíte připojit k dříve vytvořenému virtuálnímu počítači pomocníka AKS Engine.

Virtuální počítač by měl mít veřejnou IP adresu a měl by být přístupný přes SSH (port 22/TCP).

![Stránka s přehledem virtuálního počítače AKS Engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> K připojení k virtuálnímu počítači se systémem Linux pomocí SSH můžete použít libovolný nástroj, třeba MobaXterm, výstup nebo PowerShell ve Windows 10.

```console
ssh <username>@<ipaddress>
```

Po připojení spusťte příkaz `aks-engine` . Další informace o modulu AKS a verzích Kubernetes najdete v části [podporované verze modulu AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) .

![příklad příkazového řádku AKS-Engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Nasazení clusteru Kubernetes

Samotný pomocný virtuální počítač modulu AKS ještě nevytvořil cluster Kubernetes v našem centru Azure Stack. Vytvoření clusteru je první akce, která se má provést na virtuálním počítači pomocníka s modulem AKS.

Podrobný postup je popsán zde:

* [Nasazení clusteru Kubernetes s modulem AKS v centru Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Konečný výsledek `aks-engine deploy` příkazu a přípravy v předchozích krocích je plně funkční cluster Kubernetes nasazený do prostoru klienta první instance centra Azure Stack. Samotný cluster se skládá z komponent Azure IaaS, jako jsou virtuální počítače, nástroje pro vyrovnávání zatížení, virtuální sítě, disky a tak dále.

![Komponenty cluster IaaS Azure Stack portál centra](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Nástroj pro vyrovnávání zatížení Azure (koncový bod rozhraní K8s API)
2) Pracovní uzly (fond agentů)
3) Hlavní uzly

Cluster je teď v provozu a v dalším kroku se k němu připojíme.

## <a name="connect-to-the-kubernetes-cluster"></a>Připojení ke clusteru Kubernetes

Nyní se můžete připojit k dříve vytvořenému clusteru Kubernetes, a to buď prostřednictvím protokolu SSH (pomocí klíče SSH zadaného jako součást nasazení), nebo prostřednictvím `kubectl` (doporučeno). Nástroj příkazového řádku Kubernetes `kubectl` je k dispozici pro Windows, Linux a MacOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Je už předem nainstalovaný a nakonfigurovaný na hlavních uzlech tohoto clusteru.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Spustit kubectl na hlavním uzlu](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Nedoporučujeme používat hlavní uzel jako JumpBox pro úlohy správy. `kubectl`Konfigurace je uložena v nástroji `.kube/config` na hlavních uzlech a také na virtuálním počítači modulu AKS. Konfiguraci můžete zkopírovat do počítače správce s připojením k Kubernetes clusteru a použít `kubectl` příkaz. `.kube/config`Soubor se používá také později ke konfiguraci připojení služby v Azure Pipelines.

> [!IMPORTANT]
> Tyto soubory udržujte v bezpečí, protože obsahují přihlašovací údaje pro váš cluster Kubernetes. Útočník s přístupem k souboru má dostatek informací, aby k němu mohl získat přístup správce. Všechny akce, které se provádějí pomocí počátečního `.kube/config` souboru, se provádějí pomocí účtu správce clusteru.

Nyní můžete vyzkoušet různé příkazy pomocí nástroje `kubectl` a zkontrolovat stav clusteru.

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> Kubernetes má vlastní model *založený na rolích Access Control (RBAC)**, který umožňuje vytvářet podrobné definice rolí a vazby rolí. Toto je vhodnější způsob, jak řídit přístup ke clusteru místo ručního nastavení oprávnění správce clusteru.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Připojení Azure Pipelines k clusterům Kubernetes

K připojení Azure Pipelines k nově nasazenému clusteru Kubernetes potřebujeme svůj soubor Kube config ( `.kube/config` ), jak je vysvětleno v předchozím kroku.

* Připojte se k jednomu z hlavních uzlů clusteru Kubernetes.
* Zkopírujte obsah `.kube/config` souboru.
* Přejít na Azure DevOps > nastavení projektu > připojení služby a vytvořit nové připojení služby Kubernetes (použijte KubeConfig jako metodu ověřování)

> [!IMPORTANT]
> Azure Pipelines (nebo jeho agenti sestavení) musí mít přístup k rozhraní Kubernetes API. Pokud je k dispozici připojení k Internetu z Azure Pipelines do centra Azure Stack Kubernetes clusetr, bude nutné nasadit agenta sestavení Azure Pipelines v místním prostředí.

Při nasazování agentů pro samoobslužné hostování pro Azure Pipelines můžete nasadit buď v centru Azure Stack, nebo na počítači, který má síťové připojení ke všem požadovaným koncovým bodům správy. Podrobnosti najdete tady:

* [Agenti Azure Pipelines](/azure/devops/pipelines/agents/agents) v [systému Windows](/azure/devops/pipelines/agents/v2-windows) nebo [Linux](/azure/devops/pipelines/agents/v2-linux)

Část s [informacemi pro nasazení vzoru (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) obsahuje rozhodovací tok, který vám pomůže pochopit, jestli se mají používat agenti hostovaná Microsoftem nebo agenti s místním hostováním:

[![agenti s místním hostováním rozhodování](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

V tomto ukázkovém řešení zahrnuje topologie agent sestavení v místním prostředí v každé instanci centra Azure Stack. Agent má přístup ke koncovým bodům správy centra Azure Stack a koncovým bodům rozhraní API clusteru Kubernetes.

[![pouze odchozí přenosy](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Tento návrh splňuje společný regulativní požadavek, který má pouze odchozí připojení z řešení aplikace.

## <a name="configure-monitoring"></a>Konfigurace sledování

Můžete použít [Azure monitor](/azure/azure-monitor/) pro kontejnery k monitorování kontejnerů v řešení. Tyto body Azure Monitor clusteru Kubernetes nasazeného pro modul AKS v centru Azure Stack.

Existují dva způsoby, jak povolit Azure Monitor v clusteru. Oba způsoby vyžadují, abyste si nastavili pracovní prostor Log Analytics v Azure.

* [Metoda One](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) používá Helm graf.
* [Metoda 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) jako součást specifikace clusteru AKS Engine

V ukázkové topologii je použita "metoda One", která umožňuje snazší instalaci procesu a aktualizací.

Pro další krok potřebujete pracovní prostor Azure LogAnalytics (ID a klíč), `Helm` (verze 3) a `kubectl` na vašem počítači.

Helm je Kubernetes správce balíčků, který je dostupný jako binární soubor, který je spuštěný v macOS, Windows a Linux. Můžete si ho stáhnout tady: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm spoléhá na konfigurační soubor Kubernetes, který se používá pro `kubectl` příkaz.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Tento příkaz nainstaluje agenta Azure Monitor do clusteru Kubernetes:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Agent Operations Management Suite (OMS) v clusteru Kubernetes odešle data monitorování do vašeho pracovního prostoru Azure Log Analytics (pomocí odchozího protokolu HTTPS). Teď můžete pomocí Azure Monitor získat hlubší přehled o clusterech Kubernetes v centru Azure Stack. Tento návrh představuje účinný způsob, jak předvést výkon analýz, které se dají automaticky nasadit do clusterů vaší aplikace.

[![Clustery centra Azure Stack ve službě Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Podrobnosti o Azure Monitor clusteru](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Pokud Azure Monitor nezobrazuje žádná data centra Azure Stack, ujistěte se, že jste postupovali podle pokynů k pečlivému [přidání AzureMonitor-Containers řešení do pracovního prostoru Azure Loganalytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) .

## <a name="deploy-the-application"></a>Nasazení aplikace

Před instalací naší ukázkové aplikace je k dispozici další krok konfigurace nginx řadiče pro příchozí přenos dat v našem clusteru Kubernetes. Kontroler příchozího přenosu dat se používá jako nástroj pro vyrovnávání zatížení vrstvy 7 ke směrování provozu v našem clusteru založeném na hostiteli, cestě nebo protokolu. Nginx – příchozí je k dispozici jako graf Helm. Podrobné pokyny najdete v [úložišti GitHub Helm Chart](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

Naše ukázková aplikace je také zabalena jako Helm graf, jako je například [Agent Azure Monitoring](#configure-monitoring) v předchozím kroku. V takovém případě je jednoduché nasadit aplikaci do našeho clusteru Kubernetes. [Soubory grafu Helm najdete v doprovodném úložišti GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) .

Ukázková aplikace je tři aplikace vrstev nasazená do clusteru Kubernetes v každé ze dvou instancí centra Azure Stack. Aplikace používá databázi MongoDB. Další informace o tom, jak získat data replikovaná napříč několika instancemi, najdete v tématu popisujícím data vzorů [a úložiště](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

Po nasazení grafu Helm pro aplikaci uvidíte všechny tři úrovně vaší aplikace, které jsou reprezentovány jako nasazení a stavové sady (pro databázi), jedním pod:

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

Na straně služby najdete Nginx kontroler příchozího přenosu dat a jeho veřejnou IP adresu:

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

Adresa externí IP adresy je naším koncovým bodem aplikace. Je to způsob, jakým se uživatelé připojí k otevření aplikace a použijí se taky jako koncový bod pro náš další krok [konfigurace Traffic Manager](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Automatické škálování aplikace
Volitelně můžete nakonfigurovat automatické [škálování vodorovně pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) , abyste mohli škálovat nahoru nebo dolů na základě určitých metrik, jako je využití procesoru. Následující příkaz vytvoří horizontální horizontální navýšení, které udržuje 1 až 10 replik v Luskech kontrolovaných pomocí nasazení webu hodnocení. HPA zvýší a sníží počet replik (přes nasazení), aby se zachovalo průměrné využití CPU ve všech Luskech 80%.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Aktuální stav automatického škálování můžete kontrolovat spuštěním:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Konfigurace služby Traffic Manager

K distribuci provozu mezi dvěma (nebo více) nasazeními aplikace použijeme [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview). Azure Traffic Manager je nástroj pro vyrovnávání zatížení založený na DNS v Azure.

> [!NOTE]
> Traffic Manager používá DNS k přímému směrování požadavků klientů na nejvhodnější koncový bod služby na základě metody směrování provozu a stavu koncových bodů.

Místo používání Azure Traffic Manager můžete také použít jiná globální řešení vyrovnávání zatížení hostovaná místně. V ukázkovém scénáři použijeme Azure Traffic Manager k distribuci provozu mezi dvěma instancemi naší aplikace. Můžou běžet na instancích centra Azure Stack ve stejném nebo různých umístěních:

![místní Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

V Azure nakonfigurujeme Traffic Manager tak, aby odkazovaly na dvě různé instance naší aplikace:

[![Profil koncového bodu TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Jak vidíte, dva koncové body ukazují na dvě instance nasazené aplikace z [předchozí části](#deploy-the-application).

V tomto okamžiku:
- Byla vytvořena infrastruktura Kubernetes, včetně kontroleru příchozího přenosu dat.
- Clustery byly nasazeny v rámci dvou Azure Stackch instancí centra.
- Monitorování bylo nakonfigurováno.
- Azure Traffic Manager vyrovnává zatížení mezi dvěma instancemi centra Azure Stack.
- V horní části této infrastruktury se ukázková aplikace nasadila automatizovaným způsobem pomocí Helm grafů. 

Řešení by teď mělo být dostupné pro uživatele.

K dispozici jsou také některé provozní předpoklady po nasazení, které jsou zahrnuty v následujících dvou částech.

## <a name="upgrade-kubernetes"></a>Upgrade Kubernetes

Při upgradu clusteru Kubernetes Vezměte v úvahu následující témata:

- Upgrade clusteru Kubernetes je složitá operace dne 2, kterou je možné provést pomocí stroje AKS. Další informace najdete v tématu [upgrade clusteru Kubernetes na rozbočovači Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Modul AKS vám umožňuje upgradovat clustery na novější verze Kubernetes a základní image operačního systému. Další informace najdete v tématu [Postup upgradu na novější verzi Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Můžete také upgradovat pouze uzly v části snáška na novější verze imagí základní operační systémy. Další informace najdete v tématu [Postup upgradu image operačního systému](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Novější image základního operačního systému obsahují aktualizace zabezpečení a jádra. Je zodpovědností operátora clusteru sledovat dostupnost novějších verzí Kubernetes a imagí operačních systémů. Operátor by měl naplánovat a spustit tyto upgrady pomocí AKS Engine. Základní image operačního systému se musí stáhnout z webu centra Azure Stack pomocí operátoru centra Azure Stack.

## <a name="scale-kubernetes"></a>Škálování Kubernetes

Škálování je další operace dne 2, kterou je možné orchestrovat pomocí AKS Engine.

Příkaz Scale znovu použije váš konfigurační soubor clusteru (apimodel.json) ve výstupním adresáři jako vstup pro nové nasazení Azure Resource Manager. AKS Engine provádí operaci škálování proti určitému fondu agentů. Po dokončení operace škálování AKS modul aktualizuje definici clusteru v stejném apimodel.jsv souboru. Definice clusteru odráží nové počty uzlů, aby odrážela aktualizovanou aktuální konfiguraci clusteru.

- [Škálování clusteru Kubernetes na rozbočovači Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Další kroky

- Další informace o [požadavcích na návrh hybridní aplikace](overview-app-design-considerations.md)
- Přečtěte si a navrhněte vylepšení [kódu pro tuto ukázku na GitHubu](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).