---
title: Configuración de la identidad de nube híbrida para aplicaciones de Azure y Azure Stack Hub
description: Aprenda a configurar la identidad de nube híbrida para aplicaciones de Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477259"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="35df7-103">Configuración de la identidad de nube híbrida para aplicaciones de Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="35df7-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="35df7-104">Aprenda a configurar la identidad de nube híbrida para sus aplicaciones de Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="35df7-105">Tiene dos opciones para conceder acceso a sus aplicaciones tanto en Azure global como en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="35df7-106">Si Azure Stack Hub tiene conexión continua a Internet, puede usar Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="35df7-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="35df7-107">Si Azure Stack Hub no tiene conexión a Internet, puede usar los Servicios de federación de Active Directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="35df7-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="35df7-108">Las entidades de servicio se usan para conceder acceso a las aplicaciones de Azure Stack Hub para la implementación o configuración mediante Azure Resource Manager en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="35df7-109">En esta solución, creará un entorno de ejemplo para:</span><span class="sxs-lookup"><span data-stu-id="35df7-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="35df7-110">Establecer una identidad híbrida en Azure global y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="35df7-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="35df7-111">Recuperar un token para acceder a la API de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="35df7-112">Para seguir los pasos de esta solución, debe tener permisos de operador de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="35df7-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="35df7-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="35df7-114">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="35df7-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="35df7-115">Azure Stack Hub aporta la agilidad e innovación de la informática en la nube a su entorno local, ya que habilita la única nube híbrida que permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="35df7-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="35df7-116">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los pilares de la calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="35df7-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="35df7-117">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="35df7-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="35df7-118">Crear a una entidad de servicio para Azure AD en el portal</span><span class="sxs-lookup"><span data-stu-id="35df7-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="35df7-119">Si implementó Azure Stack Hub con Azure AD como el almacén de identidades, puede crear entidades de servicio como lo hace para Azure.</span><span class="sxs-lookup"><span data-stu-id="35df7-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="35df7-120">La sección [Uso de una identidad de aplicación para acceder a recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) muestra cómo realizar los pasos desde el portal.</span><span class="sxs-lookup"><span data-stu-id="35df7-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="35df7-121">Asegúrese de que dispone de los [permisos de Azure AD necesarios](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) antes de comenzar.</span><span class="sxs-lookup"><span data-stu-id="35df7-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="35df7-122">Creación de una entidad de servicio para AD FS mediante PowerShell</span><span class="sxs-lookup"><span data-stu-id="35df7-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="35df7-123">Si implementó Azure Stack Hub con AD FS, puede usar PowerShell para crear una entidad de servicio, asignar un rol para el acceso e iniciar sesión en PowerShell con dicha identidad.</span><span class="sxs-lookup"><span data-stu-id="35df7-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="35df7-124">La sección [Uso de una identidad de aplicación para acceder a recursos](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) muestra cómo realizar los pasos necesarios mediante PowerShell.</span><span class="sxs-lookup"><span data-stu-id="35df7-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="35df7-125">Uso de la API de Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="35df7-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="35df7-126">La solución [API de Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use.md) le guía por el proceso de recuperación de un token de acceso a la API de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="35df7-127">Conexión a Azure Stack Hub mediante PowerShell</span><span class="sxs-lookup"><span data-stu-id="35df7-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="35df7-128">La guía de inicio rápido para [empezar a trabajar con PowerShell en Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) le guía por los pasos necesarios para instalar Azure PowerShell y conectarse a su instalación de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="35df7-129">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="35df7-129">Prerequisites</span></span>

<span data-ttu-id="35df7-130">Necesita una instalación de Azure Stack Hub conectada a Azure AD con una suscripción a la que puede acceder.</span><span class="sxs-lookup"><span data-stu-id="35df7-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="35df7-131">Si no tiene una instalación de Azure Stack Hub, puede usar estas instrucciones para configurar un [Kit de desarrollo de Azure Stack (ASDK)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="35df7-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="35df7-132">Conexión a Azure Stack Hub mediante código</span><span class="sxs-lookup"><span data-stu-id="35df7-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="35df7-133">Para conectarse a Azure Stack Hub mediante código, use la API de los puntos de conexión de Azure Resource Manager para obtener los puntos de conexión de autenticación y Graph para la instalación de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="35df7-134">Luego, autentíquese mediante solicitudes de REST.</span><span class="sxs-lookup"><span data-stu-id="35df7-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="35df7-135">Puede encontrar una aplicación cliente de ejemplo en [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="35df7-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="35df7-136">A menos que el SDK de Azure correspondiente al lenguaje seleccionado admita perfiles de API de Azure, puede que el SDK no funcione con Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="35df7-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="35df7-137">Para más información acerca de los perfiles de la API de Azure, consulte el artículo [Administración de los perfiles de la versión de API](/azure-stack/user/azure-stack-version-profiles.md).</span><span class="sxs-lookup"><span data-stu-id="35df7-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="35df7-138">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="35df7-138">Next steps</span></span>

- <span data-ttu-id="35df7-139">Para más información sobre cómo se trata la identidad en Azure Stack Hub, consulte [Arquitectura de identidad para Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span><span class="sxs-lookup"><span data-stu-id="35df7-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="35df7-140">Para más información sobre los patrones de nube de Azure, consulte [Patrones de diseño en la nube](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="35df7-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
