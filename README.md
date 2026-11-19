# TunnelSafe Mesh™

**Sistema Autónomo de Monitoreo Atmosférico y Control de Ventilación Forzada**

TunnelSafe Mesh™ es un sistema autónomo de monitoreo atmosférico y control de ventilación forzada diseñado para **túneles viales en construcción, parqueaderos subterráneos, cuartos de máquinas y galerías mineras** donde no existe cableado de red ni cobertura Wi-Fi continua.

Está concebido como un producto **llave en mano** para operarios no expertos en sistemas embebidos (supervisores de seguridad industrial o administradores de obra).

---

## 1. Visión de Producto Comercial

### 1.1 Experiencia de Usuario y Operación en Campo

1. **Desembalaje y Puesta en Marcha (Bring-up guiado):** El cliente conecta la Unidad Central de Control (BeaglePlay) a la corriente eléctrica. Al detectar que es un equipo sin configurar, la central enciende un LED azul parpadeante y crea una red Wi-Fi propia (`TunnelSafe-Setup`). Desde su celular o portátil, el supervisor ingresa a un asistente web sencillo donde conecta el equipo al Wi-Fi de la caseta de control, asigna nombres legibles a cada tramo del túnel (ej. "Boca Norte", "Tramo Medio", "Frente de Obra") y fija los límites de alarma de gases.

2. **Despliegue sin Cableado de Datos:** Los módulos de medición (Nodos Sensores) y control (Nodos Extractores) se fijan a las paredes del túnel cada 10 a 15 metros. Al encenderse, se descubren y enlazan automáticamente formando una red malla autorrecuperable sin requerir configuración por consola.

3. **Operación Autónoma y Fail-Safe Inverso (Protección de Vida):**
   - **Modo Normal:** La central muestra en pantalla el estado de cada tramo en verde y los extractores permanecen apagados para ahorrar energía.
   - **Modo Alerta por Contaminación:** Si se acumula monóxido de carbono (CO), humo o compuestos orgánicos volátiles (VOC), el nodo afectado notifica al instante a la central y se encienden los extractores de aire y las balizas luminosas/sonoras hasta purgar el ambiente.
   - **Modo Fail-Safe Inverso ante Fallas:** A diferencia de un sistema de riego que cierra válvulas ante un fallo, en un túnel la pérdida de información representa riesgo de asfixia. Si un sensor se desconecta (`5.03` a los 60 s) o un nodo queda aplastado o sin energía (timeout a los 93 s), la central ordena el encendido continuo al 100% de los extractores hasta que mantenimiento revise el tramo.
   - **Supervivencia sin Internet ni Central:** Si se corta el Wi-Fi o se apaga la unidad central, los nodos dentro del túnel siguen comunicándose entre sí y activando la ventilación local de forma autónoma.

4. **Restauración de Fábrica (Factory Reset):** Si el equipo se traslada a otra obra o se olvida la contraseña, basta con mantener presionado durante 5 segundos el botón físico lateral (`RESET` / `USR`) con un clip; los LEDs confirman el borrado y el dispositivo vuelve limpio al asistente inicial.

---

## 2. Descripción del Sistema

El sistema adopta una **arquitectura distribuida de borde** que unifica el ecosistema **Linux Embebido** (en la placa BeaglePlay) con una **red malla inalámbrica Thread sobre IEEE 802.15.4** (nodos ESP32-C6), alineada con el modelo de referencia **ISO/IEC 30141**.

### 2.1 Arquitectura Hardware-Software

#### Unidad Central de Borde (SBC Linux – BeaglePlay)

- **Procesador Host (TI AM625):** Ejecuta Linux Embebido con un demonio principal escrito en C++17 / C (compilado nativamente sin máquinas virtuales ni intérpretes pesados para minimizar huella de RAM/CPU). Gestiona el enrutador de frontera (`otbr-agent`), el cliente/servidor CoAP/CBOR (`libcoap`), la lógica de control Fail-Safe, el servidor web embebido (`libmicrohttpd` / `mongoose`) y el colector de instrumentación del sistema operativo.
- **Coprocesador de Radio (RCP):** Conectado internamente por UART (TI CC1352P7 integrado en BeaglePlay o ESP32-C6 externo vía USB-UART) mediante el protocolo Spinel, operando como interfaz de capa física/MAC IEEE 802.15.4.
- **Interfaz Física y Carcasa:** Encapsulada en carcasa prototipo (PLA/acrílico) con acceso al pulsador físico USR (leído mediante la API nativa de kernel `libgpiod`) y 5 LEDs de estado controlados vía `/sys/class/leds/` (Alimentación, Malla Thread, Enlace Wi-Fi, Alarma Gas y Estado Fail-Safe).

