# Requerimientos y restricciones

**Proyecto:** Sistema de Control de Acceso con MQTT  
**Curso:** Sistemas Linux Embebidos  
**Equipo:** David, Juan Camilo y Daniela  
**Fecha de entrega del proyecto:** 9 de diciembre de 2026  
**Estado:** Borrador inicial

## 1. Objetivo

Diseñar e implementar un prototipo académico de control de acceso para una puerta de maqueta. El lector RFID conectado al ESP32-C6 captura las credenciales; la BeaglePlay, mediante un servicio Linux y comunicación MQTT, toma la decisión de acceso, registra los eventos y los presenta en un panel web.

El prototipo demostrará integración de hardware, firmware, Linux embebido, redes IoT y documentación de pruebas. No se plantea como un sistema certificado para proteger una vivienda real.

## 2. Alcance

- Una puerta de maqueta y un nodo ESP32-C6 DevKitC.
- Lector RFID como único método de identificación en la primera versión.
- Sensor magnético para detectar si la puerta está abierta o cerrada.
- Actuador de maqueta para representar el mecanismo de apertura. El tipo de actuador está por definir.
- BeaglePlay con Mosquitto, servicio de autorización, almacenamiento de eventos y panel web.
- Autorización central en la BeaglePlay durante operación normal.
- Operación degradada si se pierde la conexión, de acuerdo con la política provisional de contingencia indicada en este documento.
- Notificaciones por Telegram como función deseable, sujeta al tiempo disponible.

## 3. Requerimientos funcionales

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-01 | El sistema deberá leer una tarjeta RFID y enviar su identificador al servicio de autorización de la BeaglePlay mediante MQTT. | Alta |
| RF-02 | La BeaglePlay deberá determinar si la credencial está autorizada y responder con una decisión de acceso. | Alta |
| RF-03 | El ESP32-C6 deberá activar el actuador únicamente cuando reciba una decisión válida de autorización o cuando aplique la política local de contingencia aprobada. | Alta |
| RF-04 | El actuador deberá permanecer activo durante un intervalo limitado y configurable. Valor inicial propuesto: 1,5 segundos. | Alta |
| RF-05 | El sistema deberá detectar y publicar los cambios de estado abierto/cerrado de la puerta. | Alta |
| RF-06 | Cada intento de acceso deberá generar un evento con fecha/hora, resultado e identificador de credencial seudonimizado. | Alta |
| RF-07 | La BeaglePlay deberá almacenar los eventos para consultarlos posteriormente. | Alta |
| RF-08 | El panel web deberá mostrar el estado de conexión del nodo, el estado de la puerta y una lista de eventos recientes. | Alta |
| RF-09 | Si se pierde MQTT, el ESP32-C6 deberá indicar la desconexión y seguir la política de contingencia definida. | Alta |
| RF-10 | Al recuperar la conexión, el ESP32-C6 deberá reconectarse al broker y reportar su estado. | Alta |
| RF-11 | El broker deberá requerir autenticación y limitar los topics que cada cliente puede publicar o suscribir. | Alta |
| RF-12 | El sistema podrá enviar por Telegram notificaciones de acceso denegado, puerta abierta por tiempo prolongado o pérdida de conexión. | Deseable |

## 4. Requerimientos no funcionales y restricciones

| ID | Requerimiento / restricción |
|---|---|
| RNF-01 | El sistema deberá operar en una red local con la BeaglePlay como broker MQTT. |
| RNF-02 | En condiciones normales, la decisión de autorización deberá realizarse en la BeaglePlay. |
| RNF-03 | El prototipo deberá poder demostrarse en una maqueta de laboratorio, sin instalarse en una puerta real. |
| RNF-04 | El firmware y el software de la BeaglePlay deberán estar versionados en el repositorio GitHub del equipo. |
| RNF-05 | La documentación deberá incluir arquitectura, conexiones, configuración, pruebas y resultados observados. |
| RNF-06 | La selección del actuador deberá considerar el presupuesto aproximado disponible de 40.000 COP y el uso en una maqueta. |
| RNF-07 | La alimentación de la lógica de control y la del actuador deberán diseñarse de acuerdo con sus requisitos eléctricos. Ningún actuador de potencia se conectará directamente a un GPIO. |

## 5. Política provisional ante pérdida de red

La intención reportada por el equipo es autorizar desde la BeaglePlay y permitir que la maqueta siga funcionando ante una pérdida de conexión. Para conciliar ambos objetivos, se propone que el ESP32-C6 mantenga una copia temporal de credenciales autorizadas para contingencia y almacene localmente los eventos ocurridos sin red para enviarlos al recuperar la conexión.

Esta política es provisional y requiere aprobación del equipo. Deben definirse la vigencia de la copia, cómo se actualiza y si una credencial revocada puede seguir abriendo durante la desconexión. Si no se aprueba esta política, el sistema deberá denegar el acceso cuando no pueda consultar a la BeaglePlay.

## 6. Consideraciones de seguridad y limitaciones

- El UID de una tarjeta puede usarse como identificador para la demostración, pero no debe considerarse una credencial de alta seguridad para proteger una vivienda real.
- El sistema deberá evitar publicar el UID completo en el panel o en notificaciones; se utilizará un identificador seudonimizado.
- Se deberán usar credenciales MQTT y permisos mínimos por dispositivo.
- El prototipo no reemplaza un sistema de control de acceso certificado ni resuelve requisitos normativos de una instalación real.

## 7. Decisiones pendientes

1. Confirmar la política ante pérdida de red y la vigencia de la lista local de contingencia.
2. Seleccionar el actuador de maqueta y su circuito driver considerando el presupuesto.
3. Confirmar el modelo del lector RFID y los sensores disponibles para compra.
4. Identificar la distribución y versión Linux instalada en la BeaglePlay.
5. Definir si Telegram entra en el alcance obligatorio o queda como mejora.
6. Acordar criterios cuantitativos finales para tiempos de respuesta y estado de puerta después de las primeras pruebas.

