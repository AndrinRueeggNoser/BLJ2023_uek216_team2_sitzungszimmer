
# Arduino Code
```js

//Includments

#include <WiFi.h>
#include <PubSubClient.h>


//Zugriff auf Internet und den mqtt server damit es mit dem Noser Young Netwerk verbunden ist.
const char* ssid = "GuestWLANPortal"; 
const char* password = ""; 
const char* mqtt_server = "noseryoung.ddns.net"; 

WiFiClient espClient;
PubSubClient client(espClient);

// Die GPIO pins definieren

#define TRIG_PIN 16
#define ECHO_PIN 17

#define RED_PIN 12
#define GREEN_PIN 13
#define BLUE_PIN 14

unsigned long lastColorChangeTime = 0; 
bool isGreen = false; 

//Set up für WiFi

void setup_wifi() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
}

void setup() {
  Serial.begin(9600);

  
  setup_wifi();


  client.setServer(mqtt_server, 1983); 

 
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);

  lastColorChangeTime = millis();
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP32Client")) {
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }

//Loop Teil
  client.loop();

  
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  long duration = pulseIn(ECHO_PIN, HIGH);
  float distance = duration * 0.034 / 2;


  bool shouldGreen = distance > 8.5;
  if (isGreen != shouldGreen) {
    lastColorChangeTime = millis();
    isGreen = shouldGreen;

    const char* roomStatus = isGreen ? "Room Is Not In Use" : "Room Is In Use";
    client.publish("zuerich/Sitzungszimmer/status", roomStatus);
  }


  if (isGreen) {
    digitalWrite(RED_PIN, HIGH);
    digitalWrite(GREEN_PIN, LOW);
    digitalWrite(BLUE_PIN, LOW);
  } else {
    digitalWrite(RED_PIN, LOW);  
    digitalWrite(GREEN_PIN, HIGH);
    digitalWrite(BLUE_PIN, LOW);  
  }

 // MQTT Topic erstellen und am richtigen Ort (unter zuerich) zuteilen
  char payload[50];
  snprintf(payload, sizeof(payload), "%f", distance);
  client.publish("zuerich/Sitzungszimmer/in", payload);

  delay(100); 
}
```
