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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="0e031-103">Nasazení clusteru Kubernetes s vysokou dostupností v centru Azure Stack</span><span class="sxs-lookup"><span data-stu-id="0e031-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="0e031-104">V tomto článku se dozvíte, jak vytvořit vysoce dostupné prostředí clusteru Kubernetes nasazené na více instancích centra Azure Stack v různých fyzických umístěních.</span><span class="sxs-lookup"><span data-stu-id="0e031-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="0e031-105">V tomto průvodci nasazením řešení se dozvíte, jak:</span><span class="sxs-lookup"><span data-stu-id="0e031-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="0e031-106">Stažení a příprava modulu AKS</span><span class="sxs-lookup"><span data-stu-id="0e031-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="0e031-107">Připojení k virtuálnímu počítači pomocníka AKS Engine</span><span class="sxs-lookup"><span data-stu-id="0e031-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="0e031-108">Nasazení clusteru Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="0e031-109">Připojení ke clusteru Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="0e031-110">Připojení Azure Pipelines ke clusteru Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="0e031-111">Konfigurace sledování</span><span class="sxs-lookup"><span data-stu-id="0e031-111">Configure monitoring</span></span>
> - <span data-ttu-id="0e031-112">Nasazení aplikace</span><span class="sxs-lookup"><span data-stu-id="0e031-112">Deploy application</span></span>
> - <span data-ttu-id="0e031-113">Automatické škálování aplikace</span><span class="sxs-lookup"><span data-stu-id="0e031-113">Autoscale application</span></span>
> - <span data-ttu-id="0e031-114">Konfigurace služby Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="0e031-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="0e031-115">Upgrade Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="0e031-116">Škálování Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="0e031-117">![Hybridní pilíře](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="0e031-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="0e031-118">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="0e031-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="0e031-119">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="0e031-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="0e031-120">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="0e031-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="0e031-121">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="0e031-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="0e031-122">Předpoklady</span><span class="sxs-lookup"><span data-stu-id="0e031-122">Prerequisites</span></span>

