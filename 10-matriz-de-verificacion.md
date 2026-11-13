# Matriz de verificación

**Proyecto:** Sistema de Control de Acceso con MQTT  
**Estado:** Matriz inicial; los resultados se completarán al realizar las pruebas.

La matriz relaciona cada requisito verificable con un procedimiento y un criterio de aceptación. Se recomienda registrar en la bitácora la fecha, configuración, resultado medido y evidencia (fotografía, captura de pantalla o log) para cada ejecución.

| Prueba | Requisito | Procedimiento | Criterio de aceptación propuesto | Resultado | Evidencia |
|---|---|---|---|---|---|
| PV-01 | RF-01 | Presentar una tarjeta al lector y observar el mensaje MQTT en la BeaglePlay. | El evento llega con identificador seudonimizado y formato válido. | Pendiente | Pendiente |
| PV-02 | RF-02 | Probar una credencial autorizada y otra no autorizada. | La BeaglePlay produce la decisión esperada en ambos casos. | Pendiente | Pendiente |
| PV-03 | RF-03, RF-04 | Solicitar acceso autorizado y medir el tiempo de activación del actuador. | El actuador se activa tras autorización y se desactiva dentro del intervalo configurado (inicialmente 1,5 s). | Pendiente | Pendiente |
| PV-04 | RF-05 | Abrir y cerrar la puerta de maqueta y observar el estado publicado. | Cada cambio se refleja correctamente como abierta o cerrada. | Pendiente | Pendiente |
| PV-05 | RF-06, RF-07 | Generar varios intentos de acceso y consultar los eventos almacenados en la BeaglePlay. | Los eventos quedan guardados con fecha/hora, resultado e identificador seudonimizado. | Pendiente | Pendiente |
| PV-06 | RF-08 | Abrir el panel web desde un equipo de la misma red. | Se muestran el estado del nodo, el estado de puerta y los eventos recientes. | Pendiente | Pendiente |
| PV-07 | RF-09 | Interrumpir Wi-Fi o detener Mosquitto durante la prueba. | El nodo señala la desconexión y aplica la política de contingencia aprobada. | Pendiente | Pendiente |
| PV-08 | RF-10 | Restaurar la red o reiniciar Mosquitto. | El ESP32-C6 se reconecta y reporta disponibilidad sin intervención manual. | Pendiente | Pendiente |
| PV-09 | RF-11 | Intentar conectar con credenciales inválidas y publicar en un topic no permitido. | El broker rechaza la conexión o la publicación no autorizada. | Pendiente | Pendiente |
| PV-10 | RF-12 | Generar un evento definido como notificable. | Telegram recibe el aviso correspondiente sin exponer el UID completo. | Pendiente | Pendiente |
| PV-11 | RNF-04, RNF-05 | Revisar el repositorio y la documentación al cierre. | Código, instrucciones, pruebas y resultados están versionados y accesibles. | Pendiente | Pendiente |

## Registro de ejecución

Para cada prueba, añadir una fila de ejecución o completar los campos de resultado y evidencia con:

- Fecha y hora.
- Integrantes que ejecutaron la prueba.
- Hardware y versiones de software utilizados.
- Resultado observado y mediciones.
- Estado: aprobado, fallido o bloqueado.
- Enlace a la evidencia almacenada en el repositorio.

