# Seronitor

In een wereld anno 2021, beginnen steeds meer mensen vanuit huis te werken. Dit neemt veel voordelen met zich mee, bepaal je werktijden, eigen inrichting, pauzes zelf inplannen enz... Maar het neemt ook veiligheidsrisico's met zich mee. Zo kan je computer gestolen worden terwijl het nog ontgrendeld was of je huisgenoot ziet NDA gevoelige informatie. In dit document bevindt zich daarvoor een passende oplossing die voor iedereen geschikt is die een huis-, tuin-, keuken- kantoor heeft.

Hier onder is de eerste protoype te vinden van Seronitor op basis van arduino(esp8266), om een demostratie te kunnen bieden van het uiteindelijk beoogde product.

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
Botfather
1. Op deze link op je telefoon [t.me/botfather](t.me/botfather)
2. Type `/Start`
3. Type `/Newbot`
4. Type je gebruikers naam in
5. Als je bot succesvol is aan gemaakt krijg je een link en een TOKEN te zien, Sla deze op voor later.

Telegram gebruikers ID
1. Op deze link op je telefoon [t.me/myidbot](t.me/myidbot)
2. Type `/Getid`
3. Je ontvangt een gebruikers ID, Sla deze op voor later


## 2. Installeer de benodigde libraries
