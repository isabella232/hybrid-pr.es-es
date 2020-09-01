---
title: Implementación de una aplicación que se escala entre nubes en Azure y Azure Stack Hub
description: Aprenda a implementar una aplicación que se escala entre nubes en Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886822"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="42abc-103">Implementación de una aplicación que se escala en toda la nube con Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="42abc-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="42abc-104">Aprenda a crear una solución en toda la nube que proporcione un proceso desencadenado manualmente para cambiar de una aplicación web hospedada en Azure Stack Hub a una aplicación web hospedada en Azure con escalabilidad automática mediante Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="42abc-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="42abc-105">Este proceso garantiza la utilidad en la nube flexible y escalable con carga.</span><span class="sxs-lookup"><span data-stu-id="42abc-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="42abc-106">Con este patrón, puede que el inquilino no esté preparado para ejecutar la aplicación en la nube pública.</span><span class="sxs-lookup"><span data-stu-id="42abc-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="42abc-107">Sin embargo, puede que no sea económicamente viable para la empresa mantener la capacidad necesaria en el entorno local para controlar los picos en la demanda de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="42abc-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="42abc-108">El inquilino puede aprovechar la elasticidad de la nube pública en su solución local.</span><span class="sxs-lookup"><span data-stu-id="42abc-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="42abc-109">En esta solución, creará un entorno de ejemplo para:</span><span class="sxs-lookup"><span data-stu-id="42abc-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="42abc-110">Crear una aplicación web de varios nodos.</span><span class="sxs-lookup"><span data-stu-id="42abc-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="42abc-111">Configurar y administrar el proceso de implementación continua (CD).</span><span class="sxs-lookup"><span data-stu-id="42abc-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="42abc-112">Publicar la aplicación web en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="42abc-113">Crear una versión.</span><span class="sxs-lookup"><span data-stu-id="42abc-113">Create a release.</span></span>
> - <span data-ttu-id="42abc-114">Supervisar y realizar un seguimiento de las implementaciones.</span><span class="sxs-lookup"><span data-stu-id="42abc-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="42abc-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="42abc-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="42abc-116">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="42abc-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="42abc-117">Azure Stack Hub aporta la agilidad e innovación de la informática en la nube a su entorno local, ya que habilita la única nube híbrida que permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="42abc-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="42abc-118">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los fundamentos de calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="42abc-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="42abc-119">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="42abc-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="42abc-120">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="42abc-120">Prerequisites</span></span>

