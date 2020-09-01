---
title: Dirigir el tráfico con una aplicación distribuida geográficamente mediante Azure y Azure Stack Hub
description: Aprenda a dirigir el tráfico a puntos de conexión concretos con una solución de aplicación distribuida geográficamente mediante Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 27d07070becfa902a715b451baae7c81c7e4b46f
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886839"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="e0a1d-103">Dirigir el tráfico con una aplicación distribuida geográficamente mediante Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="e0a1d-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="e0a1d-104">Aprenda a dirigir el tráfico a puntos de conexión específicos en función de varias métricas con el patrón de aplicaciones distribuidas geográficamente.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="e0a1d-105">La creación de un perfil de Traffic Manager con la configuración de puntos de conexión y enrutamiento geográfico garantiza que la información se enruta a puntos de conexión en función de los requisitos regionales, la legislación internacional y corporativa, y su necesidad en cuanto a los datos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="e0a1d-106">En esta solución, creará un entorno de ejemplo para:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e0a1d-107">Crear una aplicación distribuida geográficamente.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="e0a1d-108">Usar Traffic Manager para orientar la aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="e0a1d-109">Uso del patrón de aplicaciones distribuidas geográficamente</span><span class="sxs-lookup"><span data-stu-id="e0a1d-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="e0a1d-110">Con el patrón de distribución geográfica, la aplicación puede abarcar regiones.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="e0a1d-111">Puede usar de forma predeterminada la nube pública, pero algunos usuarios pueden requerir que sus datos permanezcan en su región.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="e0a1d-112">Puede dirigirlos a la nube más adecuada en función de sus requisitos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="e0a1d-113">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="e0a1d-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="e0a1d-114">Consideraciones sobre escalabilidad</span><span class="sxs-lookup"><span data-stu-id="e0a1d-114">Scalability considerations</span></span>

