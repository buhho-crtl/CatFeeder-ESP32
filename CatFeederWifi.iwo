#include <FirebaseESP32.h>
#include <FirebaseESP32HTTPClient.h>
#include <jsmn.h>
#include <FirebaseJson.h>
#include <ArduinoJson.h>

#include <WiFi.h>
#include <ESP32Servo.h>

//Weight sensor config
#include "Wire.h"
#include "HX711.h"
#include "time.h"

#define DS3231_I2C_ADDRESS 0x68
#define calibration_factor -1050.0 //This value is obtained using the SparkFun_HX711_Calibration sketch

//Servo Config
#define DOUT  14
#define CLK  13
#define SERVO 12
Servo myservo;
HX711 scale;

// Set these to run example.
#define FIREBASE_HOST "" //database.firebaseio.com
#define FIREBASE_AUTH "" //XLU4DbeJm546545641Ut5dbQi123123ZKht4xfI"
#define WIFI_SSID ""
#define WIFI_PASSWORD ""

int pos = 0;    // variable to store the servo position

const char *ntpServer = "pool.ntp.org";
const long gmtOffset_sec = 0;
const int daylightOffset_sec = 3600;

FirebaseData firebaseData;

struct TimerFeer {
  public:
    int hour;
    int minutes;
    int feed;
};

TimerFeer myArray[] = {
  {9,  30, 5},
  {14, 00, 5},
  {17, 25, 1},
  {22, 30, 5}
};

void setup() {

  Serial.begin(115200);

  Wire.begin();

  scale.begin(DOUT, CLK);

  //calibrateScale();

  //connect to WiFi
  Serial.printf("Connecting to %s ", WIFI_SSID);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  int counter = 0;
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");

    if (counter % 5 == 0) {
      Serial.println("Reconect Wifi");
      WiFi.disconnect();
      delay(1000);
      WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
      WiFi.reconnect();
      delay(1000);
    }
    counter++;
  }
  Serial.println(" CONNECTED WIRELESS");
  Serial.println(WiFi.localIP());

  Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
  Firebase.reconnectWiFi(true);

  //init and get the time
  bool customData = false;

  if(customData){
    customDate();
  }else{
    configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
    configLocalTime();
  }

  
  //sendConfig(); //Initial configuration to Firebase
  //sendLog(4, 5); //Test sendLog
  //sendFeed(10);  //Test feed
  getConfig(); //Get configuration from Firebase

}

void configLocalTime() {
  struct tm timeinfo;
  while (!getLocalTime(&timeinfo)) {
    configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
  }

  Serial.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");
}

void customDate(){
  
  struct tm tm;
    tm.tm_year = 2018 - 1900;
    tm.tm_mon = 10;
    tm.tm_mday = 15;
    tm.tm_hour = 23;
    tm.tm_min = 59;
    tm.tm_sec = 50;
    time_t t = mktime(&tm);
    printf("Setting time: %s", asctime(&tm));
    struct timeval now = { .tv_sec = t };
    settimeofday(&now, NULL);
}


void calibrateScale() {
  Serial.println("Before setting up the scale:");
  Serial.print("read: \t\t");
  Serial.println(scale.read());      // print a raw reading from the ADC

  Serial.print("read average: \t\t");
  Serial.println(scale.read_average(20));   // print the average of 20 readings from the ADC

  Serial.print("get value: \t\t");
  Serial.println(
    scale.get_value(5));   // print the average of 5 readings from the ADC minus the tare weight (not set yet)

  Serial.print("get units: \t\t");
  Serial.println(scale.get_units(5),
                 1);  // print the average of 5 readings from the ADC minus tare weight (not set) divided
  // by the SCALE parameter (not set yet)

  scale.set_scale(
    2280.f);                      // this value is obtained by calibrating the scale with known weights; see the README for details
  scale.tare();               // reset the scale to 0

  Serial.println("After setting up the scale:");

  Serial.print("read: \t\t");
  Serial.println(scale.read());                 // print a raw reading from the ADC

  Serial.print("read average: \t\t");
  Serial.println(scale.read_average(20));       // print the average of 20 readings from the ADC

  Serial.print("get value: \t\t");
  Serial.println(scale.get_value(
                   5));   // print the average of 5 readings from the ADC minus the tare weight, set with tare()

  Serial.print("get units: \t\t");
  Serial.println(scale.get_units(5),
                 1);        // print the average of 5 readings from the ADC minus tare weight, divided
  // by the SCALE parameter set with set_scale

}

void displayTime() {
  byte second, minute, hour, dayOfWeek, dayOfMonth, month, year;

  struct tm timeinfo;
  if (!getLocalTime(&timeinfo)) {
    Serial.println("Failed to obtain time");
    return;
  }
  Serial.println(&timeinfo, "%A, %B %d %Y %H:%M:%S");

  second = timeinfo.tm_sec;
  minute = timeinfo.tm_min;
  hour = timeinfo.tm_hour;

  readArray();
  int n;
  int sizeArray = (sizeof(myArray) / sizeof(myArray[0]));
  
  for (n = 0; n < sizeArray; ++n) {
    if ((hour == myArray[n].hour && minute == myArray[n].minutes)) {
      sendFeed(myArray[n].feed);
    }
  }

  Serial.print("Hora ");
  Serial.print(hour, DEC);
  Serial.print(":");
  Serial.print(minute, DEC);
  Serial.print(":");
  Serial.println(second, DEC);

  readArray();

}

