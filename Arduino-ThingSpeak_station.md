#                Arduino-ThingSpeak监测站

## 一、概述

​		本项目使用adrduino单片机配合eps8266-01模块实现联网并登录ThingSpeak云平台展示数据。主要实现功能，家用小型监测站，适用地点：厨房、阳台、卫生间等易发生火灾的地方；使用dth11温湿度监测模块监测温湿度，在Thing Speak平台以实时折线图的方式展示；此外使用烟雾传感器配合两个LED灯（绿色，红色）、蜂鸣器，当烟雾正常时，绿灯亮起；当烟雾超过警戒值，LED红色亮起，蜂鸣器… -- …发出警报，同样的，Thing Speak平台会使用折线图（扇形图）实时展示烟雾浓度变化。

## 二、展示部分

![截屏2021-04-29 下午8.47.35](/Users/bigbossyifi/Desktop/截图文件暂存/截屏2021-04-29 下午8.47.35.png)




## 三、代码部分

### 温湿度代码：

```c++
#include <DHT12.h>
#include <dht11.h>

#define dht_apin 11 // Analog Pin sensor is connected to
dht11 dhtObject;

String getTemperatureValue(){

   dhtObject.read(dht_apin);
   Serial.print(" Temperature(C)= ");
   int temp = dhtObject.temperature;
   Serial.println(temp); 
   delay(50);
   return String(temp); 
  
}


String getHumidityValue(){

   dhtObject.read(dht_apin);
   Serial.print(" Humidity in %= ");
   int humidity = dhtObject.humidity;
   Serial.println(humidity);
   delay(50);
   return String(humidity); 
}
```

### 烟雾监测部分代码

```c++
#include <SoftwareSerial.h> //Arduino serial interface library
#include <Arduino.h>  //Arduino library

#define Sensor_A0 A0 //Smoke simulation port is A0
#define Sensor_D0 0  //Smoke simulation port is 0
unsigned int sensorValue = 0; //Initialize the smoke value to 0
int Buzzer=8;   //The buzzer pin is 8
int RedLed=7;      //Red LED warning light pin 7
int GreenLed = 9;   //The threshold green LED status light pin is 9

/*Smoke Monitoring Section*/
 int val; 
  val=analogRead(0);
  Serial.println(val,DEC);
  while(val>250)    //If the monitored smoke value is greater than 250
     {
        digitalWrite(Buzzer,HIGH); //Pin 8 high level buzzer sounds
        digitalWrite(RedLed,HIGH);  //The red LED is high level bright
        digitalWrite(GreenLed,LOW); //The green LED does not light at low level
        val=analogRead(0);   
        Serial.println(val,DEC);
      }
  /*Smoke values are below the normal range of 250*/
  digitalWrite(Buzzer,LOW); //Pin 8 low level buzzer does not sound
  digitalWrite(RedLed,LOW); //The red LED does not light at low level
  digitalWrite(GreenLed,HIGH);  //Green LED high voltage usually bright
}

/*Upload smoke monitoring data section*/
String SmokeValue(){
   sensorValue = analogRead(Sensor_A0);
   Serial.print(" Somke : ");
   Serial.print(sensorValue);
   int smoke = sensorValue;
   Serial.println(smoke); 
   delay(50);
   return String(smoke); 
  
}
false;
 }
```

#### 网络上传部分代码

```c++
#include <SoftwareSerial.h> //Arduino serial interface library

#define RX 2  //The Arduino development board RX pin is 2
#define TX 3  //The Arduino development board TX pin is 3

String AP = "BIGBOSS";       // Wi-Fi AP
String PASS = "02230223"; // Wi-Fi PassWord
String API = "G7T28TOHQODVIWDG";   // ThingSpeak API "Write"Key
String HOST = "api.thingspeak.com"; //HOST：api.ThingSpeak
String PORT = "80"; //PORT：80
int countTrueCommand;
int countTimeCommand; 
boolean found = false; 
int valSensor = 1;
  
SoftwareSerial esp8266(RX,TX); 
  
void setup() {
  pinMode(Sensor_D0, INPUT);  //Set the serial port D0 input mode
  Serial.begin(9600); //Set the serial port monitor baud rate (9600)
  esp8266.begin(115200);  //ESP8266 Serial Port Baud Rate (115200)
  sendCommand("AT",5,"OK");
  sendCommand("AT+CWMODE=1",5,"OK");
  sendCommand("AT+CWJAP=\""+ AP +"\",\""+ PASS +"\"",20,"OK");

  pinMode(Buzzer,OUTPUT); //Define digital port 8 as output mode
  pinMode(RedLed,OUTPUT);
  pinMode(GreenLed,OUTPUT);
}
/*Loop Port with AT*/
void loop() {
  
 String getData = "GET /update?api_key="+ API +"&field1="+getTemperatureValue()+"&field2="+getHumidityValue()+"&field3="+SmokeValue();
 sendCommand("AT+CIPMUX=1",5,"OK");
 sendCommand("AT+CIPSTART=0,\"TCP\",\""+ HOST +"\","+ PORT,15,"OK");
 sendCommand("AT+CIPSEND=0," +String(getData.length()+4),4,">");
 esp8266.println(getData);delay(1500);countTrueCommand++;
 sendCommand("AT+CIPCLOSE=0",5,"OK");

void sendCommand(String command, int maxTime, char readReplay[]) {
  Serial.print(countTrueCommand);
  Serial.print(". at command => ");
  Serial.print(command);
  Serial.print(" ");
  while(countTimeCommand < (maxTime*1))
  {
    esp8266.println(command);//at+cipsend
    if(esp8266.find(readReplay))//ok
    {
      found = true;
      break;
    }
  
    countTimeCommand++;
  }
  
  if(found == true)
  {
    Serial.println("Success");
    countTrueCommand++;
    countTimeCommand = 0;
  }
  
  if(found == false)
  {
    Serial.println("Fail");
    countTrueCommand = 0;
    countTimeCommand = 0;
  }
  
  found = false;
 }
```