#### Nodos de Campo (Microcontroladores ESP32-C6)

- **Nodos Sensores / Routers de Tramo:** Alimentados por línea eléctrica del túnel (RX Always-On para enrutar tráfico multisalto). Integran sensores de calidad de aire/gases (ej. SGP30/BME680 por bus hardware I2C para evitar jitter por interrupciones de radio, o MQ-135/MQ-2 por ADC con filtro de plausibilidad).
- **Nodos Actuadores (Extractores y Balizas):** Controlan relés de potencia para ventiladores de extracción (`/act/extractor`) y balizas de evacuación (`/act/alarm`). En caso de balizas auxiliares a batería, operan como Sleepy End Devices (SED) con T_poll = 5 s.

#### Pila de Comunicaciones y Confiabilidad (Labs 1–5 + Proyección Lab 6)

- **Capa Física y Red:** Operación estricta en Canal 25 (2475 MHz) a +20 dBm, aprovechando la ganancia de procesamiento DSSS (≈ 9 dB) y la banda de guarda frente a Wi-Fi. Enrutamiento dinámico IPv6/6LoWPAN con autorrecuperación determinista en 8.0 s ante caída de nodos intermedios y direccionamiento persistente por ML-EID.
- **Capa de Aplicación:** Transporte CoAP sobre UDP con payloads binarios CBOR (< 52 bytes por trama sin fragmentación), suscripciones CoAP Observe (reducción del 89.67% en tráfico de radio), comandos confirmados (CON) idempotentes y validación de frescura mediante Max-Age dinámico (60 s) y código `5.03 Service Unavailable`.
- **Capa de Seguridad (Integración Lab 6):** Cifrado y autenticación extremo a extremo de paquetes CoAP (mediante DTLS 1.2 PSK/ECDSA o firmas criptográficas HMAC-SHA256 en el payload CBOR) y protección de acceso al portal web para cerrar las brechas de tráfico en texto plano identificadas en el Lab 5.

---

## 3. Documentación del Proyecto

| Documento | Descripción |
|-----------|-------------|
| [`REQUERIMIENTOS.md`](./REQUERIMIENTOS.md) | Análisis de requerimientos funcionales (RF) y no funcionales (RNF). |
| [`MATRIZ_DE_VERIFICACION.md`](./MATRIZ_DE_VERIFICACION.md) | Matriz de verificación con las pruebas unitarias y de integración (PV-01 a PV-10). |

---

## 4. Equipo y Responsabilidades

Para cumplir con las exigencias de la asignatura de Linux Embebido, las responsabilidades se asignan de forma cruzada entre los 3 integrantes del equipo (quien implementa una funcionalidad no es quien la verifica):

- **Integrante 1 (I1):** Arquitecto Linux Core (Demonio C++, Bring-up, Factory Reset, Instrumentación `/proc`).
- **Integrante 2 (I2):** Ingeniero IoT & Seguridad (Firmware ESP32-C6 Thread, CoAP/CBOR, Fail-Safe, Criptografía Lab 6).
- **Integrante 3 (I3):** Ingeniero de Interfaz, Hardware & QA (Servidor Web ligero, LEDs, Carcasa prototipo, Pruebas y Video).

---

## 5. Plan de Trabajo (3 Semanas)

- **Semana 1:** Montaje del entorno de compilación cruzada/nativa en la BeaglePlay, integración de `otbr-agent` con el RCP en Canal 25, e implementación de los módulos de Bring-up y Factory Reset (`libgpiod` + LEDs).
- **Semana 2:** Desarrollo del demonio central en C++17 (`libcoap` + servidor web ligero + lectura de `/proc`), adaptación del firmware de nodos ESP32-C6 para sensores de gas/extractores con Fail-Safe inverso, e impresión/corte de carcasas.
- **Semana 3:** Integración de la seguridad de paquetes (Lab 6), ejecución completa de las 10 pruebas de la Matriz de Verificación (capturando evidencias de `valgrind`, `systemtap`/`strace` y Wireshark), redacción del manual de usuario y grabación del video demostrativo.
