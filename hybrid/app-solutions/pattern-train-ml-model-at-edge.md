---
title: Entrenamiento del modelo de Machine Learning en el patrón perimetral
description: Aprenda a realizar entrenamiento del modelo de Machine Learning en el borde con Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911950"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="1d2a4-103">Entrenamiento del modelo de Machine Learning en el patrón perimetral</span><span class="sxs-lookup"><span data-stu-id="1d2a4-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="1d2a4-104">Genere modelos de Machine Learning portables a partir de datos que solo existen en el entorno local.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="1d2a4-105">Contexto y problema</span><span class="sxs-lookup"><span data-stu-id="1d2a4-105">Context and problem</span></span>

<span data-ttu-id="1d2a4-106">Muchas organizaciones desean desbloquear información de sus datos locales o heredados mediante herramientas que entienden sus científicos de datos.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="1d2a4-107">[Azure Machine Learning](/azure/machine-learning/) proporciona herramientas nativas de la nube para entrenar, ajustar e implementar modelos de Machine Learning y aprendizaje profundo.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="1d2a4-108">Sin embargo, algunos datos son demasiado grandes para enviarlos a la nube, o bien no se pueden enviar por motivos normativos.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="1d2a4-109">Con este patrón, los científicos de datos pueden usar Azure Machine Learning para entrenar modelos mediante datos locales y proceso.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="1d2a4-110">Solución</span><span class="sxs-lookup"><span data-stu-id="1d2a4-110">Solution</span></span>

