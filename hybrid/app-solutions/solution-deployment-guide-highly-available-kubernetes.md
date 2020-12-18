---
title: Implementación de un clúster de Kubernetes de alta disponibilidad en Azure Stack Hub
description: Aprenda a implementar una solución de clúster de Kubernetes para alta disponibilidad con Azure y Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911926"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="d7ba8-103">Implementación de un clúster de Kubernetes de alta disponibilidad en Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d7ba8-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="d7ba8-104">En este artículo se muestra cómo crear un entorno de clúster de Kubernetes de alta disponibilidad, implementado en varias instancias de Azure Stack Hub, en diferentes ubicaciones físicas.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="d7ba8-105">En esta guía de implementación de la solución, obtendrá información sobre:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d7ba8-106">Descarga y preparación del motor de AKS</span><span class="sxs-lookup"><span data-stu-id="d7ba8-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="d7ba8-107">Conexión a la máquina virtual auxiliar del motor de AKS</span><span class="sxs-lookup"><span data-stu-id="d7ba8-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="d7ba8-108">Implementación de un clúster de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="d7ba8-109">Conectar al clúster de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="d7ba8-110">Conexión de Azure Pipelines al clúster de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="d7ba8-111">Configuración de la supervisión</span><span class="sxs-lookup"><span data-stu-id="d7ba8-111">Configure monitoring</span></span>
> - <span data-ttu-id="d7ba8-112">Implementación de la aplicación</span><span class="sxs-lookup"><span data-stu-id="d7ba8-112">Deploy application</span></span>
> - <span data-ttu-id="d7ba8-113">Escalado automático de la aplicación</span><span class="sxs-lookup"><span data-stu-id="d7ba8-113">Autoscale application</span></span>
> - <span data-ttu-id="d7ba8-114">Configuración del Administrador de tráfico</span><span class="sxs-lookup"><span data-stu-id="d7ba8-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="d7ba8-115">Actualización de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="d7ba8-116">Escalado de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="d7ba8-117">![Fundamentos de tecnologías híbridas](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d7ba8-118">Microsoft Azure Stack Hub es una extensión de Azure.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d7ba8-119">Azure Stack Hub aporta la agilidad y la innovación de la informática en la nube a su entorno local y hace posible la única nube híbrida que le permite crear e implementar aplicaciones híbridas en cualquier parte.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d7ba8-120">En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los fundamentos de calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d7ba8-121">Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d7ba8-122">Prerrequisitos</span><span class="sxs-lookup"><span data-stu-id="d7ba8-122">Prerequisites</span></span>

