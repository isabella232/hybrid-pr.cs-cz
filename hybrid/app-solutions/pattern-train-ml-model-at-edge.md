---
title: Výuka modelu strojového učení na hraničním vzoru
description: Naučte se, jak na hraničních zařízeních s Azure a Azure Stack hub dělat školení modelu Machine Learning.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910113"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="6fec8-103">Výuka modelu strojového učení na hraničním vzoru</span><span class="sxs-lookup"><span data-stu-id="6fec8-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="6fec8-104">Vygenerujte přenositelné modely strojového učení (ML) z dat, která existují jenom místně.</span><span class="sxs-lookup"><span data-stu-id="6fec8-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6fec8-105">Kontext a problém</span><span class="sxs-lookup"><span data-stu-id="6fec8-105">Context and problem</span></span>

<span data-ttu-id="6fec8-106">Mnoho organizací chce odemknout poznatky z místních nebo starších dat pomocí nástrojů, které jejich odborníci na data znají.</span><span class="sxs-lookup"><span data-stu-id="6fec8-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="6fec8-107">[Azure Machine Learning](/azure/machine-learning/) poskytuje cloudové nativní nástroje pro školení, ladění a nasazení modelů ml a hloubkového učení.</span><span class="sxs-lookup"><span data-stu-id="6fec8-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="6fec8-108">Některá data jsou ale příliš velká pro posílání do cloudu nebo se nedají do cloudu odeslat z důvodů regulativních předpisů.</span><span class="sxs-lookup"><span data-stu-id="6fec8-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="6fec8-109">Pomocí tohoto modelu mohou odborníci přes data využít Azure Machine Learning k učení modelů pomocí místních dat a výpočetních prostředků.</span><span class="sxs-lookup"><span data-stu-id="6fec8-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="6fec8-110">Řešení</span><span class="sxs-lookup"><span data-stu-id="6fec8-110">Solution</span></span>

<span data-ttu-id="6fec8-111">Školení na hraničním vzorci používá virtuální počítač, který běží na rozbočovači Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6fec8-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="6fec8-112">Virtuální počítač je v Azure ML zaregistrován jako výpočetní cíl, takže data přístupu k němu jsou dostupná jenom místně.</span><span class="sxs-lookup"><span data-stu-id="6fec8-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="6fec8-113">V tomto případě jsou data uložená v úložišti objektů BLOB centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6fec8-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="6fec8-114">Jakmile je model vyškolený, zaregistruje se s Azure ML a přidá se do Azure Container Registry pro nasazení.</span><span class="sxs-lookup"><span data-stu-id="6fec8-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="6fec8-115">Pro tuto iteraci vzoru musí být virtuální počítač pro školení centra Azure Stack dosažitelný přes veřejný Internet.</span><span class="sxs-lookup"><span data-stu-id="6fec8-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="6fec8-116">[![Model vlakové ML na hraniční architektuře](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="6fec8-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="6fec8-117">Jak tento vzor funguje:</span><span class="sxs-lookup"><span data-stu-id="6fec8-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="6fec8-118">Virtuální počítač centra Azure Stack je nasazený a registrovaný jako cíl pro výpočetní prostředky pomocí Azure ML.</span><span class="sxs-lookup"><span data-stu-id="6fec8-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="6fec8-119">V Azure ML se vytvoří experiment, který jako cíl výpočetní služby používá virtuální počítač centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6fec8-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="6fec8-120">Jakmile je model vyškolen, je zaregistrován a kontejnerem.</span><span class="sxs-lookup"><span data-stu-id="6fec8-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="6fec8-121">Model se teď dá nasadit do umístění, která jsou v místním prostředí nebo v cloudu.</span><span class="sxs-lookup"><span data-stu-id="6fec8-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="6fec8-122">Komponenty</span><span class="sxs-lookup"><span data-stu-id="6fec8-122">Components</span></span>

<span data-ttu-id="6fec8-123">Toto řešení používá následující komponenty:</span><span class="sxs-lookup"><span data-stu-id="6fec8-123">This solution uses the following components:</span></span>

