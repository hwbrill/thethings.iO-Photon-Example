# Photon Particle MQTT

Photon particle example to connect to http://thethings.iO. 
By using a button connected to the digital port D1, the particle sends a random value between 0 and 9 once the button is pressed. 
Here's the fritzing diagram of the connections:
![alt tag] (http://blog.thethings.io/wp-content/uploads/2016/04/Photon-thethingsiO-2.png)

## Installation

First navigate to particle build and create a new project.
![alt tag] (http://blog.thethings.io/wp-content/uploads/2016/04/thethings-particle-ide.png)

Clone this repository and copy all the code from the .ino file to the Particle IDE.

## Code Explanation

Include de header for the MQTT library.
```
#include "MQTT/MQTT.h"
```
Declare and implement the callback in ordre to receive messages from thethings.io.
```
void callback(char* topic, byte* payload, unsigned int length) {
    // handle message arrived
    String text = "";
    for (int i = 0; i < length; i++)
        text.concat((char)payload[i]);
    Serial.println(text);
}
```
Instantiate the mqtt client
``` 
MQTT client("mqtt.thethings.io", 1883, callback);
```
Change TOKEN with your token. Change the WIFI_SSID with your SSID. Change the WIFI_PASSWORD with your wifi password. 
```
int a;
String TOKEN = "YOUR TOKEN";
String TOPIC = "v2/things/"+TOKEN;
String WIFI_SSID = "yourwifiSSID";
String WIFI_PASSWORD = "yourpassword";
```
In the setup function, D1 pin is set to INPUT. Then the particle is set to connect to your wifi and to the thethings.io mqtt server. 
```
void setup() {
    pinMode(D1,INPUT_PULLDOWN);
    digitalWrite(D1, LOW);
    WiFi.on();
    WiFi.setCredentials(WIFI_SSID, WIFI_PASSWORD); 
    while(!WiFi.ready()) {
        Serial.print(".");
    }
    Serial.println("");
    Serial.print("Connected ssid");
    while (!client.connect("photonclient")) {
        Serial.println("Could not subscribe, retrying...");
    }
```
Later, it subscribes to the same topic were it publishes the messages, in order to receive messages from thethings.io. 
```
    client.subscribe(TOPIC);
}
````
In loop function, if the button is pressed, a random value between 0 and 9 is generated. A message is created with a resource key named randValue and with "a" value. Then the message is published to thethings.io. Then it assures that the mqtt client is still connected.
```
void loop() {
    if (digitalRead(D1) == HIGH) {
        a = random(0,10);
        String message = "{\"values\":[{\"key\":\"randValue\",\"value\":"+String(a, DEC)+"}]}";
        client.publish(TOPIC, message);
        delay(200);         // ADDED A LITTLE BIT DELAY TO REDUCE BOUNCES
    }
    if (client.isConnected()) {
        client.loop();
    }
}
```