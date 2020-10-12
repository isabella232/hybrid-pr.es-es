---
title: Implementar una solución MongoDB de alta disponibilidad en Azure y Azure Stack Hub
description: Aprenda a implementar una solución MongoDB de alta disponibilidad en Azure y Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852514"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="28f49-103">Implementar una solución MongoDB de alta disponibilidad en Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="28f49-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="28f49-104">Este artículo le guiará a lo largo de una implementación automatizada de un clúster de MongoDB de alta disponibilidad (HA) básico con un sitio de recuperación ante desastres (DR) entre dos entornos de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28f49-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="28f49-105">Para obtener más información acerca de MongoDB y la alta disponibilidad, consulte [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/) (Miembros del conjunto de réplicas).</span><span class="sxs-lookup"><span data-stu-id="28f49-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="28f49-106">En esta solución, creará un entorno de ejemplo para:</span><span class="sxs-lookup"><span data-stu-id="28f49-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="28f49-107">Organizar una implementación en dos instancias de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28f49-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="28f49-108">Usar Docker para minimizar los problemas de dependencia con los perfiles de la API de Azure.</span><span class="sxs-lookup"><span data-stu-id="28f49-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="28f49-109">Implementar un clúster de MongoDB de alta disponibilidad básico con un sitio de recuperación ante desastres.</span><span class="sxs-lookup"><span data-stu-id="28f49-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="28f49-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="28f49-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="28f49-111">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="28f49-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="28f49-112">Azure Stack Hub aporta la agilidad e innovación de la informática en la nube a su entorno local, ya que habilita la única nube híbrida que permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="28f49-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="28f49-113">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los pilares de la calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="28f49-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="28f49-114">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="28f49-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="28f49-115">Arquitectura de MongoDB con Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="28f49-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Arquitectura de MongoDB de alta disponibilidad en Azure Stack Hub](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="28f49-117">Requisitos previos para MongoDB con Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="28f49-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="28f49-118">Dos sistemas integrados de Azure Stack Hub (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="28f49-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="28f49-119">Esta implementación no funciona en el Kit de desarrollo de Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="28f49-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="28f49-120">Para obtener más información sobre Azure Stack Hub, consulte [¿Qué es Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="28f49-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="28f49-121">Una suscripción de inquilino en cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28f49-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="28f49-122">**Tome nota de cada identificador de suscripción y del punto de conexión de Azure Resource Manager para cada instancia de Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="28f49-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="28f49-123">Una entidad de servicio de Azure Active Directory (Azure AD) que tenga permisos para la suscripción del inquilino en cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28f49-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="28f49-124">Es posible que deba crear dos entidades de servicio si las instancias de Azure Stack Hub se implementan en diferentes inquilinos de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="28f49-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="28f49-125">Para aprender a crear una entidad de servicio para Azure Stack Hub, consulte [Uso de una identidad de aplicación para acceder a recursos de Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="28f49-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="28f49-126">**Tome nota del id. de aplicación, el secreto de cliente y el nombre de inquilino (xxxxx.onmicrosoft.com) de cada entidad de servicio.**</span><span class="sxs-lookup"><span data-stu-id="28f49-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="28f49-127">Ubuntu 16.04 sindicado en el Marketplace de cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="28f49-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="28f49-128">Para más información acerca de la redifusión de Marketplace, consulte [Descarga de elementos de Marketplace en Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="28f49-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="28f49-129">[Docker para Windows](https://docs.docker.com/docker-for-windows/) instalado en la máquina local.</span><span class="sxs-lookup"><span data-stu-id="28f49-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="28f49-130">Obtener la imagen de Docker</span><span class="sxs-lookup"><span data-stu-id="28f49-130">Get the Docker image</span></span>

<span data-ttu-id="28f49-131">Las imágenes de Docker para cada implementación eliminan los problemas de dependencia entre las distintas versiones de Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="28f49-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="28f49-132">Asegúrese de que Docker para Windows utiliza contenedores de Windows.</span><span class="sxs-lookup"><span data-stu-id="28f49-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="28f49-133">Ejecute el siguiente comando en un símbolo del sistema con privilegios elevados para obtener el contenedor de Docker con los scripts de implementación.</span><span class="sxs-lookup"><span data-stu-id="28f49-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="28f49-134">Implementar los clústeres</span><span class="sxs-lookup"><span data-stu-id="28f49-134">Deploy the clusters</span></span>

1. <span data-ttu-id="28f49-135">Una vez extraída correctamente la imagen de contenedor, inicie la imagen.</span><span class="sxs-lookup"><span data-stu-id="28f49-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="28f49-136">Una vez iniciado el contenedor, se le ofrecerá un terminal de PowerShell con privilegios elevados en el contenedor.</span><span class="sxs-lookup"><span data-stu-id="28f49-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="28f49-137">Cambie los directorios para acceder al script de implementación.</span><span class="sxs-lookup"><span data-stu-id="28f49-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="28f49-138">Ejecute la implementación.</span><span class="sxs-lookup"><span data-stu-id="28f49-138">Run the deployment.</span></span> <span data-ttu-id="28f49-139">Proporcione las credenciales y los nombres de los recursos cuando sea necesario.</span><span class="sxs-lookup"><span data-stu-id="28f49-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="28f49-140">Alta disponibilidad hace referencia a la instancia de Azure Stack Hub en la que se implementará el clúster de alta disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="28f49-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="28f49-141">Recuperación ante desastres hace referencia a la instancia de Azure Stack Hub en la que se implementará el clúster de recuperación ante desastres.</span><span class="sxs-lookup"><span data-stu-id="28f49-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. <span data-ttu-id="28f49-142">Escriba `Y` para permitir que el proveedor de NuGet se instale, lo que iniciará los módulos "2018-03-01-hybrid" del perfil de la API que se van a instalar.</span><span class="sxs-lookup"><span data-stu-id="28f49-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="28f49-143">Los recursos de alta disponibilidad se implementarán en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="28f49-143">The HA resources will deploy first.</span></span> <span data-ttu-id="28f49-144">Supervise la implementación y espere a que finalice.</span><span class="sxs-lookup"><span data-stu-id="28f49-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="28f49-145">Una vez que tenga el mensaje que indique que la implementación de alta disponibilidad ha finalizado, en el portal de Azure Stack Hub de alta disponibilidad podrá ver los recursos implementados.</span><span class="sxs-lookup"><span data-stu-id="28f49-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="28f49-146">Continúe con la implementación de recursos de DR y decida si quiere habilitar un jumpbox en Azure Stack Hub de DR para interactuar con el clúster.</span><span class="sxs-lookup"><span data-stu-id="28f49-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="28f49-147">Espere hasta que finalice la implementación de recursos de recuperación ante desastres.</span><span class="sxs-lookup"><span data-stu-id="28f49-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="28f49-148">Una vez que haya finalizado la implementación de recursos de recuperación ante desastres, salga del contenedor.</span><span class="sxs-lookup"><span data-stu-id="28f49-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="28f49-149">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="28f49-149">Next steps</span></span>

- <span data-ttu-id="28f49-150">Si habilitó la VM de jumpbox en Azure Stack Hub de DR, puede conectarse mediante SSH e interactuar con el clúster de MongoDB mediante la instalación de la CLI de mongo.</span><span class="sxs-lookup"><span data-stu-id="28f49-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="28f49-151">Para obtener más información sobre cómo interactuar con MongoDB, consulte [The mongo Shell](https://docs.mongodb.com/manual/mongo/) (El shell mongo).</span><span class="sxs-lookup"><span data-stu-id="28f49-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="28f49-152">Para más información sobre las aplicaciones en la nube híbrida, consulte [Soluciones en la nube híbrida.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="28f49-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="28f49-153">Modifique el código según este ejemplo de [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="28f49-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>