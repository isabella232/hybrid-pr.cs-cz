---
title: Nasazení řešení pro detekci Footfall založené na AI v Azure a centra Azure Stack
description: Naučte se, jak nasadit řešení pro detekci Footfall založené na AI pro analýzu provozu návštěvníků v prodejnách pomocí Azure a centra Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5f2e18e164e54f60b1bb7a14026a0c75c7d7ce69
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477163"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="1fa4b-103">Nasazení řešení pro detekci Footfall založené na AI pomocí Azure a centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="1fa4b-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="1fa4b-104">Tento článek popisuje, jak nasadit řešení založené na AI, které generuje přehledy z reálných akcí pomocí Azure, centra Azure Stack a Custom Vision AI dev Kit.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="1fa4b-105">V tomto řešení se dozvíte, jak:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="1fa4b-106">Nasaďte na hranici cloudové nativní sady aplikací (CNAB).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="1fa4b-107">Nasaďte aplikaci, která zahrnuje hranice cloudu.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="1fa4b-108">Pro odvození na hranici použijte Custom Vision AI dev Kit.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="1fa4b-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="1fa4b-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="1fa4b-110">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="1fa4b-111">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="1fa4b-112">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="1fa4b-113">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="1fa4b-114">Požadavky</span><span class="sxs-lookup"><span data-stu-id="1fa4b-114">Prerequisites</span></span>