<span data-ttu-id="0e031-123">Než začnete s tímto průvodcem nasazením, nezapomeňte:</span><span class="sxs-lookup"><span data-stu-id="0e031-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="0e031-124">Projděte si článek o [vzoru clusteru Kubernetes s vysokou dostupností](pattern-highly-available-kubernetes.md) .</span><span class="sxs-lookup"><span data-stu-id="0e031-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="0e031-125">Zkontrolujte obsah [doprovodného úložiště GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), které obsahuje další prostředky, na které se odkazuje v tomto článku.</span><span class="sxs-lookup"><span data-stu-id="0e031-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="0e031-126">Mít účet, který má přístup k [portálu pro uživatele centra Azure Stack](/azure-stack/user/azure-stack-use-portal), s alespoň [oprávněními Přispěvatel](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="0e031-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="0e031-127">Stažení a příprava AKS Engine</span><span class="sxs-lookup"><span data-stu-id="0e031-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="0e031-128">Modul AKS je binární soubor, který se dá použít v jakémkoli hostiteli se systémem Windows nebo Linux, který se může dostat k koncovým bodům Azure Resource Manager centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="0e031-129">Tato příručka popisuje nasazení nového virtuálního počítače se systémem Linux (nebo Windows) v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="0e031-130">Později se použije v případě, že AKS modul nasadí clustery Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="0e031-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="0e031-131">Můžete také použít stávající virtuální počítač se systémem Windows nebo Linux k nasazení clusteru Kubernetes v Azure Stackovém centru pomocí modulu AKS.</span><span class="sxs-lookup"><span data-stu-id="0e031-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="0e031-132">Podrobný proces a požadavky na modul AKS jsou popsané tady:</span><span class="sxs-lookup"><span data-stu-id="0e031-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="0e031-133">[Instalace modulu AKS v systému Linux do centra Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (nebo pomocí [systému Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="0e031-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="0e031-134">Modul AKS je pomocný nástroj pro nasazení a provozování (nespravovaných) Kubernetes clusterů (v Azure a centru Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="0e031-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="0e031-135">Podrobnosti a rozdíly modulu AKS na rozbočovači Azure Stack jsou popsány zde:</span><span class="sxs-lookup"><span data-stu-id="0e031-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="0e031-136">Co je modul AKS v centru Azure Stack?</span><span class="sxs-lookup"><span data-stu-id="0e031-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="0e031-137">[AKS Engine v centru Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (na GitHubu)</span><span class="sxs-lookup"><span data-stu-id="0e031-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="0e031-138">Ukázkové prostředí použije Terraformu k automatizaci nasazení virtuálního počítače modulu AKS.</span><span class="sxs-lookup"><span data-stu-id="0e031-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="0e031-139">[Podrobnosti a kód můžete najít v doprovodném úložišti GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="0e031-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="0e031-140">Výsledkem tohoto kroku je nová skupina prostředků v Azure Stack hub, která obsahuje pomocný virtuální počítač pro modul AKS a související prostředky:</span><span class="sxs-lookup"><span data-stu-id="0e031-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Prostředky virtuálních počítačů AKS Engine v centru Azure Stack](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="0e031-142">Pokud potřebujete nasadit modul AKS v odpojeném prostředí gapped, přečtěte si téma [odpojené instance centra Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) , kde se dozvíte víc.</span><span class="sxs-lookup"><span data-stu-id="0e031-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="0e031-143">V dalším kroku použijeme nově nasazený virtuální počítač AKS Engine k nasazení clusteru Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="0e031-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="0e031-144">Připojení k virtuálnímu počítači pomocníka AKS Engine</span><span class="sxs-lookup"><span data-stu-id="0e031-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="0e031-145">Nejprve se musíte připojit k dříve vytvořenému virtuálnímu počítači pomocníka AKS Engine.</span><span class="sxs-lookup"><span data-stu-id="0e031-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="0e031-146">Virtuální počítač by měl mít veřejnou IP adresu a měl by být přístupný přes SSH (port 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="0e031-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Stránka s přehledem virtuálního počítače AKS Engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="0e031-148">K připojení k virtuálnímu počítači se systémem Linux pomocí SSH můžete použít libovolný nástroj, třeba MobaXterm, výstup nebo PowerShell ve Windows 10.</span><span class="sxs-lookup"><span data-stu-id="0e031-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="0e031-149">Po připojení spusťte příkaz `aks-engine` .</span><span class="sxs-lookup"><span data-stu-id="0e031-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="0e031-150">Další informace o modulu AKS a verzích Kubernetes najdete v části [podporované verze modulu AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) .</span><span class="sxs-lookup"><span data-stu-id="0e031-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![příklad příkazového řádku AKS-Engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="0e031-152">Nasazení clusteru Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="0e031-153">Samotný pomocný virtuální počítač modulu AKS ještě nevytvořil cluster Kubernetes v našem centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="0e031-154">Vytvoření clusteru je první akce, která se má provést na virtuálním počítači pomocníka s modulem AKS.</span><span class="sxs-lookup"><span data-stu-id="0e031-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="0e031-155">Podrobný postup je popsán zde:</span><span class="sxs-lookup"><span data-stu-id="0e031-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="0e031-156">Nasazení clusteru Kubernetes s modulem AKS v centru Azure Stack</span><span class="sxs-lookup"><span data-stu-id="0e031-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="0e031-157">Konečný výsledek `aks-engine deploy` příkazu a přípravy v předchozích krocích je plně funkční cluster Kubernetes nasazený do prostoru klienta první instance centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="0e031-158">Samotný cluster se skládá z komponent Azure IaaS, jako jsou virtuální počítače, nástroje pro vyrovnávání zatížení, virtuální sítě, disky a tak dále.</span><span class="sxs-lookup"><span data-stu-id="0e031-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Komponenty cluster IaaS Azure Stack portál centra](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="0e031-160">Nástroj pro vyrovnávání zatížení Azure (koncový bod rozhraní K8s API)</span><span class="sxs-lookup"><span data-stu-id="0e031-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="0e031-161">Pracovní uzly (fond agentů)</span><span class="sxs-lookup"><span data-stu-id="0e031-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="0e031-162">Hlavní uzly</span><span class="sxs-lookup"><span data-stu-id="0e031-162">Master Nodes</span></span>

<span data-ttu-id="0e031-163">Cluster je teď v provozu a v dalším kroku se k němu připojíme.</span><span class="sxs-lookup"><span data-stu-id="0e031-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="0e031-164">Připojení ke clusteru Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="0e031-165">Nyní se můžete připojit k dříve vytvořenému clusteru Kubernetes, a to buď prostřednictvím protokolu SSH (pomocí klíče SSH zadaného jako součást nasazení), nebo prostřednictvím `kubectl` (doporučeno).</span><span class="sxs-lookup"><span data-stu-id="0e031-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="0e031-166">Nástroj příkazového řádku Kubernetes `kubectl` je k dispozici pro Windows, Linux a MacOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="0e031-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="0e031-167">Je už předem nainstalovaný a nakonfigurovaný na hlavních uzlech tohoto clusteru.</span><span class="sxs-lookup"><span data-stu-id="0e031-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Spustit kubectl na hlavním uzlu](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="0e031-169">Nedoporučujeme používat hlavní uzel jako JumpBox pro úlohy správy.</span><span class="sxs-lookup"><span data-stu-id="0e031-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="0e031-170">`kubectl`Konfigurace je uložena v nástroji `.kube/config` na hlavních uzlech a také na virtuálním počítači modulu AKS.</span><span class="sxs-lookup"><span data-stu-id="0e031-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="0e031-171">Konfiguraci můžete zkopírovat do počítače správce s připojením k Kubernetes clusteru a použít `kubectl` příkaz.</span><span class="sxs-lookup"><span data-stu-id="0e031-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="0e031-172">`.kube/config`Soubor se používá také později ke konfiguraci připojení služby v Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="0e031-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="0e031-173">Tyto soubory udržujte v bezpečí, protože obsahují přihlašovací údaje pro váš cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="0e031-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="0e031-174">Útočník s přístupem k souboru má dostatek informací, aby k němu mohl získat přístup správce.</span><span class="sxs-lookup"><span data-stu-id="0e031-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="0e031-175">Všechny akce, které se provádějí pomocí počátečního `.kube/config` souboru, se provádějí pomocí účtu správce clusteru.</span><span class="sxs-lookup"><span data-stu-id="0e031-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="0e031-176">Nyní můžete vyzkoušet různé příkazy pomocí nástroje `kubectl` a zkontrolovat stav clusteru.</span><span class="sxs-lookup"><span data-stu-id="0e031-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

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
> <span data-ttu-id="0e031-177">Kubernetes má vlastní model \*založený na rolích Access Control (RBAC)\*\*, který umožňuje vytvářet podrobné definice rolí a vazby rolí.</span><span class="sxs-lookup"><span data-stu-id="0e031-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="0e031-178">Toto je vhodnější způsob, jak řídit přístup ke clusteru místo ručního nastavení oprávnění správce clusteru.</span><span class="sxs-lookup"><span data-stu-id="0e031-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="0e031-179">Připojení Azure Pipelines k clusterům Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="0e031-180">K připojení Azure Pipelines k nově nasazenému clusteru Kubernetes potřebujeme svůj soubor Kube config ( `.kube/config` ), jak je vysvětleno v předchozím kroku.</span><span class="sxs-lookup"><span data-stu-id="0e031-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="0e031-181">Připojte se k jednomu z hlavních uzlů clusteru Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="0e031-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="0e031-182">Zkopírujte obsah `.kube/config` souboru.</span><span class="sxs-lookup"><span data-stu-id="0e031-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="0e031-183">Přejít na Azure DevOps > nastavení projektu > připojení služby a vytvořit nové připojení služby Kubernetes (použijte KubeConfig jako metodu ověřování)</span><span class="sxs-lookup"><span data-stu-id="0e031-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="0e031-184">Azure Pipelines (nebo jeho agenti sestavení) musí mít přístup k rozhraní Kubernetes API.</span><span class="sxs-lookup"><span data-stu-id="0e031-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="0e031-185">Pokud je k dispozici připojení k Internetu z Azure Pipelines do centra Azure Stack Kubernetes clusetr, bude nutné nasadit agenta sestavení Azure Pipelines v místním prostředí.</span><span class="sxs-lookup"><span data-stu-id="0e031-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="0e031-186">Při nasazování agentů pro samoobslužné hostování pro Azure Pipelines můžete nasadit buď v centru Azure Stack, nebo na počítači, který má síťové připojení ke všem požadovaným koncovým bodům správy.</span><span class="sxs-lookup"><span data-stu-id="0e031-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="0e031-187">Podrobnosti najdete tady:</span><span class="sxs-lookup"><span data-stu-id="0e031-187">See the details here:</span></span>

* <span data-ttu-id="0e031-188">[Agenti Azure Pipelines](/azure/devops/pipelines/agents/agents) v [systému Windows](/azure/devops/pipelines/agents/v2-windows) nebo [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="0e031-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="0e031-189">Část s [informacemi pro nasazení vzoru (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) obsahuje rozhodovací tok, který vám pomůže pochopit, jestli se mají používat agenti hostovaná Microsoftem nebo agenti s místním hostováním:</span><span class="sxs-lookup"><span data-stu-id="0e031-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="0e031-190">[![agenti s místním hostováním rozhodování](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="0e031-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="0e031-191">V tomto ukázkovém řešení zahrnuje topologie agent sestavení v místním prostředí v každé instanci centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="0e031-192">Agent má přístup ke koncovým bodům správy centra Azure Stack a koncovým bodům rozhraní API clusteru Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="0e031-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="0e031-193">[![pouze odchozí přenosy](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="0e031-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="0e031-194">Tento návrh splňuje společný regulativní požadavek, který má pouze odchozí připojení z řešení aplikace.</span><span class="sxs-lookup"><span data-stu-id="0e031-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="0e031-195">Konfigurace sledování</span><span class="sxs-lookup"><span data-stu-id="0e031-195">Configure monitoring</span></span>

<span data-ttu-id="0e031-196">Můžete použít [Azure monitor](/azure/azure-monitor/) pro kontejnery k monitorování kontejnerů v řešení.</span><span class="sxs-lookup"><span data-stu-id="0e031-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="0e031-197">Tyto body Azure Monitor clusteru Kubernetes nasazeného pro modul AKS v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="0e031-198">Existují dva způsoby, jak povolit Azure Monitor v clusteru.</span><span class="sxs-lookup"><span data-stu-id="0e031-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="0e031-199">Oba způsoby vyžadují, abyste si nastavili pracovní prostor Log Analytics v Azure.</span><span class="sxs-lookup"><span data-stu-id="0e031-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="0e031-200">[Metoda One](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) používá Helm graf.</span><span class="sxs-lookup"><span data-stu-id="0e031-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="0e031-201">[Metoda 2](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) jako součást specifikace clusteru AKS Engine</span><span class="sxs-lookup"><span data-stu-id="0e031-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="0e031-202">V ukázkové topologii je použita "metoda One", která umožňuje snazší instalaci procesu a aktualizací.</span><span class="sxs-lookup"><span data-stu-id="0e031-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="0e031-203">Pro další krok potřebujete pracovní prostor Azure LogAnalytics (ID a klíč), `Helm` (verze 3) a `kubectl` na vašem počítači.</span><span class="sxs-lookup"><span data-stu-id="0e031-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="0e031-204">Helm je Kubernetes správce balíčků, který je dostupný jako binární soubor, který je spuštěný v macOS, Windows a Linux.</span><span class="sxs-lookup"><span data-stu-id="0e031-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="0e031-205">Můžete si ho stáhnout tady: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm spoléhá na konfigurační soubor Kubernetes, který se používá pro `kubectl` příkaz.</span><span class="sxs-lookup"><span data-stu-id="0e031-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="0e031-206">Tento příkaz nainstaluje agenta Azure Monitor do clusteru Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="0e031-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="0e031-207">Agent Operations Management Suite (OMS) v clusteru Kubernetes odešle data monitorování do vašeho pracovního prostoru Azure Log Analytics (pomocí odchozího protokolu HTTPS).</span><span class="sxs-lookup"><span data-stu-id="0e031-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="0e031-208">Teď můžete pomocí Azure Monitor získat hlubší přehled o clusterech Kubernetes v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="0e031-209">Tento návrh představuje účinný způsob, jak předvést výkon analýz, které se dají automaticky nasadit do clusterů vaší aplikace.</span><span class="sxs-lookup"><span data-stu-id="0e031-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="0e031-210">[![Clustery centra Azure Stack ve službě Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="0e031-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="0e031-211">[![Podrobnosti o Azure Monitor clusteru](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="0e031-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="0e031-212">Pokud Azure Monitor nezobrazuje žádná data centra Azure Stack, ujistěte se, že jste postupovali podle pokynů k pečlivému [přidání AzureMonitor-Containers řešení do pracovního prostoru Azure Loganalytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) .</span><span class="sxs-lookup"><span data-stu-id="0e031-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="0e031-213">Nasazení aplikace</span><span class="sxs-lookup"><span data-stu-id="0e031-213">Deploy the application</span></span>

<span data-ttu-id="0e031-214">Před instalací naší ukázkové aplikace je k dispozici další krok konfigurace nginx řadiče pro příchozí přenos dat v našem clusteru Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="0e031-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="0e031-215">Kontroler příchozího přenosu dat se používá jako nástroj pro vyrovnávání zatížení vrstvy 7 ke směrování provozu v našem clusteru založeném na hostiteli, cestě nebo protokolu.</span><span class="sxs-lookup"><span data-stu-id="0e031-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="0e031-216">Nginx – příchozí je k dispozici jako graf Helm.</span><span class="sxs-lookup"><span data-stu-id="0e031-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="0e031-217">Podrobné pokyny najdete v [úložišti GitHub Helm Chart](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="0e031-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="0e031-218">Naše ukázková aplikace je také zabalena jako Helm graf, jako je například [Agent Azure Monitoring](#configure-monitoring) v předchozím kroku.</span><span class="sxs-lookup"><span data-stu-id="0e031-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="0e031-219">V takovém případě je jednoduché nasadit aplikaci do našeho clusteru Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="0e031-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="0e031-220">[Soubory grafu Helm najdete v doprovodném úložišti GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) .</span><span class="sxs-lookup"><span data-stu-id="0e031-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="0e031-221">Ukázková aplikace je tři aplikace vrstev nasazená do clusteru Kubernetes v každé ze dvou instancí centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="0e031-222">Aplikace používá databázi MongoDB.</span><span class="sxs-lookup"><span data-stu-id="0e031-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="0e031-223">Další informace o tom, jak získat data replikovaná napříč několika instancemi, najdete v tématu popisujícím data vzorů [a úložiště](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="0e031-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="0e031-224">Po nasazení grafu Helm pro aplikaci uvidíte všechny tři úrovně vaší aplikace, které jsou reprezentovány jako nasazení a stavové sady (pro databázi), jedním pod:</span><span class="sxs-lookup"><span data-stu-id="0e031-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

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

<span data-ttu-id="0e031-225">Na straně služby najdete Nginx kontroler příchozího přenosu dat a jeho veřejnou IP adresu:</span><span class="sxs-lookup"><span data-stu-id="0e031-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

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

<span data-ttu-id="0e031-226">Adresa externí IP adresy je naším koncovým bodem aplikace.</span><span class="sxs-lookup"><span data-stu-id="0e031-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="0e031-227">Je to způsob, jakým se uživatelé připojí k otevření aplikace a použijí se taky jako koncový bod pro náš další krok [konfigurace Traffic Manager](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="0e031-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="0e031-228">Automatické škálování aplikace</span><span class="sxs-lookup"><span data-stu-id="0e031-228">Autoscale the application</span></span>
<span data-ttu-id="0e031-229">Volitelně můžete nakonfigurovat automatické [škálování vodorovně pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) , abyste mohli škálovat nahoru nebo dolů na základě určitých metrik, jako je využití procesoru.</span><span class="sxs-lookup"><span data-stu-id="0e031-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="0e031-230">Následující příkaz vytvoří horizontální horizontální navýšení, které udržuje 1 až 10 replik v Luskech kontrolovaných pomocí nasazení webu hodnocení.</span><span class="sxs-lookup"><span data-stu-id="0e031-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="0e031-231">HPA zvýší a sníží počet replik (přes nasazení), aby se zachovalo průměrné využití CPU ve všech Luskech 80%.</span><span class="sxs-lookup"><span data-stu-id="0e031-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="0e031-232">Aktuální stav automatického škálování můžete kontrolovat spuštěním:</span><span class="sxs-lookup"><span data-stu-id="0e031-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="0e031-233">Konfigurace služby Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="0e031-233">Configure Traffic Manager</span></span>

<span data-ttu-id="0e031-234">K distribuci provozu mezi dvěma (nebo více) nasazeními aplikace použijeme [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="0e031-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="0e031-235">Azure Traffic Manager je nástroj pro vyrovnávání zatížení založený na DNS v Azure.</span><span class="sxs-lookup"><span data-stu-id="0e031-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="0e031-236">Traffic Manager používá DNS k přímému směrování požadavků klientů na nejvhodnější koncový bod služby na základě metody směrování provozu a stavu koncových bodů.</span><span class="sxs-lookup"><span data-stu-id="0e031-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="0e031-237">Místo používání Azure Traffic Manager můžete také použít jiná globální řešení vyrovnávání zatížení hostovaná místně.</span><span class="sxs-lookup"><span data-stu-id="0e031-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="0e031-238">V ukázkovém scénáři použijeme Azure Traffic Manager k distribuci provozu mezi dvěma instancemi naší aplikace.</span><span class="sxs-lookup"><span data-stu-id="0e031-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="0e031-239">Můžou běžet na instancích centra Azure Stack ve stejném nebo různých umístěních:</span><span class="sxs-lookup"><span data-stu-id="0e031-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![místní Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="0e031-241">V Azure nakonfigurujeme Traffic Manager tak, aby odkazovaly na dvě různé instance naší aplikace:</span><span class="sxs-lookup"><span data-stu-id="0e031-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="0e031-242">[![Profil koncového bodu TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="0e031-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="0e031-243">Jak vidíte, dva koncové body ukazují na dvě instance nasazené aplikace z [předchozí části](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="0e031-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="0e031-244">V tomto okamžiku:</span><span class="sxs-lookup"><span data-stu-id="0e031-244">At this point:</span></span>
- <span data-ttu-id="0e031-245">Byla vytvořena infrastruktura Kubernetes, včetně kontroleru příchozího přenosu dat.</span><span class="sxs-lookup"><span data-stu-id="0e031-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="0e031-246">Clustery byly nasazeny v rámci dvou Azure Stackch instancí centra.</span><span class="sxs-lookup"><span data-stu-id="0e031-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="0e031-247">Monitorování bylo nakonfigurováno.</span><span class="sxs-lookup"><span data-stu-id="0e031-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="0e031-248">Azure Traffic Manager vyrovnává zatížení mezi dvěma instancemi centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="0e031-249">V horní části této infrastruktury se ukázková aplikace nasadila automatizovaným způsobem pomocí Helm grafů.</span><span class="sxs-lookup"><span data-stu-id="0e031-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="0e031-250">Řešení by teď mělo být dostupné pro uživatele.</span><span class="sxs-lookup"><span data-stu-id="0e031-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="0e031-251">K dispozici jsou také některé provozní předpoklady po nasazení, které jsou zahrnuty v následujících dvou částech.</span><span class="sxs-lookup"><span data-stu-id="0e031-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="0e031-252">Upgrade Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="0e031-253">Při upgradu clusteru Kubernetes Vezměte v úvahu následující témata:</span><span class="sxs-lookup"><span data-stu-id="0e031-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="0e031-254">Upgrade clusteru Kubernetes je složitá operace dne 2, kterou je možné provést pomocí stroje AKS.</span><span class="sxs-lookup"><span data-stu-id="0e031-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="0e031-255">Další informace najdete v tématu [upgrade clusteru Kubernetes na rozbočovači Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="0e031-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="0e031-256">Modul AKS vám umožňuje upgradovat clustery na novější verze Kubernetes a základní image operačního systému.</span><span class="sxs-lookup"><span data-stu-id="0e031-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="0e031-257">Další informace najdete v tématu [Postup upgradu na novější verzi Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="0e031-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="0e031-258">Můžete také upgradovat pouze uzly v části snáška na novější verze imagí základní operační systémy.</span><span class="sxs-lookup"><span data-stu-id="0e031-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="0e031-259">Další informace najdete v tématu [Postup upgradu image operačního systému](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="0e031-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="0e031-260">Novější image základního operačního systému obsahují aktualizace zabezpečení a jádra.</span><span class="sxs-lookup"><span data-stu-id="0e031-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="0e031-261">Je zodpovědností operátora clusteru sledovat dostupnost novějších verzí Kubernetes a imagí operačních systémů.</span><span class="sxs-lookup"><span data-stu-id="0e031-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="0e031-262">Operátor by měl naplánovat a spustit tyto upgrady pomocí AKS Engine.</span><span class="sxs-lookup"><span data-stu-id="0e031-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="0e031-263">Základní image operačního systému se musí stáhnout z webu centra Azure Stack pomocí operátoru centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="0e031-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="0e031-264">Škálování Kubernetes</span><span class="sxs-lookup"><span data-stu-id="0e031-264">Scale Kubernetes</span></span>

<span data-ttu-id="0e031-265">Škálování je další operace dne 2, kterou je možné orchestrovat pomocí AKS Engine.</span><span class="sxs-lookup"><span data-stu-id="0e031-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="0e031-266">Příkaz Scale znovu použije váš konfigurační soubor clusteru (apimodel.json) ve výstupním adresáři jako vstup pro nové nasazení Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="0e031-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="0e031-267">AKS Engine provádí operaci škálování proti určitému fondu agentů.</span><span class="sxs-lookup"><span data-stu-id="0e031-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="0e031-268">Po dokončení operace škálování AKS modul aktualizuje definici clusteru v stejném apimodel.jsv souboru.</span><span class="sxs-lookup"><span data-stu-id="0e031-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="0e031-269">Definice clusteru odráží nové počty uzlů, aby odrážela aktualizovanou aktuální konfiguraci clusteru.</span><span class="sxs-lookup"><span data-stu-id="0e031-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="0e031-270">Škálování clusteru Kubernetes na rozbočovači Azure Stack</span><span class="sxs-lookup"><span data-stu-id="0e031-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="0e031-271">Další kroky</span><span class="sxs-lookup"><span data-stu-id="0e031-271">Next steps</span></span>

- <span data-ttu-id="0e031-272">Další informace o [požadavcích na návrh hybridní aplikace](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="0e031-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="0e031-273">Přečtěte si a navrhněte vylepšení [kódu pro tuto ukázku na GitHubu](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="0e031-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>