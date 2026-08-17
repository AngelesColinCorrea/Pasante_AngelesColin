# TeleCom-UAV

Sistema de enlace RF de 900 MHz con diversidad de antena y control de potencia adaptativo para telemetría y comando-control (C2) de un vehículo aéreo no tripulado (UAV).

## Descripción

TeleCom-UAV es un sistema de radiofrecuencia pensado para el enlace de telemetría y comando y control (C2, *Command and Control*) de un vehículo aéreo no tripulado (UAV), en la banda ISM de 902-928 MHz. Es el enlace más crítico de todo el vuelo: por ahí van los comandos de la estación terrena hacia la aeronave, y de regreso viene la telemetría (posición, altitud, velocidad, batería, etc.) en tiempo real. El protocolo que se usa para este intercambio, y que ya es el estándar en el mundo de los drones open-source, es MAVLink; lo usan estaciones terrenas como QGroundControl y Mission Planner para hablar con controladores de vuelo tipo PX4 o ArduPilot.

El sistema usa dos antenas trabajando en diversidad: una omnidireccional que da cobertura de respaldo en todas direcciones, y una direccional de mayor ganancia que apunta hacia la aeronave para estirar el alcance. Dos switchers conmutan entre ellas según el RSSI (la fuerza de señal), tanto en transmisión como en recepción. En tierra hay un amplificador bidireccional de 10 W, mientras que el transceptor a bordo trabaja a 1 W. Todo lo que llega se procesa y se muestra en una PC que hace de estación de control terrestre.

## Alcance

Lo que sí cubre el proyecto:

- Arquitectura del sistema: diagrama de bloques y por qué se eligió cada componente
- Presupuesto de enlace (link budget) para saber qué tan lejos puede llegar la señal en teoría
- Lógica de conmutación de antenas basada en RSSI, con un margen de histéresis para que no ande cambiando de antena a cada rato
- Selección del amplificador, antenas y transceptor, respetando lo que permite el IFT en la banda de 900 MHz
- Construcción física del sistema
- Pruebas de campo con una aeronave real, para checar que el enlace y la conmutación de antenas funcionen como se espera

Lo que no cubre:

- El enlace de video (el proyecto es solo telemetría/C2)

## Revisión de literatura

**Marco regulatorio.** En México la banda de 902-928 MHz es espectro de uso libre según el IFT, pero con límites: 1 W de potencia máxima entregada a la antena, 6 dBi de ganancia máxima si la antena es direccional, y una PIRE máxima de 4 W. Si la antena direccional tiene más ganancia de 6 dBi, hay que bajarle esa misma cantidad a la potencia de entrada. Estos números son los que van a limitar cuánto se le puede meter al amplificador y qué antenas se pueden usar.

**Protocolo de comunicación.** MAVLink es un protocolo binario ligero, pensado justo para enlaces de poco ancho de banda como el de radio. Define un montón de tipos de mensaje ya estandarizados para telemetría y comandos, y no depende de si el enlace es WiFi, radio de 900 MHz o lo que sea.

**Fundamentos de enlace de RF.** El link budget se calcula con la ecuación de Friis, que dice cuánta potencia le llega al receptor tomando en cuenta la potencia transmitida, las ganancias de las antenas y lo que se pierde por la distancia y la frecuencia. Es lo básico de cualquier curso de comunicaciones por RF para saber hasta dónde puede llegar un enlace.

**Diversidad de antena.** La idea de usar más de una antena para no perder la señal por desvanecimiento viene desde esquemas clásicos como el de Alamouti, que mostró que con solo dos antenas transmisoras se puede lograr casi lo mismo que con sistemas de recepción mucho más complicados. Para que la conmutación entre antenas no ande brincando con cualquier fluctuación, la industria usa temporizadores y umbrales de histéresis — eso evita que el switcher se la pase cambiando de antena y termine empeorando el enlace en vez de ayudarlo. En drones específicamente, todavía hay investigación reciente sobre cómo aplicar mejor la diversidad de antena en enlaces con desvanecimiento.

## Estado del arte

Tres radios comerciales de 900 MHz que ya se usan en drones de código abierto, para comparar:

| Sistema | Banda | Potencia | Diversidad de antena | Notas |
|---|---|---|---|---|
| Microhard P900 | 902-928 MHz | 100 mW – 1 W | No nativa | FHSS, comunicación MAVLink |
| RFD900x | 902-928 MHz | Hasta 1 W (pasos de 1 dB) | Sí, automática (en tiempo real, a nivel de paquete) | Alcance >40 km, RSSI vía MAVLink |
| Digi XBee-Pro 900HP | 902-928 MHz | Hasta 250 mW | No nativa | Orientado a redes de sensores (DigiMesh) |

El RFD900x ya trae diversidad de antena automática de fábrica, pero en los tres casos la potencia se ajusta a mano o en pasos fijos, no de forma dinámica. Ahí está el punto diferente de TeleCom-UAV: que el amplificador ajuste su potencia solo, según el RSSI que va reportando el enlace, y que el switcher de antenas use histéresis para no estar cambiando a cada rato. En teoría eso debería traducirse en menos consumo de energía y un enlace más estable que el de estos sistemas comerciales.

## Fuentes

- [Inventario de Bandas de Frecuencias Clasificadas como Espectro Libre (IFT)](https://www.ift.org.mx/sites/default/files/contenidogeneral/espectro-radioelectrico/inventariodebandasdefrecuenciasclasificadascomoespectrolibre-mayo2025.pdf)
- [Protocol Overview - MAVLink Guide](https://mavlink.io/en/about/overview.html)
- [Holybro Microhard P900 Radio - PX4 Guide](https://docs.px4.io/main/en/telemetry/holybro_microhard_p900_radio)
- [RFD900x and RFD868x Radio Modem Datasheet](https://files.rfdesign.com.au/Files/documents/RFD900x%20DataSheet%20V1.2.pdf)
- [XBee-PRO 900HP Product Datasheet - Digi International](https://www.digi.com/resources/library/data-sheets/ds_xbeepro900hp)
- [A Simple Transmit Diversity Technique for Wireless Communications (Alamouti, 1998)](https://www.ece.tufts.edu/ee/108/alamouti.pdf)

## Autor

Angy — Universidad Aeronáutica en Querétaro (UNAQ)
