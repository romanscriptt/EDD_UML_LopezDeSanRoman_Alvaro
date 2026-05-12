# Sistema de Gestión de Pedidos - Modelado de Comportamiento UML 2.5
**Módulo:** Entornos de Desarrollo · DAM Superior  

**Autores:** Álvaro López de San Román · Santiago González González

**Centro:** FP Superior - Cámara De Comercio De Sevilla

**Trimestre:** 3º Trimestre
---

## Índice
1. [Introducción y Contexto](#1-contexto-y-empresa-auditada)
2. [Diagrama de Actividad (Fase 2)](#2-diagrama-de--actividad-(fase 2)


---
## 1. Introducción y Contexto
Este proyecto lo presentamos en el modelado técnico del proceso que hay detras de una Confirmación de Pedido para una plataforma de E-commerce. Siguiendo los pasos que lleva la industria y el **UML 2.5**, el diseño se enfoca en la optimización de procesos mediante la ejecución concurrente de tareas osea el tema de facturación, ventas, pedidos, tramitaciones.. permitiendo una aplicación más eficiente y una mejor experiencia de usuario al evitar esperas innecesarias en tareas de segundo plano ya que si hacemos esperar un minuto mas al usuario no comprara nuestro producto y seria dinero perdido lo cual no nos intresa perder ventas y sobretodo a usuarios.

## 2. Diagrama de Actividad (Fase 2)
El siguiente diagrama detalla el flujo de control, desde que el usuario finaliza la compra hasta la confirmación final, destacando los puntos de bifurcación y sincronización, lo hemos hecho a través de una extension draw.io integration lo cual nostros lo hemos exportado mediante un png todo se integra autoamticamente.

/////////////// FOTO DIAGRAMA 

## 3. Explicación Técnica del Proceso
El flujo de trabajo ha sido diseñado bajo una lógica de negocio robusta que se divide en tres etapas principales loe hemos elbaorado tal como el suuario entraria y haria toda la gestion desde el que el usuario pulsa el boton hasta el mensaje de confirmación de tal forma que estamos en 1:1 con el usuario.

### A. Validación Concurrente Inicial
Al pulsar "Finalizar compra" como aparece en varias plataforma de ecom, el sistema no actúa de forma lineal que sginifica esto dentro del diagrama. Se utiliza la herramienta **Fork Node** para verificar simultáneamente dos cosas a la vez:
* **Stock de productos:** Consulta al motor de inventario que hay dentro del almacen del invenatrio de la empresa haciendo recuentos simultaneos.
* **Validez de la Sesión:** Comprobación de seguridad del usuario.
Ambos hilos se sincronizan en un **Join Node** antes de proceder al pago, garantizando la integridad de la transacción de la compra de dicho producto.

### B. Gestión de Decisiones
Se han implementado **Decision Nodes** en si lo que son los rombos dentro del digrama loq ue hace esta función es para gestionar los flujos de error:
* Si el stock o la sesión fallan, el flujo se desvía a un **Flow Final Node** (marcado con una X), terminando esa rama sin afectar el resto del sistema si hubiera otros procesos activos.
* El pago cuenta con su propio nodo de decisión para validar el éxito de la pasarela segura.
