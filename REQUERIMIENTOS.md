# Análisis de Requerimientos — TunnelSafe Mesh™

Este documento recoge los requerimientos funcionales (RF) y no funcionales (RNF) del sistema.

---

## 3.1 Requerimientos Funcionales (RF)

| ID | Nombre | Descripción Detallada | Origen |
|----|--------|-----------------------|--------|
| **RF-01** | Asistente de Bring-up Inicial | Si no existe archivo de configuración en `/etc/tunnelsafe/config.conf`, la BeaglePlay debe iniciar automáticamente en modo Punto de Acceso Wi-Fi (`hostapd` + `dnsmasq`) y servir un portal web de configuración inicial (red Wi-Fi local, nombres de zonas y umbrales de gas) sin requerir acceso SSH/consola. | Linux Embebido |
| **RF-02** | Restauración de Fábrica (Factory Reset) | Al mantener presionado el botón físico USR de la BeaglePlay durante ≥ 5.0 segundos (detectado por interrupción `libgpiod`), el sistema debe borrar la configuración persistente, ejecutar `ot factoryreset` en el RCP, parpadear los LEDs de confirmación y reiniciar en estado Bring-up (RF-01). | Linux Embebido |
| **RF-03** | Comunicación Host-RCP y Malla Thread | La BeaglePlay debe comunicarse vía UART/USB (Spinel) con el RCP 802.15.4 mediante `otbr-agent`, conformando una malla Thread en Canal 25 a +20 dBm con enrutamiento IPv6/SLAAC y persistencia por ML-EID. | IoT (Labs 1, 2, 5) |
| **RF-04** | Monitoreo Atmosférico por Eventos | Los nodos sensores deben muestrear concentración de gases/VOC (`/env/air`), aplicar filtros de plausibilidad local y notificar cambios significativos o latidos periódicos a la central usando CoAP Observe sobre payloads CBOR (< 52 bytes). | IoT (Labs 3, 4) |
| **RF-05** | Control Idempotente de Ventilación | La central y los nodos locales deben accionar extractores (`/act/extractor`) y balizas (`/act/alarm`) mediante peticiones CoAP PUT confirmadas (CON) e idempotentes, garantizando estado estable ante retransmisiones. | IoT (Lab 3) |
| **RF-06** | Contrato de Frescura y Fail-Safe Inverso | El recurso `/env/air` debe incluir Max-Age dinámico (60 s). Ante desconexión de sonda (`5.03 Service Unavailable` a los 60 s) o caída de nodo (timeout a los 93 s), la central debe activar automáticamente los extractores al 100% (Fail-Safe inverso) y emitir alarma de mantenimiento. | IoT (Lab 4) |
| **RF-07** | Autonomía Local sin Pasarela | Si la BeaglePlay pierde conexión Wi-Fi o se apaga, los nodos dentro del túnel deben mantener el enrutamiento interno y activar localmente los extractores ante niveles críticos de gas. | IoT (Lab 5) |
| **RF-08** | Interfaz Web y Señalización LED | El demonio nativo en C/C++ debe servir una interfaz web ligera con lecturas en tiempo real, estado de nodos, control manual de extractores e historial de alertas, sincronizada con los 5 LEDs físicos de la BeaglePlay. | Linux Embebido |
| **RF-09** | Instrumentación Soft del Sistema | El sistema debe recolectar y exponer en tiempo real desde `/proc` y `/sys`: porcentaje de uso de CPU, consumo de memoria RAM (RSS/VmSize), bytes RX/TX en interfaces `wpan0` y `wlan0`, puertos UDP6 activos y estado del descriptor UART del RCP. | Linux Embebido |
| **RF-10** | Autenticación e Integridad de Paquetes | Todos los mensajes CoAP de telemetría y actuación entre nodos y central deben estar autenticados/cifrados (DTLS o firma HMAC-SHA256) para rechazar paquetes inyectados o suplantados. | IoT (Lab 6) |

---

## 3.2 Requerimientos No Funcionales (RNF)

| ID | Categoría | Especificación Cuantitativa / Criterio |
|----|-----------|----------------------------------------|
| **RNF-01** | Eficiencia de Recursos (Producto Fabricable) | El demonio principal en la BeaglePlay debe estar escrito en lenguaje compilado (C/C++17), consumir < 15 MB de RAM (RSS), < 10% de CPU en estado estable y cero fugas de memoria (0 bytes *definitely lost* en Valgrind). |
| **RNF-02** | Desempeño RF y Latencia de Red | Enlaces entre nodos con RSSI ≥ −70 dBm y PER < 1% a espaciamiento nominal de 15 m (8–10 m en zonas con obstáculos); latencia de pasarela OTBR < 100 ms (30.71 ms baseline). |
| **RNF-03** | Resiliencia y Autorrecuperación | Ante la caída de un nodo Router intermedio del túnel, la malla Thread debe reconverger y restablecer la ruta en ≤ 8.0 segundos. |
| **RNF-04** | Usabilidad para No Expertos | El flujo completo de Bring-up (desde encendido hasta ver el primer nodo reportando en el panel web) debe completarse en < 3 minutos siguiendo el manual ilustrado, sin uso de terminal. |
| **RNF-05** | Integración Mecánica (Carcasa) | La unidad central y los nodos de prueba deben alojarse en carcasas prototipo (PLA/ABS o acrílico) con ventilación adecuada, acceso libre al botón de Factory Reset y visibilidad directa de los LEDs de estado. |