- <span data-ttu-id="42abc-121">Suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="42abc-121">Azure subscription.</span></span> <span data-ttu-id="42abc-122">Si es necesario, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de comenzar.</span><span class="sxs-lookup"><span data-stu-id="42abc-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="42abc-123">Un sistema integrado de Azure Stack Hub o la implementación del Kit de desarrollo de Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="42abc-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="42abc-124">Para obtener instrucciones para la instalación de Azure Stack Hub, consulte [Instalación del Kit de desarrollo de Azure Stack](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="42abc-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="42abc-125">Para ver un script de automatización posterior a la implementación de ASDK, vaya a: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="42abc-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="42abc-126">Esta instalación puede tardar algunas horas en completarse.</span><span class="sxs-lookup"><span data-stu-id="42abc-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="42abc-127">Implemente los servicios PaaS de [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="42abc-128">[Cree planes u ofertas](/azure-stack/operator/service-plan-offer-subscription-overview.md) en el entorno de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="42abc-129">[Cree una suscripción de inquilino](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) dentro del entorno de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="42abc-130">Cree una aplicación web dentro de la suscripción de inquilino.</span><span class="sxs-lookup"><span data-stu-id="42abc-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="42abc-131">Anote la nueva dirección URL de aplicación web, ya que la usará más adelante.</span><span class="sxs-lookup"><span data-stu-id="42abc-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="42abc-132">Implemente la máquina virtual (VM) de Azure Pipelines en la suscripción del inquilino.</span><span class="sxs-lookup"><span data-stu-id="42abc-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="42abc-133">Se requiere una máquina virtual Windows Server 2016 con .NET 3.5.</span><span class="sxs-lookup"><span data-stu-id="42abc-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="42abc-134">Esta máquina virtual se compilará en la suscripción del inquilino en Azure Stack Hub como agente de compilación privado.</span><span class="sxs-lookup"><span data-stu-id="42abc-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="42abc-135">[Windows Server 2016 con la imagen de máquina virtual de SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) está disponible en Marketplace de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="42abc-136">Si esta imagen no está disponible, trabaje con un operador de Azure Stack Hub para garantizar que se agrega al entorno.</span><span class="sxs-lookup"><span data-stu-id="42abc-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="42abc-137">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="42abc-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="42abc-138">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="42abc-138">Scalability</span></span>

<span data-ttu-id="42abc-139">El componente clave del escalado de toda la nube es la posibilidad de proporcionar un escalado inmediato a petición entre la infraestructura en la nube pública y local que ofrezca unos servicios coherentes y confiables.</span><span class="sxs-lookup"><span data-stu-id="42abc-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="42abc-140">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="42abc-140">Availability</span></span>

<span data-ttu-id="42abc-141">Asegúrese de que las aplicaciones implementadas localmente están configuradas para una alta disponibilidad mediante la configuración del hardware local y la implementación de software.</span><span class="sxs-lookup"><span data-stu-id="42abc-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="42abc-142">Facilidad de uso</span><span class="sxs-lookup"><span data-stu-id="42abc-142">Manageability</span></span>

<span data-ttu-id="42abc-143">La solución para toda la nube garantiza una administración sin problemas y una interfaz familiar entre entornos.</span><span class="sxs-lookup"><span data-stu-id="42abc-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="42abc-144">Se recomienda usar PowerShell para la administración multiplataforma.</span><span class="sxs-lookup"><span data-stu-id="42abc-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="42abc-145">Escalado de toda la nube</span><span class="sxs-lookup"><span data-stu-id="42abc-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="42abc-146">Obtención de un dominio personalizado y configuración de DNS</span><span class="sxs-lookup"><span data-stu-id="42abc-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="42abc-147">Actualice el archivo de zona DNS para el dominio.</span><span class="sxs-lookup"><span data-stu-id="42abc-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="42abc-148">Azure AD comprobará la propiedad del nombre de dominio personalizado.</span><span class="sxs-lookup"><span data-stu-id="42abc-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="42abc-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) para los registros DNS de Azure/Microsoft 365/externo dentro de Azure, o bien agregue la entrada DNS a [un registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="42abc-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="42abc-150">Registre un dominio personalizado con un registrador público.</span><span class="sxs-lookup"><span data-stu-id="42abc-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="42abc-151">Inicie sesión en el registrador de nombres de dominio para el dominio.</span><span class="sxs-lookup"><span data-stu-id="42abc-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="42abc-152">Puede que se requiera un administrador autorizado para realizar las actualizaciones de DNS.</span><span class="sxs-lookup"><span data-stu-id="42abc-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="42abc-153">Actualice el archivo de zona DNS para el dominio. Para ello, agregue la entrada DNS que Azure AD proporcione</span><span class="sxs-lookup"><span data-stu-id="42abc-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="42abc-154">(la entrada del DNS no afectará al enrutamiento de correo electrónico ni a los comportamientos de hospedaje web).</span><span class="sxs-lookup"><span data-stu-id="42abc-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="42abc-155">Creación de una aplicación web predeterminada de varios nodos en Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="42abc-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="42abc-156">Configure la canalización híbrida de integración continua e implementación continua (CI/CD) para implementar aplicaciones web en Azure y Azure Stack Hub e insertar automáticamente los cambios en ambas nubes.</span><span class="sxs-lookup"><span data-stu-id="42abc-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="42abc-157">Se requiere Azure Stack Hub con las imágenes adecuadas sindicadas para ejecutarse (Windows Server y SQL) y la implementación de App Service.</span><span class="sxs-lookup"><span data-stu-id="42abc-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="42abc-158">Para más información, revise en la documentación de App Service [Requisitos previos para implementar App Service en Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="42abc-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="42abc-159">Adición de código a Azure Repos</span><span class="sxs-lookup"><span data-stu-id="42abc-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="42abc-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="42abc-160">Azure Repos</span></span>

1. <span data-ttu-id="42abc-161">Inicie sesión en Azure Repos con una cuenta que tenga derechos de creación de proyectos en Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="42abc-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="42abc-162">La canalización de CI/CD híbrida se puede aplicar al código de aplicación y al código de infraestructura.</span><span class="sxs-lookup"><span data-stu-id="42abc-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="42abc-163">Use [plantillas de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) tanto para el desarrollo en la nube hospedado como para el privado.</span><span class="sxs-lookup"><span data-stu-id="42abc-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Conectarse a un proyecto en Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="42abc-165">**Clone el repositorio**; para ello, cree y abra la aplicación web predeterminada.</span><span class="sxs-lookup"><span data-stu-id="42abc-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonar el repositorio en la aplicación web de Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="42abc-167">Creación de una implementación de aplicación web autocontenida para App Services en ambas nubes</span><span class="sxs-lookup"><span data-stu-id="42abc-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="42abc-168">Edite el archivo **WebApplication.csproj**.</span><span class="sxs-lookup"><span data-stu-id="42abc-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="42abc-169">Seleccione `Runtimeidentifier` y agregue `win10-x64`</span><span class="sxs-lookup"><span data-stu-id="42abc-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="42abc-170">(consulte el documento acerca de las [implementaciones autocontenidas](/dotnet/core/deploying/deploy-with-vs#simpleSelf)).</span><span class="sxs-lookup"><span data-stu-id="42abc-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Editar el archivo del proyecto de aplicación web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="42abc-172">Inserte el código en Azure Repos mediante Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="42abc-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="42abc-173">Confirme que el código de la aplicación se ha insertado en el repositorio en Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="42abc-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="42abc-174">Creación de la definición de compilación</span><span class="sxs-lookup"><span data-stu-id="42abc-174">Create the build definition</span></span>

1. <span data-ttu-id="42abc-175">Inicie sesión en Azure Pipelines para confirmar la posibilidad de crear definiciones de compilación.</span><span class="sxs-lookup"><span data-stu-id="42abc-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="42abc-176">Agregue el código **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="42abc-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="42abc-177">Esta adición es necesaria para activar una implementación autocontenida con .Net Core.</span><span class="sxs-lookup"><span data-stu-id="42abc-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Agregar código a la aplicación web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="42abc-179">Ejecute la compilación.</span><span class="sxs-lookup"><span data-stu-id="42abc-179">Run the build.</span></span> <span data-ttu-id="42abc-180">El proceso de [compilación de implementación autocontenida](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará los artefactos que se pueden ejecutar en Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="42abc-181">Uso de un agente hospedado de Azure</span><span class="sxs-lookup"><span data-stu-id="42abc-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="42abc-182">El uso de un agente de compilación hospedado en Azure Pipelines es una opción adecuada para compilar e implementar aplicaciones web.</span><span class="sxs-lookup"><span data-stu-id="42abc-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="42abc-183">Microsoft Azure realiza automáticamente el mantenimiento y las actualizaciones, lo que permite un ciclo de desarrollo ininterrumpido y continuo.</span><span class="sxs-lookup"><span data-stu-id="42abc-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="42abc-184">Administración y configuración del proceso de CD</span><span class="sxs-lookup"><span data-stu-id="42abc-184">Manage and configure the CD process</span></span>

<span data-ttu-id="42abc-185">Azure Pipelines y Azure DevOps Services proporcionan una canalización con una gran capacidad de configuración y administración para versiones para varios entornos, como desarrollo, almacenamiento provisional, control de calidad y producción, lo que incluye la solicitud de aprobaciones en etapas concretas.</span><span class="sxs-lookup"><span data-stu-id="42abc-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="42abc-186">Creación de la definición de versión</span><span class="sxs-lookup"><span data-stu-id="42abc-186">Create release definition</span></span>

1. <span data-ttu-id="42abc-187">Seleccione el botón con el **signo más** para agregar una versión nueva en la pestaña **Releases** (Versiones) de la sección **Build and Release** (Compilación y versión) de Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="42abc-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Creación de una definición de versión](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="42abc-189">Aplique la plantilla Implementación de Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="42abc-189">Apply the Azure App Service Deployment template.</span></span>

   ![Aplicar la plantilla Implementación de Azure App Service](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="42abc-191">En **Agregar artefacto**, agregue el artefacto para la aplicación de compilación de nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="42abc-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Agregar el artefacto a la compilación en la nube de Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="42abc-193">En la pestaña Canalización, seleccione el vínculo **Fase, Tarea** del entorno y establezca los valores del entorno de nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="42abc-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Definir los valores de entorno de nube de Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="42abc-195">Establezca el **nombre del entorno** y seleccione la **suscripción de Azure** como el punto de conexión de la nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="42abc-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Seleccione la suscripción de Azure para el punto de conexión en la nube de Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="42abc-197">En **Nombre de App Service**, establezca el nombre del servicio de aplicación de Azure necesario.</span><span class="sxs-lookup"><span data-stu-id="42abc-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Definir el nombre del servicio de App Service](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="42abc-199">Escriba "Hospedado VS2017" en **Cola de agentes** para el entorno hospedado en la nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="42abc-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Defina la Cola de agentes para el entorno hospedado de la nube de Azure.](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="42abc-201">En el menú de implementación de Azure App Service, seleccione el **paquete o carpeta** válidos para el entorno.</span><span class="sxs-lookup"><span data-stu-id="42abc-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="42abc-202">Seleccione **Aceptar** para la **ubicación de carpeta**.</span><span class="sxs-lookup"><span data-stu-id="42abc-202">Select **OK** to **folder location**.</span></span>
  
      ![Seleccionar el paquete o la carpeta del entorno de Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Seleccionar el paquete o la carpeta del entorno de Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="42abc-205">Guarde todos los cambios y vuelva a la **canalización de versión**.</span><span class="sxs-lookup"><span data-stu-id="42abc-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Guardar los cambios en la canalización de versión](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="42abc-207">Agregue un nuevo artefacto mediante la selección de la compilación para la aplicación de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Agregar nuevo artefacto para la aplicación de Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="42abc-209">Agregue un entorno más; para ello, aplique Implementación de Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="42abc-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Agregar entorno en implementación de Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="42abc-211">Asigne al nuevo entorno el nombre "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="42abc-211">Name the new environment "Azure Stack".</span></span>

    ![Nombrar entorno en implementación de Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="42abc-213">Busque el entorno de Azure Stack en la pestaña **Tarea**.</span><span class="sxs-lookup"><span data-stu-id="42abc-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Entorno de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="42abc-215">Seleccione la suscripción para el punto de conexión de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="42abc-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Seleccionar la suscripción para el punto de conexión de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="42abc-217">Establezca el nombre de la aplicación web de Azure Stack como el nombre del servicio de aplicación.</span><span class="sxs-lookup"><span data-stu-id="42abc-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="42abc-218">![Establecer el nombre de la aplicación web de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="42abc-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="42abc-219">Seleccione el agente de Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="42abc-219">Select the Azure Stack agent.</span></span>

    ![Seleccionar el agente de Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="42abc-221">En la sección de implementación de Azure App Service, seleccione el **paquete o carpeta** válidos para el entorno.</span><span class="sxs-lookup"><span data-stu-id="42abc-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="42abc-222">Seleccione **Aceptar** para la ubicación de carpeta.</span><span class="sxs-lookup"><span data-stu-id="42abc-222">Select **OK** to folder location.</span></span>

    ![Seleccionar carpeta para la implementación de Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Seleccionar carpeta para la implementación de Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="42abc-225">En la pestaña Variable, agregue una variable denominada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, establezca su valor como **true** y defina el ámbito en Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="42abc-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Agregar variable a la implementación de App de Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="42abc-227">Seleccione el icono del desencadenador de implementación **Continuo** en ambos artefactos y habilite el desencadenador de implementación **Continua**.</span><span class="sxs-lookup"><span data-stu-id="42abc-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Seleccionar desencadenador de implementación continua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="42abc-229">Seleccione el icono de condiciones **previas a la implementación** en el entorno de Azure Stack y establezca el desencadenador en **After release** (Tras el lanzamiento).</span><span class="sxs-lookup"><span data-stu-id="42abc-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Seleccionar condiciones anteriores a la implementación](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="42abc-231">Guarde todos los cambios.</span><span class="sxs-lookup"><span data-stu-id="42abc-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="42abc-232">Algunos valores de configuración de las tareas se han definido automáticamente como [variables de entorno](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) cuando se creó una definición de versión desde una plantilla.</span><span class="sxs-lookup"><span data-stu-id="42abc-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="42abc-233">Estos valores de configuración no se pueden modificar en la configuración de tareas; para hacerlo, debe seleccionar el elemento del entorno principal.</span><span class="sxs-lookup"><span data-stu-id="42abc-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="42abc-234">Publicación en Azure Stack Hub mediante Visual Studio</span><span class="sxs-lookup"><span data-stu-id="42abc-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="42abc-235">Mediante la creación de puntos de conexión, una compilación de Azure DevOps Services puede implementar aplicaciones de Azure Service en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="42abc-236">Azure Pipelines se conecta con el agente de compilación, que, a su vez, se conecta con Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="42abc-237">Inicie sesión en Azure DevOps Services y vaya a la página de configuración de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="42abc-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="42abc-238">En **Configuración**, seleccione **Seguridad**.</span><span class="sxs-lookup"><span data-stu-id="42abc-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="42abc-239">En **Grupos de VSTS**, seleccione **Creadores de puntos de conexión**.</span><span class="sxs-lookup"><span data-stu-id="42abc-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="42abc-240">En la pestaña **Miembros**, seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="42abc-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="42abc-241">En **Agregar usuarios y grupos**, escriba un nombre de usuario y seleccione ese usuario en la lista de usuarios.</span><span class="sxs-lookup"><span data-stu-id="42abc-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="42abc-242">Seleccione **Save changes** (Guardar los cambios).</span><span class="sxs-lookup"><span data-stu-id="42abc-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="42abc-243">En la lista **Grupos de VSTS**, seleccione **Administradores de puntos de conexión**.</span><span class="sxs-lookup"><span data-stu-id="42abc-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="42abc-244">En la pestaña **Miembros**, seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="42abc-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="42abc-245">En **Agregar usuarios y grupos**, escriba un nombre de usuario y seleccione ese usuario en la lista de usuarios.</span><span class="sxs-lookup"><span data-stu-id="42abc-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="42abc-246">Seleccione **Save changes** (Guardar los cambios).</span><span class="sxs-lookup"><span data-stu-id="42abc-246">Select **Save changes**.</span></span>

<span data-ttu-id="42abc-247">Ahora que existe la información del punto de conexión, la conexión de Azure Pipelines con Azure Stack Hub está lista para su uso.</span><span class="sxs-lookup"><span data-stu-id="42abc-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="42abc-248">El agente de compilación de Azure Stack Hub obtiene instrucciones de Azure Pipelines y, luego, transmite la información del punto de conexión para la comunicación con Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="42abc-249">Desarrollo de la compilación de la aplicación</span><span class="sxs-lookup"><span data-stu-id="42abc-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="42abc-250">Se requiere Azure Stack Hub con las imágenes adecuadas sindicadas para ejecutarse (Windows Server y SQL) y la implementación de App Service.</span><span class="sxs-lookup"><span data-stu-id="42abc-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="42abc-251">Para más información, consulte [Requisitos previos para implementar App Service en Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="42abc-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="42abc-252">Use [plantillas de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) como código de aplicación web de Azure Repos para implementar en ambas nubes.</span><span class="sxs-lookup"><span data-stu-id="42abc-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="42abc-253">Incorporación de código a un proyecto de Azure Repos</span><span class="sxs-lookup"><span data-stu-id="42abc-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="42abc-254">Inicie sesión en Azure Repos con una cuenta que tenga derechos de creación de proyectos en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="42abc-255">**Clone el repositorio**; para ello, cree y abra la aplicación web predeterminada.</span><span class="sxs-lookup"><span data-stu-id="42abc-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="42abc-256">Creación de una implementación de aplicación web autocontenida para App Services en ambas nubes</span><span class="sxs-lookup"><span data-stu-id="42abc-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="42abc-257">Edite el archivo **WebApplication.csproj**: Seleccione `Runtimeidentifier` y, después, agregue `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="42abc-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="42abc-258">Para más información, consulte la documentación de la [implementación independiente](/dotnet/core/deploying/deploy-with-vs#simpleSelf).</span><span class="sxs-lookup"><span data-stu-id="42abc-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="42abc-259">Compruebe el código en Azure Repos mediante Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="42abc-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="42abc-260">Confirme que el código de la aplicación se ha insertado en Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="42abc-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="42abc-261">Creación de la definición de compilación</span><span class="sxs-lookup"><span data-stu-id="42abc-261">Create the build definition</span></span>

1. <span data-ttu-id="42abc-262">Inicie sesión en Azure Pipelines con una cuenta que pueda crear una definición de compilación.</span><span class="sxs-lookup"><span data-stu-id="42abc-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="42abc-263">Vaya a la página **Build Web Application** (Compilar aplicación web) del proyecto.</span><span class="sxs-lookup"><span data-stu-id="42abc-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="42abc-264">En **Argumentos**, agregue el código **-r win10-x64**.</span><span class="sxs-lookup"><span data-stu-id="42abc-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="42abc-265">Esta adición es necesaria para desencadenar una implementación independiente con .NET Core.</span><span class="sxs-lookup"><span data-stu-id="42abc-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="42abc-266">Ejecute la compilación.</span><span class="sxs-lookup"><span data-stu-id="42abc-266">Run the build.</span></span> <span data-ttu-id="42abc-267">El proceso de [compilación de implementación autocontenida](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará los artefactos que se pueden ejecutar en Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="42abc-268">Uso de un agente de compilación hospedado en Azure</span><span class="sxs-lookup"><span data-stu-id="42abc-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="42abc-269">El uso de un agente de compilación hospedado en Azure Pipelines es una opción adecuada para compilar e implementar aplicaciones web.</span><span class="sxs-lookup"><span data-stu-id="42abc-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="42abc-270">Microsoft Azure realiza automáticamente el mantenimiento y las actualizaciones, lo que permite un ciclo de desarrollo ininterrumpido y continuo.</span><span class="sxs-lookup"><span data-stu-id="42abc-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="42abc-271">Configuración del proceso de implementación continua (CD)</span><span class="sxs-lookup"><span data-stu-id="42abc-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="42abc-272">Azure Pipelines y Azure DevOps Services proporcionan una canalización con gran capacidad de configuración y administración para diferentes versiones de varios entornos, como desarrollo, almacenamiento provisional, control de calidad (QA) y producción.</span><span class="sxs-lookup"><span data-stu-id="42abc-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="42abc-273">Este proceso puede incluir la solicitud de aprobaciones en determinadas fases del ciclo de vida de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="42abc-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="42abc-274">Creación de la definición de versión</span><span class="sxs-lookup"><span data-stu-id="42abc-274">Create release definition</span></span>

<span data-ttu-id="42abc-275">La creación de una definición de versión es el último paso en el proceso de compilación de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="42abc-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="42abc-276">Esta definición de versión se usa para crear una versión e implementar una compilación.</span><span class="sxs-lookup"><span data-stu-id="42abc-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="42abc-277">Inicie sesión en Azure Pipelines y vaya a **Build and Release** (Compilación y versión) en el proyecto.</span><span class="sxs-lookup"><span data-stu-id="42abc-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="42abc-278">En la pestaña **Versiones**, seleccione **[ + ]** y, a continuación, elija **Crear definición de versión**.</span><span class="sxs-lookup"><span data-stu-id="42abc-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="42abc-279">En **Seleccionar una plantilla**, elija **Implementación de Azure App Service** y, a continuación, seleccione **Aplicar**.</span><span class="sxs-lookup"><span data-stu-id="42abc-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="42abc-280">En **Agregar artefacto**, en **Origen (definición de compilación)** , seleccione la aplicación de compilación en la nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="42abc-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="42abc-281">En la pestaña **Canalización**, seleccione el vínculo **1 fase**, **1 tarea** para **Ver tareas de entorno**.</span><span class="sxs-lookup"><span data-stu-id="42abc-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="42abc-282">En la pestaña **Tareas**, escriba Azure como **Nombre del entorno** y seleccione AzureCloud Traders-Web EP en la lista **Suscripción de Azure**.</span><span class="sxs-lookup"><span data-stu-id="42abc-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="42abc-283">Escriba el **nombre de la instancia de Azure App Service**, que es `northwindtraders` en la siguiente captura de pantalla.</span><span class="sxs-lookup"><span data-stu-id="42abc-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="42abc-284">Para la fase de agente, seleccione **VS2017 hospedado** en la lista **Cola de agentes**.</span><span class="sxs-lookup"><span data-stu-id="42abc-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="42abc-285">En **Implementar Azure App Service**, seleccione el valor de **Paquete o carpeta** válido para el entorno.</span><span class="sxs-lookup"><span data-stu-id="42abc-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="42abc-286">En **Seleccionar archivo o carpeta**, seleccione **Aceptar** para **Ubicación**.</span><span class="sxs-lookup"><span data-stu-id="42abc-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="42abc-287">Guarde todos los cambios y vuelva a **Canalización**.</span><span class="sxs-lookup"><span data-stu-id="42abc-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="42abc-288">En la pestaña **Canalización**, seleccione **Agregar artefacto** y elija **NorthwindCloud Traders-Vessel** en la lista **Origen (definición de compilación)** .</span><span class="sxs-lookup"><span data-stu-id="42abc-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="42abc-289">En **Seleccionar una plantilla**, agregue otro entorno.</span><span class="sxs-lookup"><span data-stu-id="42abc-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="42abc-290">Elija **Implementación de Azure App Service** y, a continuación, seleccione **Aplicar**.</span><span class="sxs-lookup"><span data-stu-id="42abc-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="42abc-291">Escriba `Azure Stack Hub` como **Nombre del entorno**.</span><span class="sxs-lookup"><span data-stu-id="42abc-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="42abc-292">En la pestaña **Tasks** (Tareas), busque y seleccione Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="42abc-293">En la lista **Azure subscription** (Suscripción de Azure), seleccione **AzureStack Traders-Vessel EP** como el punto de conexión de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="42abc-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="42abc-294">Establezca el nombre de la aplicación web de Azure Stack Hub como **nombre del servicio de aplicación**.</span><span class="sxs-lookup"><span data-stu-id="42abc-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="42abc-295">En **Selección de agente**, elija **AzureStack - b Douglas Fir** en la lista **Cola de agentes**.</span><span class="sxs-lookup"><span data-stu-id="42abc-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="42abc-296">En **Implementar Azure App Service**, seleccione el valor de **Paquete o carpeta** válido para el entorno.</span><span class="sxs-lookup"><span data-stu-id="42abc-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="42abc-297">En **Seleccionar archivo o carpeta**, seleccione **Aceptar** para la **Ubicación** de la carpeta.</span><span class="sxs-lookup"><span data-stu-id="42abc-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="42abc-298">En la pestaña **Variables**, busque la variable denominada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span><span class="sxs-lookup"><span data-stu-id="42abc-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="42abc-299">Establezca el valor de la variable en **true** y establezca su ámbito en **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="42abc-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="42abc-300">En la pestaña **Canalización**, seleccione el icono del **Desencadenador de implementación continua** para el artefacto NorthwindCloud Traders-Web y establezca **Desencadenador de implementación continua** en **Habilitado**.</span><span class="sxs-lookup"><span data-stu-id="42abc-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="42abc-301">Lo mismo para el artefacto **NorthwindCloud Traders-Vessel**.</span><span class="sxs-lookup"><span data-stu-id="42abc-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="42abc-302">En el entorno de Azure Stack Hub, seleccione el icono de **condiciones previas a la implementación** y establezca el desencadenador en **After release** (Tras el lanzamiento).</span><span class="sxs-lookup"><span data-stu-id="42abc-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="42abc-303">Guarde todos los cambios.</span><span class="sxs-lookup"><span data-stu-id="42abc-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="42abc-304">Algunos valores de configuración de las tareas se han definido automáticamente como [variables de entorno](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) cuando se creó una definición de versión desde una plantilla.</span><span class="sxs-lookup"><span data-stu-id="42abc-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="42abc-305">Esta configuración no puede modificarse en la configuración de la tarea, pero sí en los elementos del entorno primario.</span><span class="sxs-lookup"><span data-stu-id="42abc-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="42abc-306">Creación de una versión</span><span class="sxs-lookup"><span data-stu-id="42abc-306">Create a release</span></span>

1. <span data-ttu-id="42abc-307">En la pestaña **Canalización**, abra la lista **Versión** y seleccione **Crear versión**.</span><span class="sxs-lookup"><span data-stu-id="42abc-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="42abc-308">Escriba la descripción de la versión, compruebe que se han seleccionado los artefactos correctos y seleccione **Create** (Crear).</span><span class="sxs-lookup"><span data-stu-id="42abc-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="42abc-309">Transcurridos unos instantes, aparece un mensaje emergente que indica que se ha creado la nueva versión y el nombre de la versión se muestra como un vínculo.</span><span class="sxs-lookup"><span data-stu-id="42abc-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="42abc-310">Seleccione el vínculo para ver la página de resumen de la versión.</span><span class="sxs-lookup"><span data-stu-id="42abc-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="42abc-311">La página de resumen de la versión muestra detalles acerca de la versión.</span><span class="sxs-lookup"><span data-stu-id="42abc-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="42abc-312">En la siguiente captura de pantalla de "Release-2", en la sección **Environments** (Entornos), en la columna **Deployment status** (Estado de implementación) de Azure aparece el valor "IN PROGRESS" (EN CURSO) y el estado de Azure Stack Hub es "SUCCEEDED" (CORRECTO).</span><span class="sxs-lookup"><span data-stu-id="42abc-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="42abc-313">Cuando el estado de implementación para el entorno de Azure cambie a "Correcto", aparece un mensaje emergente que indica que la versión está lista para su aprobación.</span><span class="sxs-lookup"><span data-stu-id="42abc-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="42abc-314">Cuando una implementación está pendiente o se ha producido un error, se muestra un icono de información **(i)** azul.</span><span class="sxs-lookup"><span data-stu-id="42abc-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="42abc-315">Desplácese sobre el icono para ver un elemento emergente que contiene la razón del retraso o los errores.</span><span class="sxs-lookup"><span data-stu-id="42abc-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="42abc-316">Otras vistas, como la lista de versiones, también muestran un icono que indica que está pendiente la aprobación.</span><span class="sxs-lookup"><span data-stu-id="42abc-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="42abc-317">El elemento emergente para este icono muestra el nombre del entorno y más detalles relacionados con la implementación.</span><span class="sxs-lookup"><span data-stu-id="42abc-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="42abc-318">Los administradores pueden ver fácilmente el progreso general de las versiones, así como qué versiones están pendientes de aprobación.</span><span class="sxs-lookup"><span data-stu-id="42abc-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="42abc-319">Supervisión y seguimiento de implementaciones</span><span class="sxs-lookup"><span data-stu-id="42abc-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="42abc-320">En la página de resumen **Release-2**, seleccione **Registros**.</span><span class="sxs-lookup"><span data-stu-id="42abc-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="42abc-321">Durante una implementación, esta página muestra el registro en directo del agente.</span><span class="sxs-lookup"><span data-stu-id="42abc-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="42abc-322">El panel izquierdo muestra el estado de cada operación de la implementación para cada entorno.</span><span class="sxs-lookup"><span data-stu-id="42abc-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="42abc-323">Seleccione el icono de persona en la columna **Acción** correspondiente a una aprobación previa o posterior a la implementación para ver quién aprobó (o rechazó) la implementación y el mensaje que especificó.</span><span class="sxs-lookup"><span data-stu-id="42abc-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="42abc-324">Una vez finalizada la implementación, se muestra el archivo de registro completo en el panel derecho.</span><span class="sxs-lookup"><span data-stu-id="42abc-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="42abc-325">Seleccione cualquier **paso** en el panel izquierdo para ver el archivo de registro de un paso individual, como **Initialize Job** (Inicializar trabajo).</span><span class="sxs-lookup"><span data-stu-id="42abc-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="42abc-326">La posibilidad de ver registros individuales facilita realizar el seguimiento y la depuración de partes individuales de la implementación general.</span><span class="sxs-lookup"><span data-stu-id="42abc-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="42abc-327">**Guarde** el archivo de registro de un paso, o bien seleccione**Download all logs as zip** (Descargar todos los registros como zip).</span><span class="sxs-lookup"><span data-stu-id="42abc-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="42abc-328">Abra la pestaña **Resumen** para ver información general sobre la versión.</span><span class="sxs-lookup"><span data-stu-id="42abc-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="42abc-329">Esta vista muestra detalles de la compilación, los entornos en los que se implementó, el estado de implementación y otra información sobre la versión.</span><span class="sxs-lookup"><span data-stu-id="42abc-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="42abc-330">Seleccione un vínculo de entorno (**Azure** o **Azure Stack Hub**) para ver información sobre las implementaciones existentes y pendientes en un entorno específico.</span><span class="sxs-lookup"><span data-stu-id="42abc-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="42abc-331">Use estas vistas como una forma rápida de comprobar que la misma compilación se ha implementado en ambos entornos.</span><span class="sxs-lookup"><span data-stu-id="42abc-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="42abc-332">Abra la **aplicación de producción implementada** en un explorador.</span><span class="sxs-lookup"><span data-stu-id="42abc-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="42abc-333">Por ejemplo, para el sitio web de Azure App Services, abra la dirección URL `https://[your-app-name\].azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="42abc-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="42abc-334">La integración de Azure y Azure Stack Hub proporciona una solución escalable entre nubes</span><span class="sxs-lookup"><span data-stu-id="42abc-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="42abc-335">Un servicio de nube múltiple flexible y sólido proporciona seguridad de los datos, copia de seguridad y redundancia, disponibilidad coherente y rápida, almacenamiento y distribución escalables, y compatibilidad con el enrutamiento con replicación geográfica.</span><span class="sxs-lookup"><span data-stu-id="42abc-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="42abc-336">Este proceso desencadenado manualmente garantiza una conmutación confiable y eficaz entre las aplicaciones web hospedadas y una disponibilidad inmediata de los datos más importantes.</span><span class="sxs-lookup"><span data-stu-id="42abc-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="42abc-337">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="42abc-337">Next steps</span></span>

- <span data-ttu-id="42abc-338">Para más información sobre los patrones de nube de Azure, consulte [Patrones de diseño en la nube](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="42abc-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
