# NODE-RED-CON-LOS-DOS-SENSORES
PROGRAMACION CON UN SENSOR DE TEMPERATURA Y HUMEDAD, Y UN SENSOR ULTRASONICO, ATRAVES DE UN SIMULADOR (WOKWI) Y UNSERVIDOR (NODE-RED).
## INSTRUCCIONES
1. Abrimos **WOKWI** y colocamos la siguiente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 0;   
const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "35.172.255.228";
String username_mqtt="RAUL CASTANEDA";
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
  delay(1000);          //Hacemos una pausa de 100ms
delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
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
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"]= String(d);
     doc["nombre"] = "RAUL CASTANEDA";
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("CUERNAVACA", output.c_str());
  }
}
```

2. Abrimos **CMD** y enseguida colocamos **node-red**, seguido de **enter**:
![](https://raw.githubusercontent.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/79c8785182c819f0f1be52018963d8dfa88fb32c/Captura%20de%20pantalla%202024-12-19%20134556.png)

3. En el explorador colocamos **localhost:1880** para abrir **node-red**:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20134844.png?raw=true)

4. Agregamos las siguientes 3 librerias:
- **ArduinoJson**
- **PubSubClient**
- **DHT sensor library for ESPx**

5. Cambiamos usuario y contraseña en el código que pusimos:
!][(https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20135514.png?raw=true)

6. En el buscador colocamos **www.emqx.com** y copiamos el broker que aparece:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-18%20154658.png?raw=true)

7. Abrimos **CDM** y escribimos **nslookup** y pegamos el broker copiado en el paso anterior, enseguida damos enter:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20140327.png?raw=true)

8. Copiamos el **adress** que aparece en el paso anterior y lo pegamos en el código:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20140610.png?raw=true)

9. Conectamos los componentes en **WOKWI**:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20140902.png?raw=true)

10. En el código agregmos otro dato a mostrar en la simulación, en este caso sería **nuestros nombre** y el **topico**:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20141156.png?raw=true)

11. Nos dirigimos a **NODE-RED** que abrimos anteriormente, agregaremos el bloque **mqtt in** y lo configuramos, tanto con el **topico** como con el **mqtt server** copiado anteriormente:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20141631.png?raw=true)

12. Colocamos el bloque **json** y lo cofiguramos seleccionando la siguiente opción:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20141929.png?raw=true)

13. Colocamos el bloque **debug** que nos dira todo lo que hace y lo conectamos a los bloques anteriores:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20142312.png?raw=true)

14. Después de conectar estos 3 bloques le damos click en **deploy** y en la simulación le damos **play**, y nos mostrara los datos tanto en la **ESP** como en **node-red**:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20143728.png?raw=true)
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20143852.png?raw=true)
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-19%20143829.png?raw=true)

15. Agregamos 2 bloques de funciones, en uno configuramos con el código de temperatura y en el otro con el código de humedad, y los conectamos al bloque **json**:
![](https://github.com/RaulCasS/NODE-RED-CON-LOS-DOS-SENSORES/blob/main/Captura%20de%20pantalla%202024-12-18%20162107.png?raw=true)
![]()