| <span data-ttu-id="6fec8-124">Vrstva</span><span class="sxs-lookup"><span data-stu-id="6fec8-124">Layer</span></span> | <span data-ttu-id="6fec8-125">Součást</span><span class="sxs-lookup"><span data-stu-id="6fec8-125">Component</span></span> | <span data-ttu-id="6fec8-126">Description</span><span class="sxs-lookup"><span data-stu-id="6fec8-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="6fec8-127">Azure</span><span class="sxs-lookup"><span data-stu-id="6fec8-127">Azure</span></span> | <span data-ttu-id="6fec8-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="6fec8-128">Azure Machine Learning</span></span> | <span data-ttu-id="6fec8-129">[Azure Machine Learning](/azure/machine-learning/) orchestruje školení modelu ml.</span><span class="sxs-lookup"><span data-stu-id="6fec8-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="6fec8-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="6fec8-130">Azure Container Registry</span></span> | <span data-ttu-id="6fec8-131">Azure ML zabalí model do kontejneru a uloží ho do [Azure Container Registry](/azure/container-registry/) pro nasazení.</span><span class="sxs-lookup"><span data-stu-id="6fec8-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="6fec8-132">Centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="6fec8-132">Azure Stack Hub</span></span> | <span data-ttu-id="6fec8-133">App Service</span><span class="sxs-lookup"><span data-stu-id="6fec8-133">App Service</span></span> | <span data-ttu-id="6fec8-134">[Azure Stack centrum s App Service](/azure-stack/operator/azure-stack-app-service-overview) poskytuje základ pro komponenty na hraničních zařízeních.</span><span class="sxs-lookup"><span data-stu-id="6fec8-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="6fec8-135">Compute</span><span class="sxs-lookup"><span data-stu-id="6fec8-135">Compute</span></span> | <span data-ttu-id="6fec8-136">Pro výuku modelu ML se používá virtuální počítač centra Azure Stack se systémem Ubuntu s Docker.</span><span class="sxs-lookup"><span data-stu-id="6fec8-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="6fec8-137">Storage</span><span class="sxs-lookup"><span data-stu-id="6fec8-137">Storage</span></span> | <span data-ttu-id="6fec8-138">Soukromá data můžou být hostovaná v úložišti objektů BLOB centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="6fec8-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="6fec8-139">Problémy a důležité informace</span><span class="sxs-lookup"><span data-stu-id="6fec8-139">Issues and considerations</span></span>

<span data-ttu-id="6fec8-140">Při rozhodování, jak implementovat toto řešení, vezměte v úvahu následující body:</span><span class="sxs-lookup"><span data-stu-id="6fec8-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="6fec8-141">Škálovatelnost</span><span class="sxs-lookup"><span data-stu-id="6fec8-141">Scalability</span></span>

<span data-ttu-id="6fec8-142">Pokud chcete toto řešení povolit pro škálování, budete muset vytvořit vhodný velikost virtuálního počítače na Azure Stackovém centru pro školení.</span><span class="sxs-lookup"><span data-stu-id="6fec8-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="6fec8-143">Dostupnost</span><span class="sxs-lookup"><span data-stu-id="6fec8-143">Availability</span></span>

<span data-ttu-id="6fec8-144">Ujistěte se, že školicí skripty a virtuální počítač centra Azure Stack mají přístup k místním datům používaným pro školení.</span><span class="sxs-lookup"><span data-stu-id="6fec8-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="6fec8-145">Možnosti správy</span><span class="sxs-lookup"><span data-stu-id="6fec8-145">Manageability</span></span>

<span data-ttu-id="6fec8-146">Zajistěte, aby modely a experimenty byly správně registrovány, označeny verzí a označily, aby nedocházelo k záměně při nasazení modelu.</span><span class="sxs-lookup"><span data-stu-id="6fec8-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="6fec8-147">Zabezpečení</span><span class="sxs-lookup"><span data-stu-id="6fec8-147">Security</span></span>

<span data-ttu-id="6fec8-148">Tento model umožňuje službě Azure ML přístup k možným citlivým datům v místním prostředí.</span><span class="sxs-lookup"><span data-stu-id="6fec8-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="6fec8-149">Ujistěte se, že účet používaný k SSH do virtuálního počítače centra Azure Stack obsahuje silné heslo a školicí skripty neuchovávají ani neodesílají data do cloudu.</span><span class="sxs-lookup"><span data-stu-id="6fec8-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6fec8-150">Další kroky</span><span class="sxs-lookup"><span data-stu-id="6fec8-150">Next steps</span></span>

<span data-ttu-id="6fec8-151">Další informace o tématech zavedených v tomto článku:</span><span class="sxs-lookup"><span data-stu-id="6fec8-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="6fec8-152">Přehled ML a Příbuzná témata najdete v [dokumentaci k Azure Machine Learning](/azure/machine-learning) .</span><span class="sxs-lookup"><span data-stu-id="6fec8-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="6fec8-153">V tématu [Azure Container Registry](/azure/container-registry/) se dozvíte, jak vytvářet, ukládat a spravovat image pro nasazení kontejnerů.</span><span class="sxs-lookup"><span data-stu-id="6fec8-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="6fec8-154">Další informace o poskytovateli prostředků a způsobu nasazení najdete [v tématu App Service v centru Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) .</span><span class="sxs-lookup"><span data-stu-id="6fec8-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="6fec8-155">Další informace o osvědčených postupech a o tom, jak získat odpovědi na další otázky, najdete v tématu [aspekty návrhu hybridní aplikace](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="6fec8-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="6fec8-156">Další informace o celém portfoliu produktů a řešení najdete v [Azure Stack rodině produktů a řešení](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="6fec8-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="6fec8-157">Až budete připraveni otestovat příklad řešení, pokračujte v [modelu vlakové ml v Průvodci nasazením Edge](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="6fec8-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="6fec8-158">Průvodce nasazením poskytuje podrobné pokyny pro nasazení a testování jeho komponent.</span><span class="sxs-lookup"><span data-stu-id="6fec8-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
