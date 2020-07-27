---
title: Implementación de una aplicación híbrida con datos locales que se escalan en la nube
description: Aprenda a implementar aplicaciones que usan datos locales y que se escalan en la nube mediante Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477327"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="d6e51-103">Implementación de una aplicación híbrida con datos locales que se escalan en la nube</span><span class="sxs-lookup"><span data-stu-id="d6e51-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="d6e51-104">En esta guía de soluciones se muestra cómo implementar una aplicación híbrida que abarca Azure y Azure Stack Hub, y que usa un solo origen de datos local.</span><span class="sxs-lookup"><span data-stu-id="d6e51-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="d6e51-105">Mediante el uso de una solución en la nube híbrida, puede combinar las ventajas de cumplimiento de una nube privada con la escalabilidad de la nube pública.</span><span class="sxs-lookup"><span data-stu-id="d6e51-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="d6e51-106">Los desarrolladores pueden aprovechar el ecosistema de desarrolladores de Microsoft y aplicar sus aptitudes a los entornos en la nube y locales.</span><span class="sxs-lookup"><span data-stu-id="d6e51-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="d6e51-107">Introducción y supuestos</span><span class="sxs-lookup"><span data-stu-id="d6e51-107">Overview and assumptions</span></span>

<span data-ttu-id="d6e51-108">Siga este tutorial para configurar un flujo de trabajo que permita a los desarrolladores implementar una aplicación web idéntica en una nube pública y en una privada.</span><span class="sxs-lookup"><span data-stu-id="d6e51-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="d6e51-109">Esta aplicación puede acceder a una red enrutable sin conexión a Internet hospedada en la nube privada.</span><span class="sxs-lookup"><span data-stu-id="d6e51-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="d6e51-110">Estas aplicaciones web se supervisan y, cuando hay un pico de tráfico, un programa modifica los registros del DNS para redirigir el tráfico a la nube pública.</span><span class="sxs-lookup"><span data-stu-id="d6e51-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="d6e51-111">Cuando el tráfico disminuye hasta el nivel anterior al pico, el tráfico se enruta de vuelta a la nube privada.</span><span class="sxs-lookup"><span data-stu-id="d6e51-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="d6e51-112">En este tutorial se describen las tareas siguientes:</span><span class="sxs-lookup"><span data-stu-id="d6e51-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d6e51-113">Implementar un servidor de base de datos de SQL Server con conexión híbrida.</span><span class="sxs-lookup"><span data-stu-id="d6e51-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="d6e51-114">Conectar una aplicación web de Azure global a una red híbrida.</span><span class="sxs-lookup"><span data-stu-id="d6e51-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="d6e51-115">Configurar DNS para el escalado entre nubes.</span><span class="sxs-lookup"><span data-stu-id="d6e51-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d6e51-116">Configurar certificados SSL para el escalado entre nubes.</span><span class="sxs-lookup"><span data-stu-id="d6e51-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d6e51-117">Configurar e implementar la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="d6e51-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="d6e51-118">Crear un perfil de Traffic Manager y configurarlo para el escalado entre nubes.</span><span class="sxs-lookup"><span data-stu-id="d6e51-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="d6e51-119">Configurar la supervisión y las alertas de Application Insights para un aumento del tráfico.</span><span class="sxs-lookup"><span data-stu-id="d6e51-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="d6e51-120">Configuración de la conmutación automática del tráfico entre Azure global y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="d6e51-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d6e51-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d6e51-122">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d6e51-123">Azure Stack Hub aporta la agilidad y la innovación de la informática en la nube a su entorno local y hace posible la única nube híbrida que le permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="d6e51-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d6e51-124">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los fundamentos de calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="d6e51-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d6e51-125">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="d6e51-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="d6e51-126">Supuestos</span><span class="sxs-lookup"><span data-stu-id="d6e51-126">Assumptions</span></span>

<span data-ttu-id="d6e51-127">En este tutorial se da por supuesto que tiene conocimientos básicos de Azure global y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d6e51-128">Para más información antes de iniciar el tutorial, consulte los siguientes artículos:</span><span class="sxs-lookup"><span data-stu-id="d6e51-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="d6e51-129">Introducción a Azure</span><span class="sxs-lookup"><span data-stu-id="d6e51-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="d6e51-130">Conceptos clave de Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="d6e51-131">En este tutorial también se supone que tiene una suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="d6e51-132">Si no tiene una suscripción, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.</span><span class="sxs-lookup"><span data-stu-id="d6e51-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d6e51-133">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="d6e51-133">Prerequisites</span></span>