void sendFeed(long addLimit) {
  scale.power_up();
  calibrateScale();
  Serial.print("Reading: ");
  Serial.print(scale.get_units(), 1); //scale.get_units() returns a float
  Serial.print(" lbs"); //You can change this to kg but you'll need to refactor the calibration_factor
  Serial.println();

  boolean exit = true;
  myservo.attach(SERVO);
  float pounds = scale.get_units(5);
  float limit = 0;
  bool reverse = false;

  limit = pounds + addLimit;

  Serial.print("pounds ");
  Serial.println(pounds, DEC);
  Serial.print("Limit ");
  Serial.println(limit, DEC);

  delay(1500);
  float weigh = 0;

  while (exit) {
    Serial.println("scan");
    float pounds = scale.get_units(5);
    Serial.println(pounds, 1);

    int angle = 0;

    //rotate back
     for (angle = 30; angle >= 1; angle--)     // command to move from 180 degrees to 0 degrees 
        {
            float weighNow = scale.get_units(5);
            Serial.print("weitgh ");
            Serial.println(weighNow, 1);
            if (limit <= weighNow) {
                exit = false;
                Serial.print("weitgh ");
                Serial.println(weighNow, 1);
                Serial.print("Limit ");
                Serial.println(limit, DEC);
                Serial.println("weighNow<limit exit");
                myservo.detach();
                break;
            } else {
                Serial.println("weighNow<limit false");
                myservo.write(angle);              //command to rotate the servo to the specified angle
                delay(250);
            }

            for (pos = 0; pos <= 40; pos += 1) {
                myservo.write(pos);
                delay(15);
            }
      weigh = weighNow;
    }
    //servoBack();
  }

  scale.power_down();

  sendLog(limit, weigh);
  getConfig();
}

void getConfig() {
 if (Firebase.get(firebaseData,"config")){

      // printResult(firebaseData);
      Serial.println();
        //get array data from FirebaseData using FirebaseJsonArray object
        FirebaseJsonArray &arr = firebaseData.jsonArray();
        //Print all array values
        Serial.println("Pretty printed Array:");
        String arrStr;
        arr.toString(arrStr, true);
        Serial.println(arrStr);
        Serial.println();
        Serial.println("Iterate array values:");
        Serial.println();
  
        myArray[arr.size()]  = { };
        
        StaticJsonDocument<1500> doc;
        // Deserialize the JSON document
          DeserializationError error = deserializeJson(doc, arrStr);
        
          // Test if parsing succeeds.
          if (error) {
            Serial.print(F("deserializeJson() failed: "));
            Serial.println(error.c_str());
            return;
          }

          for (size_t i = 0; i < arr.size(); i++){
               int hour = doc[i]["feed"];
               myArray[i] = {
                doc[i]["hour"],
                doc[i]["minutes"],
                doc[i]["feed"]
                };
          }
               
    }else{
      Serial.println("ERROR: " + firebaseData.errorReason());
      Serial.println();
    }

 readArray();
}

void readArray() {
  Serial.println("------------------------------------------------------------------------------------");

  int sizeArray = (sizeof(myArray) / sizeof(myArray[0]));
  for (size_t i = 0; i < sizeArray; i++) {
    Serial.println("Element: " + String(i));
    Serial.println("\t Hour: " + String(myArray[i].hour));
    Serial.println("\t Minutes: " + String(myArray[i].minutes));
    Serial.println("\t feed: " + String(myArray[i].feed));
  }

  Serial.println("------------------------------------------------------------------------------------");
}


void sendConfig() {
 //InitialConfig with firebase
  FirebaseJson jsonBuffer;
  FirebaseJsonArray arr;

  Serial.println("Generate JSON");

  int sizeArray = (sizeof(myArray) / sizeof(myArray[0]));
  for (byte i = 0; i < sizeArray; i++) {
    FirebaseJson jsonBuffer;

    jsonBuffer.add("hour", myArray[i].hour);
    jsonBuffer.add("minutes", myArray[i].minutes);
    jsonBuffer.add("feed", myArray[i].feed);

    arr.set("/[" + String(i) + "]", jsonBuffer);
  }

  String jsonStr;
  arr.toString(jsonStr, true);
  Serial.println(jsonStr);


  if (Firebase.set(firebaseData, "/config", arr)) {
    Serial.println("PASSED");
    Serial.println("PATH: " + firebaseData.dataPath());
    Serial.print("PUSH NAME: ");
    Serial.println(firebaseData.pushName());
    Serial.println("ETag: " + firebaseData.ETag());
    Serial.println("------------------------------------");
    Serial.println();
  } else {
    Serial.println("FAILED");
    Serial.println("REASON: " + firebaseData.errorReason());
    Serial.println("------------------------------------");
    Serial.println();
  }
}

void sendLog(int limit, float weigh) {

  FirebaseJson jsonBuffer;

  time_t now;
  time(&now);
  Serial.println(now);

  jsonBuffer.add("limit", limit);
  jsonBuffer.add("feed", weigh);
  jsonBuffer.add("time", String(now).toFloat());

  String jsonStr;
  jsonBuffer.toString(jsonStr, true);
  Serial.println(jsonStr);

  if (Firebase.pushJSON(firebaseData, "measures", jsonBuffer)) {
    Serial.println("PASSED");
    Serial.println("PATH: " + firebaseData.dataPath());
    Serial.print("PUSH NAME: ");
    Serial.println(firebaseData.pushName());
    Serial.println("ETag: " + firebaseData.ETag());
    Serial.println("------------------------------------");
    Serial.println();
  } else {
    Serial.println("FAILED");
    Serial.println("REASON: " + firebaseData.errorReason());
    Serial.println("------------------------------------");
    Serial.println();
  }
}

void loop() {
  esp_sleep_disable_wakeup_source(ESP_SLEEP_WAKEUP_ALL);
  displayTime();
  delay(60000);
}
