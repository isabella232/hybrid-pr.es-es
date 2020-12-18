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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Implementación de un clúster de Kubernetes de alta disponibilidad en Azure Stack Hub

En este artículo se muestra cómo crear un entorno de clúster de Kubernetes de alta disponibilidad, implementado en varias instancias de Azure Stack Hub, en diferentes ubicaciones físicas.

En esta guía de implementación de la solución, obtendrá información sobre:

> [!div class="checklist"]
> - Descarga y preparación del motor de AKS
> - Conexión a la máquina virtual auxiliar del motor de AKS
> - Implementación de un clúster de Kubernetes
> - Conectar al clúster de Kubernetes
> - Conexión de Azure Pipelines al clúster de Kubernetes
> - Configuración de la supervisión
> - Implementación de la aplicación
> - Escalado automático de la aplicación
> - Configuración del Administrador de tráfico
> - Actualización de Kubernetes
> - Escalado de Kubernetes

> [!Tip]  
> ![Fundamentos de tecnologías híbridas](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub es una extensión de Azure. Azure Stack Hub aporta la agilidad y la innovación de la informática en la nube a su entorno local y hace posible la única nube híbrida que le permite crear e implementar aplicaciones híbridas en cualquier parte.  
> 
> En el artículo [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md) se examinan los fundamentos de calidad del software (selección de ubicación, escalabilidad, disponibilidad, resistencia, manejabilidad y seguridad) para diseñar, implementar y usar aplicaciones híbridas. Las consideraciones de diseño ayudan a optimizar el diseño de aplicaciones híbridas y reducen los desafíos en los entornos de producción.

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar a usar esta guía de implementación, asegúrese de:

- Revisar el artículo [Patrón de clúster de Kubernetes de alta disponibilidad](pattern-highly-available-kubernetes.md).
- Revisar el contenido del [repositorio complementario de GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), que contiene recursos adicionales a los que se hace referencia en este artículo.
- Disponer de una cuenta que pueda acceder al [portal de usuarios de Azure Stack Hub](/azure-stack/user/azure-stack-use-portal), con [permisos de "colaborador"](/azure-stack/user/azure-stack-manage-permissions) como mínimo.

## <a name="download-and-prepare-aks-engine"></a>Descarga y preparación del motor de AKS

El motor de AKS es un archivo binario que se puede usar desde cualquier host Windows o Linux que pueda acceder a los puntos de conexión de Azure Resource Manager de Azure Stack Hub. En esta guía se describe la implementación de una nueva máquina virtual Linux (o Windows) en Azure Stack Hub. Se usará más adelante cuando el motor de AKS implemente los clústeres de Kubernetes.

> [!NOTE]
> También puede usar una máquina virtual Windows o Linux existente para implementar un clúster de Kubernetes en Azure Stack Hub con el motor de AKS.

Los requisitos y el proceso paso a paso para el motor de AKS se documentan aquí:

* [Instalación del motor de AKS en Linux para Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (o con [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

El motor de AKS es una herramienta auxiliar para implementar y usar clústeres de Kubernetes (no administrados) (en Azure y Azure Stack Hub).

Aquí se describen los detalles y las diferencias del motor de AKS en Azure Stack Hub:

* [Descripción del motor de AKS en Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Motor de AKS en Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (en GitHub)

El entorno de ejemplo usará Terraform para automatizar la implementación de la máquina virtual del motor de AKS. Puede encontrar los [detalles y el código en el repositorio de GitHub complementario](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

El resultado de este paso es un nuevo grupo de recursos en Azure Stack Hub que contiene la máquina virtual auxiliar del motor de AKS y los recursos relacionados:

![Recursos de la máquina virtual del motor de AKS en Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Si tiene que implementar el motor de AKS en un entorno desconectado del exterior, consulte [Instancias desconectadas de Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) para más información.

En el paso siguiente, usaremos la máquina virtual del motor de AKS recién implementada para implementar un clúster de Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Conexión a la máquina virtual auxiliar del motor de AKS

En primer lugar, debe conectarse a la máquina virtual auxiliar del motor de AKS creada anteriormente.

La máquina virtual debe tener una dirección IP pública y debe ser accesible mediante SSH (puerto 22/TCP).

![Página de información general de la máquina virtual del motor de AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Puede usar una herramienta de su elección, como MobaXterm, puTTY o PowerShell en Windows 10 para conectarse a una máquina virtual Linux mediante SSH.

```console
ssh <username>@<ipaddress>
```

Después de conectarse, ejecute el comando `aks-engine`. Vaya a [Versiones admitidas del motor de AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) para más información sobre las versiones del motor de AKS y Kubernetes.

![Ejemplo del comando aks-engine en la línea de comandos](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Implementación de un clúster de Kubernetes

La propia máquina virtual auxiliar del motor de AKS no ha creado aún un clúster de Kubernetes en Azure Stack Hub. La creación del clúster es la primera acción que se realiza en la máquina virtual auxiliar del motor de AKS.

El proceso paso a paso se documenta aquí:

* [Implementación de un clúster de Kubernetes con el motor de AKS en Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

El resultado final del comando `aks-engine deploy` y los preparativos de los pasos anteriores es un clúster de Kubernetes con todas las características implementado en el espacio de inquilinos de la primera instancia de Azure Stack Hub. El clúster se compone de componentes de IaaS de Azure, como máquinas virtuales, equilibradores de carga, redes virtuales, discos, etc.

![Componentes de IaaS del clúster en el portal de Azure Stack Hub](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure Load Balancer (punto de conexión de API K8s)
2) Nodos de trabajo (grupo de agentes)
3) Nodos maestros

El clúster está ahora en funcionamiento y, en el paso siguiente, se conectará a él.

## <a name="connect-to-the-kubernetes-cluster"></a>Conectar al clúster de Kubernetes

Ahora puede conectarse al clúster de Kubernetes creado anteriormente, ya sea mediante SSH (con la clave SSH especificada como parte de la implementación) o mediante `kubectl` (recomendado). La herramienta de la línea de comandos de Kubernetes `kubectl` está disponible para Windows, Linux y macOS [aquí](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Ya está preinstalada y configurada en los nodos maestros del clúster.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Ejecución de kubectl en el nodo maestro](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

No se recomienda usar el nodo maestro como equipo de acceso para las tareas administrativas. La configuración de `kubectl` se almacena en `.kube/config` en los nodos maestros y en la máquina virtual del motor de AKS. Puede copiar la configuración en un equipo de administración con conectividad con el clúster de Kubernetes y usar el comando `kubectl` desde allí. El archivo `.kube/config` se usa también más adelante para configurar una conexión de servicio en Azure Pipelines.

> [!IMPORTANT]
> Mantenga estos archivos protegidos, porque contienen las credenciales del clúster de Kubernetes. Un atacante con acceso al archivo tiene información suficiente para obtener acceso de administrador al clúster. Todas las acciones que se realizan con el archivo `.kube/config` inicial se realizan con una cuenta de administrador del clúster.

Ahora puede probar varios comandos mediante `kubectl` para comprobar el estado del clúster.

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
> Kubernetes tiene su propio modelo de *control de acceso basado en roles (RBAC)* * que le permite crear definiciones de roles detalladas y enlaces de roles. Esta es la manera preferible de controlar el acceso al clúster en lugar de entregar los permisos de administrador del clúster.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Conexión de Azure Pipelines a clústeres de Kubernetes

Para conectar Azure Pipelines al clúster de Kubernetes recién implementado, necesitamos su archivo kube config (`.kube/config`), tal como se explicó en el paso anterior.

* Conéctese a uno de los nodos maestros del clúster de Kubernetes.
* Copie el contenido del archivo `.kube/config`.
* Vaya a Azure DevOps > Configuración del proyecto > Conexiones de servicio para crear la nueva conexión de servicio "Kubernetes" (use KubeConfig como método de autenticación).

> [!IMPORTANT]
> Azure Pipelines (o sus agentes de compilación) deben tener acceso a la API de Kubernetes. Si hay una conexión a Internet desde Azure Pipelines al clúster de Kubernetes de Azure Stack Hub, deberá implementar un agente de compilación de Azure Pipelines autohospedado.

Al implementar agentes autohospedados para Azure Pipelines, puede realizar la implementación en Azure Stack Hub o en un equipo con conectividad de red con todos los puntos de conexión de administración necesarios. Consulte los detalles aquí:

* [Agentes de Azure Pipelines](/azure/devops/pipelines/agents/agents) en [Windows](/azure/devops/pipelines/agents/v2-windows) o [Linux](/azure/devops/pipelines/agents/v2-linux)

La sección [Consideraciones de implementación (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) del patrón contiene un flujo de decisión que le ayuda a decidir si usar agentes hospedados por Microsoft o agentes autohospedados:

[![Flujo de decisión para agentes autohospedados](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

En esta solución de ejemplo, la topología incluye un agente de compilación autohospedado en cada instancia de Azure Stack Hub. El agente puede acceder a los puntos de conexión de administración de Azure Stack Hub y a los puntos de conexión de API del clúster de Kubernetes.

[![Solo tráfico de salida](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Este diseño cumple un requisito normativo común, que consiste en tener solo conexiones salientes desde la solución de la aplicación.

## <a name="configure-monitoring"></a>Configuración de la supervisión

Puede usar [Azure Monitor](/azure/azure-monitor/) para contenedores para supervisar los contenedores de la solución. Esto hace que Azure Monitor apunte al clúster de Kubernetes implementado por el motor de AKS en Azure Stack Hub.

Hay dos maneras de habilitar Azure Monitor en el clúster. Ambos métodos requieren la configuración de un área de trabajo de Log Analytics en Azure.

* El [método uno](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa un gráfico de Helm.
* El [método dos](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) se trata como parte de la especificación del clúster del motor de AKS.

En la topología de ejemplo se usa el "método uno", que permite la automatización del proceso y las actualizaciones se pueden instalar más fácilmente.

En el siguiente paso, necesitará un área de trabajo de Azure LogAnalytics (identificador y clave), `Helm` (versión 3) y `kubectl` en el equipo.

Helm es un administrador de paquetes de Kubernetes, disponible como un archivo binario que se ejecuta en macOS, Windows y Linux. Se puede descargar aquí: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm se basa en el archivo de configuración de Kubernetes que se usa para el comando `kubectl`.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Este comando instalará el agente de Azure Monitor en el clúster de Kubernetes:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

El agente de Operations Management Suite (OMS) del clúster de Kubernetes enviará datos de supervisión al área de trabajo de Azure Log Analytics (mediante HTTPS de salida). Ahora puede usar Azure Monitor para obtener información más detallada sobre los clústeres de Kubernetes en Azure Stack Hub. Este diseño es una manera eficaz de demostrar la eficacia de los análisis que se pueden implementar automáticamente con los clústeres de la aplicación.

[![Clústeres de Azure Stack Hub en Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Detalles del clúster de Azure Monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Si Azure Monitor no muestra ningún dato de Azure Stack Hub, asegúrese de que ha seguido cuidadosamente las instrucciones descritas en [Adición de la solución "AzureMonitor-Containers" a un área de trabajo de Azure Log Analytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).

## <a name="deploy-the-application"></a>Implementación de la aplicación

Antes de instalar la aplicación de ejemplo, hay otro paso para configurar el controlador de entrada basado en nginx en el clúster de Kubernetes. El controlador de entrada se usa como equilibrador de carga de capa 7 para enrutar el tráfico en el clúster en función del host, la ruta de acceso o el protocolo. Nginx-ingress está disponible como un gráfico de Helm. Para obtener instrucciones detalladas, consulte el [repositorio de GitHub del gráfico de Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

La aplicación de ejemplo también está empaquetada como un gráfico de Helm, como el [agente de supervisión de Azure](#configure-monitoring) del paso anterior. Por lo tanto, es sencillo implementar la aplicación en el clúster de Kubernetes. Puede encontrar los [archivos del gráfico de Helm en el repositorio de GitHub complementario](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm).

La aplicación de ejemplo es una aplicación de tres niveles, implementada en un clúster de Kubernetes en cada una de las dos instancias de Azure Stack Hub. La aplicación utiliza una base de datos MongoDB. Para más información sobre cómo hacer que los datos se repliquen en varias instancias, consulte la sección [Consideraciones sobre los datos y el almacenamiento](pattern-highly-available-kubernetes.md#data-and-storage-considerations) del patrón.

Después de implementar el gráfico de Helm de la aplicación, verá los tres niveles de la aplicación representados como implementaciones y conjuntos con estado (para la base de datos) con un único pod:

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

En los servicios, encontrará el controlador de entrada basado en nginx y su dirección IP pública:

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

La dirección "IP externa" es nuestro "punto de conexión de la aplicación". Es el modo con el que los usuarios se conectarán para abrir la aplicación y también se usará como punto de conexión para el paso siguiente sobre [Configuración de Traffic Manager](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Escalado automático de la aplicación
Opcionalmente, puede configurar [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) para escalar o reducir verticalmente en función de ciertas métricas, como el uso de la CPU. El siguiente comando creará una instancia de Horizontal Pod Autoscaler que mantiene entre 1 y 10 réplicas de los pods controlados por la implementación de clasificación web. HPA aumentará y disminuirá el número de réplicas (mediante la implementación) para mantener un uso promedio de la CPU en todos los pods del 80 %.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Puede comprobar el estado actual del escalador automático mediante la ejecución de:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Configuración del Administrador de tráfico

Para distribuir el tráfico entre dos (o más) implementaciones de la aplicación, usaremos [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview). Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS en Azure.

> [!NOTE]
> Traffic Manager usa DNS para dirigir las solicitudes del cliente al punto de conexión de servicio más adecuado en función de un método de enrutamiento del tráfico y del estado de los puntos de conexión.

En lugar de usar Azure Traffic Manager, también puede usar otras soluciones de equilibrio de carga globales hospedadas en el entorno local. En el escenario de ejemplo, usaremos Azure Traffic Manager para distribuir el tráfico entre dos instancias de la aplicación. Se pueden ejecutar en instancias de Azure Stack Hub en la misma ubicación o en ubicaciones diferentes:

![Traffic Manager local](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

En Azure, vamos a configurar Traffic Manager para que apunte a las dos instancias diferentes de la aplicación:

[![Perfil de punto de conexión de TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Como puede ver, los dos puntos de conexión apuntan a las dos instancias de la aplicación implementada en la sección [anterior](#deploy-the-application).

En este punto:
- Se ha creado la infraestructura de Kubernetes, incluido un controlador de entrada.
- Los clústeres se han implementado en dos instancias de Azure Stack Hub.
- Se ha configurado la supervisión.
- Azure Traffic Manager equilibrará la carga del tráfico entre las dos instancias de Azure Stack Hub.
- Sobre esta infraestructura, la aplicación de tres niveles de ejemplo se ha implementado de forma automatizada mediante gráficos de Helm. 

Ahora la solución debería estar activa y accesible para los usuarios.

También hay algunas consideraciones operativas posteriores a la implementación que merece la pena analizar, que se tratan en las dos secciones siguientes.

## <a name="upgrade-kubernetes"></a>Actualización de Kubernetes

Tenga en cuenta los siguientes temas al actualizar el clúster de Kubernetes:

- La actualización de un clúster de Kubernetes es una operación compleja de una segunda etapa que se puede realizar mediante el motor de AKS. Para más información, consulte [Actualización de un clúster de Kubernetes en Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- El motor de AKS permite actualizar los clústeres a las versiones más recientes de Kubernetes y de la imagen del sistema operativo de base. Para más información, consulte [Pasos para actualizar a una versión más reciente de Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- También puede actualizar solo los nodos subyacentes a las versiones más recientes de la imagen del sistema operativo de base. Para más información, consulte [Pasos para actualizar únicamente la imagen del sistema operativo](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Las imágenes del sistema operativo de base más recientes contienen actualizaciones de seguridad y del kernel. Es responsabilidad del operador del clúster supervisar la disponibilidad de las versiones más recientes de Kubernetes y de las imágenes del sistema operativo. El operador debe planear y ejecutar estas actualizaciones mediante el motor de AKS. El operador de Azure Stack Hub debe descargar las imágenes del sistema operativo de base desde Marketplace de Azure Stack Hub.

## <a name="scale-kubernetes"></a>Escalado de Kubernetes

El escalado es otra operación de una segunda etapa que se puede organizar mediante el motor de AKS.

El comando de escalado reutiliza el archivo de configuración del clúster (apimodel.json) del directorio de salida como entrada para una nueva implementación de Azure Resource Manager. El motor de AKS ejecuta la operación de escalado en un grupo de agentes específico. Una vez completada la operación de escalado, el motor de AKS actualiza la definición del clúster en el mismo archivo apimodel.json. La definición del clúster refleja el nuevo número de nodos para reflejar la configuración del clúster actual actualizada.

- [Escalado de un clúster de Kubernetes en Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Consideraciones de diseño de aplicaciones híbridas](overview-app-design-considerations.md).
- Revise y proponga mejoras en [el código de este ejemplo en GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).