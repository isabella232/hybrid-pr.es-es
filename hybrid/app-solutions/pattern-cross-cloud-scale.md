---
title: Patrón de escalado entre nubes en Azure Stack Hub
description: Obtenga información sobre cómo crear una aplicación multiplataforma escalable en Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911963"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="212c9-103">Patrón de escalado de toda la nube</span><span class="sxs-lookup"><span data-stu-id="212c9-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="212c9-104">Agregue recursos automáticamente a una aplicación existente para dar cabida a un aumento de la carga.</span><span class="sxs-lookup"><span data-stu-id="212c9-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="212c9-105">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="212c9-105">Context and problem</span></span>

<span data-ttu-id="212c9-106">La aplicación no puede aumentar la capacidad para satisfacer un aumento inesperado de la demanda.</span><span class="sxs-lookup"><span data-stu-id="212c9-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="212c9-107">Esta falta de escalabilidad hace que los usuarios no puedan acceder a la aplicación durante los momentos de uso máximo.</span><span class="sxs-lookup"><span data-stu-id="212c9-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="212c9-108">La aplicación puede atender a un número fijo de usuarios.</span><span class="sxs-lookup"><span data-stu-id="212c9-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="212c9-109">Las empresas globales requieren aplicaciones basadas en la nube seguras, confiables y disponibles.</span><span class="sxs-lookup"><span data-stu-id="212c9-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="212c9-110">Por este motivo, es vital satisfacer los aumentos de la demanda y usar la infraestructura adecuada para admitir esa demanda.</span><span class="sxs-lookup"><span data-stu-id="212c9-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="212c9-111">Las empresas luchan por equilibrar los costos y el mantenimiento con la seguridad, el almacenamiento y la disponibilidad en tiempo real de los datos empresariales.</span><span class="sxs-lookup"><span data-stu-id="212c9-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="212c9-112">Es posible que no pueda ejecutar la aplicación en la nube pública.</span><span class="sxs-lookup"><span data-stu-id="212c9-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="212c9-113">Sin embargo, puede que no sea económicamente viable para la empresa mantener la capacidad necesaria en el entorno local para controlar los picos en la demanda de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="212c9-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="212c9-114">Con este patrón, puede usar la elasticidad de la nube pública con la solución local.</span><span class="sxs-lookup"><span data-stu-id="212c9-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="212c9-115">Solución</span><span class="sxs-lookup"><span data-stu-id="212c9-115">Solution</span></span>

<span data-ttu-id="212c9-116">El patrón de escalado de toda la nube amplía una aplicación ubicada en una nube local con los recursos de nube pública.</span><span class="sxs-lookup"><span data-stu-id="212c9-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="212c9-117">El patrón se desencadena por un aumento o una disminución de la demanda y, respectivamente, agrega recursos a la nube o los quita de esta.</span><span class="sxs-lookup"><span data-stu-id="212c9-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="212c9-118">Estos recursos proporcionan redundancia, disponibilidad rápida y enrutamiento compatible con las geoáreas.</span><span class="sxs-lookup"><span data-stu-id="212c9-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Patrón de escalado de toda la nube](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="212c9-120">Este patrón solo se aplica a los componentes sin estado de su aplicación.</span><span class="sxs-lookup"><span data-stu-id="212c9-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="212c9-121">Componentes</span><span class="sxs-lookup"><span data-stu-id="212c9-121">Components</span></span>

<span data-ttu-id="212c9-122">El patrón de escalado entre nubes consta de los siguientes componentes.</span><span class="sxs-lookup"><span data-stu-id="212c9-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="212c9-123">Fuera de la nube</span><span class="sxs-lookup"><span data-stu-id="212c9-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="212c9-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="212c9-124">Traffic Manager</span></span>

