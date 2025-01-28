
//Ejemplo de código:
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






