# Seronitor

In een wereld anno 2021, beginnen steeds meer mensen vanuit huis te werken. Dit neemt veel voordelen met zich mee, bepaal je werktijden, eigen inrichting, pauzes zelf inplannen enz... Maar het neemt ook veiligheidsrisico's met zich mee. Zo kan je computer gestolen worden terwijl het nog ontgrendeld was of je huisgenoot ziet NDA gevoelige informatie. In dit document bevindt zich daarvoor een passende oplossing die voor iedereen geschikt is die een huis-, tuin-, keuken- kantoor heeft.

Hier onder is de eerste protoype te vinden van Seronitor op basis van arduino(esp8266), om een demostratie te kunnen bieden van het uiteindelijk beoogde product.

![Seronitor](https://github.com/LarsHVA/Project-IOT-HVA/blob/main/Doc/b7320ad7-1d14-48b5-bbd4-fd32b3015a20.jpg?raw=true)

## Wat heb je nodig
### Hardware
- Arduino(esp8266)
- HC-SR04 Ultrasonic Sensor
- Ledstrip (HUE)
### Software
- [Arduino IDE](https://www.arduino.cc/en/software)
- [Telegram App store](https://apps.apple.com/app/telegram-messenger/id686449807), [Telegram Play store](https://play.google.com/store/apps/details?id=org.telegram.messenger) 

## 1. instaleer Arduino
- [Instaleer Arduino](https://www.arduino.cc/en/guide/windows)
- [Instaleer ESP32 Board](https://www.arduino.cc/en/guide/windows)

## 2. Telegram bot aanmaken
### Botfather
1. Op deze link op je telefoon [t.me/botfather](t.me/botfather)
2. Type `/Start`
3. Type `/Newbot`
4. Type je gebruikers naam in
5. Als je bot succesvol is aan gemaakt krijg je een link en een TOKEN te zien, Sla deze op voor later.

![Telegram bot aanmaken](https://github.com/LarsHVA/Project-IOT-HVA/blob/main/Doc/4590c48c-6330-45d4-8777-9dc5d78fcc7f.jpg?raw=true)

### Telegram gebruikers ID
1. Op deze link op je telefoon [t.me/myidbot](t.me/myidbot)
2. Type `/Getid`
3. Je ontvangt een gebruikers ID, Sla deze op voor later

![Telegram gebruikers ID](https://github.com/LarsHVA/Project-IOT-HVA/blob/main/Doc/6777f4db-d5ab-4f25-9495-fd2b9f32a496.jpg?raw=true)


## 3. Installeer de benodigde libraries
### Telegram library
1. Download [De Telegram library](https://github.com/witnessmenow/Universal-Arduino-Telegram-Bot/archive/master.zip)
2. Ga naar *Sketch > Include Library > Add.ZIP Library..* in de Arduino IDE
3. En upload de net gedownloade library

### ArduinoJson Library
1. Ga naar *Skech > Include Library > Manage Libraries* in de Arduino IDE
2. Zoek naar *ArduinoJson by Benoit BlancHon* Versie > 6.15.2
3. Instaleer de Library

![ArduinoJson Library](https://github.com/LarsHVA/Project-IOT-HVA/blob/main/Doc/Install-ArduinoJson-Library.png?raw=true)

## 4. Hardware verbinden
### HC-SR04 Ultrasonic Sensor naar Arduino
1. Verbind Vcc met Vin
2. Verbind Trig met D6
3. Verbind Echo met D7
4. Verbind Gnd met Gnd

### HC-SR04 Ultrasonic Sensor naar Arduino
1. Verbind +5v met Vin
2. Verbind Din met D5
4. Verbind Gnd met Gnd

## 5. upload code
1. Kopieer de volgende code

```
#ifdef ESP32
  #include <WiFi.h>
#else
  #include <ESP8266WiFi.h>
#endif
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>   // Universal Telegram Bot Library written by Brian Lough: https://github.com/witnessmenow/Universal-Arduino-Telegram-Bot
#include <ArduinoJson.h>

#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
 #include <avr/power.h>
#endif
#define PIN        D5
#define NUMPIXELS !!
Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
#define DELAYVAL 500

// Replace with your network credentials
const char* ssid = "!!!!!!!!!!!!!";
const char* password = "!!!!!!!!!!!!!";

// Initialize Telegram BOT
#define BOTtoken "!!!!!!!!!!!!!" 
#define CHAT_ID "!!!!!!!!!!!!!"

#ifdef ESP8266
  X509List cert(TELEGRAM_CERTIFICATE_ROOT);
#endif

WiFiClientSecure client;
UniversalTelegramBot bot(BOTtoken, client);

// Checks for new messages every 1 second.
int botRequestDelay = 1000;
unsigned long lastTimeBotRan;

const int ledPin = 2;
bool ledState = LOW;

byte doOnce = 0;

const int trigPin = D6;
const int echoPin = D7;

//define sound velocity in cm/uS
#define SOUND_VELOCITY 0.034

long duration;
float distanceCm;

// Handle what happens when you receive new messages
void handleNewMessages(int numNewMessages) {
  Serial.println("handleNewMessages");
  Serial.println(String(numNewMessages));

  for (int i=0; i<numNewMessages; i++) {
    
    #if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
      clock_prescale_set(clock_div_1);
    #endif
    pixels.begin();
    
    // Chat id of the requester
    String chat_id = String(bot.messages[i].chat_id);
    if (chat_id != CHAT_ID){
      bot.sendMessage(chat_id, "Unauthorized user", "");
      continue;
    }
    
    // Print the received message
    String text = bot.messages[i].text;
    Serial.println(text);

    String from_name = bot.messages[i].from_name;

    if (text == "/start") {
      String welcome = "Welkom, " + from_name + ".\n";
      welcome += "Gebruik de volgende commando's om je computer aan of uit te zetten\n\n";
      welcome += "/Aan Zet Computer aan \n";
      welcome += "/Uit Zet computer uit \n";
      bot.sendMessage(chat_id, welcome, "");
    }

    if (text == "/Aan") {
      bot.sendMessage(chat_id, "Computer is aangezet", "");
      Serial.println("Computer aan");
      pixels.clear();
      for(int i=0; i<NUMPIXELS; i++) {
        pixels.setPixelColor(i, pixels.Color(0, 0, 255));
        pixels.show();
      }
    }
    
    if (text == "/Uit") {
      bot.sendMessage(chat_id, "Computer is uitgezet", "");
      Serial.println("Computer uit");
      pixels.clear();
      pixels.show();
    }
    
  }

    // Check distance is below 150 or not
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    // Sets the trigPin on HIGH state for 10 micro seconds
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);
    
    // Reads the echoPin, returns the sound wave travel time in microseconds
    duration = pulseIn(echoPin, HIGH);
    
    // Calculate the distance
    distanceCm = duration * SOUND_VELOCITY/2;
   
    
    // Prints the distance on the Serial Monitor
    Serial.print("Distance (cm): ");
    Serial.println(distanceCm);

    if (distanceCm <= 150) {
      Serial.println("Computer uit");
      pixels.clear();
      pixels.show();
    }
  
}

void setup() {
  Serial.begin(115200);

  pinMode(trigPin, OUTPUT); // Sets the trigPin as an Output
  pinMode(echoPin, INPUT); // Sets the echoPin as an Input

  #ifdef ESP8266
    configTime(0, 0, "pool.ntp.org");      // get UTC time via NTP
    client.setTrustAnchors(&cert); // Add root certificate for api.telegram.org
  #endif

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, ledState);
  
  // Connect to Wi-Fi
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  #ifdef ESP32
    client.setCACert(TELEGRAM_CERTIFICATE_ROOT); // Add root certificate for api.telegram.org
  #endif
  while (WiFi.status() != WL_CONNECTED) {
    if (doOnce == 0) {
      delay(1000);
      Serial.print("Connecting to WiFi.. [-_-]Z");
      doOnce = 1;
    } 
    if (doOnce == 1){
      delay(1000); 
      Serial.print("z");
    }
  }
  
  // Print ESP32 Local IP Address
  Serial.print(" IP:");
  Serial.println(WiFi.localIP());
}

void loop() {

    int numNewMessages = bot.getUpdates(bot.last_message_received + 1);

    while(numNewMessages) {
      Serial.println("got response");
      handleNewMessages(numNewMessages);
      numNewMessages = bot.getUpdates(bot.last_message_received + 1);
    }

    handleNewMessages(numNewMessages);
  
}
```

2. Vul de volgende stukjes aan in de code op de !
  1. Het aantal Ledjes op je strip `#define NUMPIXELS !!`
  2. De SSID van je netwerk `const char* ssid = "!!!!!!!!!!!!!";` (Moet een 2.4 GHZ verbinding zijn!)
  3. Het wachtwoord van je netwerk `const char* password = "!!!!!!!!!!!!!";` (Moet een 2.4 GHZ verbinding zijn!)
  4. Bot token die je hebt gekregen van de Botfather bot `#define BOTtoken "!!!!!!!!!!!!!"`
  5. Chat ID die je hebt gekregen van de myidbot Bot `#define CHAT_ID "!!!!!!!!!!!!!"`

## Code uploaden en Testen
1. Upload je code door op het groene ronde pijltje boven in je Arduino IDE te klikken als je je Arduino hebt aangesloten
2. Als dit zonder errors gaat, klik dan op Serial Monitor, zoniet check de velden die je moest invullen nog een keer of check de error log.
3. In de Serial Monitor selecteer je
4. Als het goed is zie je nu de tekst `Connecting to WiFi.. [-_-]Z` verschijnen
5. Tijdens het verbinden komen er `z` bij. (Als dit langer dan een minuut duurt check je netwerk inlog gegevens die je hebt ingevult)
6. Als de `z` zijn beeindicht ben je succesvol verbonden met je netwerk
7. Hierna zie je gegeven van de HC-SR04 Ultrasonic Sensor opkomen

![Serial Monitor](https://github.com/LarsHVA/Project-IOT-HVA/blob/main/Doc/Schermafbeelding%202021-10-27%20232838.png?raw=true)

9. Ga naar je eigen bot via de link die je van de Botfather hebt gekregen.
10. Type `/Start` in om de Commando's in te zien
11. Type `/Aan` of `/Uit` in om de Ledstrip aan of uit te zetten
12. Als de HC-SR04 Ultrasonic Sensor een afstand meet die korter is dan 150 CM zet de Arduino zelf de Ledstrip uit

![Resultaat](https://github.com/LarsHVA/Project-IOT-HVA/blob/main/Doc/01a659ea-c32f-40d4-9c14-417e3e60cd97.jpg?raw=true)

![Resultaat](https://github.com/LarsHVA/Project-IOT-HVA/blob/main/Doc/ezgif-6-d0c740dc9b33.gif?raw=true)

## Bronnen
- [Telegram: Control ESP32/ESP8266 Outputs (Arduino IDE)](https://randomnerdtutorials.com/telegram-control-esp32-esp8266-nodemcu-outputs/)
- [ESP8266 NodeMCU with HC-SR04 Ultrasonic Sensor with Arduino IDE](https://randomnerdtutorials.com/esp8266-nodemcu-hc-sr04-ultrasonic-arduino/)
