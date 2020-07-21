---
title: Nasazení hybridní aplikace s místními daty, která škálují mezi cloudy
description: Naučte se, jak nasadit aplikaci, která používá místní data, a škálujte mezi cloudy pomocí Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477316"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="1a099-103">Nasazení hybridní aplikace s místními daty, která škálují mezi cloudy</span><span class="sxs-lookup"><span data-stu-id="1a099-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="1a099-104">V tomto průvodci se dozvíte, jak nasadit hybridní aplikaci, která zahrnuje Azure i Azure Stack hub a používá jeden místní zdroj dat.</span><span class="sxs-lookup"><span data-stu-id="1a099-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="1a099-105">Pomocí hybridního cloudového řešení můžete kombinovat výhody dodržování předpisů privátního cloudu s škálovatelností veřejného cloudu.</span><span class="sxs-lookup"><span data-stu-id="1a099-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="1a099-106">Vývojáři můžou také využít výhody Microsoft Developer ekosystému a využívat jejich dovednosti v cloudových i místních prostředích.</span><span class="sxs-lookup"><span data-stu-id="1a099-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="1a099-107">Přehled a předpoklady</span><span class="sxs-lookup"><span data-stu-id="1a099-107">Overview and assumptions</span></span>

<span data-ttu-id="1a099-108">Postupujte podle tohoto kurzu a nastavte pracovní postup, který vývojářům umožní nasadit identickou webovou aplikaci do veřejného cloudu a privátního cloudu.</span><span class="sxs-lookup"><span data-stu-id="1a099-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="1a099-109">Tato aplikace má přístup k síti směrovatelné přes Internet, která je hostovaná v privátním cloudu.</span><span class="sxs-lookup"><span data-stu-id="1a099-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="1a099-110">Tyto webové aplikace jsou monitorovány a když je špička v provozu, program upraví záznamy DNS pro přesměrování provozu do veřejného cloudu.</span><span class="sxs-lookup"><span data-stu-id="1a099-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="1a099-111">Když provoz na úrovni před špičkou klesne, provoz se směruje zpátky do privátního cloudu.</span><span class="sxs-lookup"><span data-stu-id="1a099-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="1a099-112">Tento kurz se zabývá následujícími úkony:</span><span class="sxs-lookup"><span data-stu-id="1a099-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="1a099-113">Nasaďte SQL Server databázový server s hybridním připojením.</span><span class="sxs-lookup"><span data-stu-id="1a099-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="1a099-114">Připojte webovou aplikaci v globálním Azure k hybridní síti.</span><span class="sxs-lookup"><span data-stu-id="1a099-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="1a099-115">Nakonfigurujte DNS pro škálování mezi cloudy.</span><span class="sxs-lookup"><span data-stu-id="1a099-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="1a099-116">Nakonfigurujte certifikáty SSL pro škálování mezi cloudy.</span><span class="sxs-lookup"><span data-stu-id="1a099-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="1a099-117">Nakonfigurujte a nasaďte webovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1a099-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="1a099-118">Vytvořte profil Traffic Manager a nakonfigurujte ho pro škálování mezi cloudy.</span><span class="sxs-lookup"><span data-stu-id="1a099-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="1a099-119">Nastavte Application Insights monitorování a upozorňování na zvýšení provozu.</span><span class="sxs-lookup"><span data-stu-id="1a099-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="1a099-120">Nakonfigurujte automatické přepínání provozu mezi globálním centrem Azure a Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="1a099-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="1a099-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="1a099-122">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="1a099-123">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="1a099-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="1a099-124">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="1a099-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="1a099-125">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="1a099-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="1a099-126">Předpoklady</span><span class="sxs-lookup"><span data-stu-id="1a099-126">Assumptions</span></span>

