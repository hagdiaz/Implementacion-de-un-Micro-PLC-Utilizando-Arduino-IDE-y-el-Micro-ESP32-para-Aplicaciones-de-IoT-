# Implementación de un Micro PLC Utilizando Arduino IDE y el Microcontrolador ESP32 para Aplicaciones de IoT

## Introducción
En el siglo XXI, el desarrollo de proyectos de ingeniería en automática es crucial. Este proyecto tiene como objetivo la implementación de un micro PLC utilizando Arduino IDE y el microcontrolador ESP32 para aplicaciones de IoT. Este micro PLC se instalará en habitaciones hoteleras o incluso en casas de cultivo, donde se requiere la automatización de máquinas de riego de pivote central, entre otras aplicaciones.

## Descripción del Proyecto
### Empleo de la Placa LilyGo T3 S3
En el contexto del control de un entorno IoT para una habitación hotelera, se han utilizado sensores de temperatura, humedad, movimiento y apertura de puertas y ventanas para monitorear y controlar el ambiente de la habitación.

### Control de Relés
Mediante la implementación de relés, se manejan dispositivos como el aire acondicionado y las luces. El sistema es capaz de:
1. Mantener una temperatura óptima cuando no hay presencia en la habitación.
2. Apagar el aire acondicionado si se detecta que las ventanas o puertas están abiertas durante un tiempo prolongado.
3. Encender las luces automáticamente al detectar presencia en la habitación.

## Código de Implementación
A continuación se presenta un ejemplo de código en C++ que utiliza la placa LilyGo T3 S3 para integrar los sensores mencionados y controlar los relés según las condiciones detectadas.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

// Configuración de WiFi
const char* ssid = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";

// Configuración de MQTT
const char* mqttServer = "MQTT_BROKER_IP";
const int mqttPort = 1883;
const char* mqttUser = "MQTT_USER";
const char* mqttPassword = "MQTT_PASSWORD";

WiFiClient espClient;
PubSubClient client(espClient);

// Configuración del sensor DHT
#define DHTPIN 4          // Pin donde está conectado el DHT
#define DHTTYPE DHT11     // Tipo de sensor DHT (DHT11 o DHT22)
DHT dht(DHTPIN, DHTTYPE);

// Pines para relés
#define RELAY_AC 5        // Relé para aire acondicionado
#define RELAY_LIGHT 6     // Relé para luces
#define SENSOR_DOOR 7     // Sensor de puerta (abierto/cerrado)
#define SENSOR_WINDOW 8   // Sensor de ventana (abierto/cerrado)

// Variables para almacenar datos
float temperature;
float humidity;
bool presenceDetected = false; // Variable para detectar presencia

void setup() {
    Serial.begin(115200);
    dht.begin();
    pinMode(RELAY_AC, OUTPUT);
    pinMode(RELAY_LIGHT, OUTPUT);
    pinMode(SENSOR_DOOR, INPUT);
    pinMode(SENSOR_WINDOW, INPUT);
    
    setup_wifi();
    client.setServer(mqttServer, mqttPort);
}

void setup_wifi() {
    delay(10);
    Serial.println();
    Serial.print("Conectando a ");
    Serial.println(ssid);
    
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    
    Serial.println("");
    Serial.println("Conectado a WiFi");
}

void reconnect() {
    while (!client.connected()) {
        Serial.print("Intentando conexión MQTT...");
        if (client.connect("LilyGo_T3_S3", mqttUser, mqttPassword)) {
            Serial.println("conectado");
            client.subscribe("hotel/room/control");
        } else {
            Serial.print("falló, rc=");
            Serial.print(client.state());
            delay(2000);
        }
    }
}

void readSensors() {
    humidity = dht.readHumidity();
    temperature = dht.readTemperature();

    if (isnan(humidity) || isnan(temperature)) {
        Serial.println("Error al leer el sensor DHT");
        return;
    }

    // Controlar aire acondicionado
    if (presenceDetected) {
        if (digitalRead(SENSOR_DOOR) == HIGH || digitalRead(SENSOR_WINDOW) == HIGH) {
            digitalWrite(RELAY_AC, LOW); // Apagar aire acondicionado
            Serial.println("Aire acondicionado apagado por ventana/puerta abierta.");
        } else {
            if (temperature > 24.0) { // Temperatura de confort
                digitalWrite(RELAY_AC, HIGH); // Encender aire acondicionado
                Serial.println("Aire acondicionado encendido.");
            } else {
                digitalWrite(RELAY_AC, LOW); // Apagar aire acondicionado
                Serial.println("Aire acondicionado apagado.");
            }
        }
    } else {
        digitalWrite(RELAY_AC, LOW); // Apagar aire si no hay presencia
        Serial.println("Aire acondicionado apagado por falta de presencia.");
    }

    // Controlar luces
    if (presenceDetected) {
        digitalWrite(RELAY_LIGHT, HIGH); // Encender luces
        Serial.println("Luces encendidas.");
    } else {
        digitalWrite(RELAY_LIGHT, LOW); // Apagar luces
        Serial.println("Luces apagadas.");
    }
}

void loop() {
    if (!client.connected()) {
        reconnect();
    }
    
    client.loop();
    
    readSensors(); // Leer y controlar cada vez que se ejecuta el loop

    delay(5000); // Esperar 5 segundos entre lecturas
}

###Descripción de la PCB y componentes electrónicos, sus principales datos técnicos:
 
La placa LilyGo T3 S3 contiene el microcontrolador ESP32, que actúa como el cerebro del micro PLC. En la imagen se puede apreciar el pinout del dispositivo para su posterior empleo.
 
