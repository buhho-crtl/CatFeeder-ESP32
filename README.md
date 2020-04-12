# CatFeeder-ESP32

Program your ESP32 to feed your cats at the time you want. :cat:

The project initially started through the printing of this thingiverse project

https://www.thingiverse.com/thing:3411404

Components:

Motor 360º :point_down:

![](https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcSyB-TDoKKnJPtLMTtXqHcsx4P0YIFW2EITWX4VN_eUznpZhaXf&usqp=CAU)

Sensor Load Cell Weighing Sensor (1KG) :point_down:

![](https://robu.in/wp-content/uploads/2017/04/517saYIG0vL._SL1100_.jpg)

xt711 :point_down:

![](https://images-na.ssl-images-amazon.com/images/I/71o%2BzzWUG6L._SL1500_.jpg)

ESP32 :point_down:

![](https://ae01.alicdn.com/kf/HTB1BC5DQFXXXXaFXVXXq6xXFXXXj/ESP32-Rev1-ESP-32-WiFi-Modules-Bluetooth-Dual-ESP-32-ESP-32S-ESP8266.jpg)

I configured everything with arduino uno and a shield clock, but over time I thought that every time I wanted to modify the time of feeding or the amount of food dispensed on the arduino I had to plug it in the computer, so I dusted the ESP32 that I had in the drawer and reprogrammed everything from 0 to be esp32 compatible (it can also be adapted with arduino uno and 8266).

The process is easy, you have to create a firebase project and add the authentication data in the configuration. Then you can clone the project on android and configure the schedules so that the feeder works.

Thank you very much and good code!
