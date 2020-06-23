---
title: Patrón de detección de afluencia de compradores con Azure y Azure Stack Hub
description: Aprenda a usar Azure y Azure Stack Hub para implementar una solución de afluencia de compradores basada en inteligencia artificial para analizar el tráfico de las tiendas.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84912004"
---
# <a name="footfall-detection-pattern"></a><span data-ttu-id="5f94d-103">Patrón de detección de afluencia de compradores</span><span class="sxs-lookup"><span data-stu-id="5f94d-103">Footfall detection pattern</span></span>

<span data-ttu-id="5f94d-104">Este patrón proporciona información general para la implementación de una solución de detección de afluencia de compradores basada en inteligencia artificial, con el fin de analizar el tráfico de visitantes en las tiendas.</span><span class="sxs-lookup"><span data-stu-id="5f94d-104">This pattern provides an overview for implementing an AI-based footfall detection solution for analyzing visitor traffic in retail stores.</span></span> <span data-ttu-id="5f94d-105">La solución genera información de acciones del mundo real mediante Azure, Azure Stack Hub y el kit de desarrollo de inteligencia artificial de Custom Vision.</span><span class="sxs-lookup"><span data-stu-id="5f94d-105">The solution generates insights from real world actions, using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="5f94d-106">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="5f94d-106">Context and problem</span></span>

<span data-ttu-id="5f94d-107">Contoso Stores quiere obtener información sobre el modo en que los clientes reciben sus productos actuales, en relación con el diseño de la tienda.</span><span class="sxs-lookup"><span data-stu-id="5f94d-107">Contoso Stores would like to gain insights on how customers are receiving their current products in relation to store layout.</span></span> <span data-ttu-id="5f94d-108">No pueden colocar personal en cada sección, y es ineficaz que un equipo de analistas revise la grabación completa de la cámara de una tienda.</span><span class="sxs-lookup"><span data-stu-id="5f94d-108">They're unable to place staff in every section and it's inefficient to have a team of analysts review an entire store's camera footage.</span></span> <span data-ttu-id="5f94d-109">Además, ninguno de sus almacenes tiene suficiente ancho de banda como para transmitir vídeo desde todas sus cámaras a la nube para su análisis.</span><span class="sxs-lookup"><span data-stu-id="5f94d-109">In addition, none of their stores have enough bandwidth to stream video from all their cameras to the cloud for analysis.</span></span>

<span data-ttu-id="5f94d-110">Contoso quiere encontrar una manera discreta y fácil para determinar los datos demográficos, la fidelidad y las reacciones de sus clientes ante el diseño de la tienda y los productos.</span><span class="sxs-lookup"><span data-stu-id="5f94d-110">Contoso would like to find an unobtrusive, privacy-friendly way to determine their customers' demographics, loyalty, and reactions to store displays and products.</span></span>

## <a name="solution"></a><span data-ttu-id="5f94d-111">Solución</span><span class="sxs-lookup"><span data-stu-id="5f94d-111">Solution</span></span>

<span data-ttu-id="5f94d-112">Este patrón de análisis minorista usa un enfoque por niveles para la inferencia en el perímetro.</span><span class="sxs-lookup"><span data-stu-id="5f94d-112">This retail analytics pattern uses a tiered approach to inferencing at the edge.</span></span> <span data-ttu-id="5f94d-113">Mediante el kit de desarrollo de inteligencia artificial de Custom Vision, solo las imágenes con caras humanas se envían para su análisis a una instancia privada de Azure Stack Hub que ejecute Azure Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="5f94d-113">By using the Custom Vision AI Dev Kit, only images with human faces are sent for analysis to a private Azure Stack Hub that runs Azure Cognitive Services.</span></span> <span data-ttu-id="5f94d-114">Los datos agregados anonimizados se envían a Azure para su agregación en todas las tiendas y su visualización en Power BI.</span><span class="sxs-lookup"><span data-stu-id="5f94d-114">Anonymized, aggregated data is sent to Azure for aggregation across all stores and visualization in Power BI.</span></span> <span data-ttu-id="5f94d-115">La combinación de la nube perimetral y la nube pública permite a Contoso aprovechar las modernas tecnologías de inteligencia artificial, a la vez que mantiene el cumplimiento de las directivas corporativas y respeta la privacidad de sus clientes.</span><span class="sxs-lookup"><span data-stu-id="5f94d-115">Combining the edge and public cloud lets Contoso take advantage of modern AI technology while also remaining in compliance with their corporate policies and respecting their customers' privacy.</span></span>

