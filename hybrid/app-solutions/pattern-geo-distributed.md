---
title: Patrón de aplicación distribuida geográficamente en Azure Stack Hub
description: Conozca el patrón de aplicación distribuida geográficamente para la inteligencia perimetral mediante Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911855"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="7150e-103">Patrón de aplicación distribuida geográficamente</span><span class="sxs-lookup"><span data-stu-id="7150e-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="7150e-104">Sepa cómo proporcionar puntos de conexión de aplicación entre varias regiones y enrutar el tráfico de los usuarios en función de las necesidades de ubicación y cumplimiento.</span><span class="sxs-lookup"><span data-stu-id="7150e-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="7150e-105">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="7150e-105">Context and problem</span></span>

<span data-ttu-id="7150e-106">Las organizaciones con zonas geográficas de gran alcance intentan distribuir y permitir el acceso a los datos de forma segura y precisa, a la vez que garantizan los niveles necesarios de seguridad, cumplimiento y rendimiento por usuario, ubicación y dispositivo en todos los límites.</span><span class="sxs-lookup"><span data-stu-id="7150e-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="7150e-107">Solución</span><span class="sxs-lookup"><span data-stu-id="7150e-107">Solution</span></span>

<span data-ttu-id="7150e-108">El patrón de enrutamiento del tráfico geográfico de Azure Stack Hub o las aplicaciones distribuidas geográficamente, permiten que el tráfico se dirija a puntos de conexión específicos en función de varias métricas.</span><span class="sxs-lookup"><span data-stu-id="7150e-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="7150e-109">La creación de una instancia de Traffic Manager con la configuración de puntos de conexión y enrutamiento geográfico enruta el tráfico a puntos de conexión en función de los requisitos regionales, la legislación internacional y corporativa y su necesidad en cuanto a los datos.</span><span class="sxs-lookup"><span data-stu-id="7150e-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Patrón distribuido geográficamente](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="7150e-111">Componentes</span><span class="sxs-lookup"><span data-stu-id="7150e-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="7150e-112">Fuera de la nube</span><span class="sxs-lookup"><span data-stu-id="7150e-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="7150e-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="7150e-113">Traffic Manager</span></span>

<span data-ttu-id="7150e-114">En el diagrama, Traffic Manager se encuentra fuera de la nube pública, pero tiene que ser capaz de coordinar el tráfico tanto en el centro de datos local como en la nube pública.</span><span class="sxs-lookup"><span data-stu-id="7150e-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="7150e-115">El equilibrador enruta el tráfico a las ubicaciones geográficas.</span><span class="sxs-lookup"><span data-stu-id="7150e-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="7150e-116">Sistema de nombres de dominio (DNS)</span><span class="sxs-lookup"><span data-stu-id="7150e-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="7150e-117">El sistema de nombres de dominio, o DNS, es responsable de traducir (o resolver) el nombre del sitio web o del servicio en su dirección IP.</span><span class="sxs-lookup"><span data-stu-id="7150e-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="7150e-118">Nube pública</span><span class="sxs-lookup"><span data-stu-id="7150e-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="7150e-119">Punto de conexión de nube</span><span class="sxs-lookup"><span data-stu-id="7150e-119">Cloud Endpoint</span></span>

<span data-ttu-id="7150e-120">Las direcciones IP públicas se usan para enrutar el tráfico entrante mediante Traffic Manager al punto de conexión de recursos de aplicación de la nube pública.</span><span class="sxs-lookup"><span data-stu-id="7150e-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="7150e-121">Nubes locales</span><span class="sxs-lookup"><span data-stu-id="7150e-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="7150e-122">Punto de conexión local</span><span class="sxs-lookup"><span data-stu-id="7150e-122">Local endpoint</span></span>

<span data-ttu-id="7150e-123">Las direcciones IP públicas se usan para enrutar el tráfico entrante mediante Traffic Manager al punto de conexión de recursos de aplicación de la nube pública.</span><span class="sxs-lookup"><span data-stu-id="7150e-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="7150e-124">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="7150e-124">Issues and considerations</span></span>

<span data-ttu-id="7150e-125">Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:</span><span class="sxs-lookup"><span data-stu-id="7150e-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="7150e-126">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="7150e-126">Scalability</span></span>

<span data-ttu-id="7150e-127">El patrón controla el enrutamiento del tráfico geográfico en lugar de escalar para satisfacer los aumentos del tráfico.</span><span class="sxs-lookup"><span data-stu-id="7150e-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="7150e-128">Sin embargo, puede combinar este patrón con otras soluciones locales y de Azure.</span><span class="sxs-lookup"><span data-stu-id="7150e-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="7150e-129">Por ejemplo, este patrón se puede usar con el patrón de escalado de toda la nube.</span><span class="sxs-lookup"><span data-stu-id="7150e-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="7150e-130">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="7150e-130">Availability</span></span>

<span data-ttu-id="7150e-131">Asegúrese de que las aplicaciones implementadas localmente están configuradas para una alta disponibilidad mediante la configuración del hardware local y la implementación de software.</span><span class="sxs-lookup"><span data-stu-id="7150e-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="7150e-132">Facilidad de uso</span><span class="sxs-lookup"><span data-stu-id="7150e-132">Manageability</span></span>

<span data-ttu-id="7150e-133">El patrón garantiza una administración sin problemas y una interfaz familiar entre entornos.</span><span class="sxs-lookup"><span data-stu-id="7150e-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="7150e-134">Cuándo usar este patrón</span><span class="sxs-lookup"><span data-stu-id="7150e-134">When to use this pattern</span></span>

- <span data-ttu-id="7150e-135">Mi organización tiene oficinas internacionales que requieren una seguridad regional personalizada y directivas de distribución.</span><span class="sxs-lookup"><span data-stu-id="7150e-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="7150e-136">Cada una de las oficinas de mi organización extrae datos de empleados, negocio e instalaciones, que requieren informes de actividad para cada normativa local y zona horaria.</span><span class="sxs-lookup"><span data-stu-id="7150e-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="7150e-137">Se pueden cumplir los requisitos de gran escala mediante el escalado horizontal de las aplicaciones, con la realización de varias implementaciones de la aplicación en una única región, así como entre regiones, para afrontar los requisitos extremos de carga.</span><span class="sxs-lookup"><span data-stu-id="7150e-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="7150e-138">Las aplicaciones deben tener alta disponibilidad y responder a las solicitudes de los clientes incluso en caso de una interrupción en una sola región.</span><span class="sxs-lookup"><span data-stu-id="7150e-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="7150e-139">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="7150e-139">Next steps</span></span>

<span data-ttu-id="7150e-140">Para más información sobre los temas presentados en este artículo:</span><span class="sxs-lookup"><span data-stu-id="7150e-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="7150e-141">Consulte la [información general sobre Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) para más información sobre cómo funciona este equilibrador de carga de tráfico basado en DNS.</span><span class="sxs-lookup"><span data-stu-id="7150e-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="7150e-142">Consulte [Consideraciones sobre el diseño de aplicaciones híbridas](overview-app-design-considerations.md) para más información sobre los procedimientos recomendados y para obtener respuestas a preguntas adicionales.</span><span class="sxs-lookup"><span data-stu-id="7150e-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="7150e-143">Consulte la información relativa a la [familia de productos y soluciones de Azure Stack](/azure-stack) para más información sobre toda la gama de productos y soluciones.</span><span class="sxs-lookup"><span data-stu-id="7150e-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="7150e-144">Cuando esté listo para probar la solución de ejemplo, continúe con la [guía de implementación de la solución de aplicaciones distribuidas geográficamente](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="7150e-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="7150e-145">La guía de implementación proporciona instrucciones paso a paso para implementar y probar sus componentes.</span><span class="sxs-lookup"><span data-stu-id="7150e-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="7150e-146">Aprenda a dirigir el tráfico a puntos de conexión específicos en función de varias métricas con el patrón de aplicaciones distribuidas geográficamente.</span><span class="sxs-lookup"><span data-stu-id="7150e-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="7150e-147">La creación de un perfil de Traffic Manager con la configuración de puntos de conexión y enrutamiento geográfico garantiza que la información se enruta a puntos de conexión en función de los requisitos regionales, la legislación internacional y corporativa, y su necesidad en cuanto a los datos.</span><span class="sxs-lookup"><span data-stu-id="7150e-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
