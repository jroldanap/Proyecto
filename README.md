# Proyecto ESP32 con Node-Red.
Este repositorio muestra como podemos programar una ESP32 con relevadores para simular motores, realizar un proceso industrial de liquidos para llenado mezclado y vaciado de un liquido.

## Introducción

### Descripción

La Esp32 la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos relevadores para simular motores, realizar un proceso industrial de liquidos para llenado mezclado y vaciado de un liquido, con un boton que se vera reflejado en Node-Red para el mezclado ; Cabe aclarar que esta practica se usara un simulador llamado [WOKWI](https://https://wokwi.com/).


## Material Necesario

Para realizar esta practica necesitas lo siguiente

- [WOKWI](https://https://wokwi.com/)
- 1 sensor ultrasonico
- 2 Relevador con led.
- 1 resistencia.
- 1 led rojo.
- Pantalla LCD 16X32.
- Programa Node-Red (previamente instalado en https://github.com/DiegoJm10/Node-red-instalacion)



## Instrucciones

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://https://wokwi.com/).


### Instrucciones de preparación de entorno 

1. Abrir la terminal de programación y colocar la siguente programación:

```
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
#define BUILTIN_LED 2
#include <LiquidCrystal_I2C.h>
#define I2C_ADDR    0x27
#define LCD_COLUMNS 20
#define LCD_LINES   4

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqttServer = "44.195.202.69";
const int mqttPort = 1883;
const char* mqttUser = "EqJJU-1";
const char* mqttPassword = "123456";
const char* mqttTopic = "EqJJU-1";
WiFiClient espClient;
PubSubClient client(espClient);
LiquidCrystal_I2C lcd(I2C_ADDR, LCD_COLUMNS, LCD_LINES);

const int trigPin = 13;
const int echoPin = 12;
const int ledproce1 = 5 ;
int ledPin = 19; // Pin del LED
int ledPin2 = 18; // Pin del LED2
long duration;
int distance;
int safetyDistance;
const int Trigger = 13;   //Pin digital 2 para el Trigger del sensor
const int Echo = 12;   //Pin digital 3 para el Echo del sensor
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Conectando a: ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("Conectado a la red WiFi");
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Intentando conexión MQTT...");
    if (client.connect("ESP32Client", mqttUser, mqttPassword)) {
      Serial.println("Conectado");
      client.subscribe(mqttTopic);
    } else {
      Serial.print("Error de conexión, rc=");
      Serial.print(client.state());
      Serial.println(" Intentando de nuevo en 5 segundos");
      delay(5000);
    }
  }
}
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Mensaje recibido: [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  if (strcmp(topic, mqttTopic) == 0) {
    if ((char)payload[0] == '1') {
      digitalWrite(ledPin, HIGH);
    } else {
      digitalWrite(ledPin, LOW);
    }
  }
  
  }
void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  pinMode(ledPin, OUTPUT);
  pinMode(ledPin2, OUTPUT);
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqttServer, mqttPort);
  client.setCallback(callback);
    pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
    lcd.init();
  lcd.backlight();
  pinMode(ledproce1, OUTPUT);
   Serial.begin(115200);
  Serial.println("Hello, ESP32!");

  lcd.setCursor(0, 0);
  lcd.print("  Bienvenidos    ");
  lcd.setCursor(0, 1);  
  lcd.print(" Al mixer 3000     " );
delay (4000);

lcd.setCursor(0, 0);
  lcd.print("   Iniciando     ");
  lcd.setCursor(0, 1);  
  lcd.print("      ...          " );
delay (2000);
lcd.setCursor(0, 0);
  lcd.print("  Llenando  ");
  lcd.setCursor(0, 1);  
  lcd.print("      ...    " );
delay (5000);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
  delay(1000);
  long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
    lcd.setCursor(0, 0);
  lcd.print("Nivel en tanque:        " );
  lcd.setCursor(0, 1);
  lcd.print(       String(d) + " L            "     );
delay(1500);
if (safetyDistance >= 5 && safetyDistance<=350){

  digitalWrite(ledproce1, HIGH);
  lcd.setCursor(0, 0);
  lcd.print("Nivel insuficiente " );
  lcd.setCursor(0, 1);
  lcd.print(   "     ...     "     );
  delay(1000);
  lcd.setCursor(0, 0);
  lcd.print("Nivel en tanque:        " );
  lcd.setCursor(0, 1);
  lcd.print(       String(d) + " L            "     );
 
}
if (safetyDistance >= 350 && safetyDistance<=400){

  digitalWrite(ledproce1, LOW);
  lcd.setCursor(0, 0);
  lcd.print(" Tanque  Lleno   " );
  lcd.setCursor(0, 1);
  lcd.print(   "      ...        "     );
  delay(1000);
  lcd.setCursor(0, 0);
  lcd.print("Nivel en tanque:        " );
  lcd.setCursor(0, 1);
  lcd.print(       String(d) + " L            "     );
 
}
 safetyDistance = d;

 if (digitalRead (ledPin) == 1)
{
safetyDistance==1;
lcd.setCursor(0, 0);
  lcd.print("  Mezclando      " );
  lcd.setCursor(0, 1);
  lcd.print(   "              "  );
delay(10000);
digitalWrite(ledPin2, HIGH);
}
if (digitalRead (ledPin2) == 1)
{
  digitalWrite(ledPin, LOW);
  digitalWrite(ledproce1, 0);
  lcd.setCursor(0, 0);
  lcd.print("     Vaciando        " );
  lcd.setCursor(0, 1);
  lcd.print(  "      ...           "  );
  delay(10000);
  lcd.setCursor(0, 0);
  lcd.print("  Fin del proceso      " );
  lcd.setCursor(0, 1);
  lcd.print(  "    Gracias    "  );
  delay(15000);
  digitalWrite(ledPin2, LOW);
  delay(100);
}
Serial.print("Litros: ");
Serial.println(d);
delay (1000);

String output;
Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Equipo-/1", output.c_str());
}
```

2. Instalar las librerias de *PubSubClient*, *LiquidCrystal i2c*, *ArduinoJson* como se muestra en la siguente imagen.

![](https://github.com/jroldanap/Proyecto/blob/main/librerias.png?raw=true)

3. Hacer la conexion de los *relevadores la pantalla lcd, el sensor ultrasonico, el led y la resistencia*, con la *ESP32* como se muestra en la siguente imagen.

![](https://github.com/jroldanap/Proyecto/blob/main/conexion.png?raw=true)

4. Poner el bloque switch en el programa Node-Red, cambiar el topic a *mezclado*, cambiar *on payload* en la flecha a *string*, agregar el num 1, cambiar *off payload* en la flecha a *string*, agregar el num 0.

![](https://github.com/jroldanap/Basededatos/blob/main/swit.png?raw=true)



5. Añadir el bloque *mqtt out* y el server a  *44.195.202.69* y modificar el topic a *EqJJU-1*.

![](https://github.com/jroldanap/Basededatos/blob/main/out.png?raw=true)

6. Añadir el bloque *mqtt in* y el server a  *44.195.202.69* y modificar el topic a *Equipo-/1*.

![](https://github.com/jroldanap/Basededatos/blob/main/in.png?raw=true)

7. Añadir el bloque debug y poner la configuracion como se muestra en la imagen..

![](https://github.com/jroldanap/Basededatos/blob/main/debug.png?raw=true)

8. Conectar los bloques entre si de la siguiente forma.

![](https://github.com/jroldanap/Proyecto/blob/main/node.png?raw=true)


9. Dar click en flecha en rojo para ver el boton, como se muestra en la imagen.

![](https://github.com/jroldanap/esp32led/blob/main/boton.png?raw=true)




### Instrucciónes de operación

1. Iniciar simulador dando click en el boton verde de play.
2. Visualizar los datos en el monitor serial y los procesos en la pantalla lcd.
3. Dar doble click en el sensor ultrasonico y poner la distancia a 400.
3. Presionar en el boton de Node-Red *mezclado*.
4. Visualizar el procesos mezclado y vaciado.

## Resultados

Cuando haya funcionado, al poner la distancia en 400 en el sensor ultrasonico y presionar el boton mezclado en  node red, se mezclara y despues se vaciara.

![](https://github.com/jroldanap/Basededatos/blob/main/nivel%20insuficiente.jpeg?raw=true)

![](https://github.com/jroldanap/Proyecto/blob/main/tanque%20lleno.png?raw=true)

![](https://github.com/jroldanap/Proyecto/blob/main/mix.jpeg?raw=true)

![](https://github.com/jroldanap/Proyecto/blob/main/empty.jpeg?raw=true)



## Evidencias de programa corriendo

![](https://github.com/jroldanap/Basededatos/blob/main/nivel%20insuficiente.jpeg?raw=true)

![](https://github.com/jroldanap/Proyecto/blob/main/tanque%20lleno.png?raw=true)

![](https://github.com/jroldanap/Proyecto/blob/main/mix.jpeg?raw=true)

![](https://github.com/jroldanap/Proyecto/blob/main/empty.jpeg?raw=true)



# Créditos

Desarrollado por Jorge Esteban Lopez Quiroz , Uriel Mastache Juarez y Jorge Alberto Roldan Aponte.

- Jorge Esteban Lopez Quiroz [GitHub](https://github.com/jorgelopezquiroz)
- Uriel Mastache Juarez [GitHub](https://github.com/UrielMastache)
- Jorge Alberto Roldan Aponte  [GitHub](https://github.com/jroldanap)