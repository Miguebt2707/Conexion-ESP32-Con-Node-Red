
# Conexion-ESP32-Con-Node-Red

 LO QUE SE HIZO EN ESTA PRACTICA FUE EL REALIZAR UNA CONEXION DIRECTAMENTE EN LA RED PARA QUE LOGRARAN INTERACTUAR SENSORES QUE SE TIENEN EN UNA CONEXION Y VER LOS DATOS QUE ARROJAN EN UNOS DISPOSITIVOS PARECIDOS A UNOS TACOMETROS EN UNA INTERFAS EN INTERNET QUE CABE ACLARAR ES UN SISTEMA GRATUITO, PARA ESO SE TUVO QUE CONSEGUIR LA DIRECCION DE LA TERMINAL DE LA CUAL IBA A SALIR DEL WOKWI Y A LA QUE IBA A ENTRAR EL EL NODE RED, LOS DATOS QUE NOS ARROJAN O DEBERIAN ARROJARNOS SON LOS DE TEMPERATURA Y HUMEDAD YA QUE SE ESTA UTILIZANDO EL SENSOR DTH22
 
## LOS MATERIALES FUERON:

- WOKWI
- TARJETA ESP32
- SENSOR DTH22
  EN ESTE PUNTO SE PUEDE APRECIAR LAS CONEXIONES QUE SE HICIERON EN NUESTRO SISTEMA LA CUAL PERTENECE AL WOKWI
![](https://github.com/Miguebt2707/Conexion-ESP32-Con-Node-Red/blob/main/conexiones1.png?raw=true)
EN ESTA PARTE SE MUESTRA EL DIAGRAMA DE LAS NUVES QUE SE UTILIZARON PARA QUE EL WOKWI PUDIERA INTEACTUAR CON LA INTERFAS DEL NODE RED 
![](https://github.com/Miguebt2707/Conexion-ESP32-Con-Node-Red/blob/main/node-red.png?raw=true)
AQUI ES LA PARTE FUNDAMENTAL DEL PROGRAMA YA QUE EN ESTA PARTE SE COLOCA LA DIRECCION IP DEL SERVIDOR CON LA CUAL VA A ESTAR TRABAJANDO Y A SU VEZ ESTA DIRECCION VA A ESTAR REFLEJANDO LOS DATOS AL NODE RED  
![](https://github.com/Miguebt2707/Conexion-ESP32-Con-Node-Red/blob/main/miguebt1.png?raw=true)
Y FINALMENTE ASI NOS QUEDA NUESTRAS GRAFICAS CON NUESTROS PROYECTOS
![](https://github.com/Miguebt2707/Conexion-ESP32-Con-Node-Red/blob/main/graficasYtacometros.png?raw=true)


```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="Miguelon";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 3000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("MigueBT2707", output.c_str());
  }
}
```