<span data-ttu-id="212c9-125">En el diagrama esto se encuentra fuera del grupo de la nube pública, pero debería ser capaz de coordinar el tráfico tanto en el centro de datos local como en la nube pública.</span><span class="sxs-lookup"><span data-stu-id="212c9-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="212c9-126">El equilibrador ofrece alta disponibilidad para la aplicación mediante la supervisión de los puntos de conexión y la redistribución de la conmutación por error cuando es necesario.</span><span class="sxs-lookup"><span data-stu-id="212c9-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="212c9-127">Sistema de nombres de dominio (DNS)</span><span class="sxs-lookup"><span data-stu-id="212c9-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="212c9-128">El sistema de nombres de dominio, o DNS, es responsable de traducir (o resolver) el nombre del sitio web o del servicio en su dirección IP.</span><span class="sxs-lookup"><span data-stu-id="212c9-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="212c9-129">Nube</span><span class="sxs-lookup"><span data-stu-id="212c9-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="212c9-130">Servidor de compilación hospedado</span><span class="sxs-lookup"><span data-stu-id="212c9-130">Hosted build server</span></span>

<span data-ttu-id="212c9-131">Un entorno para hospedar la canalización de compilación.</span><span class="sxs-lookup"><span data-stu-id="212c9-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="212c9-132">Recursos de la aplicación</span><span class="sxs-lookup"><span data-stu-id="212c9-132">App resources</span></span>

<span data-ttu-id="212c9-133">Los recursos de la aplicación deben ser capaces de reducirse y escalarse horizontalmente, como los conjuntos de escalado de máquinas virtuales y los contenedores.</span><span class="sxs-lookup"><span data-stu-id="212c9-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="212c9-134">Nombre de dominio personalizado</span><span class="sxs-lookup"><span data-stu-id="212c9-134">Custom domain name</span></span>

<span data-ttu-id="212c9-135">Use un nombre de dominio personalizado para el enrutamiento global de las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="212c9-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="212c9-136">Direcciones IP públicas</span><span class="sxs-lookup"><span data-stu-id="212c9-136">Public IP addresses</span></span>

<span data-ttu-id="212c9-137">Las direcciones IP públicas se usan para enrutar el tráfico entrante mediante Traffic Manager al punto de conexión de recursos de aplicación de la nube pública.</span><span class="sxs-lookup"><span data-stu-id="212c9-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="212c9-138">Nube local</span><span class="sxs-lookup"><span data-stu-id="212c9-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="212c9-139">Servidor de compilación hospedado</span><span class="sxs-lookup"><span data-stu-id="212c9-139">Hosted build server</span></span>

<span data-ttu-id="212c9-140">Un entorno para hospedar la canalización de compilación.</span><span class="sxs-lookup"><span data-stu-id="212c9-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="212c9-141">Recursos de la aplicación</span><span class="sxs-lookup"><span data-stu-id="212c9-141">App resources</span></span>

<span data-ttu-id="212c9-142">Los recursos de la aplicación deben ser capaces de reducirse y escalarse horizontalmente, como los conjuntos de escalado de máquinas virtuales y los contenedores.</span><span class="sxs-lookup"><span data-stu-id="212c9-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="212c9-143">Nombre de dominio personalizado</span><span class="sxs-lookup"><span data-stu-id="212c9-143">Custom domain name</span></span>

<span data-ttu-id="212c9-144">Use un nombre de dominio personalizado para el enrutamiento global de las solicitudes.</span><span class="sxs-lookup"><span data-stu-id="212c9-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="212c9-145">Direcciones IP públicas</span><span class="sxs-lookup"><span data-stu-id="212c9-145">Public IP addresses</span></span>

<span data-ttu-id="212c9-146">Las direcciones IP públicas se usan para enrutar el tráfico entrante mediante Traffic Manager al punto de conexión de recursos de aplicación de la nube pública.</span><span class="sxs-lookup"><span data-stu-id="212c9-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="212c9-147">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="212c9-147">Issues and considerations</span></span>

<span data-ttu-id="212c9-148">Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="212c9-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="212c9-149">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="212c9-149">Scalability</span></span>

<span data-ttu-id="212c9-150">El componente clave del escalado entre nubes es la capacidad de ofrecer escalado a petición.</span><span class="sxs-lookup"><span data-stu-id="212c9-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="212c9-151">El escalado debe ocurrir entre la infraestructura en la nube pública y local, así como ofrecer un servicio coherente y de confianza de acuerdo con la demanda.</span><span class="sxs-lookup"><span data-stu-id="212c9-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="212c9-152">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="212c9-152">Availability</span></span>