<span data-ttu-id="e0a1d-115">La solución que va a crear con este artículo no admite la escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="e0a1d-116">Sin embargo, si se utiliza en combinación con otras soluciones de Azure y locales, puede dar cabida a los requisitos de escalabilidad.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="e0a1d-117">Para obtener información acerca de cómo crear una solución híbrida con ajuste de escala automático a través de Traffic Manager, consulte [Creación de soluciones de escalado en toda la nube con Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="e0a1d-118">Consideraciones sobre disponibilidad</span><span class="sxs-lookup"><span data-stu-id="e0a1d-118">Availability considerations</span></span>

<span data-ttu-id="e0a1d-119">Como sucede con las consideraciones sobre escalabilidad, esta solución no aborda directamente la disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="e0a1d-120">Sin embargo, las soluciones locales y de Azure se pueden implementar dentro de esta solución para garantizar una alta disponibilidad para todos los componentes implicados.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="e0a1d-121">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="e0a1d-121">When to use this pattern</span></span>

- <span data-ttu-id="e0a1d-122">Su organización oficinas internacionales que requieren una seguridad regional personalizada y directivas de distribución.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="e0a1d-123">Cada una de las oficinas de su organización extrae datos de los empleados, el negocio y las instalaciones, que requieren informes de actividad para cada normativa local y zona horaria.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="e0a1d-124">Para cumplir los requisitos de gran escala, es necesario escalar horizontalmente las aplicaciones mediante varias implementaciones de la aplicación en una única región, así como entre regiones, para afrontar los requisitos extremos de carga.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="e0a1d-125">Planeación de la topología</span><span class="sxs-lookup"><span data-stu-id="e0a1d-125">Planning the topology</span></span>

<span data-ttu-id="e0a1d-126">Antes de crear el entorno de una aplicación distribuida, resulta útil conocer lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="e0a1d-127">**Dominio personalizado para la aplicación:** ¿cuál es el nombre de dominio personalizado que los clientes usarán para acceder a la aplicación?</span><span class="sxs-lookup"><span data-stu-id="e0a1d-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="e0a1d-128">En el caso de la aplicación de ejemplo, el nombre de dominio personalizado es *www\.scalableasedemo.com*.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="e0a1d-129">**Dominio de Traffic Manager:** se elige nombre de dominio al crear un [perfil de Azure Traffic Manager](/azure/traffic-manager/traffic-manager-manage-profiles).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="e0a1d-130">Este nombre se combina con el sufijo *trafficmanager.net* para registrar una entrada de dominio que administra Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="e0a1d-131">Para la aplicación de ejemplo, el nombre elegido es *scalable-ase-demo*.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="e0a1d-132">Como resultado, el nombre de dominio completo administrado por Traffic Manager es *scalable-ase-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="e0a1d-133">**Estrategia para escalar la superficie de la aplicación:** Decida si la superficie de la aplicación se va a distribuir entre varios entornos de App Service Environment de una sola región, de varias regiones o de una mezcla de ambas opciones.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="e0a1d-134">La decisión debería basarse en las expectativas de dónde se vaya a originar el tráfico del cliente y de la escalabilidad del resto de la infraestructura de back-end complementaria de una aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="e0a1d-135">Por ejemplo, en el caso de una aplicación totalmente sin estado, se puede escalar una aplicación de forma masiva mediante una combinación de varios entornos de App Service Environment por región de Azure, multiplicados por los entornos de App Service Environment implementados en varias regiones de Azure.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="e0a1d-136">Con más de 15 regiones de Azure globales entre las que elegir, los clientes pueden realmente crear una superficie de aplicación de gran escala en todo el mundo.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="e0a1d-137">En el caso de la aplicación de ejemplo que se usa aquí, se crearon tres entornos de App Service Environment en una sola región de Azure (Centro-sur de EE. UU.).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="e0a1d-138">**Convención de nomenclatura de los entornos de App Service Environment:** cada entorno de App Service Environment requiere un nombre único.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="e0a1d-139">Si hay más de uno o dos entornos de App Service Environment, resulta útil disponer de una convención de nomenclatura que ayude a identificar cada uno de ellos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="e0a1d-140">Para la aplicación de ejemplo que se usa aquí, se usó una convención de nomenclatura sencilla.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="e0a1d-141">Los nombres de los tres entornos de App Service Environment son *fe1ase*, *fe2ase* y *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="e0a1d-142">**Convención de nomenclatura para las aplicaciones:** dado que se van a implementar varias instancias de la aplicación, se necesita un nombre para cada instancia de la aplicación implementada.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="e0a1d-143">Con el entorno de App Service Environment para Power Apps, el mismo nombre de aplicación se puede usar en varios entornos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="e0a1d-144">Dado que cada uno tiene un sufijo de dominio único, los desarrolladores pueden reutilizar el mismo nombre de aplicación en cada entorno.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="e0a1d-145">Por ejemplo, un desarrollador podría asignar los siguientes nombres a las aplicaciones:*miapp.foo1.p.azurewebsites.net*, *miapp.foo2.p.azurewebsites.net*, *miapp.foo3.p.azurewebsites.net, etc*.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="e0a1d-146">Para la aplicación que se usa aquí, cada instancia de la aplicación tiene un nombre único.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="e0a1d-147">Los nombres de las instancias de aplicación usados son *webfrontend1*, *webfrontend2* y *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="e0a1d-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="e0a1d-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="e0a1d-149">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="e0a1d-150">Azure Stack Hub aporta la agilidad y la innovación de la informática en la nube a su entorno local y hace posible la única nube híbrida que le permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="e0a1d-151">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los fundamentos de calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="e0a1d-152">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="e0a1d-153">Parte 1: Crear una aplicación distribuida geográficamente</span><span class="sxs-lookup"><span data-stu-id="e0a1d-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="e0a1d-154">En esta parte, creará una aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e0a1d-155">Crear y publicar aplicaciones web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="e0a1d-156">Agregar código a Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="e0a1d-157">Dirigir la compilación de la aplicación a varios destinos en la nube.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="e0a1d-158">Administración y configuración del proceso de CD.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e0a1d-159">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="e0a1d-159">Prerequisites</span></span>

<span data-ttu-id="e0a1d-160">Se requieren una suscripción de Azure y la instalación de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="e0a1d-161">Pasos para una aplicación distribuida geográficamente</span><span class="sxs-lookup"><span data-stu-id="e0a1d-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="e0a1d-162">Obtención de un dominio personalizado y configuración de DNS</span><span class="sxs-lookup"><span data-stu-id="e0a1d-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="e0a1d-163">Actualice el archivo de zona DNS para el dominio.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="e0a1d-164">Azure AD puede entonces comprobar la propiedad del nombre de dominio personalizado.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="e0a1d-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) para los registros DNS de Azure/Microsoft 365/externo dentro de Azure, o bien agregue la entrada DNS a [un registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="e0a1d-166">Registre un dominio personalizado con un registrador público.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="e0a1d-167">Inicie sesión en el registrador de nombres de dominio para el dominio.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="e0a1d-168">Puede que se requiera un administrador autorizado para realizar las actualizaciones de DNS.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="e0a1d-169">Actualice el archivo de zona DNS para el dominio. Para ello, agregue la entrada DNS que Azure AD proporcione.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="e0a1d-170">La entrada DNS no cambiará los comportamientos como el enrutamiento de correo o el hospedaje web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="e0a1d-171">Creación y publicación de aplicaciones web</span><span class="sxs-lookup"><span data-stu-id="e0a1d-171">Create web apps and publish</span></span>

<span data-ttu-id="e0a1d-172">Configure la canalización de integración y entrega continuas (CI/CD) híbrida para implementar la aplicación web en Azure y Azure Stack Hub e insertar automáticamente los cambios en ambas nubes.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="e0a1d-173">Se requiere Azure Stack Hub con las imágenes adecuadas sindicadas para ejecutarse (Windows Server y SQL) y la implementación de App Service.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="e0a1d-174">Para más información, consulte [Requisitos previos para implementar App Service en Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="e0a1d-175">Adición de código a Azure Repos</span><span class="sxs-lookup"><span data-stu-id="e0a1d-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="e0a1d-176">Inicie sesión en Visual Studio con **una cuenta que tenga derechos de creación de proyectos** en Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="e0a1d-177">La CI/CD se puede aplicar al código de aplicación y al código de infraestructura.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="e0a1d-178">Use [plantillas de Azure Resource Manager](https://azure.microsoft.com/resources/templates/) tanto para el desarrollo en la nube hospedado como para el privado.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Conectarse a un proyecto en Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="e0a1d-180">**Clone el repositorio**; para ello, cree y abra la aplicación web predeterminada.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Clonar el repositorio en Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="e0a1d-182">Creación de una implementación de aplicaciones web en ambas nubes</span><span class="sxs-lookup"><span data-stu-id="e0a1d-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="e0a1d-183">Edite el archivo **WebApplication.csproj**: Seleccione `Runtimeidentifier` y agregue `win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="e0a1d-184">(Consulte la documentación de [Implementaciones autocontenidas](/dotnet/core/deploying/deploy-with-vs#simpleSelf)).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Edición del archivo de proyecto de aplicación web en Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="e0a1d-186">**Inserte el código en Azure Repos** mediante Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="e0a1d-187">Confirme que el **código de la aplicación** se ha insertado en Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="e0a1d-188">Creación de la definición de compilación</span><span class="sxs-lookup"><span data-stu-id="e0a1d-188">Create the build definition</span></span>

1. <span data-ttu-id="e0a1d-189">**Inicie sesión en Azure Pipelines** para confirmar la posibilidad de crear definiciones de compilación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="e0a1d-190">Agregue el código `-r win10-x64`.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="e0a1d-191">Esta adición es necesaria para activar una implementación autocontenida con .Net Core.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Incorporación de código a la definición de compilación en Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="e0a1d-193">**Ejecute la compilación**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-193">**Run the build**.</span></span> <span data-ttu-id="e0a1d-194">El proceso de [compilación de implementación autocontenida](/dotnet/core/deploying/deploy-with-vs#simpleSelf) publicará los artefactos que se pueden ejecutar en Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="e0a1d-195">Uso de un agente hospedado de Azure</span><span class="sxs-lookup"><span data-stu-id="e0a1d-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="e0a1d-196">El uso de un agente hospedado en Azure Pipelines es una opción adecuada para compilar e implementar aplicaciones web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="e0a1d-197">Microsoft Azure realiza automáticamente el mantenimiento y las actualizaciones, lo que permite el desarrollo, la prueba y la implementación de forma continua e ininterrumpida.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="e0a1d-198">Administración y configuración del proceso de CD</span><span class="sxs-lookup"><span data-stu-id="e0a1d-198">Manage and configure the CD process</span></span>

<span data-ttu-id="e0a1d-199">Azure DevOps Services proporcionan una canalización con una gran capacidad de configuración y administración para versiones de varios entornos, como desarrollo, ensayo, control de calidad (QA) y producción, lo que incluye las aprobaciones necesarias en etapas específicas.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="e0a1d-200">Creación de la definición de versión</span><span class="sxs-lookup"><span data-stu-id="e0a1d-200">Create release definition</span></span>

1. <span data-ttu-id="e0a1d-201">Seleccione el botón con el **signo más** para agregar una versión nueva en la pestaña **Releases** (Versiones) de la sección **Build and Release** (Compilación y versión) de Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Creación de una definición de versión en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="e0a1d-203">Aplique la plantilla Implementación de Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-203">Apply the Azure App Service Deployment template.</span></span>

   ![Aplicación de la plantilla Implementación de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="e0a1d-205">En **Agregar artefacto**, agregue el artefacto para la aplicación de compilación de nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Incorporación de un artefacto a la compilación de Azure Cloud en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="e0a1d-207">En la pestaña Canalización, seleccione el vínculo **Fase, Tarea** del entorno y establezca los valores del entorno de nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Establecimiento de los valores del entorno de Azure Cloud en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="e0a1d-209">Establezca el **nombre del entorno** y seleccione la **suscripción de Azure** como el punto de conexión de la nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Selección de la suscripción de Azure para el punto de conexión de Azure Cloud en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="e0a1d-211">En **Nombre de App Service**, establezca el nombre del servicio de aplicación de Azure necesario.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Establecimiento del nombre de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="e0a1d-213">Escriba "Hospedado VS2017" en **Cola de agentes** para el entorno hospedado en la nube de Azure.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Establecimiento de la cola de agentes para el entorno hospedado de Azure Cloud en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="e0a1d-215">En el menú de implementación de Azure App Service, seleccione el **paquete o carpeta** válidos para el entorno.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="e0a1d-216">Seleccione **Aceptar** para la **ubicación de carpeta**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-216">Select **OK** to **folder location**.</span></span>
  
      ![Selección del paquete o la carpeta del entorno de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selección del paquete o la carpeta del entorno de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="e0a1d-219">Guarde todos los cambios y vuelva a la **canalización de versión**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Guardar cambios en la canalización de versión en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="e0a1d-221">Agregue un nuevo artefacto mediante la selección de la compilación para la aplicación de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Incorporación de un nuevo artefacto para la aplicación de Azure Stack Hub en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="e0a1d-223">Agregue un entorno más; para ello, aplique Implementación de Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Incorporación de un entorno a Implementación de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="e0a1d-225">Asigne al nuevo entorno el nombre Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-225">Name the new environment Azure Stack Hub.</span></span>

    ![Asignación de un nombre a un entorno en Implementación de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="e0a1d-227">Busque el entorno de Azure Stack Hub en la pestaña **Tarea**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Entorno de Azure Stack Hub en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="e0a1d-229">Seleccione la suscripción para el punto de conexión de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Selección de la suscripción para el punto de conexión de Azure Stack Hub en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="e0a1d-231">Establezca el nombre de la aplicación web de Azure Stack Hub como el nombre del servicio de aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Establecimiento del nombre de la aplicación web de Azure Stack Hub en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="e0a1d-233">Seleccione el agente de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-233">Select the Azure Stack Hub agent.</span></span>

    ![Selección del agente de Azure Stack Hub en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="e0a1d-235">En la sección de implementación de Azure App Service, seleccione el **paquete o carpeta** válidos para el entorno.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="e0a1d-236">Seleccione **Aceptar** para la ubicación de carpeta.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-236">Select **OK** to folder location.</span></span>

    ![Selección de la carpeta de Implementación de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selección de la carpeta de Implementación de Azure App Service en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="e0a1d-239">En la pestaña Variable, agregue una variable denominada `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, establezca su valor como **true** y defina el ámbito en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Incorporación de una variable a Implementación de aplicación de Azure AD en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="e0a1d-241">Seleccione el icono del desencadenador de implementación **Continuo** en ambos artefactos y habilite el desencadenador de implementación **Continua**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Selección del desencadenador de implementación continua en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="e0a1d-243">Seleccione el icono de condiciones **previas a la implementación** en el entorno de Azure Stack Hub y establezca el desencadenador en **Tras el lanzamiento**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Selección de las condiciones anteriores a la implementación en Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="e0a1d-245">Guarde todos los cambios.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="e0a1d-246">Algunos valores de configuración de las tareas se han definido automáticamente como [variables de entorno](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) cuando se creó una definición de versión desde una plantilla.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="e0a1d-247">Estos valores de configuración no se pueden modificar en la configuración de tareas; para hacerlo, debe seleccionar el elemento del entorno principal.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="e0a1d-248">Parte 2: Actualizar las opciones de la aplicación web</span><span class="sxs-lookup"><span data-stu-id="e0a1d-248">Part 2: Update web app options</span></span>

<span data-ttu-id="e0a1d-249">[Azure App Service](/azure/app-service/overview) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="e0a1d-251">Asigne un nombre DNS personalizado a Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="e0a1d-252">Use un **registro CNAME** o un **registro A** para asignar un nombre DNS personalizado a App Service.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="e0a1d-253">Asignar un nombre DNS personalizado a Azure Web Apps</span><span class="sxs-lookup"><span data-stu-id="e0a1d-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="e0a1d-254">Se recomienda usar un CNAME para todos los nombres DNS personalizados, excepto un dominio raíz (por ejemplo, northwind.com).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="e0a1d-255">Para migrar un sitio en vivo y su nombre de dominio DNS a App Service, consulte [Migración de un nombre DNS activo a Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e0a1d-256">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="e0a1d-256">Prerequisites</span></span>

<span data-ttu-id="e0a1d-257">Para completar esta solución:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-257">To complete this solution:</span></span>

- <span data-ttu-id="e0a1d-258">[Cree una aplicación de App Service](/azure/app-service/) o use alguna aplicación que se haya creado para otra solución.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="e0a1d-259">Compre un nombre de dominio y asegure el acceso al registro DNS para el proveedor de dominio.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="e0a1d-260">Actualice el archivo de zona DNS para el dominio.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="e0a1d-261">Azure AD comprobará la propiedad del nombre de dominio personalizado.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="e0a1d-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) para los registros DNS de Azure/Microsoft 365/externo dentro de Azure, o bien agregue la entrada DNS a [un registrador DNS diferente](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="e0a1d-263">Registre un dominio personalizado con un registrador público.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="e0a1d-264">Inicie sesión en el registrador de nombres de dominio para el dominio.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="e0a1d-265">(Puede que se requiera un administrador autorizado para realizar las actualizaciones de DNS).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="e0a1d-266">Actualice el archivo de zona DNS para el dominio. Para ello, agregue la entrada DNS que Azure AD proporcione.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="e0a1d-267">Por ejemplo, para agregar entradas DNS a northwindcloud.com y www\.northwindcloud.com, configure los valores de DNS del dominio raíz northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="e0a1d-268">Un nombre de dominio puede adquirirse mediante [Azure Portal](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="e0a1d-269">Para asignar un nombre DNS personalizado a una aplicación web, el [plan de App Service](https://azure.microsoft.com/pricing/details/app-service/) de dicha aplicación debe ser un nivel de pago (**Compartido**, **Básico**, **Estándar** o **Premium**).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="e0a1d-270">Creación y asignación de los registros D y CNAME</span><span class="sxs-lookup"><span data-stu-id="e0a1d-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="e0a1d-271">Acceso a los registros DNS con el proveedor de dominios</span><span class="sxs-lookup"><span data-stu-id="e0a1d-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="e0a1d-272">Puede utilizar Azure DNS para configurar un nombre DNS personalizado para Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="e0a1d-273">Para más información, consulte [Usar Azure DNS para proporcionar la configuración de un dominio personalizado para un servicio de Azure](/azure/dns/dns-custom-domain).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="e0a1d-274">Inicie sesión en el sitio web de su proveedor principal.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="e0a1d-275">Busque la página de administración de registros DNS.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="e0a1d-276">Cada proveedor de dominio tiene su propia interfaz de registros DNS.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="e0a1d-277">Busque áreas del sitio etiquetadas como **Nombre de dominio**, **DNS** o **Administración del servidor del nombres**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="e0a1d-278">La página de registros DNS puede verse en **Mis dominios**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="e0a1d-279">Busque el vínculo denominado **Archivo de zona**, **Registros DNS** o **Configuración avanzada**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="e0a1d-280">La captura de pantalla siguiente es un ejemplo de página de registros DNS:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-280">The following screenshot is an example of a DNS records page:</span></span>

![Página de registros DNS de ejemplo](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="e0a1d-282">En el registrador de nombres de dominio, seleccione **Agregar o crear** para crear un registro.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="e0a1d-283">Algunos proveedores tienen diferentes vínculos para agregar diferentes tipos de registros.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="e0a1d-284">Consulte la documentación del proveedor.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="e0a1d-285">Agregue un registro CNAME para asignar un subdominio al nombre de host predeterminado de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="e0a1d-286">Para el ejemplo del dominio www\.northwindcloud.com, agregue un registro CNAME que asigne el nombre a `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="e0a1d-287">Después de agregar el registro CNAME, la página de registros DNS es como la del ejemplo siguiente:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Navegación en el portal a la aplicación de Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="e0a1d-289">Habilitación de la asignación de registros CNAME en Azure</span><span class="sxs-lookup"><span data-stu-id="e0a1d-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="e0a1d-290">En otra pestaña, inicie sesión en Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="e0a1d-291">Vaya a App Services.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-291">Go to App Services.</span></span>

3. <span data-ttu-id="e0a1d-292">Seleccione la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-292">Select web app.</span></span>

4. <span data-ttu-id="e0a1d-293">En el panel de navegación izquierdo de la página de la aplicación en Azure Portal, seleccione **Dominios personalizados**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="e0a1d-294">Seleccione el icono **+** situado junto a **Agregar nombre de host**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="e0a1d-295">Escriba el nombre de dominio completo, como por ejemplo `www.northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="e0a1d-296">Seleccione **Validar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-296">Select **Validate**.</span></span>

8. <span data-ttu-id="e0a1d-297">Si se indica, agregue más registros de otros tipos (`A` o `TXT`) a los registros DNS de los registradores de nombres de dominio.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="e0a1d-298">Azure proporcionará los valores y los tipos de estos registros:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="e0a1d-299">a.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-299">a.</span></span>  <span data-ttu-id="e0a1d-300">Un registro **D** que se asigna a la dirección IP de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="e0a1d-301">b.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-301">b.</span></span>  <span data-ttu-id="e0a1d-302">Un registro **TXT** que se asigna al nombre de host predeterminado de la aplicación `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="e0a1d-303">App Service usa este registro solo durante la configuración para confirmar la propiedad del dominio personalizado.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="e0a1d-304">Después de la confirmación, elimine el registro TXT.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="e0a1d-305">Complete esta tarea en la pestaña del registrador de dominio y vuelva a validar hasta que el botón **Agregar nombre de host** se active.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="e0a1d-306">Asegúrese de que en **Tipo de registro de nombre de host** está seleccionado **CNAME** (www.example.com o cualquier subdominio).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="e0a1d-307">Seleccione **Agregar nombre de host**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="e0a1d-308">Escriba el nombre de dominio completo, como por ejemplo `northwindcloud.com`.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="e0a1d-309">Seleccione **Validar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-309">Select **Validate**.</span></span> <span data-ttu-id="e0a1d-310">Se activa **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="e0a1d-311">Asegúrese de que el **Tipo de registro de nombre de host** esté establecido en **Registro A** (example.com).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="e0a1d-312">**Agregue un nombre de host**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-312">**Add hostname**.</span></span>

    <span data-ttu-id="e0a1d-313">El nuevo nombre de host puede tardar algo en reflejarse en la página **Dominios personalizados** de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="e0a1d-314">Intente actualizar el explorador para actualizar los datos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-314">Try refreshing the browser to update the data.</span></span>
  
    ![Dominios personalizados](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="e0a1d-316">Si se produce un error, aparecerá una notificación de error de comprobación en la parte inferior de la página.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Error de comprobación de dominio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="e0a1d-318">Se pueden repetir los pasos anteriores para asignar un dominio con comodín (\*.northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="e0a1d-319">Esto permite la adición de subdominios adicionales a este servicio de aplicaciones sin tener que crear un registro CNAME independiente para cada uno.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="e0a1d-320">Siga las instrucciones del registrador para establecer esta configuración.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="e0a1d-321">Prueba en un explorador</span><span class="sxs-lookup"><span data-stu-id="e0a1d-321">Test in a browser</span></span>

<span data-ttu-id="e0a1d-322">Desplácese a los nombres DNS que configuró anteriormente (por ejemplo, `northwindcloud.com` o `www.northwindcloud.com`).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="e0a1d-323">Parte 3: Enlazar un certificado SSL personalizado</span><span class="sxs-lookup"><span data-stu-id="e0a1d-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="e0a1d-324">En esta parte, seguiremos estos pasos:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e0a1d-325">Se enlazará el certificado SSL personalizado con App Service.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="e0a1d-326">Se aplicará HTTPS para la aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="e0a1d-327">Automatizar el enlace de certificado SSL con scripts.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="e0a1d-328">Si es necesario, se obtendrá un certificado SSL de cliente en Azure Portal y se enlazará a la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="e0a1d-329">Para obtener más información, consulte el [tutorial sobre App Service Certificates](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e0a1d-330">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="e0a1d-330">Prerequisites</span></span>

<span data-ttu-id="e0a1d-331">Para realizar esta solución:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-331">To complete this  solution:</span></span>

- <span data-ttu-id="e0a1d-332">[Cree una aplicación de App Service](/azure/app-service/).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-332">[Create an App Service app.](/azure/app-service/)</span></span>
- [<span data-ttu-id="e0a1d-333">Asigne un nombre DNS personalizado a la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="e0a1d-334">Adquiera un certificado SSL de una entidad de certificación de confianza y use la clave para firmar la solicitud.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="e0a1d-335">Requisitos para el certificado SSL</span><span class="sxs-lookup"><span data-stu-id="e0a1d-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="e0a1d-336">Para usar un certificado en App Service, el certificado debe cumplir los siguientes requisitos:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="e0a1d-337">Estar firmado por una entidad de certificación de confianza.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="e0a1d-338">Haberse exportado como archivo PFX protegido por contraseña.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="e0a1d-339">Contener una clave privada con una longitud de al menos 2048 bits.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="e0a1d-340">Contener todos los certificados intermedios de la cadena de certificados.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="e0a1d-341">Los **certificados de criptografía de curva elíptica (ECC)** funcionan con App Service, pero están fuera del ámbito de esta guía.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="e0a1d-342">Consulte una entidad de certificación para obtener ayuda en la creación de certificados ECC.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="e0a1d-343">Preparación de la aplicación web</span><span class="sxs-lookup"><span data-stu-id="e0a1d-343">Prepare the web app</span></span>

<span data-ttu-id="e0a1d-344">Para enlazar un certificado SSL personalizado a la aplicación web, su [plan de App Service](https://azure.microsoft.com/pricing/details/app-service/) debe estar en el nivel **Básico**, **Estándar** o **Premium**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="e0a1d-345">Inicio de sesión en Azure</span><span class="sxs-lookup"><span data-stu-id="e0a1d-345">Sign in to Azure</span></span>

1. <span data-ttu-id="e0a1d-346">Abra [Azure Portal](https://portal.azure.com/) y vaya a la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="e0a1d-347">En el menú izquierdo, seleccione **App Services** y, después, el nombre de la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Selección de una aplicación web en Azure Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="e0a1d-349">Comprobar el plan de tarifa</span><span class="sxs-lookup"><span data-stu-id="e0a1d-349">Check the pricing tier</span></span>

1. <span data-ttu-id="e0a1d-350">En el panel de navegación izquierdo de la página de la aplicación web, desplácese a la sección **Configuración** y seleccione **Escalar verticalmente (plan de App Service)** .</span><span class="sxs-lookup"><span data-stu-id="e0a1d-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menú Escalar verticalmente en una aplicación web](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="e0a1d-352">Asegúrese de que la aplicación web no está en el nivel **Gratis** ni **Compartido**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="e0a1d-353">El nivel actual de la aplicación web aparece resaltado en un cuadro azul oscuro.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Comprobación del plan de tarifa en la aplicación web](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="e0a1d-355">El SSL personalizado no es compatible con los niveles **Gratis** y **Compartido**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="e0a1d-356">Para escalar, siga los pasos de la sección siguiente o, en la página **Elija su plan de tarifa**, vaya directamente a las secciones sobre cómo [cargar y enlazar el certificado SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="e0a1d-357">Escalar verticalmente el plan de App Service</span><span class="sxs-lookup"><span data-stu-id="e0a1d-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="e0a1d-358">Seleccione uno de los niveles, **Básico**, **Estándar** o **Premium**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="e0a1d-359">Elija **Seleccionar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-359">Select **Select**.</span></span>

![Elección del plan de tarifa de la aplicación web](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="e0a1d-361">La operación de escalado está completa cuando se muestra la notificación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-361">The scale operation is complete when notification is displayed.</span></span>

![Notificación de escalado vertical](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="e0a1d-363">Enlace del certificado SSL y combinación de certificados intermedios</span><span class="sxs-lookup"><span data-stu-id="e0a1d-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="e0a1d-364">Combine varios certificados en la cadena.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="e0a1d-365">**Abra cada certificado** que ha recibido en un editor de texto.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="e0a1d-366">Cree un archivo para el certificado combinado denominado *mergedcertificate.crt*.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="e0a1d-367">En un editor de texto, copie el contenido de cada certificado en este archivo.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="e0a1d-368">Los certificados deben seguir el orden de la cadena de certificados, comenzando por el certificado y terminando por el certificado raíz.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="e0a1d-369">Debe ser similar al ejemplo siguiente:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="e0a1d-370">Exportar el certificado a PFX</span><span class="sxs-lookup"><span data-stu-id="e0a1d-370">Export certificate to PFX</span></span>

<span data-ttu-id="e0a1d-371">Exporte el certificado SSL combinado con la clave privada generada por el certificado.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="e0a1d-372">Se crea un archivo de clave privada a través de OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="e0a1d-373">Para exportar el certificado a PFX, ejecute el comando siguiente y reemplace los marcadores de posición `<private-key-file>` y `<merged-certificate-file>` por la ruta de acceso de la clave privada y el archivo de certificado combinado:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="e0a1d-374">Cuando se le pida, defina una contraseña de exportación para cargar el certificado SSL a Azure App Service más adelante.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="e0a1d-375">Cuando se usan IIS o **Certreq.exe** para generar la solicitud de certificado, instale el certificado en una máquina local y luego [exporte el certificado a PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="e0a1d-376">Carga del certificado SSL</span><span class="sxs-lookup"><span data-stu-id="e0a1d-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="e0a1d-377">Seleccione **Configuración de SSL** en la navegación del lado izquierdo de la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="e0a1d-378">Seleccione **Cargar certificado**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="e0a1d-379">En **Archivo de certificado PFX**, seleccione el archivo PFX.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="e0a1d-380">En **Contraseña del certificado**, escriba la contraseña que se creó al exportar el archivo PFX.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="e0a1d-381">Seleccione **Cargar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-381">Select **Upload**.</span></span>

    ![Carga del certificado SSL](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="e0a1d-383">Cuando App Service termina de cargar el certificado, este aparece en la página **Configuración de SSL**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Configuración de SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="e0a1d-385">Enlazar el certificado SSL</span><span class="sxs-lookup"><span data-stu-id="e0a1d-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="e0a1d-386">En la sección **Enlaces SSL**, seleccione **Agregar enlaces**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="e0a1d-387">Si se ha cargado el certificado, pero no aparece en los nombres de dominio en la lista desplegable **Nombre de host**, pruebe a actualizar la página del explorador.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="e0a1d-388">En la página **Agregar enlace SSL**, use las listas desplegables para seleccionar el nombre de dominio que se va a proteger, así como el certificado va a utilizar.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="e0a1d-389">En **Tipo de SSL**, seleccione si se va a usar [**Indicación de nombre de servidor (SNI)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) o SSL basada en IP.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="e0a1d-390">**SSL basada en SNI**: Se pueden agregar varios enlaces SSL basados en SNI.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="e0a1d-391">Esta opción permite que varios certificados SSL protejan varios dominios en una misma dirección IP.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="e0a1d-392">Los exploradores más modernos (como Internet Explorer, Chrome, Firefox y Opera) admiten SNI (encontrará información de compatibilidad con exploradores más completa en [Indicación de nombre de servidor](https://wikipedia.org/wiki/Server_Name_Indication)).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="e0a1d-393">**SSL basada en IP**: solo puede agregarse un enlaces SSL basado en IP.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="e0a1d-394">Esta opción solo permite que un único certificado SSL proteja una dirección IP dedicada.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="e0a1d-395">Para proteger varios dominios, debe usar el mismo certificado SSL.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="e0a1d-396">La SSL basada en IP es la opción tradicional para enlaces SSL.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="e0a1d-397">Haga clic en **Agregar enlace**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-397">Select **Add Binding**.</span></span>

    ![Agregar enlace SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="e0a1d-399">Cuando App Service termina de cargar el certificado, este aparece en la sección **Enlaces SSL**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Los enlaces SSL han finalizado la carga](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="e0a1d-401">Reasignación del registro D en SSL de IP</span><span class="sxs-lookup"><span data-stu-id="e0a1d-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="e0a1d-402">Si la SSL basada en IP no se usa en la aplicación web, vaya a [Probar HTTPS para el dominio personalizado](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="e0a1d-403">De manera predeterminada, la aplicación web usa una dirección IP pública compartida.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="e0a1d-404">Cuando se enlaza un certificado con SSL basada en IP, App Service crea otra dirección IP dedicada para la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="e0a1d-405">Cuando un registro D se asigna a la aplicación web, el registro de dominio debe actualizarse con la dirección IP dedicada.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="e0a1d-406">La página **Dominio personalizado** se actualiza con la nueva dirección IP dedicada.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="e0a1d-407">Copie esta [dirección IP](/azure/app-service/app-service-web-tutorial-custom-domain) y luego reasigne el [registro D](/azure/app-service/app-service-web-tutorial-custom-domain) a esta nueva dirección IP.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="e0a1d-408">Probar HTTPS</span><span class="sxs-lookup"><span data-stu-id="e0a1d-408">Test HTTPS</span></span>

<span data-ttu-id="e0a1d-409">En diferentes exploradores, vaya a `https://<your.custom.domain>` para asegurarse de que se atiende la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Navegación a la aplicación web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="e0a1d-411">Si se producen errores de validación de certificado, la causa podría ser que hayan quedado un certificado autofirmado o certificados intermedios al exportar al archivo PFX.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="e0a1d-412">Aplicación de HTTPS</span><span class="sxs-lookup"><span data-stu-id="e0a1d-412">Enforce HTTPS</span></span>

<span data-ttu-id="e0a1d-413">De forma predeterminada, cualquier usuario puede acceder a la aplicación web mediante HTTP.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="e0a1d-414">Todas las solicitudes HTTP al puerto HTTPS se pueden redirigir.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="e0a1d-415">En la página de la aplicación web, seleccione **Configuración de SL**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="e0a1d-416">A continuación, en **Solo HTTPS**, seleccione **On**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Aplicación de HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="e0a1d-418">Una vez completada la operación, vaya a cualquiera de las direcciones URL HTTP que apuntan a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="e0a1d-419">Por ejemplo:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-419">For example:</span></span>

- <span data-ttu-id="e0a1d-420">https://<app_name>.azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="e0a1d-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="e0a1d-421">Aplicación de TLS 1.1 y 1.2</span><span class="sxs-lookup"><span data-stu-id="e0a1d-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="e0a1d-422">La aplicación permite [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 de forma predeterminada, lo cual, según los estándares del sector (como [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)), ya no se considera seguro.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="e0a1d-423">Para aplicar las versiones posteriores de TLS, siga estos pasos:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="e0a1d-424">En la página de la aplicación web, en el panel de navegación izquierdo, seleccione **Configuración de SSL**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="e0a1d-425">En **Versión de TLS**, seleccione la versión mínima de TLS que desee.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Exigir aplicación de TLS 1.1 o 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="e0a1d-427">Crear un perfil de Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="e0a1d-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="e0a1d-428">Seleccione **Crear un recurso** > **Redes** > **Perfil de Traffic Manager** > **Crear**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="e0a1d-429">En **Crear perfil de Traffic Manager**, complete los campos de la manera siguiente:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="e0a1d-430">En **Nombre**, proporcione un nombre para el perfil.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="e0a1d-431">Este nombre debe ser único en la zona traffic manager.net y genera el nombre DNS, trafficmanager.net, que se usa para acceder al perfil de Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="e0a1d-432">En **Método de enrutamiento**, seleccione el **Método de enrutamiento geográfico**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="e0a1d-433">En **Suscripción**, seleccione la suscripción en la que desea crear este perfil.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="e0a1d-434">En **Grupo de recursos**, cree un grupo de recursos nuevo en el cual colocar este perfil.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="e0a1d-435">En **Ubicación del grupo de recursos**, seleccione la ubicación del grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="e0a1d-436">Esta configuración hace referencia a la ubicación del grupo de recursos y no tiene efecto alguno sobre el perfil de Traffic Manager que se implementa globalmente.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="e0a1d-437">Seleccione **Crear**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-437">Select **Create**.</span></span>

    7. <span data-ttu-id="e0a1d-438">Una vez que se complete la implementación global del perfil de Traffic Manager, aparecerá en el grupo de recursos respectivo como uno de los recursos.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Grupos de recursos para crear un perfil de Traffic Manager](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="e0a1d-440">Incorporación de puntos de conexión de Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="e0a1d-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="e0a1d-441">En la barra de búsqueda del portal, busque el nombre del **perfil de Traffic Manager** que creó en la sección anterior y seleccione el perfil en los resultados que aparecen.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="e0a1d-442">En **Perfil de Traffic Manager**, en la sección **Configuración**, seleccione **Puntos de conexión**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="e0a1d-443">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-443">Select **Add**.</span></span>

4. <span data-ttu-id="e0a1d-444">Incorporación del punto de conexión de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="e0a1d-445">En **Tipo**, seleccione **Punto de conexión externo**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="e0a1d-446">Proporcione un **nombre** para este punto de conexión: la mejor opción es el nombre de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="e0a1d-447">En el nombre de dominio completo (**FQDN**), use la dirección URL externa de la aplicación web de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="e0a1d-448">En la asignación geográfica, seleccione la región o continente donde se encuentra el recurso.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="e0a1d-449">Por ejemplo, **Europa.**</span><span class="sxs-lookup"><span data-stu-id="e0a1d-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="e0a1d-450">En la lista desplegable de países y regiones que aparece, seleccione el país que se aplica a este punto de conexión</span><span class="sxs-lookup"><span data-stu-id="e0a1d-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="e0a1d-451">Por ejemplo, **Alemania**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="e0a1d-452">No active la opción **Agregar como deshabilitado**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="e0a1d-453">Seleccione **Aceptar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-453">Select **OK**.</span></span>

12. <span data-ttu-id="e0a1d-454">Incorporación del punto de conexión de Azure:</span><span class="sxs-lookup"><span data-stu-id="e0a1d-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="e0a1d-455">En **Tipo**, seleccione **Punto de conexión de Azure**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="e0a1d-456">Proporcione un **Nombre** para el punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="e0a1d-457">En **Tipo de recurso de destino**, seleccione **App Service**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="e0a1d-458">En **Recurso de destino**, seleccione **Elegir un servicio de aplicaciones** para mostrar la lista de Web Apps en la misma suscripción.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="e0a1d-459">En **Recurso**, elija el servicio de aplicación que se usa como primer punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="e0a1d-460">En la asignación geográfica, seleccione la región o continente donde se encuentra el recurso.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="e0a1d-461">Por ejemplo, **Norteamérica/Centroamérica/Caribe.**</span><span class="sxs-lookup"><span data-stu-id="e0a1d-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="e0a1d-462">En la lista desplegable de países o regiones que aparece, deje este espacio en blanco para seleccionar toda la agrupación regional anterior.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="e0a1d-463">No active la opción **Agregar como deshabilitado**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="e0a1d-464">Seleccione **Aceptar**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="e0a1d-465">Cree al menos un punto de conexión con el ámbito geográfico Todos (mundo) para que actúe como punto de conexión predeterminado para el recurso.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="e0a1d-466">Cuando termine de agregar ambos puntos de conexión, aparecerán en **Perfil de Traffic Manager** junto con el estado de supervisión como **En línea**.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Estado de punto de conexión de perfil de Traffic Manager](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="e0a1d-468">Las empresas globales confían en las funcionalidades de la distribución geográfica de Azure</span><span class="sxs-lookup"><span data-stu-id="e0a1d-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="e0a1d-469">Dirigir el tráfico de datos a través de Azure Traffic Manager y puntos de conexión específicos de la geografía permite a las empresas globales cumplir la legislación regional y mantener los datos conformes a la vez que seguros, lo que resulta crucial para el éxito del negocio tanto en el entorno local como en ubicaciones remotas.</span><span class="sxs-lookup"><span data-stu-id="e0a1d-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e0a1d-470">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="e0a1d-470">Next steps</span></span>

- <span data-ttu-id="e0a1d-471">Para más información sobre los patrones de nube de Azure, consulte [Patrones de diseño en la nube](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="e0a1d-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
