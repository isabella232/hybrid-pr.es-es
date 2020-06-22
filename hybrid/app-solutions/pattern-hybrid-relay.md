---
title: Patrón de retransmisión híbrida en Azure y Azure Stack Hub
description: Utilice el patrón de retransmisión híbrida en Azure y Azure Stack Hub para conectarse a recursos perimetrales protegidos por firewalls.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911969"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="55ffd-103">Patrón de retransmisión híbrida</span><span class="sxs-lookup"><span data-stu-id="55ffd-103">Hybrid relay pattern</span></span>

<span data-ttu-id="55ffd-104">Obtenga información sobre cómo conectarse a recursos perimetrales o dispositivos protegidos mediante firewalls mediante el patrón de retransmisión híbrida y el Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="55ffd-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="55ffd-105">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="55ffd-105">Context and problem</span></span>

<span data-ttu-id="55ffd-106">Los dispositivos perimetrales suelen estar detrás de un firewall corporativo o un dispositivo NAT.</span><span class="sxs-lookup"><span data-stu-id="55ffd-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="55ffd-107">Aunque son seguros, es posible que no puedan comunicarse con la nube pública o con dispositivos perimetrales de otras redes corporativas.</span><span class="sxs-lookup"><span data-stu-id="55ffd-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="55ffd-108">Puede que sea necesario exponer determinados puertos y funcionalidad a los usuarios de la nube pública de forma segura.</span><span class="sxs-lookup"><span data-stu-id="55ffd-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="55ffd-109">Solución</span><span class="sxs-lookup"><span data-stu-id="55ffd-109">Solution</span></span>