<span data-ttu-id="1a099-127">V tomto kurzu se předpokládá, že máte základní znalosti globálního centra Azure a centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="1a099-128">Pokud se chcete dozvědět víc, než začnete s kurzem, přečtěte si tyto články:</span><span class="sxs-lookup"><span data-stu-id="1a099-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="1a099-129">Úvod do Azure</span><span class="sxs-lookup"><span data-stu-id="1a099-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="1a099-130">Klíčové koncepty centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="1a099-131">V tomto kurzu se taky předpokládá, že máte předplatné Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="1a099-132">Pokud předplatné nemáte, [Vytvořte si bezplatný účet](https://azure.microsoft.com/free/) před tím, než začnete.</span><span class="sxs-lookup"><span data-stu-id="1a099-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="1a099-133">Požadavky</span><span class="sxs-lookup"><span data-stu-id="1a099-133">Prerequisites</span></span>

<span data-ttu-id="1a099-134">Než začnete s tímto řešením, ujistěte se, že splňujete následující požadavky:</span><span class="sxs-lookup"><span data-stu-id="1a099-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="1a099-135">Azure Stack Development Kit (ASDK) nebo předplatné v integrovaném systému Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="1a099-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="1a099-136">Pokud chcete nasadit ASDK, postupujte podle pokynů v tématu [nasazení ASDK pomocí instalačního programu](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="1a099-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="1a099-137">Vaše instalace centra Azure Stack by měla mít nainstalované následující:</span><span class="sxs-lookup"><span data-stu-id="1a099-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="1a099-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="1a099-138">The Azure App Service.</span></span> <span data-ttu-id="1a099-139">K nasazení a konfiguraci Azure App Service ve vašem prostředí můžete použít operátor centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="1a099-140">Tento kurz vyžaduje, aby v App Service bylo aspoň jedna (1) dostupná vyhrazená role pracovního procesu.</span><span class="sxs-lookup"><span data-stu-id="1a099-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="1a099-141">Bitová kopie systému Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="1a099-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="1a099-142">Windows Server 2016 s imagí Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="1a099-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="1a099-143">Příslušné plány a nabídky.</span><span class="sxs-lookup"><span data-stu-id="1a099-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="1a099-144">Název domény pro vaši webovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1a099-144">A domain name for your web app.</span></span> <span data-ttu-id="1a099-145">Pokud název domény nemáte, můžete si ho koupit od poskytovatele domény, jako je GoDaddy, Bluehost a InMotion.</span><span class="sxs-lookup"><span data-stu-id="1a099-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="1a099-146">Certifikát SSL pro vaši doménu od důvěryhodné certifikační autority, jako je LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="1a099-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="1a099-147">Webová aplikace, která komunikuje s databází SQL Server a podporuje Application Insights.</span><span class="sxs-lookup"><span data-stu-id="1a099-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="1a099-148">Ukázkovou aplikaci [dotnetcore-SQLDB-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) si můžete stáhnout z GitHubu.</span><span class="sxs-lookup"><span data-stu-id="1a099-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="1a099-149">Hybridní síť mezi virtuální sítí Azure a virtuální sítí centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="1a099-150">Podrobné pokyny najdete v tématu [Konfigurace připojení hybridního cloudu pomocí Azure a centra Azure Stack](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="1a099-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="1a099-151">Hybridní kanál průběžné integrace nebo průběžného nasazování (CI/CD) s privátním agentem sestavení v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="1a099-152">Podrobné pokyny najdete v tématu [Konfigurace hybridní cloudové identity s aplikacemi Azure a Azure Stack hub](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="1a099-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="1a099-153">Nasazení SQL Server databázového serveru s hybridním připojením</span><span class="sxs-lookup"><span data-stu-id="1a099-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="1a099-154">Přihlašovat se k portálu Azure Stack User Portal.</span><span class="sxs-lookup"><span data-stu-id="1a099-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="1a099-155">Na **řídicím panelu**vyberte **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="1a099-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Tržiště centra Azure Stack](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="1a099-157">V **tržišti**vyberte **COMPUTE**a pak klikněte na **Další**.</span><span class="sxs-lookup"><span data-stu-id="1a099-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="1a099-158">V části **Další**vyberte **bezplatnou licenci SQL Server: SQL Server 2017 vývojář v imagi Windows serveru** .</span><span class="sxs-lookup"><span data-stu-id="1a099-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Výběr image virtuálního počítače na portálu Azure Stack User Portal](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="1a099-160">**Licence na bezplatnou SQL Server: SQL Server 2017 vývojář na Windows serveru**vyberte **vytvořit**.</span><span class="sxs-lookup"><span data-stu-id="1a099-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="1a099-161">**Základy > nakonfigurovat základní nastavení**, zadat **název** virtuálního počítače (VM), **uživatelské jméno** pro SQL Server SA a **heslo** pro SA.</span><span class="sxs-lookup"><span data-stu-id="1a099-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="1a099-162">V rozevíracím seznamu **předplatné** vyberte předplatné, na které nasazujete.</span><span class="sxs-lookup"><span data-stu-id="1a099-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="1a099-163">V poli **Skupina prostředků** **Vyberte vybrat existující** a vložte virtuální počítač do stejné skupiny prostředků, jako je vaše webová aplikace Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="1a099-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Konfigurace základního nastavení pro virtuální počítač v portálu Azure Stack User Portal](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="1a099-165">V části **Velikost**vyberte velikost pro svůj virtuální počítač.</span><span class="sxs-lookup"><span data-stu-id="1a099-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="1a099-166">Pro tento kurz doporučujeme A2_Standard nebo DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="1a099-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="1a099-167">V části **nastavení > nakonfigurovat volitelné funkce**nakonfigurujte následující nastavení:</span><span class="sxs-lookup"><span data-stu-id="1a099-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="1a099-168">**Účet úložiště**: vytvořte nový účet, pokud ho potřebujete.</span><span class="sxs-lookup"><span data-stu-id="1a099-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="1a099-169">**Virtuální síť**:</span><span class="sxs-lookup"><span data-stu-id="1a099-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="1a099-170">Ujistěte se, že je váš virtuální počítač SQL Server nasazený ve stejné virtuální síti jako brány VPN.</span><span class="sxs-lookup"><span data-stu-id="1a099-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="1a099-171">**Veřejná IP adresa**: použijte výchozí nastavení.</span><span class="sxs-lookup"><span data-stu-id="1a099-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="1a099-172">**Skupina zabezpečení sítě**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="1a099-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="1a099-173">Vytvořte nový NSG.</span><span class="sxs-lookup"><span data-stu-id="1a099-173">Create a new NSG.</span></span>
   - <span data-ttu-id="1a099-174">**Rozšíření a monitorování**: ponechte výchozí nastavení.</span><span class="sxs-lookup"><span data-stu-id="1a099-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="1a099-175">**Účet úložiště diagnostiky**: vytvořte nový účet, pokud ho potřebujete.</span><span class="sxs-lookup"><span data-stu-id="1a099-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="1a099-176">Kliknutím na **OK** uložte svou konfiguraci.</span><span class="sxs-lookup"><span data-stu-id="1a099-176">Select **OK** to save your configuration.</span></span>

     ![Konfigurace volitelných funkcí virtuálních počítačů na portálu Azure Stack User Portal](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="1a099-178">V části **nastavení SQL Server**nakonfigurujte následující nastavení:</span><span class="sxs-lookup"><span data-stu-id="1a099-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="1a099-179">V případě **připojení SQL**vyberte **veřejné (Internet)**.</span><span class="sxs-lookup"><span data-stu-id="1a099-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="1a099-180">V poli **port**ponechte výchozí hodnotu **1433**.</span><span class="sxs-lookup"><span data-stu-id="1a099-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="1a099-181">Pro **ověřování SQL**vyberte **Povolit**.</span><span class="sxs-lookup"><span data-stu-id="1a099-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="1a099-182">Pokud povolíte ověřování SQL, mělo by se automaticky naplnit informace o "SQLAdmin", které jste nakonfigurovali v **základních**informacích.</span><span class="sxs-lookup"><span data-stu-id="1a099-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="1a099-183">U zbývajících nastavení ponechte výchozí nastavení.</span><span class="sxs-lookup"><span data-stu-id="1a099-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="1a099-184">Vyberte **OK**.</span><span class="sxs-lookup"><span data-stu-id="1a099-184">Select **OK**.</span></span>

     ![Konfigurace nastavení SQL Server v uživatelském portálu centra Azure Stack](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="1a099-186">Na stránce **Souhrn**Zkontrolujte konfiguraci virtuálních počítačů a potom kliknutím na **tlačítko OK** spusťte nasazení.</span><span class="sxs-lookup"><span data-stu-id="1a099-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Souhrn konfigurace na portálu pro uživatele centra Azure Stack](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="1a099-188">Vytvoření nového virtuálního počítače nějakou dobu trvá.</span><span class="sxs-lookup"><span data-stu-id="1a099-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="1a099-189">STAV virtuálních počítačů můžete zobrazit na **virtuálních počítačích**.</span><span class="sxs-lookup"><span data-stu-id="1a099-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Stav virtuálních počítačů v portálu User Portal centra Azure Stack](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="1a099-191">Vytváření webových aplikací v Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1a099-192">Azure App Service zjednodušuje spouštění a správu webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="1a099-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="1a099-193">Vzhledem k tomu, že centrum Azure Stack je konzistentní s Azure, může App Service běžet v obou prostředích.</span><span class="sxs-lookup"><span data-stu-id="1a099-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="1a099-194">K hostování vaší aplikace použijete App Service.</span><span class="sxs-lookup"><span data-stu-id="1a099-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="1a099-195">Vytváření webových aplikací</span><span class="sxs-lookup"><span data-stu-id="1a099-195">Create web apps</span></span>

1. <span data-ttu-id="1a099-196">Pomocí pokynů v tématu [Správa plánu App Service v Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)vytvořte webovou aplikaci v Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="1a099-197">Nezapomeňte umístit webovou aplikaci do stejného předplatného a skupiny prostředků, jako je vaše hybridní síť.</span><span class="sxs-lookup"><span data-stu-id="1a099-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="1a099-198">V Azure Stackovém centru opakujte předchozí krok (1).</span><span class="sxs-lookup"><span data-stu-id="1a099-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="1a099-199">Přidat trasu pro Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="1a099-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="1a099-200">App Service v centru Azure Stack musí být směrovatelné z veřejného Internetu a umožnit tak uživatelům přístup k vaší aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1a099-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="1a099-201">Pokud je vaše centrum Azure Stack dostupné z Internetu, poznamenejte si veřejnou IP adresu nebo adresu URL pro webovou aplikaci Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="1a099-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="1a099-202">Pokud používáte ASDK, můžete [nakonfigurovat mapování statického překladu adres (NAT)](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) , které vystavuje App Service mimo virtuální prostředí.</span><span class="sxs-lookup"><span data-stu-id="1a099-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="1a099-203">Připojení webové aplikace v Azure k hybridní síti</span><span class="sxs-lookup"><span data-stu-id="1a099-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="1a099-204">Aby bylo možné zajistit propojení mezi webovým front-end v Azure a databází SQL Server v centru Azure Stack, musí být webová aplikace připojená k hybridní síti mezi Azure a centrem Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="1a099-205">Pokud chcete povolit připojení, budete muset:</span><span class="sxs-lookup"><span data-stu-id="1a099-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="1a099-206">Nakonfigurujte připojení Point-to-site.</span><span class="sxs-lookup"><span data-stu-id="1a099-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="1a099-207">Nakonfigurujte webovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1a099-207">Configure the web app.</span></span>
- <span data-ttu-id="1a099-208">Upravte bránu místní sítě v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="1a099-209">Konfigurace virtuální sítě Azure pro připojení Point-to-site</span><span class="sxs-lookup"><span data-stu-id="1a099-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="1a099-210">Brána virtuální sítě na straně Azure hybridní sítě musí umožňovat připojení typu Point-to-site k integraci s Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="1a099-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="1a099-211">V Azure přejdete na stránku brány virtuální sítě.</span><span class="sxs-lookup"><span data-stu-id="1a099-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="1a099-212">V části **Nastavení**vyberte **Konfigurace Point-to-site**.</span><span class="sxs-lookup"><span data-stu-id="1a099-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Možnost Point-to-site v bráně virtuální sítě Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="1a099-214">Vyberte **Konfigurovat nyní** pro konfiguraci Point-to-site.</span><span class="sxs-lookup"><span data-stu-id="1a099-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Spuštění konfigurace Point-to-site v bráně virtuální sítě Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="1a099-216">Na stránce konfigurace **Point-to-site** zadejte rozsah privátních IP adres, který chcete použít ve **fondu adres**.</span><span class="sxs-lookup"><span data-stu-id="1a099-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="1a099-217">Ujistěte se, že se rozsah, který zadáte, nepřekrývá s žádným z rozsahů adres, které už jsou využívané v podsítích v globálním komponentě Azure nebo Azure Stack hub hybridní sítě.</span><span class="sxs-lookup"><span data-stu-id="1a099-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="1a099-218">V části **Typ tunelu**zrušte ověření **IKEv2 VPN**.</span><span class="sxs-lookup"><span data-stu-id="1a099-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="1a099-219">Výběrem **Uložit** dokončete konfiguraci Point-to-site.</span><span class="sxs-lookup"><span data-stu-id="1a099-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Nastavení Point-to-site v bráně virtuální sítě Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="1a099-221">Integrace aplikace Azure App Service s hybridní sítí</span><span class="sxs-lookup"><span data-stu-id="1a099-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="1a099-222">Pokud chcete připojit aplikaci k virtuální síti Azure, postupujte podle pokynů v části [Brána požadovaná integrace virtuální](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)sítě.</span><span class="sxs-lookup"><span data-stu-id="1a099-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="1a099-223">Přejít na **Nastavení** pro App Service plán hostování webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="1a099-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="1a099-224">V **Nastavení**vyberte **sítě**.</span><span class="sxs-lookup"><span data-stu-id="1a099-224">In **Settings**, select **Networking**.</span></span>

    ![Konfigurace sítě pro plán App Service](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="1a099-226">V **Integrace virtuální sítě** **pro správu vyberte kliknutím sem**.</span><span class="sxs-lookup"><span data-stu-id="1a099-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Správa integrace virtuální sítě pro plán App Service](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="1a099-228">Vyberte virtuální síť, kterou chcete nakonfigurovat.</span><span class="sxs-lookup"><span data-stu-id="1a099-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="1a099-229">V části **IP adresy SMĚROVANÉ do virtuální**sítě zadejte rozsah IP adres pro virtuální síť Azure, virtuální síť centra Azure Stack a adresní prostory Point-to-site.</span><span class="sxs-lookup"><span data-stu-id="1a099-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="1a099-230">Vyberte **Uložit** a ověřte a uložte tato nastavení.</span><span class="sxs-lookup"><span data-stu-id="1a099-230">Select **Save** to validate and save these settings.</span></span>

    ![Rozsahy IP adres, které se mají směrovat v Virtual Network Integration](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="1a099-232">Další informace o tom, jak se App Service integruje s Azure virtuální sítě, najdete v tématu [integrace aplikace s využitím azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="1a099-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="1a099-233">Konfigurace virtuální sítě centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="1a099-234">Brána místní sítě ve virtuální síti centra Azure Stack musí být nakonfigurovaná tak, aby směrovala provoz z rozsahu adres App Service Point-to-site.</span><span class="sxs-lookup"><span data-stu-id="1a099-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="1a099-235">V Azure Stackovém centru, přejít na **bránu místní sítě**.</span><span class="sxs-lookup"><span data-stu-id="1a099-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="1a099-236">V části **Nastavení** vyberte **Konfigurace**.</span><span class="sxs-lookup"><span data-stu-id="1a099-236">Under **Settings**, select **Configuration**.</span></span>

    ![Možnost konfigurace brány v bráně místní sítě centra Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="1a099-238">Do pole **adresní prostor**zadejte rozsah adres Point-to-site pro bránu virtuální sítě v Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Adresní prostor Point-to-site v bráně místní sítě centra Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="1a099-240">Vyberte **Uložit** a ověřte a uložte konfiguraci.</span><span class="sxs-lookup"><span data-stu-id="1a099-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="1a099-241">Konfigurace DNS pro škálování mezi cloudy</span><span class="sxs-lookup"><span data-stu-id="1a099-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="1a099-242">Díky správné konfiguraci DNS pro různé cloudové aplikace můžou uživatelé přistupovat k globálním instancím Azure a Azure Stack hub vaší webové aplikace.</span><span class="sxs-lookup"><span data-stu-id="1a099-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="1a099-243">Konfigurace DNS pro tento kurz také umožňuje službě Azure Traffic Manager směrovat provoz, když se zatížení zvýší nebo sníží.</span><span class="sxs-lookup"><span data-stu-id="1a099-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="1a099-244">Tento kurz používá Azure DNS ke správě DNS, protože App Service domény nebudou fungovat.</span><span class="sxs-lookup"><span data-stu-id="1a099-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="1a099-245">Vytvořit subdomény</span><span class="sxs-lookup"><span data-stu-id="1a099-245">Create subdomains</span></span>

<span data-ttu-id="1a099-246">Vzhledem k tomu, že Traffic Manager spoléhá na záznamy CNAME DNS, je pro správné směrování provozu do koncových bodů nutná subdoména.</span><span class="sxs-lookup"><span data-stu-id="1a099-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="1a099-247">Další informace o záznamech DNS a mapování domén najdete v tématu [mapování domén pomocí Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="1a099-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="1a099-248">Pro koncový bod Azure vytvoříte subdoménu, kterou můžou uživatelé používat pro přístup k vaší webové aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1a099-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="1a099-249">Pro tento kurz můžete použít **App.Northwind.com**, ale tuto hodnotu byste měli přizpůsobit na základě vaší vlastní domény.</span><span class="sxs-lookup"><span data-stu-id="1a099-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="1a099-250">Také budete muset vytvořit subdoménu se záznamem A pro koncový bod centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="1a099-251">Můžete použít **azurestack.Northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="1a099-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="1a099-252">Konfigurace vlastní domény v Azure</span><span class="sxs-lookup"><span data-stu-id="1a099-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="1a099-253">Přidejte název hostitele **App.Northwind.com** do webové aplikace Azure tak, že [namapujete CNAME na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="1a099-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="1a099-254">Konfigurace vlastních domén v centru Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="1a099-255">Přidejte název hostitele **azurestack.Northwind.com** do webové aplikace centra Azure Stack tak, že [namapujete záznam a na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="1a099-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="1a099-256">Pro aplikaci App Service použijte IP adresu, kterou lze směrovat na Internet.</span><span class="sxs-lookup"><span data-stu-id="1a099-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="1a099-257">Přidejte název hostitele **App.Northwind.com** do webové aplikace centra Azure Stack tak, že [namapujete CNAME na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="1a099-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="1a099-258">Jako cíl pro záznam CNAME použijte název hostitele, který jste nakonfigurovali v předchozím kroku (1).</span><span class="sxs-lookup"><span data-stu-id="1a099-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="1a099-259">Konfigurace certifikátů SSL pro škálování mezi cloudy</span><span class="sxs-lookup"><span data-stu-id="1a099-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="1a099-260">Je důležité zajistit, aby citlivá data shromažďovaná vaší webovou aplikací byla zabezpečená při přenosu do a při ukládání do databáze SQL.</span><span class="sxs-lookup"><span data-stu-id="1a099-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="1a099-261">Webové aplikace Azure a centra Azure Stack nakonfigurujete tak, aby používaly certifikáty SSL pro všechny příchozí přenosy.</span><span class="sxs-lookup"><span data-stu-id="1a099-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="1a099-262">Přidání SSL do Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1a099-263">Přidání protokolu SSL do Azure:</span><span class="sxs-lookup"><span data-stu-id="1a099-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="1a099-264">Ujistěte se, že certifikát SSL, který získáte, je platný pro subdoménu, kterou jste vytvořili.</span><span class="sxs-lookup"><span data-stu-id="1a099-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="1a099-265">(Používání certifikátů se zástupnými znaky je v pořádku.)</span><span class="sxs-lookup"><span data-stu-id="1a099-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="1a099-266">V Azure postupujte podle pokynů v části **Příprava vaší webové aplikace** a **vytvoření vazby certifikátu SSL** ve [vazbě existujícího vlastního certifikátu SSL k Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) článku.</span><span class="sxs-lookup"><span data-stu-id="1a099-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="1a099-267">Jako **typ SSL**vyberte **SSL založené na sni** .</span><span class="sxs-lookup"><span data-stu-id="1a099-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="1a099-268">Přesměrujte veškerý provoz na port HTTPS.</span><span class="sxs-lookup"><span data-stu-id="1a099-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="1a099-269">Postupujte podle pokynů v části **vyhovět protokolu HTTPS** v tématu [vytvoření vazby EXISTUJÍCÍHO vlastního certifikátu SSL k Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) článku.</span><span class="sxs-lookup"><span data-stu-id="1a099-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="1a099-270">Postup přidání protokolu SSL do centra Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="1a099-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="1a099-271">Opakujte kroky 1-3, které jste použili pro Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="1a099-272">Konfigurace a nasazení webové aplikace</span><span class="sxs-lookup"><span data-stu-id="1a099-272">Configure and deploy the web app</span></span>

<span data-ttu-id="1a099-273">Nakonfigurujete kód aplikace pro hlášení telemetrie na správnou instanci Application Insights a nakonfigurujete webové aplikace pomocí správných připojovacích řetězců.</span><span class="sxs-lookup"><span data-stu-id="1a099-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="1a099-274">Další informace o Application Insights najdete v tématu [co je Application Insights?](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="1a099-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="1a099-275">Přidat Application Insights</span><span class="sxs-lookup"><span data-stu-id="1a099-275">Add Application Insights</span></span>

1. <span data-ttu-id="1a099-276">Otevřete webovou aplikaci v Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="1a099-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="1a099-277">[Přidejte Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) do projektu pro přenos telemetrie, kterou Application Insights používá k vytvoření výstrah při zvyšování nebo snižování webového provozu.</span><span class="sxs-lookup"><span data-stu-id="1a099-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="1a099-278">Konfigurace dynamických připojovacích řetězců</span><span class="sxs-lookup"><span data-stu-id="1a099-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="1a099-279">Každá instance webové aplikace bude používat pro připojení k databázi SQL jinou metodu.</span><span class="sxs-lookup"><span data-stu-id="1a099-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="1a099-280">Aplikace v Azure používá privátní IP adresu SQL Server virtuálního počítače a aplikace v centru Azure Stack používá veřejnou IP adresu SQL Server virtuálního počítače.</span><span class="sxs-lookup"><span data-stu-id="1a099-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="1a099-281">V integrovaném systému Azure Stack hub by veřejná IP adresa neměla být směrovatelný z Internetu.</span><span class="sxs-lookup"><span data-stu-id="1a099-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="1a099-282">V ASDK se veřejná IP adresa nesměrovatelný mimo ASDK.</span><span class="sxs-lookup"><span data-stu-id="1a099-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="1a099-283">Proměnné prostředí App Service můžete použít k předání jiného připojovacího řetězce do každé instance aplikace.</span><span class="sxs-lookup"><span data-stu-id="1a099-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="1a099-284">Otevřete aplikaci v aplikaci Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="1a099-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="1a099-285">Otevřete Startup.cs a vyhledejte následující blok kódu:</span><span class="sxs-lookup"><span data-stu-id="1a099-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="1a099-286">Nahraďte předchozí blok kódu následujícím kódem, který používá připojovací řetězec definovaný v *appsettings.js* souboru:</span><span class="sxs-lookup"><span data-stu-id="1a099-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="1a099-287">Konfigurovat nastavení aplikace App Service</span><span class="sxs-lookup"><span data-stu-id="1a099-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="1a099-288">Vytvořte připojovací řetězce pro Azure a centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="1a099-289">Řetězce by měly být stejné, s výjimkou používaných IP adres.</span><span class="sxs-lookup"><span data-stu-id="1a099-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="1a099-290">V Azure a centra Azure Stack přidejte příslušný připojovací řetězec [jako nastavení aplikace](/azure/app-service/web-sites-configure) ve webové aplikaci, a to pomocí `SQLCONNSTR\_` předpony v názvu.</span><span class="sxs-lookup"><span data-stu-id="1a099-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="1a099-291">**Uložte** nastavení webové aplikace a restartujte aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1a099-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="1a099-292">Povolit automatické škálování v globálním Azure</span><span class="sxs-lookup"><span data-stu-id="1a099-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="1a099-293">Při vytváření webové aplikace v prostředí App Service se spustí s jednou instancí.</span><span class="sxs-lookup"><span data-stu-id="1a099-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="1a099-294">Automatické horizontální navýšení kapacity můžete provést tak, že přidáte instance pro zajištění dalších výpočetních prostředků pro vaši aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1a099-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="1a099-295">Podobně můžete automaticky škálovat a snížit počet instancí, které vaše aplikace potřebuje.</span><span class="sxs-lookup"><span data-stu-id="1a099-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="1a099-296">Abyste mohli nakonfigurovat horizontální navýšení kapacity a škálování, musíte mít App Service plán.</span><span class="sxs-lookup"><span data-stu-id="1a099-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="1a099-297">Pokud nemáte plán, vytvořte ho před spuštěním dalších kroků.</span><span class="sxs-lookup"><span data-stu-id="1a099-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="1a099-298">Povolit automatické horizontální navýšení kapacity</span><span class="sxs-lookup"><span data-stu-id="1a099-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="1a099-299">V Azure Najděte App Service plán pro lokality, pro které chcete škálovat kapacitu, a pak vyberte škálování na více instancí **(App Service plán)**.</span><span class="sxs-lookup"><span data-stu-id="1a099-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Horizontální navýšení kapacity Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="1a099-301">Vyberte **Povolit automatické škálování**.</span><span class="sxs-lookup"><span data-stu-id="1a099-301">Select **Enable autoscale**.</span></span>

    ![Povolit automatické škálování v Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="1a099-303">Zadejte název pro **název nastavení automatického škálování**.</span><span class="sxs-lookup"><span data-stu-id="1a099-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="1a099-304">Pro **výchozí** pravidlo automatického škálování vyberte škálovat na **základě metriky**.</span><span class="sxs-lookup"><span data-stu-id="1a099-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="1a099-305">Nastavte **limity instancí** na **minimum: 1**, **maximum: 10**a **Výchozí hodnota: 1**.</span><span class="sxs-lookup"><span data-stu-id="1a099-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Konfigurace automatického škálování v Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="1a099-307">Vyberte **+ Přidat pravidlo**.</span><span class="sxs-lookup"><span data-stu-id="1a099-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="1a099-308">Ve **zdroji metriky**vyberte **aktuální prostředek**.</span><span class="sxs-lookup"><span data-stu-id="1a099-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="1a099-309">Pro pravidlo použijte následující kritéria a akce.</span><span class="sxs-lookup"><span data-stu-id="1a099-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="1a099-310">Kritéria</span><span class="sxs-lookup"><span data-stu-id="1a099-310">Criteria</span></span>

1. <span data-ttu-id="1a099-311">V části **Časová agregace** vyberte **průměr**.</span><span class="sxs-lookup"><span data-stu-id="1a099-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="1a099-312">V části **název metriky**vyberte **Procento procesoru**.</span><span class="sxs-lookup"><span data-stu-id="1a099-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="1a099-313">V části **operátor**vyberte **větší než**.</span><span class="sxs-lookup"><span data-stu-id="1a099-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="1a099-314">Nastavte **prahovou hodnotu** na **50**.</span><span class="sxs-lookup"><span data-stu-id="1a099-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="1a099-315">Nastavte **dobu trvání** na **10**.</span><span class="sxs-lookup"><span data-stu-id="1a099-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="1a099-316">Akce</span><span class="sxs-lookup"><span data-stu-id="1a099-316">Action</span></span>

1. <span data-ttu-id="1a099-317">V části **operace**vyberte **zvýšit počet o**.</span><span class="sxs-lookup"><span data-stu-id="1a099-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="1a099-318">Nastavte **počet instancí** na **2**.</span><span class="sxs-lookup"><span data-stu-id="1a099-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="1a099-319">Nastavte **vychladnutí dolů** na **5**.</span><span class="sxs-lookup"><span data-stu-id="1a099-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="1a099-320">Vyberte **Přidat**.</span><span class="sxs-lookup"><span data-stu-id="1a099-320">Select **Add**.</span></span>

5. <span data-ttu-id="1a099-321">Vyberte **+ Přidat pravidlo**.</span><span class="sxs-lookup"><span data-stu-id="1a099-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="1a099-322">Ve **zdroji metriky**vyberte **aktuální prostředek.**</span><span class="sxs-lookup"><span data-stu-id="1a099-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="1a099-323">Aktuální prostředek bude obsahovat název nebo identifikátor GUID vašeho plánu App Service a rozevírací seznamy pro **typ prostředku** a **prostředek** nebudou k dispozici.</span><span class="sxs-lookup"><span data-stu-id="1a099-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="1a099-324">Povolit automatické škálování v</span><span class="sxs-lookup"><span data-stu-id="1a099-324">Enable automatic scale in</span></span>

<span data-ttu-id="1a099-325">Při snížení provozu může webová aplikace Azure automaticky snížit počet aktivních instancí, aby se snížily náklady.</span><span class="sxs-lookup"><span data-stu-id="1a099-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="1a099-326">Tato akce je méně agresivní než horizontální navýšení kapacity a minimalizuje dopad na uživatele aplikace.</span><span class="sxs-lookup"><span data-stu-id="1a099-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="1a099-327">Přejít do **výchozí** podmínky horizontálního navýšení kapacity a pak vybrat **+ Přidat pravidlo**.</span><span class="sxs-lookup"><span data-stu-id="1a099-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="1a099-328">Pro pravidlo použijte následující kritéria a akce.</span><span class="sxs-lookup"><span data-stu-id="1a099-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="1a099-329">Kritéria</span><span class="sxs-lookup"><span data-stu-id="1a099-329">Criteria</span></span>

1. <span data-ttu-id="1a099-330">V části **Časová agregace** vyberte **průměr**.</span><span class="sxs-lookup"><span data-stu-id="1a099-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="1a099-331">V části **název metriky**vyberte **Procento procesoru**.</span><span class="sxs-lookup"><span data-stu-id="1a099-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="1a099-332">V části **operátor**vyberte **menší než**.</span><span class="sxs-lookup"><span data-stu-id="1a099-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="1a099-333">Nastavte **prahovou hodnotu** na **30**.</span><span class="sxs-lookup"><span data-stu-id="1a099-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="1a099-334">Nastavte **dobu trvání** na **10**.</span><span class="sxs-lookup"><span data-stu-id="1a099-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="1a099-335">Akce</span><span class="sxs-lookup"><span data-stu-id="1a099-335">Action</span></span>

1. <span data-ttu-id="1a099-336">V části **operace**vyberte **snížit počet o**.</span><span class="sxs-lookup"><span data-stu-id="1a099-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="1a099-337">Nastavte **počet instancí** na **1**.</span><span class="sxs-lookup"><span data-stu-id="1a099-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="1a099-338">Nastavte **vychladnutí dolů** na **5**.</span><span class="sxs-lookup"><span data-stu-id="1a099-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="1a099-339">Vyberte **Přidat**.</span><span class="sxs-lookup"><span data-stu-id="1a099-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="1a099-340">Vytvoření profilu Traffic Manager a konfigurace škálování mezi cloudy</span><span class="sxs-lookup"><span data-stu-id="1a099-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="1a099-341">Vytvořte v Azure profil Traffic Manager a pak nakonfigurujte koncové body, aby se povolilo škálování mezi cloudy.</span><span class="sxs-lookup"><span data-stu-id="1a099-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="1a099-342">Vytvořit profil Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="1a099-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="1a099-343">Vyberte **Vytvořit prostředek**.</span><span class="sxs-lookup"><span data-stu-id="1a099-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="1a099-344">Vyberte **Sítě**.</span><span class="sxs-lookup"><span data-stu-id="1a099-344">Select **Networking**.</span></span>
3. <span data-ttu-id="1a099-345">Vyberte **profil Traffic Manager** a nakonfigurujte následující nastavení:</span><span class="sxs-lookup"><span data-stu-id="1a099-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="1a099-346">Do **název**zadejte název profilu.</span><span class="sxs-lookup"><span data-stu-id="1a099-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="1a099-347">Tento název **musí** být v zóně trafficmanager.NET jedinečný a používá se k vytvoření nového názvu DNS (například northwindstore.trafficmanager.NET).</span><span class="sxs-lookup"><span data-stu-id="1a099-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="1a099-348">U **metody směrování**vyberte **Vážená**.</span><span class="sxs-lookup"><span data-stu-id="1a099-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="1a099-349">V části **předplatné**vyberte předplatné, ve kterém chcete vytvořit tento profil.</span><span class="sxs-lookup"><span data-stu-id="1a099-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="1a099-350">V rámci **skupiny prostředků**vytvořte novou skupinu prostředků pro tento profil.</span><span class="sxs-lookup"><span data-stu-id="1a099-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="1a099-351">V poli **Umístění skupiny prostředků** vyberte umístění skupiny prostředků.</span><span class="sxs-lookup"><span data-stu-id="1a099-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="1a099-352">Toto nastavení odkazuje na umístění skupiny prostředků a nemá žádný vliv na profil Traffic Manager, který se globálně nasazuje.</span><span class="sxs-lookup"><span data-stu-id="1a099-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="1a099-353">Vyberte **Vytvořit**.</span><span class="sxs-lookup"><span data-stu-id="1a099-353">Select **Create**.</span></span>

    ![Vytvořit profil Traffic Manager](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="1a099-355">Po dokončení globálního nasazení profilu Traffic Manager se zobrazí v seznamu prostředků pro skupinu prostředků, ve které jste ji vytvořili.</span><span class="sxs-lookup"><span data-stu-id="1a099-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="1a099-356">Přidání koncových bodů služby Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="1a099-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="1a099-357">Vyhledejte profil Traffic Manager, který jste vytvořili.</span><span class="sxs-lookup"><span data-stu-id="1a099-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="1a099-358">Pokud jste přešli na skupinu prostředků pro daný profil, vyberte profil.</span><span class="sxs-lookup"><span data-stu-id="1a099-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="1a099-359">V části **profil Traffic Manager**v části **Nastavení**vyberte **koncové body**.</span><span class="sxs-lookup"><span data-stu-id="1a099-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="1a099-360">Vyberte **Přidat**.</span><span class="sxs-lookup"><span data-stu-id="1a099-360">Select **Add**.</span></span>

4. <span data-ttu-id="1a099-361">V části **přidat koncový bod**použijte pro Azure Stack centrum následující nastavení:</span><span class="sxs-lookup"><span data-stu-id="1a099-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="1a099-362">Jako **typ**vyberte **externí koncový bod**.</span><span class="sxs-lookup"><span data-stu-id="1a099-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="1a099-363">Zadejte **název** koncového bodu.</span><span class="sxs-lookup"><span data-stu-id="1a099-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="1a099-364">Pro **plně kvalifikovaný název domény (FQDN) nebo IP**adresu zadejte externí adresu URL webové aplikace centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="1a099-365">Pro **váhu**ponechte výchozí hodnotu **1**.</span><span class="sxs-lookup"><span data-stu-id="1a099-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="1a099-366">Tato váha má za následek veškerý provoz směřující do tohoto koncového bodu, pokud je v pořádku.</span><span class="sxs-lookup"><span data-stu-id="1a099-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="1a099-367">Nechejte položku **Přidat jako zakázanou** nezaškrtnutou.</span><span class="sxs-lookup"><span data-stu-id="1a099-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="1a099-368">Výběrem **OK** uložte koncový bod centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="1a099-369">Potom nakonfigurujete koncový bod Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="1a099-370">V **Traffic Manager profil**vyberte **koncové body**.</span><span class="sxs-lookup"><span data-stu-id="1a099-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="1a099-371">Vyberte **+ Přidat**.</span><span class="sxs-lookup"><span data-stu-id="1a099-371">Select **+Add**.</span></span>
3. <span data-ttu-id="1a099-372">Na stránce **přidat koncový bod**použijte následující nastavení pro Azure:</span><span class="sxs-lookup"><span data-stu-id="1a099-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="1a099-373">Jako **typ**vyberte **koncový bod Azure**.</span><span class="sxs-lookup"><span data-stu-id="1a099-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="1a099-374">Zadejte **název** koncového bodu.</span><span class="sxs-lookup"><span data-stu-id="1a099-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="1a099-375">Jako **typ cílového prostředku**vyberte **App Service**.</span><span class="sxs-lookup"><span data-stu-id="1a099-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="1a099-376">V části **cílový prostředek**vyberte **možnost zvolit službu App Service** , ve které se zobrazí seznam Web Apps ve stejném předplatném.</span><span class="sxs-lookup"><span data-stu-id="1a099-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="1a099-377">V části **Prostředek** vyberte službu App Service, kterou chcete přidat jako první koncový bod.</span><span class="sxs-lookup"><span data-stu-id="1a099-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="1a099-378">V případě **váhy**vyberte **2**.</span><span class="sxs-lookup"><span data-stu-id="1a099-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="1a099-379">Toto nastavení způsobí, že veškerý provoz směřující do tohoto koncového bodu je v případě, že primární koncový bod není v pořádku, nebo pokud máte pravidlo nebo výstrahu, která přesměruje provoz, když se aktivuje.</span><span class="sxs-lookup"><span data-stu-id="1a099-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="1a099-380">Nechejte položku **Přidat jako zakázanou** nezaškrtnutou.</span><span class="sxs-lookup"><span data-stu-id="1a099-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="1a099-381">Výběrem **OK** uložte koncový bod Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="1a099-382">Po nakonfigurování obou koncových bodů jsou uvedeny v **Traffic Manager profilu** při výběru **koncových bodů**.</span><span class="sxs-lookup"><span data-stu-id="1a099-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="1a099-383">Příklad na následujícím snímku obrazovky ukazuje dva koncové body s informacemi o stavu a konfiguraci pro každé z nich.</span><span class="sxs-lookup"><span data-stu-id="1a099-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Koncové body v profilu Traffic Manager](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="1a099-385">Nastavení Application Insights monitorování a upozorňování</span><span class="sxs-lookup"><span data-stu-id="1a099-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="1a099-386">Azure Application Insights umožňuje monitorovat aplikaci a odesílat výstrahy na základě podmínek, které nakonfigurujete.</span><span class="sxs-lookup"><span data-stu-id="1a099-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="1a099-387">Mezi příklady patří: aplikace není k dispozici, dochází k chybám nebo zobrazuje problémy s výkonem.</span><span class="sxs-lookup"><span data-stu-id="1a099-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="1a099-388">K vytváření výstrah použijete Application Insights metriky.</span><span class="sxs-lookup"><span data-stu-id="1a099-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="1a099-389">Při aktivaci těchto výstrah se instance webové aplikace automaticky přepne z centra Azure Stack do Azure pro horizontální navýšení kapacity a potom zpátky na Azure Stack centra pro horizontální navýšení kapacity.</span><span class="sxs-lookup"><span data-stu-id="1a099-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="1a099-390">Vytvoření výstrahy z metriky</span><span class="sxs-lookup"><span data-stu-id="1a099-390">Create an alert from metrics</span></span>

<span data-ttu-id="1a099-391">Pro tento kurz vyberte skupinu prostředků a pak **Application Insights**otevřete tak, že vyberete instanci Application Insights.</span><span class="sxs-lookup"><span data-stu-id="1a099-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="1a099-393">Pomocí tohoto zobrazení můžete vytvořit upozornění na horizontální navýšení kapacity a upozornění na horizontální navýšení kapacity.</span><span class="sxs-lookup"><span data-stu-id="1a099-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="1a099-394">Vytvoření upozornění na horizontální navýšení kapacity</span><span class="sxs-lookup"><span data-stu-id="1a099-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="1a099-395">V části **Konfigurovat**vyberte **výstrahy (klasické)**.</span><span class="sxs-lookup"><span data-stu-id="1a099-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="1a099-396">Vyberte **Přidat upozornění na metriku (Classic)**.</span><span class="sxs-lookup"><span data-stu-id="1a099-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="1a099-397">V části **Přidat pravidlo**nakonfigurujte následující nastavení:</span><span class="sxs-lookup"><span data-stu-id="1a099-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="1a099-398">Jako **název**zadejte **nárůst do cloudu Azure**.</span><span class="sxs-lookup"><span data-stu-id="1a099-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="1a099-399">**Popis** je volitelný.</span><span class="sxs-lookup"><span data-stu-id="1a099-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="1a099-400">V **Source**části  >  **Výstraha**zdrojového kódu **vyberte metriky**.</span><span class="sxs-lookup"><span data-stu-id="1a099-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="1a099-401">V části **kritéria**vyberte své předplatné, skupinu prostředků pro profil Traffic Manager a název profilu Traffic Manager pro daný prostředek.</span><span class="sxs-lookup"><span data-stu-id="1a099-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="1a099-402">Jako **metrika**vyberte **rychlost požadavků**.</span><span class="sxs-lookup"><span data-stu-id="1a099-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="1a099-403">V případě **podmínky**vyberte **větší než**.</span><span class="sxs-lookup"><span data-stu-id="1a099-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="1a099-404">V případě **prahové hodnoty**zadejte **2**.</span><span class="sxs-lookup"><span data-stu-id="1a099-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="1a099-405">Jako **periodu**vyberte **za posledních 5 minut**.</span><span class="sxs-lookup"><span data-stu-id="1a099-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="1a099-406">V části **oznamovat prostřednictvím**:</span><span class="sxs-lookup"><span data-stu-id="1a099-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="1a099-407">Zaškrtněte políčko pro **vlastníky e-mailů, přispěvatele a čtenáře**.</span><span class="sxs-lookup"><span data-stu-id="1a099-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="1a099-408">Zadejte svou e- **mailovou adresu pro další e-maily správce**.</span><span class="sxs-lookup"><span data-stu-id="1a099-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="1a099-409">Na panelu nabídek vyberte **Uložit**.</span><span class="sxs-lookup"><span data-stu-id="1a099-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="1a099-410">Vytvoření upozornění na horizontální navýšení kapacity</span><span class="sxs-lookup"><span data-stu-id="1a099-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="1a099-411">V části **Konfigurovat**vyberte **výstrahy (klasické)**.</span><span class="sxs-lookup"><span data-stu-id="1a099-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="1a099-412">Vyberte **Přidat upozornění na metriku (Classic)**.</span><span class="sxs-lookup"><span data-stu-id="1a099-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="1a099-413">V části **Přidat pravidlo**nakonfigurujte následující nastavení:</span><span class="sxs-lookup"><span data-stu-id="1a099-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="1a099-414">Jako **název**zadejte horizontální navýšení **kapacity zpátky do centra Azure Stack**.</span><span class="sxs-lookup"><span data-stu-id="1a099-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="1a099-415">**Popis** je volitelný.</span><span class="sxs-lookup"><span data-stu-id="1a099-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="1a099-416">V **Source**části  >  **Výstraha**zdrojového kódu **vyberte metriky**.</span><span class="sxs-lookup"><span data-stu-id="1a099-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="1a099-417">V části **kritéria**vyberte své předplatné, skupinu prostředků pro profil Traffic Manager a název profilu Traffic Manager pro daný prostředek.</span><span class="sxs-lookup"><span data-stu-id="1a099-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="1a099-418">Jako **metrika**vyberte **rychlost požadavků**.</span><span class="sxs-lookup"><span data-stu-id="1a099-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="1a099-419">V případě **podmínky**vyberte **méně než**.</span><span class="sxs-lookup"><span data-stu-id="1a099-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="1a099-420">V případě **prahové hodnoty**zadejte **2**.</span><span class="sxs-lookup"><span data-stu-id="1a099-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="1a099-421">Jako **periodu**vyberte **za posledních 5 minut**.</span><span class="sxs-lookup"><span data-stu-id="1a099-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="1a099-422">V části **oznamovat prostřednictvím**:</span><span class="sxs-lookup"><span data-stu-id="1a099-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="1a099-423">Zaškrtněte políčko pro **vlastníky e-mailů, přispěvatele a čtenáře**.</span><span class="sxs-lookup"><span data-stu-id="1a099-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="1a099-424">Zadejte svou e- **mailovou adresu pro další e-maily správce**.</span><span class="sxs-lookup"><span data-stu-id="1a099-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="1a099-425">Na panelu nabídek vyberte **Uložit**.</span><span class="sxs-lookup"><span data-stu-id="1a099-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="1a099-426">Na následujícím snímku obrazovky vidíte výstrahy pro škálování a škálování.</span><span class="sxs-lookup"><span data-stu-id="1a099-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Výstrahy Application Insights (klasické)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="1a099-428">Přesměrování provozu mezi Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1a099-429">Můžete nakonfigurovat ruční nebo automatické přepínání provozu webové aplikace mezi Azure a Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="1a099-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="1a099-430">Konfigurace ručního přepínání mezi Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1a099-431">Když váš web dosáhne prahových hodnot, které nakonfigurujete, zobrazí se upozornění.</span><span class="sxs-lookup"><span data-stu-id="1a099-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="1a099-432">K ručnímu přesměrování provozu do Azure použijte následující postup.</span><span class="sxs-lookup"><span data-stu-id="1a099-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="1a099-433">V Azure Portal vyberte svůj profil Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="1a099-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Traffic Manager koncové body v Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="1a099-435">Vyberte **koncové body**.</span><span class="sxs-lookup"><span data-stu-id="1a099-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="1a099-436">Vyberte **koncový bod Azure**.</span><span class="sxs-lookup"><span data-stu-id="1a099-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="1a099-437">V části **stav**vyberte **povoleno**a pak vyberte **Uložit**.</span><span class="sxs-lookup"><span data-stu-id="1a099-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Povolit koncový bod Azure v Azure Portal](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="1a099-439">V části **koncové body** profilu Traffic Manager vyberte **externí koncový bod**.</span><span class="sxs-lookup"><span data-stu-id="1a099-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="1a099-440">V části **stav**vyberte **zakázáno**a pak vyberte **Uložit**.</span><span class="sxs-lookup"><span data-stu-id="1a099-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Zakázat koncový bod centra Azure Stack v Azure Portal](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="1a099-442">Po nakonfigurování koncových bodů přejde provoz aplikace do webové aplikace se škálováním na více instancí místo webové aplikace centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Změny koncových bodů v provozu webové aplikace Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="1a099-444">Pokud chcete tok vrátit zpátky do centra Azure Stack, použijte k těmto akcím předchozí kroky:</span><span class="sxs-lookup"><span data-stu-id="1a099-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="1a099-445">Povolte koncový bod centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="1a099-446">Zakažte koncový bod Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="1a099-447">Konfigurace automatického přepínání mezi Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1a099-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1a099-448">Application Insights monitorování můžete použít také v případě, že aplikace běží v prostředí bez [serveru](https://azure.microsoft.com/overview/serverless-computing/) , které poskytuje Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="1a099-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="1a099-449">V tomto scénáři můžete nakonfigurovat Application Insights pro použití Webhooku, který volá aplikaci Function App.</span><span class="sxs-lookup"><span data-stu-id="1a099-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="1a099-450">Tato aplikace automaticky povolí nebo zakáže koncový bod v reakci na výstrahu.</span><span class="sxs-lookup"><span data-stu-id="1a099-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="1a099-451">Pomocí následujících kroků můžete nakonfigurovat automatické přepínání provozu.</span><span class="sxs-lookup"><span data-stu-id="1a099-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="1a099-452">Vytvoření aplikace Azure Function App</span><span class="sxs-lookup"><span data-stu-id="1a099-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="1a099-453">Vytvoření funkce aktivované protokolem HTTP</span><span class="sxs-lookup"><span data-stu-id="1a099-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="1a099-454">Importujte sady Azure SDK pro Správce prostředků, Web Apps a Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="1a099-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="1a099-455">Vývoj kódu pro:</span><span class="sxs-lookup"><span data-stu-id="1a099-455">Develop code to:</span></span>

   - <span data-ttu-id="1a099-456">Ověřte si předplatné Azure.</span><span class="sxs-lookup"><span data-stu-id="1a099-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="1a099-457">Použijte parametr, který přepíná Traffic Manager koncové body pro směrování provozu do Azure nebo centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1a099-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="1a099-458">Uložte svůj kód a přidejte adresu URL aplikace Function App s příslušnými parametry do oddílu **Webhooku** nastavení pravidla výstrahy Application Insights.</span><span class="sxs-lookup"><span data-stu-id="1a099-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="1a099-459">Provoz se automaticky přesměruje, když se aktivuje výstraha Application Insights.</span><span class="sxs-lookup"><span data-stu-id="1a099-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1a099-460">Další kroky</span><span class="sxs-lookup"><span data-stu-id="1a099-460">Next steps</span></span>

- <span data-ttu-id="1a099-461">Další informace o vzorech cloudu Azure najdete v tématu [vzory návrhu cloudu](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="1a099-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