<span data-ttu-id="5f94d-116">[![Solución con patrón de detección de afluencia de compradores](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="5f94d-116">[![Footfall detection pattern solution](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span></span>

<span data-ttu-id="5f94d-117">A continuación se muestra un resumen de cómo funciona la solución:</span><span class="sxs-lookup"><span data-stu-id="5f94d-117">Here's a summary of how the solution works:</span></span>

1. <span data-ttu-id="5f94d-118">El kit de desarrollo de IA de Custom Vision obtiene una configuración de IoT Hub, que instala el tiempo de ejecución de IoT Edge y un modelo de ML.</span><span class="sxs-lookup"><span data-stu-id="5f94d-118">The Custom Vision AI Dev Kit gets a configuration from IoT Hub, which installs the IoT Edge Runtime and an ML model.</span></span>
2. <span data-ttu-id="5f94d-119">Si el modelo ve una persona, le hace una foto y la carga en el almacenamiento de blobs de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="5f94d-119">If the model sees a person, it takes a picture and uploads it to Azure Stack Hub blob storage.</span></span>
3. <span data-ttu-id="5f94d-120">Blob service desencadena una función de Azure en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="5f94d-120">The blob service triggers an Azure Function on Azure Stack Hub.</span></span>
4. <span data-ttu-id="5f94d-121">Azure Functions llama a un contenedor con la Face API para obtener datos demográficos y de emociones de la imagen.</span><span class="sxs-lookup"><span data-stu-id="5f94d-121">The Azure Function calls a container with the Face API to get demographic and emotion data from the image.</span></span>
5. <span data-ttu-id="5f94d-122">Los datos se anonimizan y se envían a un clúster de Azure Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="5f94d-122">The data is anonymized and sent to an Azure Event Hubs cluster.</span></span>
6. <span data-ttu-id="5f94d-123">Este, a su vez, los envía Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="5f94d-123">The Event Hubs cluster pushes the data to Stream Analytics.</span></span>
7. <span data-ttu-id="5f94d-124">Stream Analytics agrega los datos y los envía a Power BI.</span><span class="sxs-lookup"><span data-stu-id="5f94d-124">Stream Analytics aggregates the data and pushes it to Power BI.</span></span>

## <a name="components"></a><span data-ttu-id="5f94d-125">Componentes</span><span class="sxs-lookup"><span data-stu-id="5f94d-125">Components</span></span>

<span data-ttu-id="5f94d-126">Esta solución usa los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="5f94d-126">This solution uses the following components:</span></span>

| <span data-ttu-id="5f94d-127">Nivel</span><span class="sxs-lookup"><span data-stu-id="5f94d-127">Layer</span></span> | <span data-ttu-id="5f94d-128">Componente</span><span class="sxs-lookup"><span data-stu-id="5f94d-128">Component</span></span> | <span data-ttu-id="5f94d-129">Descripción</span><span class="sxs-lookup"><span data-stu-id="5f94d-129">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="5f94d-130">Hardware en la tienda</span><span class="sxs-lookup"><span data-stu-id="5f94d-130">In-store hardware</span></span> | [<span data-ttu-id="5f94d-131">Kit de desarrollo de IA de Custom Vision</span><span class="sxs-lookup"><span data-stu-id="5f94d-131">Custom Vision AI Dev Kit</span></span>](https://azure.github.io/Vision-AI-DevKit-Pages/) | <span data-ttu-id="5f94d-132">Proporciona filtrado en la tienda mediante un modelo de ML local que solo captura imágenes de personas para su análisis.</span><span class="sxs-lookup"><span data-stu-id="5f94d-132">Provides in-store filtering using a local ML model that only captures images of people for analysis.</span></span> <span data-ttu-id="5f94d-133">Aprovisionado y actualizado de forma segura a través de IoT Hub.</span><span class="sxs-lookup"><span data-stu-id="5f94d-133">Securely provisioned and updated through IoT Hub.</span></span><br><br>|
| <span data-ttu-id="5f94d-134">Azure</span><span class="sxs-lookup"><span data-stu-id="5f94d-134">Azure</span></span> | [<span data-ttu-id="5f94d-135">Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="5f94d-135">Azure Event Hubs</span></span>](/azure/event-hubs/) | <span data-ttu-id="5f94d-136">Azure Event Hubs proporciona una plataforma escalable para la ingesta de datos anonimizados que se integra perfectamente con Azure Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="5f94d-136">Azure Event Hubs provides a scalable platform for ingesting anonymized data that integrates neatly with Azure Stream Analytics.</span></span> |
|  | [<span data-ttu-id="5f94d-137">Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="5f94d-137">Azure Stream Analytics</span></span>](/azure/stream-analytics/) | <span data-ttu-id="5f94d-138">Un trabajo de Azure Stream Analytics agrega los datos anonimizados y los agrupa en ventanas de 15 segundos para su visualización.</span><span class="sxs-lookup"><span data-stu-id="5f94d-138">An Azure Stream Analytics job aggregates the anonymized data and groups it into 15-second windows for visualization.</span></span> |
|  | [<span data-ttu-id="5f94d-139">Microsoft Power BI</span><span class="sxs-lookup"><span data-stu-id="5f94d-139">Microsoft Power BI</span></span>](https://powerbi.microsoft.com/) | <span data-ttu-id="5f94d-140">Power BI proporciona una interfaz de panel fácil de usar para ver los resultados de Azure Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="5f94d-140">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="5f94d-141">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="5f94d-141">Azure Stack Hub</span></span> | [<span data-ttu-id="5f94d-142">App Service</span><span class="sxs-lookup"><span data-stu-id="5f94d-142">App Service</span></span>](/azure-stack/operator/azure-stack-app-service-overview.md) | <span data-ttu-id="5f94d-143">El proveedor de recursos App Service (RP) proporciona una base para los componentes perimetrales, incluidas las características de hospedaje y administración de aplicaciones web, API y funciones.</span><span class="sxs-lookup"><span data-stu-id="5f94d-143">The App Service resource provider (RP) provides a base for edge components, including hosting and management features for web apps/APIs and Functions.</span></span> |
| | <span data-ttu-id="5f94d-144">Clúster del [motor de](https://github.com/Azure/aks-engine) Azure Kubernetes Service (AKS)</span><span class="sxs-lookup"><span data-stu-id="5f94d-144">Azure Kubernetes Service [(AKS) Engine](https://github.com/Azure/aks-engine) cluster</span></span> | <span data-ttu-id="5f94d-145">El RP de AKS con el clúster de AKS-Engine implementado en Azure Stack Hub proporciona un motor escalable y resistente para ejecutar el contenedor de Face API.</span><span class="sxs-lookup"><span data-stu-id="5f94d-145">The AKS RP with AKS-Engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span> |
| | <span data-ttu-id="5f94d-146">[Contenedores de Face API](/azure/cognitive-services/face/face-how-to-install-containers) de Azure Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="5f94d-146">Azure Cognitive Services [Face API containers](/azure/cognitive-services/face/face-how-to-install-containers)</span></span>| <span data-ttu-id="5f94d-147">El RP de Azure Cognitive Services con contenedores de Face API proporciona detección de visitantes exclusivos, de emociones y demográfica en la red privada de Contoso.</span><span class="sxs-lookup"><span data-stu-id="5f94d-147">The Azure Cognitive Services RP with Face API containers provides demographic, emotion, and unique visitor detection on Contoso's private network.</span></span> |
| | <span data-ttu-id="5f94d-148">Blob Storage</span><span class="sxs-lookup"><span data-stu-id="5f94d-148">Blob Storage</span></span> | <span data-ttu-id="5f94d-149">Las imágenes capturadas con el kit de desarrollo de IA se cargan en el almacenamiento de blobs de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="5f94d-149">Images captured from the AI Dev Kit are uploaded to Azure Stack Hub's blob storage.</span></span> |
| | <span data-ttu-id="5f94d-150">Azure Functions</span><span class="sxs-lookup"><span data-stu-id="5f94d-150">Azure Functions</span></span> | <span data-ttu-id="5f94d-151">Una función de Azure que se ejecuta en Azure Stack Hub recibe la entrada del almacenamiento de blobs y administra las interacciones con Face API.</span><span class="sxs-lookup"><span data-stu-id="5f94d-151">An Azure Function running on Azure Stack Hub receives input from blob storage and manages the interactions with the Face API.</span></span> <span data-ttu-id="5f94d-152">Emite datos anonimizados a un clúster de Event Hubs ubicado en Azure.</span><span class="sxs-lookup"><span data-stu-id="5f94d-152">It emits anonymized data to an Event Hubs cluster located in Azure.</span></span><br><br>|

## <a name="issues-and-considerations"></a><span data-ttu-id="5f94d-153">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="5f94d-153">Issues and considerations</span></span>

<span data-ttu-id="5f94d-154">Tenga en cuenta los puntos siguientes al decidir cómo implementar esta solución:</span><span class="sxs-lookup"><span data-stu-id="5f94d-154">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="5f94d-155">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="5f94d-155">Scalability</span></span>

<span data-ttu-id="5f94d-156">Para permitir que esta solución se escale entre varias cámaras y ubicaciones, deberá asegurarse de que todos los componentes pueden controlar el aumento de la carga.</span><span class="sxs-lookup"><span data-stu-id="5f94d-156">To enable this solution to scale across multiple cameras and locations, you'll need to make sure that all of the components can handle the increased load.</span></span> <span data-ttu-id="5f94d-157">Es posible que tenga que realizar acciones como las siguientes:</span><span class="sxs-lookup"><span data-stu-id="5f94d-157">You may need to take actions like:</span></span>

- <span data-ttu-id="5f94d-158">Aumentar el número de unidades de streaming de Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="5f94d-158">Increase the number of Stream Analytics streaming units.</span></span>
- <span data-ttu-id="5f94d-159">Escalar horizontalmente la implementación de Face API.</span><span class="sxs-lookup"><span data-stu-id="5f94d-159">Scale out the Face API deployment.</span></span>
- <span data-ttu-id="5f94d-160">Aumentar el rendimiento de los clústeres de Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="5f94d-160">Increase the Event Hubs cluster throughput.</span></span>
- <span data-ttu-id="5f94d-161">En casos extremos, puede que sea necesario migrar desde Azure Functions a una máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="5f94d-161">For extreme cases, migrate from Azure Functions to a virtual machine may be necessary.</span></span>

### <a name="availability"></a><span data-ttu-id="5f94d-162">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="5f94d-162">Availability</span></span>

<span data-ttu-id="5f94d-163">Dado que esta solución está en capas, es importante pensar en cómo tratar los errores de red o de alimentación.</span><span class="sxs-lookup"><span data-stu-id="5f94d-163">Since this solution is tiered, it's important to think about how to deal with networking or power failures.</span></span> <span data-ttu-id="5f94d-164">En función de las necesidades empresariales, podría ser adecuado implementar un mecanismo para almacenar localmente en caché las imágenes y, después, reenviarlas a Azure Stack Hub cuando vuelva a tener conectividad.</span><span class="sxs-lookup"><span data-stu-id="5f94d-164">Depending on business needs, you might want to implement a mechanism to cache images locally, then forward to Azure Stack Hub when connectivity returns.</span></span> <span data-ttu-id="5f94d-165">Si la ubicación es lo suficientemente grande, podría ser una opción mejor implementar una instancia de Data Box Edge con el contenedor de Face API en esa ubicación.</span><span class="sxs-lookup"><span data-stu-id="5f94d-165">If the location is large enough, deploying a Data Box Edge with the Face API container to that location might be a better option.</span></span>

### <a name="manageability"></a><span data-ttu-id="5f94d-166">Facilidad de uso</span><span class="sxs-lookup"><span data-stu-id="5f94d-166">Manageability</span></span>

<span data-ttu-id="5f94d-167">Esta solución puede abarcar muchos dispositivos y ubicaciones, lo que podría resultar complicado.</span><span class="sxs-lookup"><span data-stu-id="5f94d-167">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="5f94d-168">[Los servicios de IoT de Azure](/azure/iot-fundamentals/) se pueden usar para poner en línea automáticamente nuevas ubicaciones y dispositivos y mantenerlos actualizados.</span><span class="sxs-lookup"><span data-stu-id="5f94d-168">[Azure's IoT services](/azure/iot-fundamentals/) can be used to automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="5f94d-169">Seguridad</span><span class="sxs-lookup"><span data-stu-id="5f94d-169">Security</span></span>

<span data-ttu-id="5f94d-170">Esta solución captura imágenes de clientes, por lo que es fundamental tener en cuenta la seguridad.</span><span class="sxs-lookup"><span data-stu-id="5f94d-170">This solution captures customer images, making security a paramount consideration.</span></span> <span data-ttu-id="5f94d-171">Asegúrese de que todas las cuentas de almacenamiento están protegidas con las directivas de acceso adecuadas y que las claves se cambian con regularidad.</span><span class="sxs-lookup"><span data-stu-id="5f94d-171">Make sure all storage accounts are secured with the proper access policies and rotate keys regularly.</span></span> <span data-ttu-id="5f94d-172">Asegúrese de que las cuentas de almacenamiento y Event Hubs tienen directivas de retención que cumplen las normas de privacidad corporativa y gubernamental.</span><span class="sxs-lookup"><span data-stu-id="5f94d-172">Ensure storage accounts and Event Hubs have retention policies that meet corporate and government privacy regulations.</span></span> <span data-ttu-id="5f94d-173">Asegúrese también de organizar los niveles de acceso de los usuarios.</span><span class="sxs-lookup"><span data-stu-id="5f94d-173">Also make sure to tier the user access levels.</span></span> <span data-ttu-id="5f94d-174">La organización por niveles garantiza que los usuarios solo tienen acceso a los datos que necesitan para su rol.</span><span class="sxs-lookup"><span data-stu-id="5f94d-174">Tiering ensures that users only have access to the data they need for their role.</span></span>

## <a name="next-steps"></a><span data-ttu-id="5f94d-175">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="5f94d-175">Next steps</span></span>

<span data-ttu-id="5f94d-176">Para más información sobre los temas presentados en este artículo:</span><span class="sxs-lookup"><span data-stu-id="5f94d-176">To learn more about the topics introduced in this article:</span></span>

- <span data-ttu-id="5f94d-177">Consulte [Patrón de datos en niveles](https://aka.ms/tiereddatadeploy), que aprovecha el patrón de detección de afluencia de compradores.</span><span class="sxs-lookup"><span data-stu-id="5f94d-177">See the [Tiered Data pattern](https://aka.ms/tiereddatadeploy), which is leveraged by the footfall detection pattern.</span></span>
- <span data-ttu-id="5f94d-178">Para más información sobre el uso de Custom Vision, consulte [Kit de desarrollo de IA de Custom Vision](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="5f94d-178">See the [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) to learn more about using custom vision.</span></span> 

<span data-ttu-id="5f94d-179">Cuando esté listo para probar la solución de ejemplo, continúe con la [guía de implementación de la detección de afluencia de compradores](solution-deployment-guide-retail-footfall-detection.md).</span><span class="sxs-lookup"><span data-stu-id="5f94d-179">When you're ready to test the solution example, continue with the [Footfall detection deployment guide](solution-deployment-guide-retail-footfall-detection.md).</span></span> <span data-ttu-id="5f94d-180">La guía de implementación proporciona instrucciones paso a paso para implementar y probar sus componentes.</span><span class="sxs-lookup"><span data-stu-id="5f94d-180">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>