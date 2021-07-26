# Arduino
This repository contain Moisture sensor code for Arduino circuit 
#include <ESP8266WiFi.h>
#include <String.h>
#include <Arduino.h>
#include <ESP8266HTTPClient.h>
const int sensor_pin = A0;  /* Connect Soil moisture analog sensor pin to A0 of NodeMCU */
const char* ssid="Contact-03047172474";
const char* password = "9fed1f1a";
const int motorPin = D0;
unsigned long interval = 10000;
int action=2;

float threshold=20.0;
float maxthreshold=52.0;
 float moisture_percentage=0.0;
int m_status=0;
void setup() {
 
  //while (!Serial) ;
  Serial.begin(9600); /* Define baud rate for serial communication */
while (!Serial) {

  Serial.print("serial connecting ");
    ; // wait for serial port to connect. Needed for native USB port only

  }

  
 pinMode(motorPin, OUTPUT);
digitalWrite(motorPin, LOW); // keep motor off initally
  moisture_percentage = ( 100.00 - ( (analogRead(sensor_pin)/1023.00) * 100.00 ) );
  
  Serial.print("main function ");
    Serial.print("Wifi connecting to ");
  Serial.println( ssid );

  WiFi.begin(ssid,password);
  delay(3000);
  Serial.print("Connecting");
while (!WiFi.status()== WL_CONNECTED) {

  Serial.println(".");
  
    ; // wait for serial port to connect. Needed for native USB port only

  }
  

  
     
 
  

 
  

 
}
int SendValue(){
 // Serial.println(String(m_status)+"   "+String(threshold)+"    "+String(maxthreshold));
   moisture_percentage = ( 100.00 - ( (analogRead(sensor_pin)/1023.00) * 100.00 ) );
   if(moisture_percentage > 2.0){
    
   
         Serial.println("Soil Moisture(in Percentage) = ");
         
    Serial.print( moisture_percentage);
    HTTPClient http;    //Declare object of class HTTPClient
    http.begin("http://192.168.10.7/smartagriapi/api/DeviceRec?did=1&m="+String(moisture_percentage)+"&ms="+String(m_status));      //Specify request destination
    http.addHeader("Content-Type", "text/plain");  //Specify content-type header
    
    int httpCode = http.GET();   //Send the request
    
    String payload = http.getString();
     Serial.println(payload);
    if(httpCode>0){
     Serial.println(httpCode);
      payload = http.getString();
      if (payload.indexOf("success",0) >0)
{
    Serial.println(payload);
//Serial.println("success");

    int s = payload.indexOf('|');
   
    int e = payload.indexOf('|',s+1) ;
   
    action =(payload.substring(s+1, e )).toInt(); //extract motor status from api response 
     s = e + 1;
        e = payload.indexOf('|', s);
    
   
    threshold=( payload.substring(s, e )).toFloat(); //extract threshold value from api response 
     s = e + 1;
        e = payload.indexOf('|', s);
    
    
    maxthreshold = (payload.substring(s, e)).toFloat(); //extract MAX threshold value from api response
    Serial.println(String(action)+"   "+String(threshold)+"    "+String(maxthreshold));
   
    //}
    //else{ //if Api response failed 
      
//Serial.println("Api Response failed ");
      // continue;
   //}
    }
      
     //Serial.print(payload);
Serial.println();
   
 
   }
   else{
    Serial.print(moisture_percentage);
      Serial.print("  Recorded Value is less than siol value so it will be discarted ");
   }





   
}}
void loop() {
  

 Serial.println("-------------------loop!---------");
 
 
  
  // delay(5000);
 
  if(WiFi.status()== WL_CONNECTED){
    
     Serial.println("Wifi status Connected !");
 // Serial.print("NodeMCU IP Address : ");
  //Serial.println(WiFi.localIP() );
    SendValue();
    delay(3000);
    if(action==1){
      //turn on motot    m_ststus=1 sleep(5 sec)
       Serial.println("action status triger to ON the motor");
      digitalWrite(motorPin, HIGH);
      m_status=1;
       delay(5000);
      while((moisture_percentage<maxthreshold)){
        Serial.println("----- while loop----");
        if((action==2)||(action==0)){
          Serial.println("motor off due to action off");
          break;
        }
        Serial.println(threshold);
         Serial.println(action);
          Serial.println(maxthreshold);
        SendValue();
        delay(4000);
        
        
      }
      if((m_status==1)&&(moisture_percentage>maxthreshold)){   // this will change the action status of db coloumn
      Serial.println("this will change the action status of db coloumn");
      HTTPClient http;
      http.begin("http://192.168.10.7/smartagriapi/api/mobile/DevicAction?deviceid=1&action=2");  "smartagriapi/api/mobile/DevicAction?deviceid=1&action=1"    //Specify request destination
      ;http.addHeader("Content-Type", "text/plain");  //Specify content-type header
      int httpCode = http.GET();   //Send the request
      String payload = http.getString();
      Serial.println(payload);
      }
      Serial.println(" action status or moisture triger to Off the motor");
     digitalWrite(motorPin, LOW);
     m_status=0;
      
    }
     else{
      Serial.println("sending values towards server with motor status off");
       // SendValue();
        
        delay(2000);
      }
  }
  else{
    Serial.println("Wifi status notConnected !");
     delay(2000);
      moisture_percentage = ( 100.00 - ( (analogRead(sensor_pin)/1023.00) * 100.00 ) );
      
      Serial.print("Soil Moisture(in Percentage) = ");
  Serial.println(moisture_percentage);
    if(moisture_percentage>5.0){
      
    
      if(moisture_percentage<threshold){
        //turn on motot    m_ststus=1 sleep(5 sec)
         digitalWrite(motorPin, HIGH);
         m_status=1;
       while(moisture_percentage<maxthreshold){
        //SendValue();
        
        delay(2000);
        moisture_percentage = ( 100.00 - ( (analogRead(sensor_pin)/1023.00) * 100.00 ) );
        Serial.println(moisture_percentage);
      }
       digitalWrite(motorPin, LOW);
       m_status=0;
      }
    }

  }
  


    
 
  
}