<span data-ttu-id="1fa4b-115">Než začnete s tímto průvodcem nasazením, nezapomeňte:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="1fa4b-116">Projděte si téma [Footfall Detection Pattern](pattern-retail-footfall-detection.md) .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="1fa4b-117">Získejte přístup uživatele k Azure Stack Development Kit (ASDK) nebo k instanci integrovaného systému centra Azure Stack pomocí:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="1fa4b-118">Azure App Service nainstalovaného [poskytovatele prostředků centra Azure Stack](/azure-stack/operator/azure-stack-app-service-overview.md) .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="1fa4b-119">K vaší instanci centra Azure Stack potřebujete přistupovat pomocí operátoru, nebo pokud chcete nainstalovat správce, spolupracujte se správcem.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="1fa4b-120">Předplatné nabídky, které poskytuje App Service a kvótu úložiště.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="1fa4b-121">Chcete-li vytvořit nabídku, potřebujete přístup k operátoru.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="1fa4b-122">Získejte přístup k předplatnému Azure.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="1fa4b-123">Pokud ještě nemáte předplatné Azure, zaregistrujte si [bezplatný zkušební účet](https://azure.microsoft.com/free/) před tím, než začnete.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="1fa4b-124">Vytvořte ve svém adresáři dva instanční objekty:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="1fa4b-125">Jedna nastavená pro použití s prostředky Azure s přístupem v oboru předplatného Azure.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="1fa4b-126">Jedna je nastavená pro použití s prostředky centra Azure Stack s přístupem v oboru předplatného centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="1fa4b-127">Další informace o vytváření instančních objektů a autorizaci přístupu najdete v tématu [použití identity aplikace pro přístup k prostředkům](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="1fa4b-128">Pokud dáváte přednost použití rozhraní příkazového řádku Azure, přečtěte si téma [Vytvoření instančního objektu Azure pomocí Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="1fa4b-129">Nasaďte Azure Cognitive Services v Azure nebo v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="1fa4b-130">Nejprve [si přečtěte další informace o Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="1fa4b-131">Potom navštivte [nasazení služby Azure Cognitive Services, abyste Azure Stack centrum](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) nasadili Cognitive Services v Azure Stackovém centru.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="1fa4b-132">Nejprve se musíte zaregistrovat pro přístup k verzi Preview.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="1fa4b-133">Naklonujte nebo Stáhněte si nenakonfigurovanou sadu Azure Custom Vision AI dev Kit.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="1fa4b-134">Podrobnosti najdete v tématu [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="1fa4b-135">Zaregistrujte si účet Power BI.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="1fa4b-136">Klíč předplatného služby Azure Cognitive Services Face API a adresa URL koncového bodu.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="1fa4b-137">Jak můžete získat [Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) bezplatnou zkušební verzi.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="1fa4b-138">Případně postupujte podle pokynů v části [Vytvoření účtu Cognitive Services](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="1fa4b-139">Nainstalujte následující prostředky pro vývoj:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="1fa4b-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="1fa4b-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="1fa4b-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="1fa4b-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="1fa4b-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="1fa4b-143">Pomocí Porter můžete nasazovat cloudové aplikace s využitím manifestů CNAB sad, které jsou pro vás k dispozici.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="1fa4b-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="1fa4b-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="1fa4b-145">Nástroje Azure IoT pro Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="1fa4b-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="1fa4b-146">Rozšíření Pythonu pro Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="1fa4b-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="1fa4b-147">Python</span><span class="sxs-lookup"><span data-stu-id="1fa4b-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="1fa4b-148">Nasazení hybridní cloudové aplikace</span><span class="sxs-lookup"><span data-stu-id="1fa4b-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="1fa4b-149">Nejprve pomocí rozhraní příkazového řádku Porter vygenerujte sadu přihlašovacích údajů a potom nasaďte cloudovou aplikaci.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="1fa4b-150">Naklonujte nebo Stáhněte ukázkový kód řešení z https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="1fa4b-151">Porter vygeneruje sadu přihlašovacích údajů, které budou automatizovat nasazení aplikace.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="1fa4b-152">Před spuštěním příkazu pro generování přihlašovacích údajů se ujistěte, že máte k dispozici následující:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="1fa4b-153">Instanční objekt pro přístup k prostředkům Azure, včetně identifikátoru ID objektu služby, klíče a DNS tenanta.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="1fa4b-154">ID předplatného pro vaše předplatné Azure.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="1fa4b-155">Instanční objekt pro přístup k prostředkům služby Azure Stack hub, včetně ID instančního objektu, klíče a DNS tenanta.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="1fa4b-156">ID předplatného pro vaše předplatné centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="1fa4b-157">Adresa URL vašeho klíče Face API a klíčového bodu prostředku služby Azure Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="1fa4b-158">Spusťte proces generování přihlašovacích údajů Porter a postupujte podle následujících pokynů:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="1fa4b-159">Porter také vyžaduje, aby bylo možné spustit sadu parametrů.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="1fa4b-160">Vytvořte textový soubor parametrů a zadejte následující páry název/hodnota.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="1fa4b-161">Pokud potřebujete pomoc s kteroukoli z požadovaných hodnot, požádejte správce centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="1fa4b-162">`resource suffix`Hodnota se používá k zajištění, že prostředky nasazení mají v rámci Azure jedinečné názvy.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="1fa4b-163">Musí se jednat o jedinečný řetězec písmen a číslic, který nesmí být delší než 8 znaků.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="1fa4b-164">Uložte textový soubor a poznamenejte si jeho cestu.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="1fa4b-165">Teď jste připraveni nasadit hybridní cloudovou aplikaci pomocí Porter.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="1fa4b-166">Spusťte příkaz Install a sledujte, jak se prostředky nasazují do Azure a centra Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="1fa4b-167">Po dokončení nasazení si poznamenejte následující hodnoty:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="1fa4b-168">Připojovací řetězec kamery.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-168">The camera's connection string.</span></span>
    - <span data-ttu-id="1fa4b-169">Připojovací řetězec účtu úložiště imagí</span><span class="sxs-lookup"><span data-stu-id="1fa4b-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="1fa4b-170">Názvy skupin prostředků.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="1fa4b-171">Příprava Custom Vision AI DevKit</span><span class="sxs-lookup"><span data-stu-id="1fa4b-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="1fa4b-172">Dále nastavte Custom Vision AI dev Kit, jak je znázorněno v kurzu [rychlý Start DevKit Vision AI](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="1fa4b-173">Pomocí připojovacího řetězce, který jste zadali v předchozím kroku, můžete také nastavit a otestovat fotoaparát.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="1fa4b-174">Nasazení aplikace kamery</span><span class="sxs-lookup"><span data-stu-id="1fa4b-174">Deploy the camera app</span></span>

<span data-ttu-id="1fa4b-175">Pomocí rozhraní příkazového řádku Porter vygenerujte sadu přihlašovacích údajů a pak nasaďte aplikaci kamery.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="1fa4b-176">Porter vygeneruje sadu přihlašovacích údajů, které budou automatizovat nasazení aplikace.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="1fa4b-177">Před spuštěním příkazu pro generování přihlašovacích údajů se ujistěte, že máte k dispozici následující:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="1fa4b-178">Instanční objekt pro přístup k prostředkům Azure, včetně identifikátoru ID objektu služby, klíče a DNS tenanta.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="1fa4b-179">ID předplatného pro vaše předplatné Azure.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="1fa4b-180">Připojovací řetězec účtu úložiště imagí, který jste zadali při nasazení cloudové aplikace.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="1fa4b-181">Spusťte proces generování přihlašovacích údajů Porter a postupujte podle následujících pokynů:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="1fa4b-182">Porter také vyžaduje, aby bylo možné spustit sadu parametrů.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="1fa4b-183">Vytvořte textový soubor parametrů a zadejte následující text.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="1fa4b-184">Pokud neznáte některé z požadovaných hodnot, požádejte správce centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="1fa4b-185">`deployment suffix`Hodnota se používá k zajištění, že prostředky nasazení mají v rámci Azure jedinečné názvy.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="1fa4b-186">Musí se jednat o jedinečný řetězec písmen a číslic, který nesmí být delší než 8 znaků.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="1fa4b-187">Uložte textový soubor a poznamenejte si jeho cestu.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="1fa4b-188">Nyní jste připraveni nasadit aplikaci kamera pomocí Porter.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="1fa4b-189">Spusťte příkaz Install a sledujte, jak je vytvořeno nasazení IoT Edge.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="1fa4b-190">Ověřte, že se nasazení kamery dokončilo, zobrazením informačního kanálu kamery v `https://<camera-ip>:3000/` umístění, kde `<camara-ip>` se nachází IP adresa kamery.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="1fa4b-191">Tento krok může trvat až 10 minut.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="1fa4b-192">Konfigurace Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="1fa4b-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="1fa4b-193">Teď, když data přecházejí z kamery Azure Stream Analytics, musíme je ručně autorizovat, aby komunikovala s Power BI.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="1fa4b-194">Z Azure Portal otevřete **všechny prostředky**a úlohu \*Process-Footfall \[ yoursuffix \] \* .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="1fa4b-195">V podokně úlohy Stream Analytics v části **Topologie úlohy** vyberte možnost **Výstupy**.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="1fa4b-196">Vyberte výstupní výstupní jímku **výstupních dat** .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="1fa4b-197">Vyberte **obnovit autorizaci** a přihlaste se ke svému účtu Power BI.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Výzva k obnovení autorizace v Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="1fa4b-199">Uložte nastavení výstupu.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-199">Save the output settings.</span></span>

6. <span data-ttu-id="1fa4b-200">V podokně **Přehled** vyberte **začít** a začněte odesílat data do Power BI.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="1fa4b-201">Vyberte **Nyní** pro čas spuštění výstupu úlohy a vyberte **Spustit**.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="1fa4b-202">Stav úlohy můžete sledovat v oznamovacím pruhu.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="1fa4b-203">Vytvoření řídicího panelu Power BI</span><span class="sxs-lookup"><span data-stu-id="1fa4b-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="1fa4b-204">Po úspěšném dokončení úlohy přejdete na [Power BI](https://powerbi.com/) a přihlaste se pomocí svého pracovního nebo školního účtu.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="1fa4b-205">Pokud je v dotazu úlohy Stream Analytics výstupem výsledků, datová sada *Footfall-DataSet* , kterou jste vytvořili, existuje na kartě **datové sady** .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="1fa4b-206">V pracovním prostoru Power BI vyberte **+ vytvořit** a vytvořte nový řídicí panel s názvem *Analýza Footfall.*</span><span class="sxs-lookup"><span data-stu-id="1fa4b-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="1fa4b-207">V horní části okna vyberte **Přidat dlaždici**.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="1fa4b-208">Potom vyberte **Vlastní streamovaná data** a **Další**.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="1fa4b-209">V **datových sadách**vyberte **Footfall-DataSet** .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="1fa4b-210">V rozevíracím seznamu **typ vizualizace** vyberte **karta** a do **polí**přidejte **věk** .</span><span class="sxs-lookup"><span data-stu-id="1fa4b-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="1fa4b-211">Vyberte **Další**, zadejte název dlaždice a pak výběrem možnosti **Použít** dlaždici vytvořte.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="1fa4b-212">Podle potřeby můžete přidat další pole a karty.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="1fa4b-213">Testování řešení</span><span class="sxs-lookup"><span data-stu-id="1fa4b-213">Test Your Solution</span></span>

<span data-ttu-id="1fa4b-214">Sledujte, jak se data na kartách, které jste vytvořili v Power BI mění jako různí lidé před fotoaparátem.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="1fa4b-215">Odvození může trvat až 20 sekund, než se objeví po nahrání.</span><span class="sxs-lookup"><span data-stu-id="1fa4b-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="1fa4b-216">Odebrání řešení</span><span class="sxs-lookup"><span data-stu-id="1fa4b-216">Remove Your Solution</span></span>

<span data-ttu-id="1fa4b-217">Pokud chcete odebrat řešení, spusťte následující příkazy pomocí Porter pomocí stejných souborů parametrů, které jste vytvořili pro nasazení:</span><span class="sxs-lookup"><span data-stu-id="1fa4b-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="1fa4b-218">Další kroky</span><span class="sxs-lookup"><span data-stu-id="1fa4b-218">Next steps</span></span>

- <span data-ttu-id="1fa4b-219">Další informace o [hlediska návrhu hybridní aplikace]. (overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="1fa4b-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="1fa4b-220">Přečtěte si a navrhněte vylepšení [kódu pro tuto ukázku na GitHubu](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="1fa4b-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
