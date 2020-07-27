---
title: Implementar un grupo de disponibilidad de SQL Server 2016 en Azure y Azure Stack Hub
description: Aprenda a implementar un grupo de disponibilidad de SQL Server 2016 en Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 85b859457b9b54a973c5fc23329b927212b60a07
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477089"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="0173d-103">Implementar un grupo de disponibilidad de SQL Server 2016 en Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="0173d-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="0173d-104">Este artículo le guiará a lo largo de una implementación automatizada de un clúster de SQL Server 2016 Enterprise de alta disponibilidad (HA) con un sitio de recuperación ante desastres (DR) asincrónica entre dos entornos de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0173d-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="0173d-105">Para obtener más información acerca de SQL Server 2016 y la alta disponibilidad, vea [Grupos de disponibilidad Always On: una solución de alta disponibilidad y recuperación ante desastres](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="0173d-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="0173d-106">En esta solución, creará un entorno de ejemplo para:</span><span class="sxs-lookup"><span data-stu-id="0173d-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="0173d-107">Organizar una implementación en dos instancias de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0173d-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="0173d-108">Usar Docker para minimizar los problemas de dependencia con los perfiles de la API de Azure.</span><span class="sxs-lookup"><span data-stu-id="0173d-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="0173d-109">Implementar un clúster de SQL Server 2016 Enterprise de alta disponibilidad básico con un sitio de recuperación ante desastres.</span><span class="sxs-lookup"><span data-stu-id="0173d-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="0173d-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="0173d-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="0173d-111">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="0173d-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="0173d-112">Azure Stack Hub aporta la agilidad e innovación de la informática en la nube a su entorno local, ya que habilita la única nube híbrida que permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="0173d-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="0173d-113">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los pilares de la calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="0173d-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="0173d-114">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="0173d-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="0173d-115">Arquitectura para SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="0173d-115">Architecture for SQL Server 2016</span></span>

![Azure Stack Hub de SQL de alta disponibilidad de SQL Server 2016](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="0173d-117">Requisitos previos para SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="0173d-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="0173d-118">Dos sistemas integrados de Azure Stack Hub (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="0173d-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="0173d-119">Este desarrollo no funciona en el Kit de desarrollo de Azure Stack (ASDK).</span><span class="sxs-lookup"><span data-stu-id="0173d-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="0173d-120">Para más información sobre Azure Stack Hub, consulte la [introducción a Azure Stack Hub](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="0173d-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="0173d-121">Una suscripción de inquilino en cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0173d-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="0173d-122">**Tome nota de cada identificador de suscripción y del punto de conexión de Azure Resource Manager para cada instancia de Azure Stack Hub.**</span><span class="sxs-lookup"><span data-stu-id="0173d-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="0173d-123">Una entidad de servicio de Azure Active Directory (Azure AD) que tenga permisos para la suscripción del inquilino en cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0173d-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="0173d-124">Es posible que deba crear dos entidades de servicio si las instancias de Azure Stack Hub se implementan en diferentes inquilinos de Azure AD.</span><span class="sxs-lookup"><span data-stu-id="0173d-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="0173d-125">Para aprender a crear una entidad de servicio para Azure Stack Hub, consulte [Creación de entidades de servicio para otorgar a las aplicaciones acceso a los recursos de Azure Stack Hub](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="0173d-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="0173d-126">**Tome nota del id. de aplicación, el secreto de cliente y el nombre de inquilino (xxxxx.onmicrosoft.com) de cada entidad de servicio.**</span><span class="sxs-lookup"><span data-stu-id="0173d-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="0173d-127">SQL Server 2016 Enterprise sindicado en el Marketplace de cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0173d-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="0173d-128">Para más información acerca de la redifusión de Marketplace, consulte [Descarga de elementos de Marketplace en Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="0173d-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="0173d-129">**Asegúrese de que su organización disponga de las licencias de SQL correspondientes.**</span><span class="sxs-lookup"><span data-stu-id="0173d-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="0173d-130">[Docker para Windows](https://docs.docker.com/docker-for-windows/) instalado en la máquina local.</span><span class="sxs-lookup"><span data-stu-id="0173d-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="0173d-131">Obtener la imagen de Docker</span><span class="sxs-lookup"><span data-stu-id="0173d-131">Get the Docker image</span></span>

<span data-ttu-id="0173d-132">Las imágenes de Docker para cada implementación eliminan los problemas de dependencia entre las distintas versiones de Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="0173d-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="0173d-133">Asegúrese de que Docker para Windows utiliza contenedores de Windows.</span><span class="sxs-lookup"><span data-stu-id="0173d-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="0173d-134">Ejecute el siguiente script en un símbolo del sistema con privilegios elevados para obtener el contenedor de Docker con los scripts de implementación.</span><span class="sxs-lookup"><span data-stu-id="0173d-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="0173d-135">Implementar el grupo de disponibilidad</span><span class="sxs-lookup"><span data-stu-id="0173d-135">Deploy the availability group</span></span>

1. <span data-ttu-id="0173d-136">Una vez extraída correctamente la imagen de contenedor, inicie la imagen.</span><span class="sxs-lookup"><span data-stu-id="0173d-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="0173d-137">Una vez iniciado el contenedor, se le ofrecerá un terminal de PowerShell con privilegios elevados en el contenedor.</span><span class="sxs-lookup"><span data-stu-id="0173d-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="0173d-138">Cambie los directorios para acceder al script de implementación.</span><span class="sxs-lookup"><span data-stu-id="0173d-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="0173d-139">Ejecute la implementación.</span><span class="sxs-lookup"><span data-stu-id="0173d-139">Run the deployment.</span></span> <span data-ttu-id="0173d-140">Proporcione las credenciales y los nombres de los recursos cuando sea necesario.</span><span class="sxs-lookup"><span data-stu-id="0173d-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="0173d-141">Alta disponibilidad hace referencia a la instancia de Azure Stack Hub en la que se instalará el clúster de alta disponibilidad.</span><span class="sxs-lookup"><span data-stu-id="0173d-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="0173d-142">Recuperación ante desastres hace referencia a la instancia de Azure Stack Hub en la que se instalará el clúster de recuperación ante desastres.</span><span class="sxs-lookup"><span data-stu-id="0173d-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="0173d-143">Escriba `Y` para permitir que el proveedor de NuGet se instale, lo que iniciará los módulos "2018-03-01-hybrid" del perfil de la API que se van a instalar.</span><span class="sxs-lookup"><span data-stu-id="0173d-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="0173d-144">Espere a que la implementación de recursos se complete.</span><span class="sxs-lookup"><span data-stu-id="0173d-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="0173d-145">Una vez completada la implementación de recursos de DR, salga del contenedor.</span><span class="sxs-lookup"><span data-stu-id="0173d-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="0173d-146">Para inspeccionar la implementación, observe los recursos del portal de cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="0173d-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="0173d-147">Conéctese a una de las instancias de SQL en el entorno de alta disponibilidad e inspeccione el grupo de disponibilidad mediante SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="0173d-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![Alta disponibilidad de SQL de SQL Server 2016](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="0173d-149">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="0173d-149">Next steps</span></span>

- <span data-ttu-id="0173d-150">Use SQL Server Management Studio para conmutar por error el clúster.</span><span class="sxs-lookup"><span data-stu-id="0173d-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="0173d-151">Consulte [Realización de una conmutación por error manual forzada de un grupo de disponibilidad Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="0173d-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="0173d-152">Más información sobre las aplicaciones en la nube híbrida.</span><span class="sxs-lookup"><span data-stu-id="0173d-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="0173d-153">Consulte [Soluciones de nube híbrida.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="0173d-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="0173d-154">Use sus propios datos o modifique el código según este ejemplo en [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="0173d-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
