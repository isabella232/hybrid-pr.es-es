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
# <a name="hybrid-relay-pattern"></a>Patrón de retransmisión híbrida

Obtenga información sobre cómo conectarse a recursos perimetrales o dispositivos protegidos mediante firewalls mediante el patrón de retransmisión híbrida y el Azure Relay.

## <a name="context-and-problem"></a>Contexto y problema

Los dispositivos perimetrales suelen estar detrás de un firewall corporativo o un dispositivo NAT. Aunque son seguros, es posible que no puedan comunicarse con la nube pública o con dispositivos perimetrales de otras redes corporativas. Puede que sea necesario exponer determinados puertos y funcionalidad a los usuarios de la nube pública de forma segura.

## <a name="solution"></a>Solución

El patrón de retransmisión híbrida usa Azure Relay para establecer un túnel de WebSockets entre dos extremos que no se pueden comunicar directamente. Los dispositivos que no están en el entorno local, pero que necesitan conectarse a un punto de conexión local, se conectarán a un punto de conexión de la nube pública. Este punto de conexión redirigirá el tráfico de las rutas predefinidas por un canal seguro. Un punto de conexión dentro del entorno local recibe el tráfico y lo enruta al destino correcto.

![arquitectura de la solución del patrón de retransmisión híbrida](media/pattern-hybrid-relay/solution-architecture.png)

Este es el funcionamiento del patrón de retransmisión híbrida:

1. Un dispositivo se conecta a la máquina virtual (VM) de Azure en un puerto predefinido.
2. El tráfico se reenvía al Azure Relay en Azure.
3. La máquina virtual en Azure Stack concentrador, que ya ha establecido una conexión de larga duración con la Azure Relay, recibe el tráfico y lo reenvía al destino.
4. El servicio o punto de conexión local procesa la solicitud.

## <a name="components"></a>Componentes

Esta solución usa los siguientes componentes:

| Nivel | Componente | Descripción |
|----------|-----------|-------------|
| Azure | Azure VM | Una máquina virtual de Azure proporciona un punto de conexión accesible públicamente para el recurso local. |
| | Azure Relay | Un [Azure Relay](/azure/azure-relay/) proporciona la infraestructura para el mantenimiento del túnel y la conexión entre la máquina virtual de Azure y la máquina virtual de Azure Stack Hub.|
| Azure Stack Hub | Proceso | Una máquina virtual de Azure Stack Hub proporciona el lado servidor del túnel de retransmisión híbrida. |
| | Storage | El clúster del motor de AKS implementado en Azure Stack Hub proporciona un motor escalable y resistente para ejecutar el contenedor de Face API.|

## <a name="issues-and-considerations"></a>Problemas y consideraciones

Tenga en cuenta los puntos siguientes al decidir cómo implementar esta solución:

### <a name="scalability"></a>Escalabilidad

Este patrón solo permite asignaciones de puerto 1:1 en el cliente y el servidor. Por ejemplo, si se tuneliza el puerto 80 para un servicio del punto de conexión de Azure, no se puede usar para otro servicio. Las asignaciones de puertos deben planearse en consecuencia. El Azure Relay y las máquinas virtuales deben escalarse adecuadamente para controlar el tráfico.

### <a name="availability"></a>Disponibilidad

Estos túneles y conexiones no son redundantes. Para garantizar alta disponibilidad, podría interesarle implementar el código de comprobación de errores. Otra opción es tener un grupo de máquinas virtuales conectadas Azure Relay detrás de un equilibrador de carga.

### <a name="manageability"></a>Facilidad de uso

Esta solución puede abarcar muchos dispositivos y ubicaciones, lo que podría resultar complicado. Los servicios de IoT de Azure pueden poner en línea automáticamente nuevas ubicaciones y dispositivos y mantenerlos actualizados.

### <a name="security"></a>Seguridad

Este patrón, tal como se muestra, permite el acceso ilimitado a un puerto en un dispositivo interno desde el perímetro. Considere la posibilidad de agregar un mecanismo de autenticación al servicio en el dispositivo interno o delante del punto de conexión de retransmisión híbrida.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los temas presentados en este artículo:

- Este patrón usa Azure Relay. Para obtener más información, consulte la [documentación de Azure Relay](/azure/azure-relay/).
- Consulte [Consideraciones sobre el diseño de aplicaciones híbridas](overview-app-design-considerations.md) para más información sobre los procedimientos recomendados y obtener respuestas a preguntas adicionales.
- Consulte la información relativa a la [familia de productos y soluciones de Azure Stack](/azure-stack) para más información sobre toda la gama de productos y soluciones.

Cuando esté listo para probar la solución de ejemplo, continúe con la [guía de implementación de soluciones de retransmisión híbrida](https://aka.ms/hybridrelaydeployment). La guía de implementación proporciona instrucciones paso a paso para implementar y probar sus componentes.