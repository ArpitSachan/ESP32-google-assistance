#include <WiFi.h>
#include "Adafruit_MQTT.h"
#include "Adafruit_MQTT_Client.h"

#define WLAN_SSID       "DESKTOP-DGV7NJ1 1312"
#define WLAN_PASS       "1A&2194t" 
#define AIO_SERVER      "io.adafruit.com"
#define AIO_SERVERPORT  1883                   
#define AIO_USERNAME    "papameraarpit"           
#define AIO_KEY         "a6c68061b3a54b28a6ff062f0dbf8745"  

WiFiClient client;
Adafruit_MQTT_Client mqtt(&client, AIO_SERVER, AIO_SERVERPORT, AIO_USERNAME, AIO_KEY);
Adafruit_MQTT_Subscribe Light1 = Adafruit_MQTT_Subscribe(&mqtt, AIO_USERNAME"/feeds/onoff");
Adafruit_MQTT_Subscribe Light2 = Adafruit_MQTT_Subscribe(&mqtt, AIO_USERNAME"/feeds/zero");
Adafruit_MQTT_Subscribe Light3 = Adafruit_MQTT_Subscribe(&mqtt, AIO_USERNAME"/feeds/one");
void MQTT_connect();
void setup() {
  Serial.begin(115200);
  pinMode(14, OUTPUT);
  pinMode(12, OUTPUT);
  pinMode(13, OUTPUT);
  Serial.println(); Serial.println();
  Serial.print("Connecting to ");
  Serial.println(WLAN_SSID);
 
  WiFi.begin(WLAN_SSID, WLAN_PASS);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  Serial.println("WiFi connected");
  Serial.println("IP address: "); 
  Serial.println(WiFi.localIP());
  
  mqtt.subscribe(&Light1);
   mqtt.subscribe(&Light2);
    mqtt.subscribe(&Light3);
}

void loop() {
 MQTT_connect();
  
  Adafruit_MQTT_Subscribe *subscription;
  while ((subscription = mqtt.readSubscription(5000))) {
    
     if (subscription == &Light1) {
      Serial.print(F("Got: "));
      Serial.println((char *)Light1.lastread);
       if (strcmp((char *)Light1.lastread, "ON") == 0) {
        digitalWrite(14, HIGH); 
      }
      if (strcmp((char *)Light1.lastread, "OFF") == 0) {
        digitalWrite(14, LOW); 
      }
    }
    if (subscription == &Light2) {
      Serial.print(F("Got: "));
      Serial.println((char *)Light2.lastread);
       if (strcmp((char *)Light2.lastread, "ON") == 0) {
        digitalWrite(12, HIGH); 
      }
      if (strcmp((char *)Light2.lastread, "OFF") == 0) {
        digitalWrite(12, LOW); 
      }
    }
     if (subscription == &Light3) {
      Serial.print(F("Got: "));
      Serial.println((char *)Light3.lastread);
       if (strcmp((char *)Light3.lastread, "ON") == 0) {
        digitalWrite(13, HIGH); 
      }
      if (strcmp((char *)Light3.lastread, "OFF") == 0) {
        digitalWrite(13, LOW); 
      }
    }
      }
}

void MQTT_connect() {
  int8_t ret;
  if (mqtt.connected()) {
    return;
  }

  Serial.print("Connecting to MQTT... ");

  uint8_t retries = 3;
  
  while ((ret = mqtt.connect()) != 0) {
    Serial.println(mqtt.connectErrorString(ret));
    Serial.println("Retrying MQTT connection in 5 seconds...");
    mqtt.disconnect();
    delay(5000);
    retries--;
    if (retries == 0) {
      while (1);
    }
  }
  Serial.println("MQTT Connected!"); // DONE
  
}