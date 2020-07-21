---
title: Konfigurace hybridní cloudové identity pro Azure a aplikace Azure Stack hub
description: Naučte se konfigurovat hybridní cloudovou identitu pro aplikace Azure a Azure Stack hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: cs-CZ
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477248"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="c619e-103">Konfigurace hybridní cloudové identity pro Azure a aplikace Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="c619e-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="c619e-104">Naučte se konfigurovat hybridní cloudovou identitu pro aplikace Azure a Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="c619e-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="c619e-105">Máte dvě možnosti, jak udělit přístup k vašim aplikacím v globálním centru Azure i Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="c619e-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="c619e-106">Když Azure Stack hub má nepřetržité připojení k Internetu, můžete použít Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="c619e-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="c619e-107">Když je rozbočovač Azure Stack odpojený od Internetu, můžete použít Azure Directory federovaného služby (AD FS).</span><span class="sxs-lookup"><span data-stu-id="c619e-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="c619e-108">Pomocí instančních objektů udělíte přístup k aplikacím centra Azure Stack pro nasazení nebo konfiguraci pomocí Azure Resource Manager v centru Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="c619e-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="c619e-109">V tomto řešení sestavíte ukázkové prostředí pro:</span><span class="sxs-lookup"><span data-stu-id="c619e-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="c619e-110">Vytvoření hybridní identity v globálním centru Azure a Azure Stack</span><span class="sxs-lookup"><span data-stu-id="c619e-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="c619e-111">Načtěte token pro přístup k rozhraní API centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="c619e-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="c619e-112">Pro kroky v tomto řešení musíte mít oprávnění operátora centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="c619e-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="c619e-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="c619e-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="c619e-114">Centrum Microsoft Azure Stack je rozšířením Azure.</span><span class="sxs-lookup"><span data-stu-id="c619e-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="c619e-115">Centrum Azure Stack přináší flexibilitu a inovace cloud computingu do místního prostředí. tím se umožní jenom hybridní cloud, který umožňuje vytvářet a nasazovat hybridní aplikace odkudkoli.</span><span class="sxs-lookup"><span data-stu-id="c619e-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="c619e-116">Články [týkající se návrhu hybridní aplikace](overview-app-design-considerations.md) prověří pilíře kvality softwaru (umístění, škálovatelnost, dostupnost, odolnost, možnosti správy a zabezpečení) pro navrhování, nasazování a provozování hybridních aplikací.</span><span class="sxs-lookup"><span data-stu-id="c619e-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="c619e-117">Pokyny k návrhu pomáhají při optimalizaci návrhu hybridní aplikace a minimalizaci výzev v produkčních prostředích.</span><span class="sxs-lookup"><span data-stu-id="c619e-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="c619e-118">Vytvoření instančního objektu pro službu Azure AD na portálu</span><span class="sxs-lookup"><span data-stu-id="c619e-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="c619e-119">Pokud jste nasadili Azure Stack hub pomocí Azure AD jako úložiště identit, můžete objekty služby vytvářet stejně jako v případě Azure.</span><span class="sxs-lookup"><span data-stu-id="c619e-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="c619e-120">[Použití identity aplikace pro přístup k prostředkům](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) ukazuje, jak provést kroky prostřednictvím portálu.</span><span class="sxs-lookup"><span data-stu-id="c619e-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="c619e-121">Před zahájením se ujistěte, že máte [požadovaná oprávnění služby Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) .</span><span class="sxs-lookup"><span data-stu-id="c619e-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="c619e-122">Vytvoření instančního objektu pro AD FS s využitím PowerShellu</span><span class="sxs-lookup"><span data-stu-id="c619e-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="c619e-123">Pokud jste nasadili Azure Stack centrum s AD FS, můžete k vytvoření instančního objektu použít PowerShell a přiřazovat roli pro přístup a přihlašovat se pomocí této identity z PowerShellu.</span><span class="sxs-lookup"><span data-stu-id="c619e-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="c619e-124">[Použití identity aplikace pro přístup k prostředkům](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) ukazuje, jak provést požadované kroky pomocí prostředí PowerShell.</span><span class="sxs-lookup"><span data-stu-id="c619e-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="c619e-125">Použití rozhraní API centra Azure Stack</span><span class="sxs-lookup"><span data-stu-id="c619e-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="c619e-126">Řešení [API centra Azure Stack](/azure-stack/user/azure-stack-rest-api-use.md) vás provede procesem Načtení tokenu pro přístup k rozhraní api centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="c619e-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="c619e-127">Připojení k centru Azure Stack pomocí PowerShellu</span><span class="sxs-lookup"><span data-stu-id="c619e-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="c619e-128">Rychlý Start, [který vám umožní začít pracovat s PowerShellem v centru Azure Stack](/azure-stack/operator/azure-stack-powershell-install.md) , vás provede kroky potřebnými k instalaci Azure PowerShell a připojení k instalaci centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="c619e-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="c619e-129">Požadavky</span><span class="sxs-lookup"><span data-stu-id="c619e-129">Prerequisites</span></span>

<span data-ttu-id="c619e-130">Budete potřebovat instalaci centra Azure Stack připojenou ke službě Azure AD s předplatným, ke kterému máte přístup.</span><span class="sxs-lookup"><span data-stu-id="c619e-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="c619e-131">Pokud nemáte instalaci centra Azure Stack, můžete použít tyto pokyny k nastavení [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="c619e-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="c619e-132">Připojení k Azure Stack centru pomocí kódu</span><span class="sxs-lookup"><span data-stu-id="c619e-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="c619e-133">Pokud se chcete připojit k Azure Stack centru pomocí kódu, použijte rozhraní API pro Azure Resource Manager koncových bodů k získání koncových bodů ověřování a grafu pro instalaci centra Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="c619e-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="c619e-134">Pak proveďte ověření pomocí požadavků REST.</span><span class="sxs-lookup"><span data-stu-id="c619e-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="c619e-135">Ukázkovou klientskou aplikaci najdete na [GitHubu](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="c619e-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="c619e-136">Pokud sada Azure SDK pro váš jazyk, kterou si vyberete, nepodporuje profily rozhraní API Azure, sada SDK nemusí fungovat s Azure Stack hub.</span><span class="sxs-lookup"><span data-stu-id="c619e-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="c619e-137">Další informace o profilech rozhraní API Azure najdete v článku [Správa profilů verzí rozhraní API](/azure-stack/user/azure-stack-version-profiles.md) .</span><span class="sxs-lookup"><span data-stu-id="c619e-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c619e-138">Další kroky</span><span class="sxs-lookup"><span data-stu-id="c619e-138">Next steps</span></span>

- <span data-ttu-id="c619e-139">Další informace o tom, jak se identita zpracovává v centru Azure Stack, najdete v tématu [Architektura identity pro centrum Azure Stack](/azure-stack/operator/azure-stack-identity-architecture.md).</span><span class="sxs-lookup"><span data-stu-id="c619e-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="c619e-140">Další informace o vzorech cloudu Azure najdete v tématu [vzory návrhu cloudu](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="c619e-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