<span data-ttu-id="d6e51-134">Antes de comenzar esta solución, asegúrese de que cumple los requisitos siguientes:</span><span class="sxs-lookup"><span data-stu-id="d6e51-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="d6e51-135">Un Kit de desarrollo de Azure Stack (ASDK) o una suscripción a un sistema integrado de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="d6e51-136">Para implementar el ASDK, siga las instrucciones que encontrará en el artículo acerca de la [implementación del Kit de desarrollo de Azure Stack mediante el instalador](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="d6e51-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="d6e51-137">Para la instalación de Azure Stack Hub es necesario tener instalado lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="d6e51-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="d6e51-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="d6e51-138">The Azure App Service.</span></span> <span data-ttu-id="d6e51-139">Trabaje con su operador de Azure Stack Hub para implementar y configurar Azure App Service en su entorno.</span><span class="sxs-lookup"><span data-stu-id="d6e51-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="d6e51-140">Este tutorial requiere que App Service tenga al menos un (1) rol de trabajo dedicado disponible.</span><span class="sxs-lookup"><span data-stu-id="d6e51-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="d6e51-141">Una imagen de Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="d6e51-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="d6e51-142">Una imagen de Windows Server 2016 con Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d6e51-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="d6e51-143">Los planes y ofertas adecuados.</span><span class="sxs-lookup"><span data-stu-id="d6e51-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="d6e51-144">Un nombre de dominio para la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="d6e51-144">A domain name for your web app.</span></span> <span data-ttu-id="d6e51-145">Si no tiene un nombre de dominio, puede comprarlo a un proveedor de dominios como GoDaddy, Bluehost o InMotion.</span><span class="sxs-lookup"><span data-stu-id="d6e51-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="d6e51-146">Un certificado SSL para el dominio de una entidad de certificación de confianza como LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="d6e51-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="d6e51-147">Una aplicación web que se comunica con una base de datos de SQL Server y es compatible con Application Insights.</span><span class="sxs-lookup"><span data-stu-id="d6e51-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="d6e51-148">Puede descargar la aplicación de ejemplo [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) de GitHub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="d6e51-149">Una red híbrida entre una red virtual de Azure y la red virtual de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="d6e51-150">Para instrucciones detalladas, consulte [Configuración de la conectividad de la nube híbrida con Azure y Azure Stack Hub](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="d6e51-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="d6e51-151">Una canalización híbrida de integración continua e implementación continua (CI/CD) con un agente de compilación privado en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="d6e51-152">Para instrucciones detalladas, consulte [Configuración de la identidad de nube híbrida con aplicaciones de Azure y Azure Stack Hub](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="d6e51-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="d6e51-153">Implementación de un servidor de base de datos de SQL Server con conexión híbrida</span><span class="sxs-lookup"><span data-stu-id="d6e51-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="d6e51-154">Inicie sesión en el portal de usuarios de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="d6e51-155">En **Dashboard** (Panel), seleccione **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Marketplace de Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="d6e51-157">En **Marketplace**, seleccione **Compute** (Proceso) y, a continuación, elija **More** (Más).</span><span class="sxs-lookup"><span data-stu-id="d6e51-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="d6e51-158">En **Más**, seleccione **Free SQL Server License: SQL Server 2017 Developer en Windows Server**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Seleccione una imagen de máquina virtual en el portal de usuarios de Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="d6e51-160">En **Free SQL Server License: SQL Server 2017 Developer en Windows Server**, seleccione **Crear**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="d6e51-161">En **Aspectos básicos > Configurar opciones básicas**, proporcione un **Nombre** para la máquina virtual (VM), un **Nombre de usuario**  para el usuario SA de SQL Server y una **Contraseña** para el usuario SA.</span><span class="sxs-lookup"><span data-stu-id="d6e51-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="d6e51-162">En la lista desplegable **Suscripción**, seleccione la suscripción en la que se va a realizar la implementación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="d6e51-163">En **Resource group** (Grupo de recursos), use **Choose existing** (Elegir existente) y coloque la máquina virtual en el mismo grupo de recursos que la aplicación web de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Configuración de los valores básicos para la máquina virtual en el portal de usuarios de Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="d6e51-165">En **Size** (Tamaño), elija un tamaño de máquina virtual.</span><span class="sxs-lookup"><span data-stu-id="d6e51-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="d6e51-166">Para este tutorial se recomienda A2_Standard o DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="d6e51-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="d6e51-167">En **Configuración > Configurar características opcionales**, configure las siguientes opciones:</span><span class="sxs-lookup"><span data-stu-id="d6e51-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="d6e51-168">**Cuenta de almacenamiento**: Si necesita una, cree una nueva cuenta.</span><span class="sxs-lookup"><span data-stu-id="d6e51-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="d6e51-169">**Red virtual**:</span><span class="sxs-lookup"><span data-stu-id="d6e51-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="d6e51-170">Asegúrese de que la máquina virtual de SQL Server se implementa en la misma red virtual que las puertas de enlace de VPN.</span><span class="sxs-lookup"><span data-stu-id="d6e51-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="d6e51-171">**Dirección IP pública**: Use la configuración predeterminada.</span><span class="sxs-lookup"><span data-stu-id="d6e51-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="d6e51-172">**Grupo de seguridad de red**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="d6e51-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="d6e51-173">Cree un nuevo grupo de seguridad de red.</span><span class="sxs-lookup"><span data-stu-id="d6e51-173">Create a new NSG.</span></span>
   - <span data-ttu-id="d6e51-174">**Extensiones y Supervisión**: Mantenga la configuración predeterminada.</span><span class="sxs-lookup"><span data-stu-id="d6e51-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="d6e51-175">**Cuenta de almacenamiento de diagnóstico**: Si necesita una, cree una nueva cuenta.</span><span class="sxs-lookup"><span data-stu-id="d6e51-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="d6e51-176">Seleccione **OK** (Aceptar) para guardar la configuración.</span><span class="sxs-lookup"><span data-stu-id="d6e51-176">Select **OK** to save your configuration.</span></span>

     ![Configuración de las características opcionales de la máquina virtual en el portal de usuarios de Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="d6e51-178">En **SQL Server settings** (Configuración de SQL Server), configure las siguientes opciones:</span><span class="sxs-lookup"><span data-stu-id="d6e51-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="d6e51-179">En **SQL connectivity** (Conectividad SQL), seleccione **Public (Internet)** (Pública [Internet]).</span><span class="sxs-lookup"><span data-stu-id="d6e51-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="d6e51-180">En **Port** (Puerto), mantenga el valor predeterminado, **1433**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="d6e51-181">En **SQL authentication** (Autenticación SQL), seleccione **Enable** (Habilitar).</span><span class="sxs-lookup"><span data-stu-id="d6e51-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="d6e51-182">Al habilitar la autenticación SQL, se rellena de forma automática con la información de "SQLAdmin" que configuró en **Aspectos básicos**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="d6e51-183">Para el resto de la configuración, conserve los valores predeterminados.</span><span class="sxs-lookup"><span data-stu-id="d6e51-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="d6e51-184">Seleccione **Aceptar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-184">Select **OK**.</span></span>

     ![Configuración de SQL Server en el portal de usuarios de Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="d6e51-186">En **Summary** (Resumen), revise la configuración de la máquina virtual y seleccione **OK** (Aceptar) para iniciar la implementación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Resumen de la configuración en el portal de usuarios de Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="d6e51-188">La nueva VM tarda algún tiempo en crearse.</span><span class="sxs-lookup"><span data-stu-id="d6e51-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="d6e51-189">Puede ver el estado de las máquinas virtuales en **Virtual machines** (Máquinas virtuales).</span><span class="sxs-lookup"><span data-stu-id="d6e51-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Estado de las máquinas virtuales en el portal de usuarios de Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="d6e51-191">Creación de aplicaciones web en Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6e51-192">Azure App Service simplifica la ejecución y administración de una aplicación web.</span><span class="sxs-lookup"><span data-stu-id="d6e51-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="d6e51-193">Dado que Azure Stack Hub es coherente con Azure, App Service se puede ejecutar en ambos entornos.</span><span class="sxs-lookup"><span data-stu-id="d6e51-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="d6e51-194">Usará App Service para hospedar la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="d6e51-195">Creación de aplicaciones web</span><span class="sxs-lookup"><span data-stu-id="d6e51-195">Create web apps</span></span>

1. <span data-ttu-id="d6e51-196">Cree una aplicación web en Azure siguiendo las instrucciones de [Administración de un plan de App Service en Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="d6e51-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="d6e51-197">Debe asegurarse de colocar la aplicación web en la misma suscripción y grupo de recursos que la red híbrida.</span><span class="sxs-lookup"><span data-stu-id="d6e51-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="d6e51-198">Repita el paso anterior (1) en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="d6e51-199">Incorporación de una ruta para Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="d6e51-200">App Service en Azure Stack Hub debe ser enrutable desde la red pública de Internet para que los usuarios puedan acceder a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="d6e51-201">Si la instancia de Azure Stack Hub es accesible desde Internet, anote la dirección IP pública o la dirección URL de la aplicación web de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="d6e51-202">Si usa un ASDK, puede [configurar una asignación NAT estática](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) para exponer App Service fuera del entorno virtual.</span><span class="sxs-lookup"><span data-stu-id="d6e51-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="d6e51-203">Conexión de una aplicación web de Azure a una red híbrida</span><span class="sxs-lookup"><span data-stu-id="d6e51-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="d6e51-204">Para proporcionar conectividad entre el front-end web en Azure y la base de datos de SQL Server en Azure Stack Hub, la aplicación web debe estar conectada a la red híbrida entre Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d6e51-205">Para habilitar la conectividad, tendrá que:</span><span class="sxs-lookup"><span data-stu-id="d6e51-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="d6e51-206">Configurar la conectividad de punto a sitio.</span><span class="sxs-lookup"><span data-stu-id="d6e51-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="d6e51-207">Configurar la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="d6e51-207">Configure the web app.</span></span>
- <span data-ttu-id="d6e51-208">Modificar la puerta de enlace de red local en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="d6e51-209">Configuración de la red virtual de Azure para la conectividad de punto a sitio</span><span class="sxs-lookup"><span data-stu-id="d6e51-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="d6e51-210">La puerta de enlace de red virtual en el lado de Azure de la red híbrida debe permitir conexiones de punto a sitio para la integración con Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="d6e51-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="d6e51-211">En Azure, vaya a la página de la puerta de enlace de red virtual.</span><span class="sxs-lookup"><span data-stu-id="d6e51-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="d6e51-212">En **Configuración**, seleccione **Configuración de punto a sitio**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Opción de punto a sitio en la puerta de enlace de red virtual de Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="d6e51-214">Seleccione **Configurar ahora** para la configuración de punto a sitio.</span><span class="sxs-lookup"><span data-stu-id="d6e51-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Iniciar configuración de punto a sitio en la puerta de enlace de red virtual de Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="d6e51-216">En la página de configuración **De punto a sitio**, escriba el intervalo de direcciones IP privadas que desea usar en **Grupo de direcciones**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="d6e51-217">Asegúrese de que el intervalo que especifique no se superponga con ninguno de los intervalos de direcciones ya utilizados por las subredes de Azure global o los componentes de Azure Stack Hub de la red híbrida.</span><span class="sxs-lookup"><span data-stu-id="d6e51-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="d6e51-218">En **Tipo de túnel**, desactive la opción **VPN IKEv2**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="d6e51-219">Seleccione **Guardar** para finalizar la configuración de punto a sitio.</span><span class="sxs-lookup"><span data-stu-id="d6e51-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Configuración de punto a sitio en la puerta de enlace de red virtual de Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="d6e51-221">Integración de la aplicación de Azure App Service con la red híbrida</span><span class="sxs-lookup"><span data-stu-id="d6e51-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="d6e51-222">Para conectar la aplicación a la red virtual de Azure, siga las instrucciones de [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration) (Integración de VNet requerida de la puerta de enlace).</span><span class="sxs-lookup"><span data-stu-id="d6e51-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="d6e51-223">Vaya a **Configuración** en el plan de App Service que hospeda la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="d6e51-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="d6e51-224">En **Configuración**, seleccione **Redes**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-224">In **Settings**, select **Networking**.</span></span>

    ![Configurar redes para el plan de App Service](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="d6e51-226">En **Integración de VNET**, seleccione **Haga clic aquí para administrar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Administración de la integración de red virtual para el plan de App Service](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="d6e51-228">Seleccione la red virtual que desea configurar.</span><span class="sxs-lookup"><span data-stu-id="d6e51-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="d6e51-229">En **DIRECCIONES IP ENRUTADAS A RED VIRTUAL**, escriba el intervalo de direcciones IP de la red virtual de Azure, la red virtual de Azure Stack Hub y los espacios de direcciones de punto a sitio.</span><span class="sxs-lookup"><span data-stu-id="d6e51-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="d6e51-230">Seleccione **Guardar** para validar y guardar la configuración.</span><span class="sxs-lookup"><span data-stu-id="d6e51-230">Select **Save** to validate and save these settings.</span></span>

    ![Intervalos de direcciones IP que se enrutan en la integración de la red virtual](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="d6e51-232">Para más información acerca de cómo se integra App Service con las redes virtuales de Azure, consulte [Integración de la aplicación con una red virtual de Azure](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="d6e51-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="d6e51-233">Configuración de la red virtual de Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="d6e51-234">La puerta de enlace de red local de la red virtual de Azure Stack Hub debe estar configurada para enrutar el tráfico desde el intervalo de direcciones de punto a sitio de App Service.</span><span class="sxs-lookup"><span data-stu-id="d6e51-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="d6e51-235">En Azure Stack Hub, vaya a **Local network gateway** (Puerta de enlace de red local).</span><span class="sxs-lookup"><span data-stu-id="d6e51-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="d6e51-236">En **Settings** (Configuración), seleccione **Configuration** (Configuración).</span><span class="sxs-lookup"><span data-stu-id="d6e51-236">Under **Settings**, select **Configuration**.</span></span>

    ![Opción de configuración de puerta de enlace en la puerta de enlace de red local de Azure Stack Hub](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="d6e51-238">En **Espacio de direcciones**, escriba el intervalo de direcciones de punto a sitio para la puerta de enlace de red virtual de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Espacio de direcciones de punto a sitio en la puerta de enlace de red local de Azure Stack Hub](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="d6e51-240">Seleccione **Guardar** para validar y guardar la configuración.</span><span class="sxs-lookup"><span data-stu-id="d6e51-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="d6e51-241">Configuración de DNS para el escalado entre nubes</span><span class="sxs-lookup"><span data-stu-id="d6e51-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="d6e51-242">Mediante la configuración adecuada de DNS en las aplicaciones de toda la nube, los usuarios pueden acceder a las instancias de Azure global y de Azure Stack Hub de la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="d6e51-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="d6e51-243">La configuración de DNS de este tutorial también permite que Azure Traffic Manager enrute el tráfico cuando la carga aumenta o disminuye.</span><span class="sxs-lookup"><span data-stu-id="d6e51-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="d6e51-244">En este tutorial, se usa Azure DNS para administrar el DNS porque los dominios de App Service no funcionarían.</span><span class="sxs-lookup"><span data-stu-id="d6e51-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="d6e51-245">Creación de subdominios</span><span class="sxs-lookup"><span data-stu-id="d6e51-245">Create subdomains</span></span>

<span data-ttu-id="d6e51-246">Dado que Traffic Manager se basa en registros CNAME de DNS, se necesita un subdominio para enrutar correctamente el tráfico a los puntos de conexión.</span><span class="sxs-lookup"><span data-stu-id="d6e51-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="d6e51-247">Para más información acerca de la asignación de dominios y registros DNS, consulte [Asignación de dominios con Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="d6e51-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="d6e51-248">Para el punto de conexión de Azure, creará un subdominio que los usuarios pueden utilizar para acceder a la aplicación web.</span><span class="sxs-lookup"><span data-stu-id="d6e51-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="d6e51-249">Para este tutorial, puede usar **app.northwind.com**, pero este valor se debe personalizar en función de su propio dominio.</span><span class="sxs-lookup"><span data-stu-id="d6e51-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="d6e51-250">También deberá crear un subdominio con un registro A para el punto de conexión de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="d6e51-251">Puede usar **azurestack.northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="d6e51-252">Configuración de un dominio personalizado en Azure</span><span class="sxs-lookup"><span data-stu-id="d6e51-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="d6e51-253">Agregue el nombre de host **app.northwind.com** a la aplicación web de Azure mediante la [Asignación de un registro CNAME a Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="d6e51-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="d6e51-254">Configuración de dominios personalizados en Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="d6e51-255">Agregue el nombre de host **azurestack.northwind.com** a la aplicación web de Azure Stack Hub mediante [Asignación de un registro A para Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="d6e51-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="d6e51-256">Use la dirección IP enrutable por Internet para la aplicación de App Service.</span><span class="sxs-lookup"><span data-stu-id="d6e51-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="d6e51-257">Agregue el nombre de host **app.northwind.com** a la aplicación web de Azure Stack Hub mediante [Asignación de un registro CNAME a Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="d6e51-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="d6e51-258">Utilice el nombre de host que configuró en el paso anterior (1) como destino para el registro CNAME.</span><span class="sxs-lookup"><span data-stu-id="d6e51-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="d6e51-259">Configuración de certificados SSL para el escalado entre nubes</span><span class="sxs-lookup"><span data-stu-id="d6e51-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="d6e51-260">Es importante asegurarse de que los datos confidenciales que recopila la aplicación web están seguros en reposo y en tránsito cuando se almacenan en la base de datos SQL.</span><span class="sxs-lookup"><span data-stu-id="d6e51-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="d6e51-261">Configurará las aplicaciones web de Azure y Azure Stack Hub para que usen certificados SSL con todo el tráfico entrante.</span><span class="sxs-lookup"><span data-stu-id="d6e51-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="d6e51-262">Adición de SSL en Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6e51-263">Para agregar SSL en Azure:</span><span class="sxs-lookup"><span data-stu-id="d6e51-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="d6e51-264">Asegúrese de que el certificado SSL que obtiene es válido para el subdominio que ha creado.</span><span class="sxs-lookup"><span data-stu-id="d6e51-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="d6e51-265">(Es correcto usar certificados con caracteres comodín).</span><span class="sxs-lookup"><span data-stu-id="d6e51-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="d6e51-266">En Azure, siga las instrucciones de las secciones **Preparación de la aplicación web** y **Enlace del certificado SSL** del artículo [Enlace de un certificado SSL personalizado existente con Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="d6e51-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="d6e51-267">Seleccione **SSL basado en SNI** como el **Tipo de SSL**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="d6e51-268">Redirija todo el tráfico al puerto HTTPS.</span><span class="sxs-lookup"><span data-stu-id="d6e51-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="d6e51-269">Siga las instrucciones de la sección **Aplicación de HTTPS** del artículo [Enlace de un certificado SSL personalizado existente con Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="d6e51-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="d6e51-270">Para agregar SSL a Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="d6e51-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="d6e51-271">Repita los pasos 1 a 3 que usó para Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="d6e51-272">Configurar e implementar la aplicación web</span><span class="sxs-lookup"><span data-stu-id="d6e51-272">Configure and deploy the web app</span></span>

<span data-ttu-id="d6e51-273">Deberá configurar el código de la aplicación para notificar los datos de telemetría a la instancia de Application Insights correcta y configurar las aplicaciones web con las cadenas de conexión apropiadas.</span><span class="sxs-lookup"><span data-stu-id="d6e51-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="d6e51-274">Para más información sobre Application Insights, consulte [¿Qué es Application Insights?](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="d6e51-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="d6e51-275">Adición de Application Insights</span><span class="sxs-lookup"><span data-stu-id="d6e51-275">Add Application Insights</span></span>

1. <span data-ttu-id="d6e51-276">Abra la aplicación web en Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="d6e51-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="d6e51-277">[Agregue Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) al proyecto para transmitir los datos de telemetría que utiliza Application Insights para crear alertas cuando el tráfico web aumenta o disminuye.</span><span class="sxs-lookup"><span data-stu-id="d6e51-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="d6e51-278">Configuración de cadenas de conexión dinámicas</span><span class="sxs-lookup"><span data-stu-id="d6e51-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="d6e51-279">Cada instancia de la aplicación web usará un método diferente para conectarse a la base de datos SQL.</span><span class="sxs-lookup"><span data-stu-id="d6e51-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="d6e51-280">La aplicación de Azure usa la dirección IP privada de la máquina virtual de SQL Server y la aplicación de Azure Stack Hub usa la dirección IP pública de la máquina virtual con SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d6e51-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="d6e51-281">En un sistema integrado de Azure Stack Hub, la dirección IP pública no debería ser enrutable en Internet.</span><span class="sxs-lookup"><span data-stu-id="d6e51-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="d6e51-282">En un Kit de desarrollo de Azure Stack, la dirección IP pública no se puede enrutar fuera del ASDK.</span><span class="sxs-lookup"><span data-stu-id="d6e51-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="d6e51-283">Puede usar variables de entorno de App Service para pasar una cadena de conexión distinta a cada instancia de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="d6e51-284">Abra la aplicación en Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="d6e51-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="d6e51-285">Abra Startup.cs y busque el bloque de código siguiente:</span><span class="sxs-lookup"><span data-stu-id="d6e51-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="d6e51-286">Reemplace el bloque de código anterior por el código siguiente, que usa una cadena de conexión definida en el archivo *appsettings.json*:</span><span class="sxs-lookup"><span data-stu-id="d6e51-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="d6e51-287">Configurar los parámetros de la aplicación en App Service</span><span class="sxs-lookup"><span data-stu-id="d6e51-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="d6e51-288">Cree cadenas de conexión para Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="d6e51-289">Las cadenas deben ser iguales, excepto las direcciones IP utilizadas.</span><span class="sxs-lookup"><span data-stu-id="d6e51-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="d6e51-290">En Azure y Azure Stack Hub, agregue la cadena de conexión adecuada [como una configuración de la aplicación](/azure/app-service/web-sites-configure) en la aplicación web y use `SQLCONNSTR\_` como prefijo en el nombre.</span><span class="sxs-lookup"><span data-stu-id="d6e51-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="d6e51-291">**Guarde** la configuración de la aplicación web y reinicie la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="d6e51-292">Habilitación del escalado automático en Azure global</span><span class="sxs-lookup"><span data-stu-id="d6e51-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="d6e51-293">Cuando se crea la aplicación web en un entorno de App Service, esta se inicia con una instancia.</span><span class="sxs-lookup"><span data-stu-id="d6e51-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="d6e51-294">Puede escalar horizontalmente de manera automática para agregar instancias y proporcionar más recursos de proceso a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="d6e51-295">De forma similar, puede reducir horizontalmente de manera automática y reducir el número de instancias que necesita la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="d6e51-296">Debe tener un plan de App Service para configurar el escalado y la reducción horizontales.</span><span class="sxs-lookup"><span data-stu-id="d6e51-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="d6e51-297">Si no tiene un plan, cree uno antes de iniciar los pasos siguientes.</span><span class="sxs-lookup"><span data-stu-id="d6e51-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="d6e51-298">Habilitación de la escalabilidad horizontal automática</span><span class="sxs-lookup"><span data-stu-id="d6e51-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="d6e51-299">En Azure, busque el plan de App Service para los sitios que desea escalar horizontalmente y, a continuación, seleccione **Escalar horizontalmente (plan de App Service)** .</span><span class="sxs-lookup"><span data-stu-id="d6e51-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Escalar horizontalmente Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="d6e51-301">Seleccione **Habilitar escalado automático**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-301">Select **Enable autoscale**.</span></span>

    ![Habilitar la escalabilidad automática en Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="d6e51-303">Escriba un nombre para **Nombre de la configuración de escalado automático**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="d6e51-304">Para la regla de escalado automático **Predeterminada**, seleccione **Escalado basado en una métrica**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="d6e51-305">Establezca el valor de **Límites de instancia** en **Mínimo: 1**, **Máximo: 10** y **Predeterminado: 1**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Configurar la escalabilidad automática en Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="d6e51-307">Seleccione **+Agregar una regla**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="d6e51-308">En **Origen de métrica**, seleccione **Recurso actual**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="d6e51-309">Utilice los siguientes criterios y acciones para la regla.</span><span class="sxs-lookup"><span data-stu-id="d6e51-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="d6e51-310">Criterios</span><span class="sxs-lookup"><span data-stu-id="d6e51-310">Criteria</span></span>

1. <span data-ttu-id="d6e51-311">En **Agregación de tiempo**, seleccione **Media**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="d6e51-312">En **Nombre de métrica**, seleccione **Porcentaje de CPU**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="d6e51-313">En **Operador**, seleccione **Mayor que**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="d6e51-314">Establezca **Umbral** en **50**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="d6e51-315">Establezca **Duración** en **10**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="d6e51-316">Acción</span><span class="sxs-lookup"><span data-stu-id="d6e51-316">Action</span></span>

1. <span data-ttu-id="d6e51-317">En **Operación**, seleccione **Aumentar recuento en**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="d6e51-318">Establezca **Recuento de instancias** en **2**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="d6e51-319">Establezca **Tiempo de finalización** en **5**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="d6e51-320">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-320">Select **Add**.</span></span>

5. <span data-ttu-id="d6e51-321">Seleccione **+Agregar una regla**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="d6e51-322">En **Origen de métrica**, seleccione **Recurso actual**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="d6e51-323">El recurso actual contendrá el nombre o el identificador único del plan de App Service y las listas desplegables **Tipo de recurso** y **Recurso** no estarán disponibles.</span><span class="sxs-lookup"><span data-stu-id="d6e51-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="d6e51-324">Habilitación de la reducción horizontal automática</span><span class="sxs-lookup"><span data-stu-id="d6e51-324">Enable automatic scale in</span></span>

<span data-ttu-id="d6e51-325">Cuando el tráfico disminuye, la aplicación web de Azure puede reducir automáticamente el número de instancias activas para reducir los costos.</span><span class="sxs-lookup"><span data-stu-id="d6e51-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="d6e51-326">Esta acción es menos agresiva que la escalabilidad horizontal y reduce el impacto sobre los usuarios de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d6e51-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="d6e51-327">Vaya a la condición del escalado horizontal **Predeterminada** y seleccione **+ Agregar una regla**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="d6e51-328">Utilice los siguientes criterios y acciones para la regla.</span><span class="sxs-lookup"><span data-stu-id="d6e51-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="d6e51-329">Criterios</span><span class="sxs-lookup"><span data-stu-id="d6e51-329">Criteria</span></span>

1. <span data-ttu-id="d6e51-330">En **Agregación de tiempo**, seleccione **Media**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="d6e51-331">En **Nombre de métrica**, seleccione **Porcentaje de CPU**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="d6e51-332">En **Operador**, seleccione **Menor que**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="d6e51-333">Establezca **Umbral** en **30**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="d6e51-334">Establezca **Duración** en **10**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="d6e51-335">Acción</span><span class="sxs-lookup"><span data-stu-id="d6e51-335">Action</span></span>

1. <span data-ttu-id="d6e51-336">En **Operación**, seleccione **Reducir recuento en**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="d6e51-337">Establezca **Recuento de instancias** en **1**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="d6e51-338">Establezca **Tiempo de finalización** en **5**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="d6e51-339">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="d6e51-340">Creación de un perfil de Traffic Manager y configuración del escalado entre nubes</span><span class="sxs-lookup"><span data-stu-id="d6e51-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="d6e51-341">Cree un perfil de Traffic Manager en Azure y, a continuación, configure puntos de conexión para habilitar el escalado entre nubes.</span><span class="sxs-lookup"><span data-stu-id="d6e51-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="d6e51-342">Creación de un perfil de Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="d6e51-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="d6e51-343">Seleccione **Crear un recurso**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="d6e51-344">Seleccionar **Redes**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-344">Select **Networking**.</span></span>
3. <span data-ttu-id="d6e51-345">Seleccione **Perfil de Traffic Manager** y configure las opciones siguientes:</span><span class="sxs-lookup"><span data-stu-id="d6e51-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="d6e51-346">En **Nombre**, escriba un nombre para el perfil.</span><span class="sxs-lookup"><span data-stu-id="d6e51-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="d6e51-347">Este nombre **debe** ser único en la zona trafficmanager.net y se utiliza para crear un nuevo nombre de DNS (por ejemplo, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="d6e51-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="d6e51-348">En **Método de enrutamiento**, seleccione **Ponderado**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="d6e51-349">En **Suscripción**, seleccione la suscripción en la que desea crear este perfil.</span><span class="sxs-lookup"><span data-stu-id="d6e51-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="d6e51-350">En **Grupo de recursos**, cree un grupo de recursos nuevo para este perfil.</span><span class="sxs-lookup"><span data-stu-id="d6e51-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="d6e51-351">En **Ubicación del grupo de recursos**, seleccione la ubicación del grupo de recursos.</span><span class="sxs-lookup"><span data-stu-id="d6e51-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="d6e51-352">Este valor hace referencia a la ubicación del grupo de recursos y no tiene efecto alguno sobre el perfil de Traffic Manager que se implementa globalmente.</span><span class="sxs-lookup"><span data-stu-id="d6e51-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="d6e51-353">Seleccione **Crear**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-353">Select **Create**.</span></span>

    ![Creación de un perfil de Traffic Manager](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="d6e51-355">Una vez que se completa la implementación global del perfil de Traffic Manager, aparece en la lista de recursos del grupo de recursos en el que se creó.</span><span class="sxs-lookup"><span data-stu-id="d6e51-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="d6e51-356">Incorporación de puntos de conexión de Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="d6e51-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="d6e51-357">Busque el perfil de Traffic Manager que creó.</span><span class="sxs-lookup"><span data-stu-id="d6e51-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="d6e51-358">Si estaba en el grupo de recursos del perfil, seleccione el perfil.</span><span class="sxs-lookup"><span data-stu-id="d6e51-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="d6e51-359">En **Perfil de Traffic Manager**, en **Configuración**, seleccione **Puntos de conexión**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="d6e51-360">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-360">Select **Add**.</span></span>

4. <span data-ttu-id="d6e51-361">En **Agregar punto de conexión**, use la siguiente configuración para Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="d6e51-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="d6e51-362">En **Tipo**, seleccione **Punto de conexión externo**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="d6e51-363">Escriba el **Nombre** del punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="d6e51-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="d6e51-364">En **Nombre de dominio completo (FQDN) o dirección IP**, escriba la dirección URL externa de la aplicación web de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="d6e51-365">En **Peso**, mantenga el valor predeterminado, **1**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="d6e51-366">Este peso hace que todo el tráfico vaya a este punto de conexión si funciona correctamente.</span><span class="sxs-lookup"><span data-stu-id="d6e51-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="d6e51-367">No active la opción **Agregar como deshabilitado**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="d6e51-368">Seleccione **Aceptar** para guardar el punto de conexión de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="d6e51-369">A continuación, configurará el punto de conexión de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="d6e51-370">En **Perfil de Traffic Manager**, seleccione **Puntos de conexión**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="d6e51-371">Seleccione **+Agregar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-371">Select **+Add**.</span></span>
3. <span data-ttu-id="d6e51-372">En **Agregar punto de conexión**, utilice la siguiente configuración para Azure:</span><span class="sxs-lookup"><span data-stu-id="d6e51-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="d6e51-373">En **Tipo**, seleccione **Punto de conexión de Azure**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="d6e51-374">Escriba el **Nombre** del punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="d6e51-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="d6e51-375">En **Tipo de recurso de destino**, seleccione **App Service**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="d6e51-376">En **Recurso de destino**, seleccione **Elegir un servicio de aplicaciones** para mostrar la lista de Web Apps en la misma suscripción.</span><span class="sxs-lookup"><span data-stu-id="d6e51-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="d6e51-377">En **Recursos**, elija el servicio de aplicación que quiere agregar como el primer punto de conexión.</span><span class="sxs-lookup"><span data-stu-id="d6e51-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="d6e51-378">En **Peso**, seleccione **2**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="d6e51-379">Esta opción hace que todo el tráfico vaya a este punto de conexión si el punto de conexión principal está en mal estado o si tiene una regla o alerta que redirige el tráfico cuando se desencadena.</span><span class="sxs-lookup"><span data-stu-id="d6e51-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="d6e51-380">No active la opción **Agregar como deshabilitado**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="d6e51-381">Seleccione **Aceptar** para guardar el punto de conexión de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="d6e51-382">Después de que ambos puntos de conexión están configurados, aparecerán en el **Perfil de Traffic Manager** al seleccionar **Puntos de conexión**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="d6e51-383">El ejemplo de la captura de pantalla siguiente muestra dos puntos de conexión con la información de estado y configuración de cada uno de ellos.</span><span class="sxs-lookup"><span data-stu-id="d6e51-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Puntos de conexión del perfil de Traffic Manager](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="d6e51-385">Configuración de la supervisión y las alertas de Application Insights</span><span class="sxs-lookup"><span data-stu-id="d6e51-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="d6e51-386">Azure Application Insights permite supervisar la aplicación y enviar alertas basadas en las condiciones que configure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="d6e51-387">Algunos ejemplos son: la aplicación no está disponible, experimenta errores o muestra problemas de rendimiento.</span><span class="sxs-lookup"><span data-stu-id="d6e51-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="d6e51-388">Usará las métricas de Application Insights para crear alertas.</span><span class="sxs-lookup"><span data-stu-id="d6e51-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="d6e51-389">Cuando se desencadenen estas alertas, la instancia de la aplicación web cambiará automáticamente de Azure Stack Hub a Azure para escalarse horizontalmente y volverá a Azure Stack Hub para reducirse horizontalmente.</span><span class="sxs-lookup"><span data-stu-id="d6e51-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="d6e51-390">Creación de una alerta a partir de métricas</span><span class="sxs-lookup"><span data-stu-id="d6e51-390">Create an alert from metrics</span></span>

<span data-ttu-id="d6e51-391">Vaya al grupo de recursos de este tutorial y seleccione la instancia de Application Insights para abrir **Application Insights**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="d6e51-393">Esta vista se usará para crear una alerta de escalabilidad horizontal y una alerta de reducción horizontal.</span><span class="sxs-lookup"><span data-stu-id="d6e51-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="d6e51-394">Creación de la alerta de escalabilidad horizontal</span><span class="sxs-lookup"><span data-stu-id="d6e51-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="d6e51-395">En **Configurar**, seleccione **Alertas (clásica)** .</span><span class="sxs-lookup"><span data-stu-id="d6e51-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="d6e51-396">Seleccione **Agregar una alerta de métrica (clásica)** .</span><span class="sxs-lookup"><span data-stu-id="d6e51-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="d6e51-397">En **Agregar regla**, configure las opciones siguientes:</span><span class="sxs-lookup"><span data-stu-id="d6e51-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="d6e51-398">En **Nombre**, escriba **Burst into Azure Cloud**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="d6e51-399">La **Descripción** es opcional.</span><span class="sxs-lookup"><span data-stu-id="d6e51-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="d6e51-400">En **Origen** > **Alerta sobre**, seleccione **Métricas**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="d6e51-401">En **Criterios**, seleccione la suscripción, el grupo de recursos del perfil de Traffic Manager y el nombre del perfil de Traffic Manager para el recurso.</span><span class="sxs-lookup"><span data-stu-id="d6e51-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="d6e51-402">En **Métrica**, seleccione **Velocidad de solicitudes de**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="d6e51-403">En **Condición**, seleccione **Mayor que**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="d6e51-404">En **Umbral**, escriba **2**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="d6e51-405">En **Período**, seleccione **En los últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="d6e51-406">En **Notificar mediante**:</span><span class="sxs-lookup"><span data-stu-id="d6e51-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="d6e51-407">Active la casilla de verificación para **Correo electrónico a propietarios, colaboradores y lectores**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="d6e51-408">En **Correos electrónicos adicionales del administrador**, escriba su dirección de correo electrónico.</span><span class="sxs-lookup"><span data-stu-id="d6e51-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="d6e51-409">En la barra de menús, seleccione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="d6e51-410">Creación de la alerta de reducción horizontal</span><span class="sxs-lookup"><span data-stu-id="d6e51-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="d6e51-411">En **Configurar**, seleccione **Alertas (clásica)** .</span><span class="sxs-lookup"><span data-stu-id="d6e51-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="d6e51-412">Seleccione **Agregar una alerta de métrica (clásica)** .</span><span class="sxs-lookup"><span data-stu-id="d6e51-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="d6e51-413">En **Agregar regla**, configure las opciones siguientes:</span><span class="sxs-lookup"><span data-stu-id="d6e51-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="d6e51-414">En **Nombre**, escriba **Scale back into Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="d6e51-415">La **Descripción** es opcional.</span><span class="sxs-lookup"><span data-stu-id="d6e51-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="d6e51-416">En **Origen** > **Alerta sobre**, seleccione **Métricas**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="d6e51-417">En **Criterios**, seleccione la suscripción, el grupo de recursos del perfil de Traffic Manager y el nombre del perfil de Traffic Manager para el recurso.</span><span class="sxs-lookup"><span data-stu-id="d6e51-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="d6e51-418">En **Métrica**, seleccione **Velocidad de solicitudes de**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="d6e51-419">En **Condición**, seleccione **Menor que**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="d6e51-420">En **Umbral**, escriba **2**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="d6e51-421">En **Período**, seleccione **En los últimos 5 minutos**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="d6e51-422">En **Notificar mediante**:</span><span class="sxs-lookup"><span data-stu-id="d6e51-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="d6e51-423">Active la casilla de verificación para **Correo electrónico a propietarios, colaboradores y lectores**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="d6e51-424">En **Correos electrónicos adicionales del administrador**, escriba su dirección de correo electrónico.</span><span class="sxs-lookup"><span data-stu-id="d6e51-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="d6e51-425">En la barra de menús, seleccione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="d6e51-426">En la captura de pantalla siguiente se muestran las alertas de escalabilidad horizontal y reducción horizontal.</span><span class="sxs-lookup"><span data-stu-id="d6e51-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Alertas de Application Insights (clásico)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d6e51-428">Redirección del tráfico entre Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6e51-429">Puede configurar la conmutación manual o automática del tráfico de la aplicación web entre Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d6e51-430">Configuración de la conmutación manual entre Azure y Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d6e51-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6e51-431">Cuando el sitio web llegue a los umbrales que configuró, recibirá una alerta.</span><span class="sxs-lookup"><span data-stu-id="d6e51-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="d6e51-432">Utilice los pasos siguientes para redirigir el tráfico a Azure manualmente.</span><span class="sxs-lookup"><span data-stu-id="d6e51-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="d6e51-433">En Azure Portal, seleccione el perfil de Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="d6e51-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Puntos de conexión de Traffic Manager en Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="d6e51-435">Seleccione **Puntos de conexión**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="d6e51-436">Seleccione el **punto de conexión de Azure**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="d6e51-437">En **Estado**, seleccione **Habilitado** y, a continuación, seleccione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Habilitar punto de conexión de Azure en Azure Portal](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="d6e51-439">En **Puntos de conexión** del perfil de Traffic Manager, seleccione **Punto de conexión externo**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="d6e51-440">En **Estado**, seleccione **Deshabilitado** y, a continuación, seleccione **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="d6e51-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Deshabilitar punto de conexión de Azure Stack Hub en Azure Portal](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="d6e51-442">Después de configurar los puntos de conexión, el tráfico de la aplicación se dirige a la aplicación web de escalabilidad horizontal de Azure, y no a la aplicación web de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Puntos de conexión cambiados en el tráfico de aplicaciones web de Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="d6e51-444">Para invertir el flujo de vuelta a Azure Stack Hub, use los pasos anteriores para:</span><span class="sxs-lookup"><span data-stu-id="d6e51-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="d6e51-445">Habilitar el punto de conexión de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="d6e51-446">Deshabilitar el punto de conexión de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="d6e51-447">Configurar la conmutación automática entre Azure y Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d6e51-448">También puede usar la supervisión de Application Insights si la aplicación se ejecuta en un entorno [sin servidor](https://azure.microsoft.com/overview/serverless-computing/) proporcionado por Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="d6e51-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="d6e51-449">En este escenario, puede configurar Application Insights para que use un webhook que llama a una aplicación de función.</span><span class="sxs-lookup"><span data-stu-id="d6e51-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="d6e51-450">Esta aplicación habilita o deshabilita automáticamente un punto de conexión en respuesta a una alerta.</span><span class="sxs-lookup"><span data-stu-id="d6e51-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="d6e51-451">Use los pasos siguientes como guía para configurar la conmutación automática del tráfico.</span><span class="sxs-lookup"><span data-stu-id="d6e51-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="d6e51-452">Cree una aplicación de función de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="d6e51-453">Cree una función desencadenada por HTTP.</span><span class="sxs-lookup"><span data-stu-id="d6e51-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="d6e51-454">Importe los SDK de Azure para Resource Manager, Web Apps y Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="d6e51-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="d6e51-455">Desarrolle código para:</span><span class="sxs-lookup"><span data-stu-id="d6e51-455">Develop code to:</span></span>

   - <span data-ttu-id="d6e51-456">Autenticarse en la suscripción de Azure.</span><span class="sxs-lookup"><span data-stu-id="d6e51-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="d6e51-457">Usar un parámetro que alterne los puntos de conexión de Traffic Manager para dirigir el tráfico a Azure o Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d6e51-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="d6e51-458">Guarde el código y agregue la dirección URL de la aplicación de función con los parámetros adecuados en la sección **Webhook** de la configuración de la regla de alertas de Application Insights.</span><span class="sxs-lookup"><span data-stu-id="d6e51-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="d6e51-459">El tráfico se redirige automáticamente cuando se desencadena una alerta de Application Insights.</span><span class="sxs-lookup"><span data-stu-id="d6e51-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d6e51-460">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="d6e51-460">Next steps</span></span>

- <span data-ttu-id="d6e51-461">Para más información sobre los patrones de nube de Azure, consulte [Patrones de diseño en la nube](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="d6e51-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
