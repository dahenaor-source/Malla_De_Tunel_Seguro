# Matriz de Verificación — TunnelSafe Mesh™

**Pruebas Unitarias y de Integración**

Para cumplir con las exigencias de la asignatura de Linux Embebido, la matriz asigna **responsabilidades cruzadas** entre los 3 integrantes del equipo (quien implementa una funcionalidad no es quien la verifica):

- **Integrante 1 (I1):** Arquitecto Linux Core (Demonio C++, Bring-up, Factory Reset, Instrumentación `/proc`).
- **Integrante 2 (I2):** Ingeniero IoT & Seguridad (Firmware ESP32-C6 Thread, CoAP/CBOR, Fail-Safe, Criptografía Lab 6).
- **Integrante 3 (I3):** Ingeniero de Interfaz, Hardware & QA (Servidor Web ligero, LEDs, Carcasa prototipo, Pruebas y Video).

---

| ID | Req. | Tipo | Caso de Prueba y Procedimiento | Criterio de Aceptación | Herramientas | Impl. | Verif. |
|----|------|------|--------------------------------|------------------------|--------------|:-----:|:------:|
| **PV-01** | RF-01, RNF-04 | Integración (Bring-up) | Arrancar BeaglePlay sin `/etc/tunnelsafe/config.conf`. Conectarse al AP `TunnelSafe-Setup`, configurar Wi-Fi local y nombres de tramo vía web, y reiniciar. | El AP levanta en < 20 s; tras guardar, el equipo se conecta al Wi-Fi configurado y comisiona la malla sin usar consola. | `systemctl status`, `iw dev`, navegador web, `ot-ctl state` | I1 | I3 |
| **PV-02** | RF-02 | Integración (Factory Reset) | Con el sistema operando normalmente, mantener pulsado el botón USR durante 5 s. Medir tiempo de respuesta y estado posterior al reinicio. | LEDs parpadean 3 veces; se borra `/etc/tunnelsafe/`, el RCP ejecuta `ot factoryreset` y vuelve al modo AP de Bring-up. | `gpiod` / `gpiomon`, `strace`, `ls -la /etc/tunnelsafe` | I1 | I2 |
| **PV-03** | RF-04, RF-05 | Unitaria (Firmware / App) | Validar codificación/decodificación CBOR de `/env/air` y `/act/extractor`, además de la idempotencia al recibir 3 paquetes PUT CON duplicados. | Payload CBOR < 52 bytes; el actuador mantiene estado ON (`2.04 Changed`) sin conmutaciones erráticas. | Framework Unity / GoogleTest, Wireshark (disector CoAP/CBOR) | I2 | I1 |
| **PV-04** | RF-04, RNF-02 | Integración (Lazo Cerrado) | Inyectar gas frío de encendedor o vapor de alcohol en la cámara del túnel a escala; observar disparo de CoAP Observe y activación del extractor. | Tiempo desde detección hasta encendido del extractor < 2.0 s; el extractor se apaga automáticamente al bajar la concentración. | Cronómetro, `tcpdump -i wpan0`, panel web | I2 | I3 |
| **PV-05** | RF-06 | Integración (Fail-Safe Inverso) | Caso A: Desconectar cable del sensor de gas. Caso B: Apagar completamente el nodo sensor. Medir tiempos exactos de reacción en la central. | Caso A: Error `5.03` a los 60 s. Caso B: Timeout a los 93 s. En ambos casos, el extractor se enciende al 100% automáticamente. | Logs del demonio C++, Wireshark, multímetro en relé | I2 | I1 |
| **PV-06** | RF-03, RNF-03 | Integración (Malla Thread) | Configurar cadena lineal A (Central) ↔ B (Router) ↔ C (Sensor fondo) en Canal 25. Apagar nodo B durante transmisión activa. | La malla reconverge por ruta de respaldo en ≤ 8.0 s manteniendo las direcciones ML-EID sin pérdida permanente de telemetría. | `ot-ctl router table`, `ot-ctl neighbor table`, `ot ping` | I2 | I3 |
| **PV-07** | RF-07 | Integración (Autonomía) | Desconectar alimentación/red de la BeaglePlay (OTBR) e inyectar gas en el nodo sensor del túnel. | El nodo sensor envía PUT `/act/extractor` directamente al nodo actuador dentro de la malla y enciende el extractor. | Inspección visual de extractor y LEDs de nodos ESP32-C6 | I2 | I1 |
| **PV-08** | RF-08, RNF-05 | Unitaria / Sistema (UI & Carcasa) | Verificar sincronización entre los 5 LEDs físicos (`/sys/class/leds/`) y los estados de la web; validar ensamblaje mecánico de la carcasa. | Latencia visual web < 100 ms; LEDs reflejan estado real; botón USR accesible y puertos libres en carcasa. | Inspección física, DevTools navegador, `/sys/class/leds/` | I3 | I1 |
| **PV-09** | RF-09, RNF-01 | Unitaria / Sistema (Instrumentación) | Ejecutar el demonio C++ bajo carga de tráfico CoAP durante 30 min monitoreando fugas de memoria, llamadas al sistema y contadores de red. | RAM RSS < 15 MB, CPU < 10%, 0 bytes perdidos en Valgrind; contadores RX/TX de `wpan0`/`wlan0` coinciden con tráfico real. | `valgrind --leak-check=full`, `systemtap` / `perf` / `strace`, `/proc` | I1 | I3 |
| **PV-10** | RF-10 | Integración (Seguridad Lab 6) | Intentar enviar un paquete PUT `/act/extractor v=0` (apagar ventilación) desde un cliente no autorizado sin clave PSK/HMAC válida. | El nodo actuador y la central rechazan el paquete no autenticado y registran un intento de intrusión en la bitácora. | `libcoap` client malicioso, Wireshark (tramas cifradas/HMAC) | I2 | I3 |