Partiendo de los sistemas de entrada/salida especificados, se enumeran 7 entradas y 5 salidas. Estos bloques de entrada-salida comprenden dos diseños típicos que se repiten para ambos procesos. La Figura 3.1 representa el canal de entrada, simulado en Proteus. Dado que se tendrán 7 canales en el circuito electrónico de la PCB, cada canal está equipado con optoacopladores para desacoplar ruidos o aislar las variaciones de voltaje que puedan surgir debido a interferencias. La señal de entrada que llega es de 24VDC. Dado que es una señal de 24 voltios, se introduce una resistencia de 6.8K ohm para regular la corriente que llega al optoacoplador. Además, se incorpora un fusible de protección de 50mA y un LED para notificar visualmente que la entrada está activada. Este circuito se conecta hasta un pin de entrada previamente descrito hasta la T3 S3. 

La Figura 3.2 representa el canal de salida simulado en Proteus, y se replicaron 5 canales idénticos en el circuito electrónico de la PCB. Este canal está equipado con un relé que se utiliza para accionar contactores magnéticos responsables del control de elementos dentro de la habitación de hotel. Para lograr esta acción, se emplea un optoacoplador que se activa cuando el pin de salida envía una señal de 3.3 voltios. Del otro lado, permite el paso de los 24VDC hasta el transistor, el cual se excita y permite el flujo de corriente a través del bobinado del relé. Con el objetivo de proteger el circuito, se incorpora el diodo D2, que cumple dos funciones cruciales: protección contra retroalimentación del relé debido a la energía almacenada en la bobina del mismo y supresión de picos de voltaje. Además, se utiliza un diodo LED para indicar el estado de encendido del magnético en relación con el circuito de salida. Se incluyen resistencias para limitar altas corrientes por el circuito, y se agrega un fusible que regula la entrada a toda la placa hasta 2A de corriente. Este circuito se conecta hasta un pin de salida previamente descrito en la Tabla 2.2 hasta la T3 S3.

 

La fuente utilizada en este diseño es una de 24 VDC proveniente del exterior, que alimenta todo el bloque de salida, incluyendo los relés de potencia y el regulador de voltaje MEZD71202A-G. Este regulador se utiliza para estabilizar el voltaje de entrada de 24VDC, aspecto crucial para el funcionamiento de los optoacopladores U1 hasta U7 y la alimentación de la placa IoT T3 S3. El regulador acepta un rango de voltaje desde 4.5VDC hasta 24VDC y suministra una salida fija de 5VDC, con una capacidad máxima de corriente de hasta 2A. Se monta en la PCB a través de orificios (TH) y se conecta mediante 3 pines, como se ilustra en la Figura 3.3. En la placa, el regulador está acompañado de tres capacitores que contribuyen al completo filtrado de la entrada y salida de corriente del regulador, así se suprime la gran mayoría de los ruidos eléctricos. Con el fin de garantizar el buen funcionamiento de la placa IoT T3 S3, alimentada por este regulador, se añadió un diodo en la entrada para asegurar que la polaridad siempre sea correcta.
 

###Diseño en KiCAD:
El esquemático presentado fue diseñado en la zona de trabajo de KiCAD y sirve como base para la creación del PCB. En él, se pueden identificar todos los bloques de entrada y salida previamente especificados, conectados a sus respectivos pines como se describe en la Tabla 2.1 y Tabla 2.2. Además de estos bloques, se incluyen bloques adicionales. El primer bloque, etiquetado como "LilyGo TTGO Lora T3 S3 ESP32 Pin Conector", representa las salidas y entradas de los pines de conexión de la T3 S3 en la placa. El siguiente bloque, llamado Power input, se refiere a la entrada de 24VDC con su correspondiente fusible reiniciable de 24VDC y 2A, diseñado para proteger el circuito. También se presenta el bloque Voltaje Regulator, explicado anteriormente, incluyendo el conector J15 para alimentar la placa IoT desde ese puerto. También aparece un bloque con los protocolos de comunicación I2C y UART, para instalar diferentes sensores al PCB.

###Conclusiones:
Al terminar con la realización de este trabajo final de Ingeniería Automática, se ha podido concluir la importancia del estudio de otras posibilidades de creación y diseño de PCB con la funcionalidad de micro PLC para ser empleados en una amplia gama de servicios donde se necesite la supervisión y control de tareas y procesos, garantizando no solo una calidad excelente sino también un precio mucho más módico en comparación a los PLC que ofrecen firmas reconocidas en el mercado internacional. Se han afianzado más las habilidades en el uso de la herramienta KiCAD para el diseño de placas electrónicas con prestaciones específicas para cada proyecto que se proponga realizar, y se ha aprendido más sobre el entorno de desarrollo integrado de Arduino y sobre la programación específicamente de los microcontroladores ESP32.

###Recomendaciones
1.	Expansión de Funcionalidades: Se recomienda expandir el sistema para incluir más funcionalidades, como la integración con sistemas de gestión hotelera. Esto permitiría una administración centralizada y eficiente de las habitaciones del hotel.
2.	Notificaciones y Alertas: Implementar la capacidad de enviar notificaciones y alertas al personal del hotel sobre el estado de las habitaciones. Esto podría incluir alertas sobre ventanas o puertas abiertas, fallos en el sistema de aire acondicionado, o cualquier otra condición que requiera atención inmediata.
3.	Pruebas y Validación: Realizar pruebas exhaustivas y validación del sistema en diferentes entornos para asegurar su robustez y fiabilidad. Esto incluye pruebas de estrés, pruebas de compatibilidad y pruebas de seguridad.
4.	Documentación y Soporte: Desarrollar una documentación detallada y proporcionar soporte técnico para facilitar la implementación y el mantenimiento del sistema.
5.	Actualizaciones y Mantenimiento: Asegurar que el sistema se mantenga actualizado y en optimo funcionamiento.





