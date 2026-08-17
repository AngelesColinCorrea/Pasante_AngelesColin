# TeleCom-UAV

Sistema de enlace RF de 900 MHz con diversidad de antena y control de potencia adaptativo para telemetría y comando-control (C2) de un vehículo aéreo no tripulado (UAV).

## Descripción

El proyecto consiste en el diseño de **TeleCom-UAV**, un sistema de radiofrecuencia para el enlace de telemetría y comando y control (C2, *Command and Control*) de un vehículo aéreo no tripulado (UAV), que opera en la banda ISM de 902-928 MHz. Este tipo de enlace constituye la columna vertebral de la operación segura de un UAV: a través de él se transmiten los comandos de vuelo desde la estación terrena hacia la aeronave, y se reciben en tiempo real los datos de telemetría (posición, altitud, velocidad, estado de batería, entre otros). El protocolo estándar de facto para este intercambio de información en el ecosistema de drones de código abierto es MAVLink, empleado por estaciones terrenas como QGroundControl y Mission Planner para comunicarse con controladores de vuelo basados en PX4 o ArduPilot.

La arquitectura de TeleCom-UAV integra dos antenas en configuración de diversidad —una omnidireccional de cobertura constante y una direccional de mayor ganancia— conmutadas mediante dos switchers según la calidad de señal (RSSI), un amplificador bidireccional de 10 W en la estación terrena y un transceptor de a bordo que opera a 1 W. La información recibida se procesa y despliega en una PC que funge como estación de control terrestre.

## Alcance

El proyecto abarca el diseño, la construcción física y la validación del sistema TeleCom-UAV, e incluye:

- Arquitectura del sistema (diagrama de bloques y justificación de cada componente)
- Cálculo del presupuesto de enlace (link budget) para estimar el alcance teórico bajo los límites de potencia permitidos
- Lógica de conmutación de antenas basada en RSSI, con margen de histéresis
- Selección y justificación del amplificador, las antenas y el transceptor propuestos, dentro del marco regulatorio aplicable a la banda de 900 MHz en México
- Construcción física del sistema
- Pruebas de campo con una aeronave real, para validar el desempeño del enlace y de la lógica de conmutación de antenas

Queda fuera del alcance:

- El enlace de video downlink (el proyecto se limita al enlace de telemetría/C2)

## Revisión de literatura

**Marco regulatorio.** En México, la banda de 902-928 MHz está clasificada como espectro de uso libre por el Instituto Federal de Telecomunicaciones (IFT), con una potencia máxima de transmisión entregada a la antena de 1 W, una ganancia máxima de antena direccional de 6 dBi, y una PIRE máxima de 4 W; si se utiliza una antena direccional con ganancia mayor a 6 dBi, la potencia de entrada debe reducirse en la misma proporción en que la ganancia la exceda. Este marco normativo acota los cálculos de link budget y la selección del amplificador y las antenas de TeleCom-UAV.

**Protocolo de comunicación.** MAVLink es un protocolo de mensajería ligero, binario, diseñado específicamente para enlaces de bajo ancho de banda como los de radio, y define cientos de tipos de mensajes estandarizados para telemetría y comando, sin depender de la tecnología subyacente del enlace.

**Fundamentos de enlace de RF.** El presupuesto de enlace se basa en la ecuación de Friis de transmisión en espacio libre, que relaciona la potencia recibida con la potencia transmitida, las ganancias de antena y la pérdida por trayectoria en función de la distancia y la frecuencia; es el punto de partida estándar en la ingeniería de comunicaciones por RF para estimar el alcance máximo de un enlace.

**Diversidad de antena.** La técnica de diversidad de antena para mitigar el desvanecimiento de la señal tiene su origen conceptual en esquemas como el propuesto por Alamouti, que demostró que combinar dos antenas puede alcanzar el mismo orden de diversidad que sistemas de recepción de mayor complejidad. Para la conmutación práctica entre antenas, la literatura de patentes de la industria describe el uso de temporizadores de espera y umbrales de histéresis para reducir el número de conmutaciones y evitar degradar el desempeño del enlace. En el ámbito específico de UAVs, investigación reciente continúa explorando esquemas de diversidad de antena para enlaces UAV en canales con desvanecimiento.

## Estado del arte

Se comparan tres sistemas comerciales de radio de telemetría en 900 MHz utilizados en el ecosistema de drones de código abierto:

| Sistema | Banda | Potencia | Diversidad de antena | Notas |
|---|---|---|---|---|
| Microhard P900 | 902-928 MHz | 100 mW – 1 W | No nativa | FHSS, comunicación MAVLink |
| RFD900x | 902-928 MHz | Hasta 1 W (pasos de 1 dB) | Sí, automática (en tiempo real, a nivel de paquete) | Alcance >40 km, RSSI vía MAVLink |
| Digi XBee-Pro 900HP | 902-928 MHz | Hasta 250 mW | No nativa | Orientado a redes de sensores (DigiMesh) |

El RFD900x ya resuelve la diversidad de antena de forma automática, pero en los tres sistemas la potencia de transmisión se configura de forma fija o manual, no dinámica. Ahí radica el aporte de TeleCom-UAV: un control de potencia adaptativo según el RSSI reportado, combinado con conmutación de antena con histéresis — una mejora en eficiencia energética y estabilidad de enlace sobre los sistemas comerciales revisados.

## Fuentes

- [Inventario de Bandas de Frecuencias Clasificadas como Espectro Libre (IFT)](https://www.ift.org.mx/sites/default/files/contenidogeneral/espectro-radioelectrico/inventariodebandasdefrecuenciasclasificadascomoespectrolibre-mayo2025.pdf)
- [Protocol Overview - MAVLink Guide](https://mavlink.io/en/about/overview.html)
- [Holybro Microhard P900 Radio - PX4 Guide](https://docs.px4.io/main/en/telemetry/holybro_microhard_p900_radio)
- [RFD900x and RFD868x Radio Modem Datasheet](https://files.rfdesign.com.au/Files/documents/RFD900x%20DataSheet%20V1.2.pdf)
- [XBee-PRO 900HP Product Datasheet - Digi International](https://www.digi.com/resources/library/data-sheets/ds_xbeepro900hp)
- [A Simple Transmit Diversity Technique for Wireless Communications (Alamouti, 1998)](https://www.ece.tufts.edu/ee/108/alamouti.pdf)

## Autor

Ángeles Colín Correa — Universidad Aeronáutica en Querétaro (UNAQ)