<span data-ttu-id="55ffd-110">El patrón de retransmisión híbrida usa Azure Relay para establecer un túnel de WebSockets entre dos extremos que no se pueden comunicar directamente.</span><span class="sxs-lookup"><span data-stu-id="55ffd-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="55ffd-111">Los dispositivos que no están en el entorno local, pero que necesitan conectarse a un punto de conexión local, se conectarán a un punto de conexión de la nube pública.</span><span class="sxs-lookup"><span data-stu-id="55ffd-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="55ffd-112">Este punto de conexión redirigirá el tráfico de las rutas predefinidas por un canal seguro.</span><span class="sxs-lookup"><span data-stu-id="55ffd-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="55ffd-113">Un punto de conexión dentro del entorno local recibe el tráfico y lo enruta al destino correcto.</span><span class="sxs-lookup"><span data-stu-id="55ffd-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![arquitectura de la solución del patrón de retransmisión híbrida](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="55ffd-115">Este es el funcionamiento del patrón de retransmisión híbrida:</span><span class="sxs-lookup"><span data-stu-id="55ffd-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="55ffd-116">Un dispositivo se conecta a la máquina virtual (VM) de Azure en un puerto predefinido.</span><span class="sxs-lookup"><span data-stu-id="55ffd-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="55ffd-117">El tráfico se reenvía al Azure Relay en Azure.</span><span class="sxs-lookup"><span data-stu-id="55ffd-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="55ffd-118">La máquina virtual en Azure Stack concentrador, que ya ha establecido una conexión de larga duración con la Azure Relay, recibe el tráfico y lo reenvía al destino.</span><span class="sxs-lookup"><span data-stu-id="55ffd-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="55ffd-119">El servicio o punto de conexión local procesa la solicitud.</span><span class="sxs-lookup"><span data-stu-id="55ffd-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="55ffd-120">Componentes</span><span class="sxs-lookup"><span data-stu-id="55ffd-120">Components</span></span>

<span data-ttu-id="55ffd-121">Esta solución usa los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="55ffd-121">This solution uses the following components:</span></span>

| <span data-ttu-id="55ffd-122">Nivel</span><span class="sxs-lookup"><span data-stu-id="55ffd-122">Layer</span></span> | <span data-ttu-id="55ffd-123">Componente</span><span class="sxs-lookup"><span data-stu-id="55ffd-123">Component</span></span> | <span data-ttu-id="55ffd-124">Descripción</span><span class="sxs-lookup"><span data-stu-id="55ffd-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="55ffd-125">Azure</span><span class="sxs-lookup"><span data-stu-id="55ffd-125">Azure</span></span> | <span data-ttu-id="55ffd-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="55ffd-126">Azure VM</span></span> | <span data-ttu-id="55ffd-127">Una máquina virtual de Azure proporciona un punto de conexión accesible públicamente para el recurso local.</span><span class="sxs-lookup"><span data-stu-id="55ffd-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="55ffd-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="55ffd-128">Azure Relay</span></span> | <span data-ttu-id="55ffd-129">Un [Azure Relay](/azure/azure-relay/) proporciona la infraestructura para el mantenimiento del túnel y la conexión entre la máquina virtual de Azure y la máquina virtual de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="55ffd-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="55ffd-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="55ffd-130">Azure Stack Hub</span></span> | <span data-ttu-id="55ffd-131">Proceso</span><span class="sxs-lookup"><span data-stu-id="55ffd-131">Compute</span></span> | <span data-ttu-id="55ffd-132">Una máquina virtual de Azure Stack Hub proporciona el lado servidor del túnel de retransmisión híbrida.</span><span class="sxs-lookup"><span data-stu-id="55ffd-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="55ffd-133">Storage</span><span class="sxs-lookup"><span data-stu-id="55ffd-133">Storage</span></span> | <span data-ttu-id="55ffd-134">El clúster del motor de AKS implementado en Azure Stack Hub proporciona un motor escalable y resistente para ejecutar el contenedor de Face API.</span><span class="sxs-lookup"><span data-stu-id="55ffd-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="55ffd-135">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="55ffd-135">Issues and considerations</span></span>

<span data-ttu-id="55ffd-136">Tenga en cuenta los puntos siguientes al decidir cómo implementar esta solución:</span><span class="sxs-lookup"><span data-stu-id="55ffd-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="55ffd-137">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="55ffd-137">Scalability</span></span>

<span data-ttu-id="55ffd-138">Este patrón solo permite asignaciones de puerto 1:1 en el cliente y el servidor.</span><span class="sxs-lookup"><span data-stu-id="55ffd-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="55ffd-139">Por ejemplo, si se tuneliza el puerto 80 para un servicio del punto de conexión de Azure, no se puede usar para otro servicio.</span><span class="sxs-lookup"><span data-stu-id="55ffd-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="55ffd-140">Las asignaciones de puertos deben planearse en consecuencia.</span><span class="sxs-lookup"><span data-stu-id="55ffd-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="55ffd-141">El Azure Relay y las máquinas virtuales deben escalarse adecuadamente para controlar el tráfico.</span><span class="sxs-lookup"><span data-stu-id="55ffd-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="55ffd-142">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="55ffd-142">Availability</span></span>

<span data-ttu-id="55ffd-143">Estos túneles y conexiones no son redundantes.</span><span class="sxs-lookup"><span data-stu-id="55ffd-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="55ffd-144">Para garantizar alta disponibilidad, podría interesarle implementar el código de comprobación de errores.</span><span class="sxs-lookup"><span data-stu-id="55ffd-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="55ffd-145">Otra opción es tener un grupo de máquinas virtuales conectadas Azure Relay detrás de un equilibrador de carga.</span><span class="sxs-lookup"><span data-stu-id="55ffd-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="55ffd-146">Facilidad de uso</span><span class="sxs-lookup"><span data-stu-id="55ffd-146">Manageability</span></span>

<span data-ttu-id="55ffd-147">Esta solución puede abarcar muchos dispositivos y ubicaciones, lo que podría resultar complicado.</span><span class="sxs-lookup"><span data-stu-id="55ffd-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="55ffd-148">Los servicios de IoT de Azure pueden poner en línea automáticamente nuevas ubicaciones y dispositivos y mantenerlos actualizados.</span><span class="sxs-lookup"><span data-stu-id="55ffd-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="55ffd-149">Seguridad</span><span class="sxs-lookup"><span data-stu-id="55ffd-149">Security</span></span>

<span data-ttu-id="55ffd-150">Este patrón, tal como se muestra, permite el acceso ilimitado a un puerto en un dispositivo interno desde el perímetro.</span><span class="sxs-lookup"><span data-stu-id="55ffd-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="55ffd-151">Considere la posibilidad de agregar un mecanismo de autenticación al servicio en el dispositivo interno o delante del punto de conexión de retransmisión híbrida.</span><span class="sxs-lookup"><span data-stu-id="55ffd-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="55ffd-152">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="55ffd-152">Next steps</span></span>

<span data-ttu-id="55ffd-153">Para más información sobre los temas presentados en este artículo:</span><span class="sxs-lookup"><span data-stu-id="55ffd-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="55ffd-154">Este patrón usa Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="55ffd-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="55ffd-155">Para obtener más información, consulte la [documentación de Azure Relay](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="55ffd-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="55ffd-156">Consulte [Consideraciones sobre el diseño de aplicaciones híbridas](overview-app-design-considerations.md) para más información sobre los procedimientos recomendados y obtener respuestas a preguntas adicionales.</span><span class="sxs-lookup"><span data-stu-id="55ffd-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="55ffd-157">Consulte la información relativa a la [familia de productos y soluciones de Azure Stack](/azure-stack) para más información sobre toda la gama de productos y soluciones.</span><span class="sxs-lookup"><span data-stu-id="55ffd-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="55ffd-158">Cuando esté listo para probar la solución de ejemplo, continúe con la [guía de implementación de soluciones de retransmisión híbrida](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="55ffd-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="55ffd-159">La guía de implementación proporciona instrucciones paso a paso para implementar y probar sus componentes.</span><span class="sxs-lookup"><span data-stu-id="55ffd-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>