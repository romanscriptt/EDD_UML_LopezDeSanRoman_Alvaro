# Sistema de Gestión de Pedidos - Modelado de Comportamiento UML 2.5

**Módulo:** Entornos de Desarrollo · DAM Superior  

**Autores:** Álvaro López de San Román | Santiago González González

**Centro:** FP Superior - Cámara De Comercio De Sevilla

**Trimestre:** 3º Trimestre

---

## Índice
1. [Introducción y Contexto](#1-introducción-y-contexto)
2. [Diagrama de Actividad (Fase 2)](#2-diagrama-de-actividad-fase-2)
3. [Explicación Técnica del Proceso](#3-explicación-técnica-del-proceso)
4. [Justificación de los Nodos de Sincronización](#4-justificación-de-los-nodos-de-sincronización)
5. [Bibliografía (Formato IEEE)](#5-bibliografía-formato-ieee)
   
---
## 1. Introducción y Contexto
Este proyecto lo presentamos en el modelado técnico del proceso que hay detras de una Confirmación de Pedido para una plataforma de E-commerce. Siguiendo los pasos que lleva la industria y el **UML 2.5**, el diseño se enfoca en la optimización de procesos mediante la ejecución concurrente de tareas osea el tema de facturación, ventas, pedidos, tramitaciones.. permitiendo una aplicación más eficiente y una mejor experiencia de usuario al evitar esperas innecesarias en tareas de segundo plano ya que si hacemos esperar un minuto mas al usuario no comprara nuestro producto y seria dinero perdido lo cual no nos intresa perder ventas y sobretodo a usuarios.

## 2. Diagrama de Actividad (Fase 2)
El siguiente diagrama detalla el flujo de control, desde que el usuario finaliza la compra hasta la confirmación final, destacando los puntos de bifurcación y sincronización, lo hemos hecho a través de una extension draw.io integration lo cual nostros lo hemos exportado mediante un png todo se integra autoamticamente.

/////////////// FOTO DIAGRAMA 

## 3. Explicación Técnica del Proceso
El flujo de trabajo ha sido diseñado bajo una lógica de negocio robusta que se divide en tres etapas principales loe hemos elbaorado tal como el suuario entraria y haria toda la gestion desde el que el usuario pulsa el boton hasta el mensaje de confirmación de tal forma que estamos en 1:1 con el usuario.

### A. Validación Concurrente Inicial y Subactividad de Stock

En el desarrollo de plataformas e-commerce en este nicho esta muy explotado ultimamente, el sistema no debe actuar de forma estrictamente lineal lo que significa dentro del sistema que cada tarea espera a la anterior y tendriamos mucho teiempo de espera. En este modelo, la no linealidad se implementa mediante un **Fork Node**, lo que permite que el sistema dispare dos hilos de ejecución paralelos en el momento en que el usuario pulsa "Finalizar compra":

* **Validez de la Sesión:** Se realiza una comprobación de seguridad y tokens del usuario de forma asíncrona.
* **Subactividad: Verificar Stock:** Debido a la complejidad técnica que implica consultar múltiples almacenes físicos y realizar recuentos simultáneos, esta acción se ha modelado como una **subactividad**, lo hemos identifcado con el simbolo del rastrillo.

#### Lógica de la Subactividad "Verificar Stock":
Esta subactividad encapsula una lógica interna de búsqueda distribuida. 
* **Si el stock es insuficiente:** Se activa un **Flow Final Node** el cual lo podemos mirar como un circulo con uan X dentro, esto permite finalizar esta línea de actividad específica osea la validación de stock sin detener otros procesos del sistema que podrían estar en ejecución, enviando una señal de "Verificación fallida" al nodo de decisión.
* **Si el stock es correcto:** El flujo continúa hacia la sincronización.

#### Sincronización e Integridad:
Ambos hilos los de Sesión y Stock, deben ir obligatoriamente en un **Join Node** antes de proceder al pago. Esa barrera de sincronización garantiza la integridad de la transacción: el sistema prohíbe el acceso a la pasarela de pago segura si alguna de las dos validaciones paralelas no ha retornado un token de éxito.

#### Terminación del proceso
Cuando algunas de las dos verficaciones son fallidas se crea una subactividad, donde se termian el proceso manndando un mensaje de error

### B. Gestión de Decisiones
Se han implementado **Decision Nodes** en si lo que son los rombos dentro del digrama loq ue hace esta función es para gestionar los flujos de error:
* Si el stock o la sesión fallan, el flujo se desvía a un **Flow Final Node** (marcado con una X), terminando esa rama sin afectar el resto del sistema si hubiera otros procesos activos.
* El pago cuenta con su propio nodo de decisión para validar el éxito de la pasarela segura.

### C. Post-Pago Concurrencia Crítica
Una vez el pago es marcado como exitoso depues de haber pasado todos los filtros que hay dentro desl sistema, el propio sistema entra en una fase de **concurrencia crítica** esto seria la fase ultima y final para cual todo seri apara la tramitación del pedido. Se utiliza un segundo **Fork Node** para disparar tres acciones que no dependen entre sí, para que el usuario tengo por completo toda la facturación del prodcuto a la hora de que sea seguro y comodo ,para tener esa empatía que es buena al usuario ,para que se siuenta com odo pagando y lo vea de forma segura:
1. **Registro en Base de Datos:** Persistencia del pedido.
2. **Generación de Factura:** Creación del archivo PDF legal.
3. **Notificación:** Envío del correo electrónico de confirmación.

El uso de un **Join Node** final es incluso obligatorio ya que el mensaje de "Confirmación" que aparece no es solo lo que se le muestra al cliente cuando el sistema garantiza que la factura ha sido generada sino tambien pedido está registrado correctamente dentro del sector ecom.

## 4. Justificación de los Nodos de Sincronización
* **Fork Node:** Se justifica para mejorar el rendimiento interno que ocuure dentro del sistema. En entornos distribuidos con muchas acciones, lanzar tareas en paralelo reduce la latencia percibida por el cliente, esto siginfica que a la hora del usuario entrar dentro del sistema de confirmacón del pedido, tener varias acciones hace que el filtro se vuelve mas inteligente y ayude al sistema.
* **Join Node:** Se utiliza como barrera de sincronización. Asegura que el flujo de control no progrese hasta que todos los procesos paralelos hayan retornado un token de éxito, token dentro de estas paginas de compra significa una llave de un solo uso, evitando así condiciones de datos incompletos en la confirmación final.
* **Activity Final Node:** A diferencia del Flow Final, este nodo se sitúa al final del camino tras el mensaje de "Mostrar Confirmación". Su activación indica que toda la instancia de la actividad "Procesamiento de Pedido" se ha completado satisfactoriamente y todos los tokens dentro del diagrama han sido consumidos.

## 5. Bibliografía (Formato IEEE)

Para la elaboración de esta actividad y la correcta implementación de la sintaxis UML 2.5, se han consultado las siguientes fuentes:

* **[1] Object Management Group (OMG)**, “Unified Modeling Language (UML) Specification Version 2.5.1 - Section 15.3: Control Nodes,” Mar. 2021. [En línea]. Disponible en: [https://www.omg.org/spec/UML/2.5.1/Formal-Specification](https://www.omg.org/spec/UML/2.5.1/Formal-Specification) (Consultar específicamente la semántica de *ForkNode* y *JoinNode* en la sección de especificación formal).

* **[2] IBM Documentation**, “UML activity diagrams: Synchronization bars and control nodes,” Rational Rhapsody, 2021. [En línea]. Disponible en: [https://www.ibm.com/docs/en/rhapsody/9.0.1?topic=diagrams-uml-activity#control-nodes](https://www.ibm.com/docs/en/rhapsody/9.0.1?topic=diagrams-uml-activity#control-nodes) (Documentación técnica sobre la división y unión de flujos concurrentes).

* **[3] Visual Paradigm**, “UML Activity Diagram Notation Guide: Fork and Join Nodes,” 2024. [En línea]. Disponible en: [https://www.visual-paradigm.com/guide/uml/what-is-activity-diagram/#activity-diagram-notation-guide](https://www.visual-paradigm.com/guide/uml/what-is-activity-diagram/#activity-diagram-notation-guide) (Guía de notación visual para la implementación de barras de sincronización).

---

**Desarrollado por:** Álvaro López | Santiago González

**Asignatura:** Entornos de Desarrollo
