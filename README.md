# Base-de-datos
Este repositorio muestra como podemos programar una ESP32 con el sensor HC-SR04 para medir distancia,de temperatura y humedad con DHT22, posteriormente se visualizara en un dashboard en tiempo real los datos simulados y en un host online
## Introducción

### Descripción

La ```Esp32``` la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos la platafoma  (```node-red```) para adquirir la temperatura y humedad del entorno, tambien la Distancia;esta practica se usara un simulador llamado [WOKWI](https://https://wokwi.com/). y plataforma [node-red] (http://localhost:1880/ui/#!/0?socketid=GastYhmOZGDVaQArAAAN)  para realizar la visualización en tiempo real de los datos simulados y una pagina host online para visualizar los datos


## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- [node-red] (http://localhost:1880/ui/#!/0?socketid=GastYhmOZGDVaQArAAAN)
- [node-red] (http://localhost:1880/#flow/33551a14c9f5e89f)
- DHT22
- HC-SR04
- host online
- Esp32



## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/)
y as la plataforma  [node-red] (http://localhost:1880/ui/#!/0?socketid=GastYhmOZGDVaQArAAAN)()
 [node-red] (http://localhost:1880/#flow/33551a14c9f5e89f)


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"


const int DHT_PIN = 15;
DHTesp dhtSensor;

const int Trigger = 13;   //Pin digital 2 para el Trigger del sensor
const int Echo = 12; 
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="serg2784";
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
dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
   pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);
  
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  
}

void loop() {

TempAndHumidity  data = dhtSensor.getTempAndHumidity();
 
long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("m");
  Serial.println();
  delay(1000);       

delay(1000);

  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    doc["DISTANCIA"] = String(d);
     doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("ElUriel/Programador", output.c_str());
  }
}

```
2. Hacer la conexion de HC-SR04 DHT22 con la ESP32 como se muestra en la siguente imagen.

![](https://github.com/ser2784/Base-de-datos/blob/main/Base%20de%20datos%201.png)

3. Agregaremos los siguientes bloques en nuestro espacio de trabajo de Nod Red. (mqtt in, json, funcion, mysql, gauge,chart).

![](https://github.com/ser2784/Base-de-datos/blob/main/Base%20de%20Datos%2021.png)


### Instrucciónes de operación
1. Iniciar simulador.
2. Para poder ver los datos en tiempo real tienes que darle al cuadro superior derecho (cuadro con una flecha apuntando hacia arriba-derecha) para que te mande a tu dashboard 

## Resultados

Cuando haya funcionado, verás los valores dentro del monitor serial como se muestra en la siguente imagen.
## Resultados y Graficas 
 
 Valores dashboard

![](https://github.com/ser2784/Base-de-datos/blob/main/Base%20de%20Datos%203.png)

![](https://github.com/ser2784/Base-de-datos/blob/main/Base%20de%20Datos%204.png)

![]()


## Evidencias

[Video de Youtube](no contamos con el video ))


# Créditos

Desarrollado por Ing. Sergio RB

- [GitHub](https://github.com/ser2784)