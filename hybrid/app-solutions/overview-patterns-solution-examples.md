---
title: Ejemplos de soluciones y patrones híbridos de Azure y Azure Stack Hub
description: Información general sobre ejemplos de soluciones y patrones híbridos para aprender y compilar soluciones híbridas en Azure y Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343865"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Patrones y ejemplos de soluciones híbridas para Azure y Azure Stack

Microsoft proporciona soluciones y productos de Azure y Azure Stack como un ecosistema coherente de Azure. La familia de productos de Microsoft Azure Stack es una extensión de Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Nube híbrida y aplicaciones híbridas

Azure Stack lleva la agilidad de la informática en la nube a su entorno local y al perímetro mediante una *nube híbrida*. Azure Stack Hub, Azure Stack HCI y Azure Stack Edge llevan Azure desde la nube a sus centros de datos soberanos, sucursales, centros de trabajo, etc. Con este conjunto diverso de funcionalidades, puede:

- Reutilizar el código y ejecutar aplicaciones nativas en la nube de forma coherente entre Azure y los entornos locales.
- Ejecutar cargas de trabajo virtualizadas tradicionales con conexiones opcionales a los servicios de Azure.
- Transferir datos a la nube o mantenerlos en su centro de datos soberano para mantener el cumplimiento.
- Ejecutar cargas de trabajo virtualizadas o en contenedores, de aprendizaje automático y aceleradas mediante hardware, y todo ello con la inteligencia perimetral.

Las aplicaciones que abarcan las nubes también se conocen como *aplicaciones híbridas*. Puede compilar las aplicaciones en la nube híbrida en Azure e implementarlas en su centro de datos conectado o desconectado ubicado en cualquier lugar.

Los escenarios de aplicaciones híbridas varían significativamente según los recursos que haya disponibles para el desarrollo. También abarcan aspectos como la región geográfica, la seguridad, el acceso a Internet, etc. Aunque es posible que los patrones y ejemplos de soluciones no cumplan todos los requisitos, proporcionan directrices y ejemplos para explorar y reusar al implementar soluciones híbridas.

## <a name="solution-patterns"></a>Patrones de soluciones

Los patrones de soluciones ofrecen instrucciones de diseño repetibles y generalizadas a partir de escenarios y experiencias de clientes reales. Un patrón es abstracto, lo que permite que se pueda aplicar a distintos tipos de escenarios o sectores verticales. Cada patrón documenta el contexto y el problema, y proporciona información general sobre una solución de ejemplo. La solución de ejemplo está pensada como una posible implementación del patrón.

Hay dos tipos de artículos sobre patrones:

- Patrón único: proporciona una guía de diseño para un único escenario de uso general.
- Varios patrones: proporciona una guía de diseño en la que se usa la aplicación de varios patrones. Este patrón suele ser necesario para resolver escenarios más complejos o problemas específicos del sector.

## <a name="solution-deployment-guides"></a>Guías de implementación de soluciones

Las guías de implementación paso a paso ayudan en la implementación de una solución de ejemplo. La guía también puede hacer referencia a un ejemplo de código complementario que está almacenado en el [repositorio de soluciones de ejemplo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) de GitHub.

## <a name="next-steps"></a>Pasos siguientes

- Consulte la información relativa a la [familia de productos y soluciones de Azure Stack](/azure-stack) para más información sobre toda la gama de productos y soluciones.
- Explore las secciones sobre patrones y guías de implementación de soluciones de la tabla de contenido para más información sobre cada una de ellas.
- Lea el artículo sobre [Consideraciones de diseño para aplicaciones híbridas](overview-app-design-considerations.md) para revisar los pilares de la calidad del software para diseñar, implementar y usar aplicaciones híbridas.
- [Configure un entorno de desarrollo en Azure Stack](/azure-stack/user/azure-stack-dev-start) e [implemente su primera aplicación](/azure-stack/user/azure-stack-dev-start-deploy-app) en Azure Stack.
