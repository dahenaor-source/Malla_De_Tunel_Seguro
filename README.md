# Sistema de Control de Acceso con MQTT

Proyecto académico de Ingeniería Electrónica para la asignatura de Sistemas Linux Embebidos.

## Equipo

- David
- Juan Camilo
- Daniela

## Descripción

Prototipo de laboratorio para controlar el acceso a una puerta de maqueta mediante RFID. Un ESP32-C6 DevKitC lee las tarjetas y controla el actuador. La BeaglePlay ejecuta Linux, el broker MQTT, el servicio que autoriza credenciales, el registro de eventos y el panel web.

```text
Tarjeta RFID ──► ESP32-C6 ── Wi-Fi / MQTT ──► BeaglePlay
                     │                         ├── Mosquitto
Sensor de puerta ────┤                         ├── Servicio de autorización
Actuador ◄───────────┘                         ├── Registro de eventos
                                               └── Panel web
```

El objetivo es demostrar la integración entre microcontrolador, hardware, Linux embebido y comunicación IoT. El prototipo es una maqueta académica; no se considera un sistema certificado para proteger una vivienda real.

## Documentos

- [Requerimientos y restricciones](02-requerimientos-y-restricciones.md)
- [Matriz de verificación](10-matriz-de-verificacion.md)

## Estado del documento

Borrador inicial. Los criterios cuantitativos y las decisiones marcadas como pendientes deben confirmarse con el equipo y actualizarse a partir de las pruebas.

