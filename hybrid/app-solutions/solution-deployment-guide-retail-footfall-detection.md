---
title: Implementación de una solución de detección de afluencia de personas basada en IA con Azure y Azure Stack Hub
description: Aprenda a implementar una solución de detección de afluencia de personas basada en IA para analizar el tráfico de visitantes en las tiendas mediante Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5f2e18e164e54f60b1bb7a14026a0c75c7d7ce69
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477174"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="4d400-103">Implementación de una solución de detección de afluencia de público basada en IA con Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="4d400-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="4d400-104">En este artículo se describe cómo implementar una solución basada en inteligencia artificial que genere información útil a partir de acciones reales, mediante Azure, Azure Stack Hub y el Kit de desarrollo de inteligencia artificial de Custom Vision.</span><span class="sxs-lookup"><span data-stu-id="4d400-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="4d400-105">En esta solución, aprenderá cómo:</span><span class="sxs-lookup"><span data-stu-id="4d400-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="4d400-106">Implementar agrupaciones de aplicaciones nativas en la nube (CNAB) en el perímetro.</span><span class="sxs-lookup"><span data-stu-id="4d400-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="4d400-107">Implementar una aplicación que abarque los límites de la nube.</span><span class="sxs-lookup"><span data-stu-id="4d400-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="4d400-108">Use el kit de desarrollo de IA de Custom Vision para inferencia en el perímetro.</span><span class="sxs-lookup"><span data-stu-id="4d400-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="4d400-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="4d400-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="4d400-110">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="4d400-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="4d400-111">Azure Stack Hub aporta la agilidad y la innovación de la informática en la nube a su entorno local y hace posible la única nube híbrida que le permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="4d400-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="4d400-112">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los pilares de la calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="4d400-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="4d400-113">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="4d400-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="4d400-114">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="4d400-114">Prerequisites</span></span>