<span data-ttu-id="d7ba8-123">Antes de empezar a usar esta guía de implementación, asegúrese de:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="d7ba8-124">Revisar el artículo [Patrón de clúster de Kubernetes de alta disponibilidad](pattern-highly-available-kubernetes.md).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="d7ba8-125">Revisar el contenido del [repositorio complementario de GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), que contiene recursos adicionales a los que se hace referencia en este artículo.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="d7ba8-126">Disponer de una cuenta que pueda acceder al [portal de usuarios de Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), con [permisos de "colaborador"](/azure-stack/user/azure-stack-manage-permissions) como mínimo.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="d7ba8-127">Descarga y preparación del motor de AKS</span><span class="sxs-lookup"><span data-stu-id="d7ba8-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="d7ba8-128">El motor de AKS es un archivo binario que se puede usar desde cualquier host Windows o Linux que pueda acceder a los puntos de conexión de Azure Resource Manager de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="d7ba8-129">En esta guía se describe la implementación de una nueva máquina virtual Linux (o Windows) en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="d7ba8-130">Se usará más adelante cuando el motor de AKS implemente los clústeres de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="d7ba8-131">También puede usar una máquina virtual Windows o Linux existente para implementar un clúster de Kubernetes en Azure Stack Hub con el motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="d7ba8-132">Los requisitos y el proceso paso a paso para el motor de AKS se documentan aquí:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="d7ba8-133">[Instalación del motor de AKS en Linux para Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (o con [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="d7ba8-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="d7ba8-134">El motor de AKS es una herramienta auxiliar para implementar y usar clústeres de Kubernetes (no administrados) (en Azure y Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="d7ba8-135">Aquí se describen los detalles y las diferencias del motor de AKS en Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="d7ba8-136">Descripción del motor de AKS en Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d7ba8-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="d7ba8-137">[Motor de AKS en Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (en GitHub)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="d7ba8-138">El entorno de ejemplo usará Terraform para automatizar la implementación de la máquina virtual del motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="d7ba8-139">Puede encontrar los [detalles y el código en el repositorio de GitHub complementario](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="d7ba8-140">El resultado de este paso es un nuevo grupo de recursos en Azure Stack Hub que contiene la máquina virtual auxiliar del motor de AKS y los recursos relacionados:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Recursos de la máquina virtual del motor de AKS en Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="d7ba8-142">Si tiene que implementar el motor de AKS en un entorno desconectado del exterior, consulte [Instancias desconectadas de Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) para más información.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="d7ba8-143">En el paso siguiente, usaremos la máquina virtual del motor de AKS recién implementada para implementar un clúster de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="d7ba8-144">Conexión a la máquina virtual auxiliar del motor de AKS</span><span class="sxs-lookup"><span data-stu-id="d7ba8-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="d7ba8-145">En primer lugar, debe conectarse a la máquina virtual auxiliar del motor de AKS creada anteriormente.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="d7ba8-146">La máquina virtual debe tener una dirección IP pública y debe ser accesible mediante SSH (puerto 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Página de información general de la máquina virtual del motor de AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="d7ba8-148">Puede usar una herramienta de su elección, como MobaXterm, puTTY o PowerShell en Windows 10 para conectarse a una máquina virtual Linux mediante SSH.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="d7ba8-149">Después de conectarse, ejecute el comando `aks-engine`.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="d7ba8-150">Vaya a [Versiones admitidas del motor de AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) para más información sobre las versiones del motor de AKS y Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![Ejemplo del comando aks-engine en la línea de comandos](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="d7ba8-152">Implementación de un clúster de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="d7ba8-153">La propia máquina virtual auxiliar del motor de AKS no ha creado aún un clúster de Kubernetes en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="d7ba8-154">La creación del clúster es la primera acción que se realiza en la máquina virtual auxiliar del motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="d7ba8-155">El proceso paso a paso se documenta aquí:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="d7ba8-156">Implementación de un clúster de Kubernetes con el motor de AKS en Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d7ba8-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="d7ba8-157">El resultado final del comando `aks-engine deploy` y los preparativos de los pasos anteriores es un clúster de Kubernetes con todas las características implementado en el espacio de inquilinos de la primera instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="d7ba8-158">El clúster se compone de componentes de IaaS de Azure, como máquinas virtuales, equilibradores de carga, redes virtuales, discos, etc.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Componentes de IaaS del clúster en el portal de Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="d7ba8-160">Azure Load Balancer (punto de conexión de API K8s)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="d7ba8-161">Nodos de trabajo (grupo de agentes)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="d7ba8-162">Nodos maestros</span><span class="sxs-lookup"><span data-stu-id="d7ba8-162">Master Nodes</span></span>

<span data-ttu-id="d7ba8-163">El clúster está ahora en funcionamiento y, en el paso siguiente, se conectará a él.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="d7ba8-164">Conectar al clúster de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="d7ba8-165">Ahora puede conectarse al clúster de Kubernetes creado anteriormente, ya sea mediante SSH (con la clave SSH especificada como parte de la implementación) o mediante `kubectl` (recomendado).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="d7ba8-166">La herramienta de la línea de comandos de Kubernetes `kubectl` está disponible para Windows, Linux y macOS [aquí](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="d7ba8-167">Ya está preinstalada y configurada en los nodos maestros del clúster.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Ejecución de kubectl en el nodo maestro](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="d7ba8-169">No se recomienda usar el nodo maestro como equipo de acceso para las tareas administrativas.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="d7ba8-170">La configuración de `kubectl` se almacena en `.kube/config` en los nodos maestros y en la máquina virtual del motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="d7ba8-171">Puede copiar la configuración en un equipo de administración con conectividad con el clúster de Kubernetes y usar el comando `kubectl` desde allí.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="d7ba8-172">El archivo `.kube/config` se usa también más adelante para configurar una conexión de servicio en Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="d7ba8-173">Mantenga estos archivos protegidos, porque contienen las credenciales del clúster de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="d7ba8-174">Un atacante con acceso al archivo tiene información suficiente para obtener acceso de administrador al clúster.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="d7ba8-175">Todas las acciones que se realizan con el archivo `.kube/config` inicial se realizan con una cuenta de administrador del clúster.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="d7ba8-176">Ahora puede probar varios comandos mediante `kubectl` para comprobar el estado del clúster.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="d7ba8-177">Kubernetes tiene su propio modelo de *control de acceso basado en roles (RBAC)* \* que le permite crear definiciones de roles detalladas y enlaces de roles.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="d7ba8-178">Esta es la manera preferible de controlar el acceso al clúster en lugar de entregar los permisos de administrador del clúster.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="d7ba8-179">Conexión de Azure Pipelines a clústeres de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="d7ba8-180">Para conectar Azure Pipelines al clúster de Kubernetes recién implementado, necesitamos su archivo kube config (`.kube/config`), tal como se explicó en el paso anterior.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="d7ba8-181">Conéctese a uno de los nodos maestros del clúster de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="d7ba8-182">Copie el contenido del archivo `.kube/config`.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="d7ba8-183">Vaya a Azure DevOps > Configuración del proyecto > Conexiones de servicio para crear la nueva conexión de servicio "Kubernetes" (use KubeConfig como método de autenticación).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="d7ba8-184">Azure Pipelines (o sus agentes de compilación) deben tener acceso a la API de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="d7ba8-185">Si hay una conexión a Internet desde Azure Pipelines al clúster de Kubernetes de Azure Stack Hub, deberá implementar un agente de compilación de Azure Pipelines autohospedado.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="d7ba8-186">Al implementar agentes autohospedados para Azure Pipelines, puede realizar la implementación en Azure Stack Hub o en un equipo con conectividad de red con todos los puntos de conexión de administración necesarios.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="d7ba8-187">Consulte los detalles aquí:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-187">See the details here:</span></span>

* <span data-ttu-id="d7ba8-188">[Agentes de Azure Pipelines](/azure/devops/pipelines/agents/agents) en [Windows](/azure/devops/pipelines/agents/v2-windows) o [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="d7ba8-189">La sección [Consideraciones de implementación (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) del patrón contiene un flujo de decisión que le ayuda a decidir si usar agentes hospedados por Microsoft o agentes autohospedados:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="d7ba8-190">[![Flujo de decisión para agentes autohospedados](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="d7ba8-191">En esta solución de ejemplo, la topología incluye un agente de compilación autohospedado en cada instancia de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="d7ba8-192">El agente puede acceder a los puntos de conexión de administración de Azure Stack Hub y a los puntos de conexión de API del clúster de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="d7ba8-193">[![Solo tráfico de salida](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="d7ba8-194">Este diseño cumple un requisito normativo común, que consiste en tener solo conexiones salientes desde la solución de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="d7ba8-195">Configuración de la supervisión</span><span class="sxs-lookup"><span data-stu-id="d7ba8-195">Configure monitoring</span></span>

<span data-ttu-id="d7ba8-196">Puede usar [Azure Monitor](/azure/azure-monitor/) para contenedores para supervisar los contenedores de la solución.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="d7ba8-197">Esto hace que Azure Monitor apunte al clúster de Kubernetes implementado por el motor de AKS en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="d7ba8-198">Hay dos maneras de habilitar Azure Monitor en el clúster.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="d7ba8-199">Ambos métodos requieren la configuración de un área de trabajo de Log Analytics en Azure.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="d7ba8-200">El [método uno](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa un gráfico de Helm.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="d7ba8-201">El [método dos](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) se trata como parte de la especificación del clúster del motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="d7ba8-202">En la topología de ejemplo se usa el "método uno", que permite la automatización del proceso y las actualizaciones se pueden instalar más fácilmente.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="d7ba8-203">En el siguiente paso, necesitará un área de trabajo de Azure LogAnalytics (identificador y clave), `Helm` (versión 3) y `kubectl` en el equipo.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="d7ba8-204">Helm es un administrador de paquetes de Kubernetes, disponible como un archivo binario que se ejecuta en macOS, Windows y Linux.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="d7ba8-205">Se puede descargar aquí: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm se basa en el archivo de configuración de Kubernetes que se usa para el comando `kubectl`.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="d7ba8-206">Este comando instalará el agente de Azure Monitor en el clúster de Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="d7ba8-207">El agente de Operations Management Suite (OMS) del clúster de Kubernetes enviará datos de supervisión al área de trabajo de Azure Log Analytics (mediante HTTPS de salida).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="d7ba8-208">Ahora puede usar Azure Monitor para obtener información más detallada sobre los clústeres de Kubernetes en Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="d7ba8-209">Este diseño es una manera eficaz de demostrar la eficacia de los análisis que se pueden implementar automáticamente con los clústeres de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="d7ba8-210">[![Clústeres de Azure Stack Hub en Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="d7ba8-211">[![Detalles del clúster de Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="d7ba8-212">Si Azure Monitor no muestra ningún dato de Azure Stack Hub, asegúrese de que ha seguido cuidadosamente las instrucciones descritas en [Adición de la solución "AzureMonitor-Containers" a un área de trabajo de Azure Log Analytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="d7ba8-213">Implementación de la aplicación</span><span class="sxs-lookup"><span data-stu-id="d7ba8-213">Deploy the application</span></span>

<span data-ttu-id="d7ba8-214">Antes de instalar la aplicación de ejemplo, hay otro paso para configurar el controlador de entrada basado en nginx en el clúster de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="d7ba8-215">El controlador de entrada se usa como equilibrador de carga de capa 7 para enrutar el tráfico en el clúster en función del host, la ruta de acceso o el protocolo.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="d7ba8-216">Nginx-ingress está disponible como un gráfico de Helm.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="d7ba8-217">Para obtener instrucciones detalladas, consulte el [repositorio de GitHub del gráfico de Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="d7ba8-218">La aplicación de ejemplo también está empaquetada como un gráfico de Helm, como el [agente de supervisión de Azure](#configure-monitoring) del paso anterior.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="d7ba8-219">Por lo tanto, es sencillo implementar la aplicación en el clúster de Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="d7ba8-220">Puede encontrar los [archivos del gráfico de Helm en el repositorio de GitHub complementario](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="d7ba8-221">La aplicación de ejemplo es una aplicación de tres niveles, implementada en un clúster de Kubernetes en cada una de las dos instancias de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="d7ba8-222">La aplicación utiliza una base de datos MongoDB.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="d7ba8-223">Para más información sobre cómo hacer que los datos se repliquen en varias instancias, consulte la sección [Consideraciones sobre los datos y el almacenamiento](pattern-highly-available-kubernetes.md#data-and-storage-considerations) del patrón.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="d7ba8-224">Después de implementar el gráfico de Helm de la aplicación, verá los tres niveles de la aplicación representados como implementaciones y conjuntos con estado (para la base de datos) con un único pod:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="d7ba8-225">En los servicios, encontrará el controlador de entrada basado en nginx y su dirección IP pública:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="d7ba8-226">La dirección "IP externa" es nuestro "punto de conexión de la aplicación".</span><span class="sxs-lookup"><span data-stu-id="d7ba8-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="d7ba8-227">Es el modo con el que los usuarios se conectarán para abrir la aplicación y también se usará como punto de conexión para el paso siguiente sobre [Configuración de Traffic Manager](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="d7ba8-228">Escalado automático de la aplicación</span><span class="sxs-lookup"><span data-stu-id="d7ba8-228">Autoscale the application</span></span>
<span data-ttu-id="d7ba8-229">Opcionalmente, puede configurar [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) para escalar o reducir verticalmente en función de ciertas métricas, como el uso de la CPU.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="d7ba8-230">El siguiente comando creará una instancia de Horizontal Pod Autoscaler que mantiene entre 1 y 10 réplicas de los pods controlados por la implementación de clasificación web.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="d7ba8-231">HPA aumentará y disminuirá el número de réplicas (mediante la implementación) para mantener un uso promedio de la CPU en todos los pods del 80 %.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="d7ba8-232">Puede comprobar el estado actual del escalador automático mediante la ejecución de:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="d7ba8-233">Configuración del Administrador de tráfico</span><span class="sxs-lookup"><span data-stu-id="d7ba8-233">Configure Traffic Manager</span></span>

<span data-ttu-id="d7ba8-234">Para distribuir el tráfico entre dos (o más) implementaciones de la aplicación, usaremos [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="d7ba8-235">Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS en Azure.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="d7ba8-236">Traffic Manager usa DNS para dirigir las solicitudes del cliente al punto de conexión de servicio más adecuado en función de un método de enrutamiento del tráfico y del estado de los puntos de conexión.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="d7ba8-237">En lugar de usar Azure Traffic Manager, también puede usar otras soluciones de equilibrio de carga globales hospedadas en el entorno local.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="d7ba8-238">En el escenario de ejemplo, usaremos Azure Traffic Manager para distribuir el tráfico entre dos instancias de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="d7ba8-239">Se pueden ejecutar en instancias de Azure Stack Hub en la misma ubicación o en ubicaciones diferentes:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![Traffic Manager local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="d7ba8-241">En Azure, vamos a configurar Traffic Manager para que apunte a las dos instancias diferentes de la aplicación:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="d7ba8-242">[![Perfil de punto de conexión de TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="d7ba8-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="d7ba8-243">Como puede ver, los dos puntos de conexión apuntan a las dos instancias de la aplicación implementada en la sección [anterior](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="d7ba8-244">En este punto:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-244">At this point:</span></span>
- <span data-ttu-id="d7ba8-245">Se ha creado la infraestructura de Kubernetes, incluido un controlador de entrada.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="d7ba8-246">Los clústeres se han implementado en dos instancias de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="d7ba8-247">Se ha configurado la supervisión.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="d7ba8-248">Azure Traffic Manager equilibrará la carga del tráfico entre las dos instancias de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="d7ba8-249">Sobre esta infraestructura, la aplicación de tres niveles de ejemplo se ha implementado de forma automatizada mediante gráficos de Helm.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="d7ba8-250">Ahora la solución debería estar activa y accesible para los usuarios.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="d7ba8-251">También hay algunas consideraciones operativas posteriores a la implementación que merece la pena analizar, que se tratan en las dos secciones siguientes.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="d7ba8-252">Actualización de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="d7ba8-253">Tenga en cuenta los siguientes temas al actualizar el clúster de Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="d7ba8-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="d7ba8-254">La actualización de un clúster de Kubernetes es una operación compleja de una segunda etapa que se puede realizar mediante el motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="d7ba8-255">Para más información, consulte [Actualización de un clúster de Kubernetes en Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="d7ba8-256">El motor de AKS permite actualizar los clústeres a las versiones más recientes de Kubernetes y de la imagen del sistema operativo de base.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="d7ba8-257">Para más información, consulte [Pasos para actualizar a una versión más reciente de Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="d7ba8-258">También puede actualizar solo los nodos subyacentes a las versiones más recientes de la imagen del sistema operativo de base.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="d7ba8-259">Para más información, consulte [Pasos para actualizar únicamente la imagen del sistema operativo](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="d7ba8-260">Las imágenes del sistema operativo de base más recientes contienen actualizaciones de seguridad y del kernel.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="d7ba8-261">Es responsabilidad del operador del clúster supervisar la disponibilidad de las versiones más recientes de Kubernetes y de las imágenes del sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="d7ba8-262">El operador debe planear y ejecutar estas actualizaciones mediante el motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="d7ba8-263">El operador de Azure Stack Hub debe descargar las imágenes del sistema operativo de base desde Marketplace de Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="d7ba8-264">Escalado de Kubernetes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-264">Scale Kubernetes</span></span>

<span data-ttu-id="d7ba8-265">El escalado es otra operación de una segunda etapa que se puede organizar mediante el motor de AKS.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="d7ba8-266">El comando de escalado reutiliza el archivo de configuración del clúster (apimodel.json) del directorio de salida como entrada para una nueva implementación de Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="d7ba8-267">El motor de AKS ejecuta la operación de escalado en un grupo de agentes específico.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="d7ba8-268">Una vez completada la operación de escalado, el motor de AKS actualiza la definición del clúster en el mismo archivo apimodel.json.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="d7ba8-269">La definición del clúster refleja el nuevo número de nodos para reflejar la configuración del clúster actual actualizada.</span><span class="sxs-lookup"><span data-stu-id="d7ba8-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="d7ba8-270">Escalado de un clúster de Kubernetes en Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="d7ba8-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="d7ba8-271">Pasos siguientes</span><span class="sxs-lookup"><span data-stu-id="d7ba8-271">Next steps</span></span>

- <span data-ttu-id="d7ba8-272">Más información sobre [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="d7ba8-273">Revise y proponga mejoras en [el código de este ejemplo en GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="d7ba8-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>