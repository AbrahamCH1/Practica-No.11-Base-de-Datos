# Practica-No.11-Base-de-Datos
Dentro de este repositorio se muestra la manera de programar una ESP32 con el sensor **DHT11**, el sensor **Ultrasonico** y además, que los datos obtenidos se vean reflejados en el programa **Node-RED** a través de una conexión WIFI. Además, se generará una base de datos con ayuda de un programa llamado **XAMPP**.
## Introducción
### Descripción
La **ESP32** se utiliza en un entorno de adquisición de datos, por lo cual en esta práctica utilizaremos un sensor **DHT22** para obtener el registro de temperatura y humedad y el sensor **Ultrasonico** para el registro de distancia. Dichos registros serán mostrados de manera visual en **Node-RED**. En base a los datos obtenidos, se realizará una base de datos. Para esta practica se hace uso del programa **Node-RED**, el programa **XAMPP** y de un simulador llamado [WOKWI](https://wokwi.com/projects/new/esp32).
## Material necesario
Para realizar esta práctica necesitas lo siguiente

- [WOKWI](https://wokwi.com/projects/new/esp32)
- Tarjeta ESP32
- sensor DHT11
- Sensor Ultrasonico
- Programa *Node-RED*
- Programa *XAMPP*

## Instrucciones
### Requisitos previos
Para poder hacer uso de este repositorio se requiere entrar a **Node-RED**, a **XAMPP** y a la plataforma de [WOKWI](https://wokwi.com/projects/new/esp32).
### Instrucciones de preparación del entorno
1. Abrir la terminal de programación y colocar el siguiente código:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 2;   //Pin digital 3 para el Echo del sensor
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "18.193.219.109";
String username_mqtt="educatronicosiot";
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
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
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
Serial.print("cm");
Serial.println();
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
    //doc["Anho"] = 2022;
    doc["DISTANCIA"] = String(d);
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("AbrahamDHT", output.c_str());
  }
}
```
2. Instalar la libreria de **ArduinoJson**, **PubSubClient** y **DHT sensor library for ESPx** como se muestra en la siguiente imagen.
![](https://github.com/AbrahamCH1/Practica-No.8-Node-RED-con-DHT22/blob/main/Captura%20de%20pantalla%20(322).png?raw=true)

3. Hacer la conexion del sensor **DHT11** y **Ultrasonico** con la **ESP32** como se muestra en la siguente imagen.
![](https://github.com/AbrahamCH1/Practica-No.9-Node-RED-con-Ultrasonico/blob/main/Captura%20de%20pantalla%20(331).png?raw=true)

4. Abrir **Node-RED** y hacer la conexión de las herramientas como se muestra en la imagen.
![](https://github.com/AbrahamCH1/Practica-No.9-Node-RED-con-Ultrasonico/blob/main/Captura%20de%20pantalla%20(332).png?raw=true)

5. Colocar el servidor y el topico dando *doble click* en el recuadro de *mqtt in**.
![](https://github.com/AbrahamCH1/Practica-No.8-Node-RED-con-DHT22/blob/main/Captura%20de%20pantalla%20(325).png?raw=true)

6. Agregar las lineas de código en los recuadros de *function*.
![](https://github.com/AbrahamCH1/Practica-No.8-Node-RED-con-DHT22/blob/main/Captura%20de%20pantalla%20(329).png?raw=true)
![](https://github.com/AbrahamCH1/Practica-No.8-Node-RED-con-DHT22/blob/main/Captura%20de%20pantalla%20(330).png?raw=true)
![](https://github.com/AbrahamCH1/Practica-No.9-Node-RED-con-Ultrasonico/blob/main/Captura%20de%20pantalla%20(333).png?raw=true)

8. Modificar los parametros de las gráficas de temperatura, humedad y distancia dando *doble click* en los recuadros.
![](https://github.com/AbrahamCH1/Practica-No.8-Node-RED-con-DHT22/blob/main/Captura%20de%20pantalla%20(326).png?raw=true)
![](https://github.com/AbrahamCH1/Practica-No.8-Node-RED-con-DHT22/blob/main/Captura%20de%20pantalla%20(327).png?raw=true)

9. Abrir el programa **XAMPP**.

10. Dar **clik* en los módulos **Apache* y **MySQL* y dar click en **admin*.
![]()

11. Crear una base de datos como la que se muestra en la imagen.
![]()

12. En Node-RED agregar la herramienta de **MySQL* y una función. Agregar el código y la información que se muestra en las imágenes.
![]()
![]()

### Instrucciones de operación
1. Iniciar simulador.
2. Visualizar los datos en el monitor serial.
3. Visualizar los datos en la interfaz de **Node-RED**.
5. Colocar los valores de temperatura y humedad dando *doble click* al sensor **DHT11**.
6. Colocar los valores de distancia dando *doble click* al sensor **Ultrasonico**.
7. Visualizar los datos en **XAMPP**. 
## Resultados
Cuando haya funcionado, se podrán observar los valores en **XAMPP**
![](https://github.com/AbrahamCH1/Practica-No.9-Node-RED-con-Ultrasonico/blob/main/Captura%20de%20pantalla%20(334).png?raw=true)
![]()


# Créditos
Desarrollado por Ing. Abraham Contreras Herrera
[GITHUB](https://github.com/AbrahamCH1)