<span data-ttu-id="212c9-153">Asegúrese de que las aplicaciones implementadas localmente están configuradas para una alta disponibilidad mediante la configuración del hardware local y la implementación de software.</span><span class="sxs-lookup"><span data-stu-id="212c9-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="212c9-154">Facilidad de uso</span><span class="sxs-lookup"><span data-stu-id="212c9-154">Manageability</span></span>

<span data-ttu-id="212c9-155">El patrón de toda la nube garantiza una administración sin problemas y una interfaz familiar entre entornos.</span><span class="sxs-lookup"><span data-stu-id="212c9-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="212c9-156">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="212c9-156">When to use this pattern</span></span>

<span data-ttu-id="212c9-157">Use este patrón:</span><span class="sxs-lookup"><span data-stu-id="212c9-157">Use this pattern:</span></span>

- <span data-ttu-id="212c9-158">Cuando necesite aumentar la capacidad de la aplicación con demandas inesperadas o peticiones periódicas en la demanda.</span><span class="sxs-lookup"><span data-stu-id="212c9-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="212c9-159">Si no desea invertir en recursos que solo se utilizarán durante los picos.</span><span class="sxs-lookup"><span data-stu-id="212c9-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="212c9-160">Pague por lo que usa.</span><span class="sxs-lookup"><span data-stu-id="212c9-160">Pay for what you use.</span></span>

<span data-ttu-id="212c9-161">No se recomienda este patrón si:</span><span class="sxs-lookup"><span data-stu-id="212c9-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="212c9-162">La solución requiere que los usuarios se conecten mediante Internet.</span><span class="sxs-lookup"><span data-stu-id="212c9-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="212c9-163">Su empresa tiene normativas locales que requieren que la conexión de origen proceda de una llamada in situ.</span><span class="sxs-lookup"><span data-stu-id="212c9-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="212c9-164">La red experimenta cuellos de botella regulares que restringirían el rendimiento del escalado.</span><span class="sxs-lookup"><span data-stu-id="212c9-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="212c9-165">Su entorno está desconectado de Internet y no puede acceder a la nube pública.</span><span class="sxs-lookup"><span data-stu-id="212c9-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="212c9-166">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="212c9-166">Next steps</span></span>

<span data-ttu-id="212c9-167">Para más información sobre los temas presentados en este artículo:</span><span class="sxs-lookup"><span data-stu-id="212c9-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="212c9-168">Consulte la [información general sobre Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) para más información sobre cómo funciona este equilibrador de carga de tráfico basado en DNS.</span><span class="sxs-lookup"><span data-stu-id="212c9-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="212c9-169">Consulte [Consideraciones sobre el diseño de aplicaciones híbridas](overview-app-design-considerations.md) para más información sobre los procedimientos recomendados y obtener respuestas a preguntas adicionales.</span><span class="sxs-lookup"><span data-stu-id="212c9-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="212c9-170">Consulte la información relativa a la [familia de productos y soluciones de Azure Stack](/azure-stack) para más información sobre toda la gama de productos y soluciones.</span><span class="sxs-lookup"><span data-stu-id="212c9-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="212c9-171">Cuando esté listo para probar la solución de ejemplo, continúe con la [guía de implementación de soluciones de escalado entre nubes](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="212c9-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="212c9-172">La guía de implementación proporciona instrucciones paso a paso para implementar y probar sus componentes.</span><span class="sxs-lookup"><span data-stu-id="212c9-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="212c9-173">Aprenderá a crear una solución entre nubes que proporcione un proceso desencadenado manualmente para cambiar de una aplicación web hospedada en Azure Stack Hub a una aplicación web hospedada en Azure.</span><span class="sxs-lookup"><span data-stu-id="212c9-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="212c9-174">También aprenderá a usar el escalado automático a través de Traffic Manager, garantizando una utilidad en la nube flexible y escalable bajo carga.</span><span class="sxs-lookup"><span data-stu-id="212c9-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