<span data-ttu-id="1d2a4-111">El entrenamiento en el patrón perimetral usa una máquina virtual (VM) que se ejecuta en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="1d2a4-112">La máquina virtual se registra como destino de proceso en Azure Machine Learning, lo que le permite obtener acceso a datos disponibles solo en el entorno local.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="1d2a4-113">En este caso, los datos se almacenan en Blob Storage de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="1d2a4-114">Una vez entrenado el modelo, se registra con Azure ML, con contenedores, y se agrega a una instancia de Azure Container Registry para su implementación.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="1d2a4-115">Para esta iteración del patrón, se debe poder acceder a la máquina virtual de entrenamiento de Azure Stack Hub a través de la red pública de Internet.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="1d2a4-116">[![Entrenamiento del modelo de Machine Learning en la arquitectura perimetral](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="1d2a4-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="1d2a4-117">Así es como funciona el patrón:</span><span class="sxs-lookup"><span data-stu-id="1d2a4-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="1d2a4-118">La máquina virtual de Azure Stack Hub se implementa y se registra como destino de proceso con Azure ML.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="1d2a4-119">Se crea un experimento en Azure ML que usa la máquina virtual de Azure Stack Hub como destino de proceso.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="1d2a4-120">Una vez que el modelo se ha entrenado, se registra y se incluye en un contenedor.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="1d2a4-121">Ahora se puede implementar el modelo en ubicaciones del entorno local o la nube.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="1d2a4-122">Componentes</span><span class="sxs-lookup"><span data-stu-id="1d2a4-122">Components</span></span>

<span data-ttu-id="1d2a4-123">Esta solución usa los siguientes componentes:</span><span class="sxs-lookup"><span data-stu-id="1d2a4-123">This solution uses the following components:</span></span>

| <span data-ttu-id="1d2a4-124">Nivel</span><span class="sxs-lookup"><span data-stu-id="1d2a4-124">Layer</span></span> | <span data-ttu-id="1d2a4-125">Componente</span><span class="sxs-lookup"><span data-stu-id="1d2a4-125">Component</span></span> | <span data-ttu-id="1d2a4-126">Descripción</span><span class="sxs-lookup"><span data-stu-id="1d2a4-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="1d2a4-127">Azure</span><span class="sxs-lookup"><span data-stu-id="1d2a4-127">Azure</span></span> | <span data-ttu-id="1d2a4-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="1d2a4-128">Azure Machine Learning</span></span> | <span data-ttu-id="1d2a4-129">[Azure Machine Learning](/azure/machine-learning/) orquesta el entrenamiento del modelo de ML.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="1d2a4-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="1d2a4-130">Azure Container Registry</span></span> | <span data-ttu-id="1d2a4-131">Azure ML empaqueta el modelo en un contenedor y lo almacena en una instancia de [Azure Container Registry](/azure/container-registry/) para su implementación.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="1d2a4-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="1d2a4-132">Azure Stack Hub</span></span> | <span data-ttu-id="1d2a4-133">App Service</span><span class="sxs-lookup"><span data-stu-id="1d2a4-133">App Service</span></span> | <span data-ttu-id="1d2a4-134">[Azure Stack Hub con App Service](/azure-stack/operator/azure-stack-app-service-overview) proporciona la base para los componentes en el perímetro.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="1d2a4-135">Proceso</span><span class="sxs-lookup"><span data-stu-id="1d2a4-135">Compute</span></span> | <span data-ttu-id="1d2a4-136">Una máquina virtual de Azure Stack Hub que ejecuta Ubuntu con Docker se usa para entrenar el modelo de ML.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="1d2a4-137">Storage</span><span class="sxs-lookup"><span data-stu-id="1d2a4-137">Storage</span></span> | <span data-ttu-id="1d2a4-138">Los datos privados se pueden hospedar en Blob Storage de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="1d2a4-139">Problemas y consideraciones</span><span class="sxs-lookup"><span data-stu-id="1d2a4-139">Issues and considerations</span></span>

<span data-ttu-id="1d2a4-140">Tenga en cuenta los puntos siguientes al decidir cómo implementar esta solución:</span><span class="sxs-lookup"><span data-stu-id="1d2a4-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="1d2a4-141">Escalabilidad</span><span class="sxs-lookup"><span data-stu-id="1d2a4-141">Scalability</span></span>

<span data-ttu-id="1d2a4-142">Con el fin de habilitar esta solución para escalarla, tendrá que crear una máquina virtual del tamaño adecuado en Azure Stack Hub para su entrenamiento.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="1d2a4-143">Disponibilidad</span><span class="sxs-lookup"><span data-stu-id="1d2a4-143">Availability</span></span>

<span data-ttu-id="1d2a4-144">Asegúrese de que los scripts de entrenamiento y la máquina virtual de Azure Stack Hub tienen acceso a los datos locales usados para el entrenamiento.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="1d2a4-145">Facilidad de uso</span><span class="sxs-lookup"><span data-stu-id="1d2a4-145">Manageability</span></span>

<span data-ttu-id="1d2a4-146">Asegúrese de que los modelos y experimentos se registran y etiquetan, y de que se crean versiones de los mismos, a fin de evitar confusiones durante la implementación de modelo.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="1d2a4-147">Seguridad</span><span class="sxs-lookup"><span data-stu-id="1d2a4-147">Security</span></span>

<span data-ttu-id="1d2a4-148">Este patrón permite a Azure Machine Learning obtener acceso a posibles datos confidenciales en el entorno local.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="1d2a4-149">Asegúrese de que la cuenta que se usa para SSH en la máquina virtual de Azure Stack Hub tiene una contraseña segura y de que los scripts de entrenamiento no conservan ni cargan datos en la nube.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1d2a4-150">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="1d2a4-150">Next steps</span></span>

<span data-ttu-id="1d2a4-151">Para más información sobre los temas presentados en este artículo:</span><span class="sxs-lookup"><span data-stu-id="1d2a4-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="1d2a4-152">Consulte la [documentación de Azure Machine Learning](/azure/machine-learning) para obtener información general de ML y temas relacionados.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="1d2a4-153">Consulte [Azure Container Registry](/azure/container-registry/) para obtener información sobre cómo compilar, almacenar y administrar imágenes para implementaciones de contenedores.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="1d2a4-154">Consulte [App Service en Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) para obtener más información sobre el proveedor de recursos y cómo realizar la implementación.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="1d2a4-155">Consulte [Consideraciones sobre el diseño de aplicaciones híbridas](overview-app-design-considerations.md) para más información sobre los procedimientos recomendados y para obtener respuestas a más preguntas.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="1d2a4-156">Consulte la información relativa a la [familia de productos y soluciones de Azure Stack](/azure-stack) para más información sobre toda la gama de productos y soluciones.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="1d2a4-157">Cuando esté listo para probar la solución de ejemplo, continúe con la [guía de implementación Entrenamiento del modelo de aprendizaje automático en el perímetro](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="1d2a4-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="1d2a4-158">La guía de implementación proporciona instrucciones paso a paso para implementar y probar sus componentes.</span><span class="sxs-lookup"><span data-stu-id="1d2a4-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