<span data-ttu-id="4d400-115">Antes de empezar a usar esta guía de implementación, asegúrese de:</span><span class="sxs-lookup"><span data-stu-id="4d400-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="4d400-116">Consultar el tema [Patrón de detección de afluencia de compradores](pattern-retail-footfall-detection.md).</span><span class="sxs-lookup"><span data-stu-id="4d400-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="4d400-117">Obtener acceso de usuario a un Kit de desarrollo de Azure Stack (ASDK) o una instancia de sistema integrada de Azure Stack Hub con:</span><span class="sxs-lookup"><span data-stu-id="4d400-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="4d400-118">[Azure App Service en el proveedor de recursos de Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview.md) instalado.</span><span class="sxs-lookup"><span data-stu-id="4d400-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="4d400-119">Necesita acceso de operador a la instancia de Azure Stack Hub o trabajar con su administrador para realizar la instalación.</span><span class="sxs-lookup"><span data-stu-id="4d400-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="4d400-120">Una suscripción a una oferta que proporcione App Service y la cuota de Storage.</span><span class="sxs-lookup"><span data-stu-id="4d400-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="4d400-121">Necesita acceso de operador para crear una oferta.</span><span class="sxs-lookup"><span data-stu-id="4d400-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="4d400-122">Obtener acceso a una suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="4d400-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="4d400-123">Si no tiene ninguna suscripción de Azure, regístrese para una [cuenta de evaluación gratuita](https://azure.microsoft.com/free/) antes de empezar.</span><span class="sxs-lookup"><span data-stu-id="4d400-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="4d400-124">Crear dos entidades de servicio en el directorio:</span><span class="sxs-lookup"><span data-stu-id="4d400-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="4d400-125">Una configurada para usarla con recursos de Azure, con acceso en el ámbito de la suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="4d400-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="4d400-126">Otra configurada para usarla con recursos de Azure Stack Hub, con acceso en el ámbito de la suscripción de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4d400-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="4d400-127">Para más información sobre la creación de entidades de servicio y la autorización de acceso, consulte [Uso de una identidad de aplicación para acceder a recursos](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="4d400-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="4d400-128">Si prefiere usar la CLI de Azure, vea [Creación de una entidad de servicio de Azure con la CLI de Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="4d400-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="4d400-129">Implemente Azure Cognitive Services en Azure o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4d400-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="4d400-130">En primer lugar [obtenga más información sobre Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="4d400-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="4d400-131">Luego, visite [Implementación de Azure Cognitive Services en Azure Stack](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) para implementar Cognitive Services en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4d400-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="4d400-132">Primero debe registrarse para acceder a la versión preliminar.</span><span class="sxs-lookup"><span data-stu-id="4d400-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="4d400-133">Clonar o descargar un kit de desarrollo de IA de Azure Custom Vision.</span><span class="sxs-lookup"><span data-stu-id="4d400-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="4d400-134">Para detalles, consulte el [kit de desarrollo de IA de Vision](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="4d400-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="4d400-135">Registrarse para obtener una cuenta de Power BI.</span><span class="sxs-lookup"><span data-stu-id="4d400-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="4d400-136">Una clave de suscripción de Face API de Azure Cognitive Services y una dirección URL de punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="4d400-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="4d400-137">Puede obtener ambas con la evaluación gratuita de [Pruebe Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api).</span><span class="sxs-lookup"><span data-stu-id="4d400-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="4d400-138">O bien, siga las instrucciones de [Creación de un recurso de Cognitive Services con Azure Portal](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="4d400-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="4d400-139">Instalar los siguientes recursos de desarrollo:</span><span class="sxs-lookup"><span data-stu-id="4d400-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="4d400-140">CLI de Azure 2.0</span><span class="sxs-lookup"><span data-stu-id="4d400-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="4d400-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="4d400-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="4d400-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="4d400-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="4d400-143">Use Porter para implementar aplicaciones en la nube mediante los manifiestos del conjunto de CNAB que se proporcionan.</span><span class="sxs-lookup"><span data-stu-id="4d400-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="4d400-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="4d400-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="4d400-145">Azure IoT Tools para Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="4d400-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="4d400-146">Extensión de Python para Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="4d400-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="4d400-147">Python</span><span class="sxs-lookup"><span data-stu-id="4d400-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="4d400-148">Implementación de la aplicación en la nube híbrida</span><span class="sxs-lookup"><span data-stu-id="4d400-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="4d400-149">En primer lugar, use la CLI de Porter para generar un conjunto de credenciales y luego implemente la aplicación en la nube.</span><span class="sxs-lookup"><span data-stu-id="4d400-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="4d400-150">Clone o descargue el código de ejemplo de la solución desde https://github.com/azure-samples/azure-intelligent-edge-patterns.</span><span class="sxs-lookup"><span data-stu-id="4d400-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="4d400-151">Porter generará un conjunto de credenciales que automatizarán la implementación de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4d400-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="4d400-152">Antes de ejecutar el comando de generación de credenciales, asegúrese de que dispone de lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="4d400-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="4d400-153">Una entidad de servicio para acceder a los recursos de Azure, incluido el identificador de la entidad de servicio, la clave y el DNS del inquilino.</span><span class="sxs-lookup"><span data-stu-id="4d400-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="4d400-154">El identificador de suscripción de su suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="4d400-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="4d400-155">Una entidad de servicio para acceder a los recursos de Azure Stack Hub, incluido el identificador de la entidad de servicio, la clave y el DNS del inquilino.</span><span class="sxs-lookup"><span data-stu-id="4d400-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="4d400-156">El identificador de suscripción de su suscripción de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="4d400-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="4d400-157">Su clave de Face API de Azure Cognitive Services y una dirección URL de punto de conexión de recurso.</span><span class="sxs-lookup"><span data-stu-id="4d400-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="4d400-158">Ejecute el proceso de generación de credenciales de Porter y siga las indicaciones:</span><span class="sxs-lookup"><span data-stu-id="4d400-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="4d400-159">Porter también requiere que se ejecute un conjunto de parámetros.</span><span class="sxs-lookup"><span data-stu-id="4d400-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="4d400-160">Cree un archivo de texto de parámetro y escriba los siguientes pares de nombre/valor.</span><span class="sxs-lookup"><span data-stu-id="4d400-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="4d400-161">Pregunte al administrador de Azure Stack Hub si necesita ayuda con cualquiera de los valores necesarios.</span><span class="sxs-lookup"><span data-stu-id="4d400-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="4d400-162">El valor `resource suffix` se usa para asegurarse de que los recursos de la implementación tienen nombres únicos en Azure.</span><span class="sxs-lookup"><span data-stu-id="4d400-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="4d400-163">Debe ser una cadena única de letras y números, de no más de ocho caracteres.</span><span class="sxs-lookup"><span data-stu-id="4d400-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

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
   <span data-ttu-id="4d400-164">Guarde el archivo de texto y tome nota de su ruta de acceso.</span><span class="sxs-lookup"><span data-stu-id="4d400-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="4d400-165">Ya está listo para implementar la aplicación en la nube híbrida mediante Porter.</span><span class="sxs-lookup"><span data-stu-id="4d400-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="4d400-166">Ejecute el comando de instalación y vea cómo se implementan los recursos en Azure y Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="4d400-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="4d400-167">Una vez completada la implementación, anote los siguientes valores:</span><span class="sxs-lookup"><span data-stu-id="4d400-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="4d400-168">La cadena de conexión de la cámara.</span><span class="sxs-lookup"><span data-stu-id="4d400-168">The camera's connection string.</span></span>
    - <span data-ttu-id="4d400-169">La cadena de conexión de la cuenta de almacenamiento de imágenes.</span><span class="sxs-lookup"><span data-stu-id="4d400-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="4d400-170">Los nombres del grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="4d400-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="4d400-171">Preparación del Kit de desarrollo de inteligencia artificial de Custom Vision</span><span class="sxs-lookup"><span data-stu-id="4d400-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="4d400-172">A continuación, configure el kit de desarrollo de AI de Custom Vision, tal como se muestra en el [inicio rápido del kit de desarrollo de IA de Vision](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="4d400-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="4d400-173">También debe configurar y probar la cámara mediante la cadena de conexión proporcionada en el paso anterior.</span><span class="sxs-lookup"><span data-stu-id="4d400-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="4d400-174">Implementación de la aplicación de la cámara</span><span class="sxs-lookup"><span data-stu-id="4d400-174">Deploy the camera app</span></span>

<span data-ttu-id="4d400-175">Use la CLI de Porter para generar un conjunto de credenciales y luego implemente la aplicación de la cámara.</span><span class="sxs-lookup"><span data-stu-id="4d400-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="4d400-176">Porter generará un conjunto de credenciales que automatizarán la implementación de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="4d400-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="4d400-177">Antes de ejecutar el comando de generación de credenciales, asegúrese de que dispone de lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="4d400-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="4d400-178">Una entidad de servicio para acceder a los recursos de Azure, incluido el identificador de la entidad de servicio, la clave y el DNS del inquilino.</span><span class="sxs-lookup"><span data-stu-id="4d400-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="4d400-179">El identificador de suscripción de su suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="4d400-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="4d400-180">La cadena de conexión de la cuenta de almacenamiento de imágenes que se proporciona al implementar la aplicación en la nube.</span><span class="sxs-lookup"><span data-stu-id="4d400-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="4d400-181">Ejecute el proceso de generación de credenciales de Porter y siga las indicaciones:</span><span class="sxs-lookup"><span data-stu-id="4d400-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="4d400-182">Porter también requiere que se ejecute un conjunto de parámetros.</span><span class="sxs-lookup"><span data-stu-id="4d400-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="4d400-183">Cree un archivo de texto de parámetro y escriba el texto siguiente.</span><span class="sxs-lookup"><span data-stu-id="4d400-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="4d400-184">Pregunte al administrador de Azure Stack Hub si no conoce algunos de los valores requeridos.</span><span class="sxs-lookup"><span data-stu-id="4d400-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="4d400-185">El valor `deployment suffix` se usa para asegurarse de que los recursos de la implementación tienen nombres únicos en Azure.</span><span class="sxs-lookup"><span data-stu-id="4d400-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="4d400-186">Debe ser una cadena única de letras y números, de no más de ocho caracteres.</span><span class="sxs-lookup"><span data-stu-id="4d400-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="4d400-187">Guarde el archivo de texto y tome nota de su ruta de acceso.</span><span class="sxs-lookup"><span data-stu-id="4d400-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="4d400-188">Ya está listo para implementar la aplicación de la cámara mediante Porter.</span><span class="sxs-lookup"><span data-stu-id="4d400-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="4d400-189">Ejecute el comando de instalación y vea cómo se crea la implementación de IoT Edge.</span><span class="sxs-lookup"><span data-stu-id="4d400-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="4d400-190">Para comprobar que la implementación de la cámara ha finalizado viendo la fuente de cámara en `https://<camera-ip>:3000/`, donde `<camara-ip>` es la dirección IP de la cámara.</span><span class="sxs-lookup"><span data-stu-id="4d400-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="4d400-191">Este paso puede tardar hasta diez minutos.</span><span class="sxs-lookup"><span data-stu-id="4d400-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="4d400-192">Configuración de Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="4d400-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="4d400-193">Ahora que los datos fluyen hacia Azure Stream Analytics desde la cámara, es necesario que lo autorice manualmente para que se comuniquen con Power BI.</span><span class="sxs-lookup"><span data-stu-id="4d400-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="4d400-194">En Azure Portal, abra **Todos los recursos** y el trabajo *process-footfall\[yoursuffix\]* .</span><span class="sxs-lookup"><span data-stu-id="4d400-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="4d400-195">En la sección **Topología de trabajo** del panel de trabajos de Stream Analytics, seleccione la opción **Salidas**.</span><span class="sxs-lookup"><span data-stu-id="4d400-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="4d400-196">Seleccione el receptor de salida de **salida de tráfico**.</span><span class="sxs-lookup"><span data-stu-id="4d400-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="4d400-197">Seleccione **Renovar autorización** e inicie sesión en su cuenta de Power BI.</span><span class="sxs-lookup"><span data-stu-id="4d400-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Mensaje de renovación de autorización en Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="4d400-199">Guarde la configuración de salida.</span><span class="sxs-lookup"><span data-stu-id="4d400-199">Save the output settings.</span></span>

6. <span data-ttu-id="4d400-200">Vaya al panel **Información general** y seleccione **Iniciar** para empezar a enviar datos a Power BI.</span><span class="sxs-lookup"><span data-stu-id="4d400-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="4d400-201">Seleccione **Ahora** como la hora de inicio de la salida del trabajo y seleccione **Iniciar**.</span><span class="sxs-lookup"><span data-stu-id="4d400-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="4d400-202">El estado del trabajo se puede ver en la barra de notificación.</span><span class="sxs-lookup"><span data-stu-id="4d400-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="4d400-203">Creación de un panel de Power BI</span><span class="sxs-lookup"><span data-stu-id="4d400-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="4d400-204">Una vez que el trabajo se inicia correctamente, vaya a [Power BI](https://powerbi.com/) e inicie sesión con su cuenta profesional o educativa.</span><span class="sxs-lookup"><span data-stu-id="4d400-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="4d400-205">Si la consulta del trabajo de Stream Analytics genera resultados, el conjunto de datos *footfall-dataset* que creó existe en la pestaña **Conjuntos de datos**.</span><span class="sxs-lookup"><span data-stu-id="4d400-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="4d400-206">En el área de trabajo de Power BI, seleccione **+ Crear** para crear un nuevo panel denominado *Footfall Analysis* (Análisis de afluencia de público).</span><span class="sxs-lookup"><span data-stu-id="4d400-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="4d400-207">En la parte superior de la ventana, seleccione **Agregar icono**.</span><span class="sxs-lookup"><span data-stu-id="4d400-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="4d400-208">Luego, seleccione **Datos de transmisión personalizados** y **Siguiente**.</span><span class="sxs-lookup"><span data-stu-id="4d400-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="4d400-209">Elija **footfall-dataset** en **Sus conjuntos de datos**.</span><span class="sxs-lookup"><span data-stu-id="4d400-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="4d400-210">Seleccione **Tarjeta** en la lista desplegable **Tipo de visualización** y agregue **age** a **Campos**.</span><span class="sxs-lookup"><span data-stu-id="4d400-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="4d400-211">Seleccione **Siguiente** para escribir el nombre del icono y, después, seleccione **Aplicar** para crear el icono.</span><span class="sxs-lookup"><span data-stu-id="4d400-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="4d400-212">Puede agregar campos y tarjetas adicionales según sea necesario.</span><span class="sxs-lookup"><span data-stu-id="4d400-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="4d400-213">Prueba de la solución</span><span class="sxs-lookup"><span data-stu-id="4d400-213">Test Your Solution</span></span>

<span data-ttu-id="4d400-214">Observe cómo cambian los datos en las tarjetas que creó en Power BI cuando diferentes personas caminan frente a la cámara.</span><span class="sxs-lookup"><span data-stu-id="4d400-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="4d400-215">Las inferencias pueden tardar hasta veinte segundos en aparecer una vez grabadas.</span><span class="sxs-lookup"><span data-stu-id="4d400-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="4d400-216">Protección de la solución</span><span class="sxs-lookup"><span data-stu-id="4d400-216">Remove Your Solution</span></span>

<span data-ttu-id="4d400-217">Si desea quitar la solución, ejecute los siguientes comandos mediante Porter y utilice los mismos archivos de parámetro que creó para la implementación:</span><span class="sxs-lookup"><span data-stu-id="4d400-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="4d400-218">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="4d400-218">Next steps</span></span>

- <span data-ttu-id="4d400-219">Más información sobre [consideraciones de diseño de aplicaciones híbridas]. (overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="4d400-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="4d400-220">Revise y proponga mejoras en [el código de este ejemplo en GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="4d400-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
